---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# ServeMate Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

_{ list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well }_

--------------------------------------------------------------------------------------------------------------------

## **Setting up, getting started**

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design**

### Architecture

<puml src="diagrams/ArchitectureDiagram.puml" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of classes [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java)) is in charge of the app launch and shut down.
* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<puml src="diagrams/ArchitectureSequenceDiagram.puml" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<puml src="diagrams/ComponentManagers.puml" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

<puml src="diagrams/UiClassDiagram.puml" alt="Structure of the UI Component"/>

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<puml src="diagrams/LogicClassDiagram.puml" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking `execute("delete 1")` API call as an example.

<puml src="diagrams/DeleteSequenceDiagram.puml" alt="Interactions Inside the Logic Component for the `delete 1` Command" />

<box type="info" seamless>

**Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</box>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<puml src="diagrams/ModelClassDiagram.puml" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<box type="info" seamless>

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.address.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedAddressBook#commit()` — Saves the current address book state in its history.
* `VersionedAddressBook#undo()` — Restores the previous address book state from its history.
* `VersionedAddressBook#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()` and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

<puml src="diagrams/UndoRedoState5.puml" alt="UndoRedoState5" />

The following activity diagram summarizes what happens when a user executes a new command:

<puml src="diagrams/CommitActivityDiagram.puml" width="250" />

#### Design considerations:

**Aspect: How undo & redo executes:**

* **Alternative 1 (current choice):** Saves the entire address book.
  * Pros: Easy to implement.
  * Cons: May have performance issues in terms of memory usage.

* **Alternative 2:** Individual command knows how to undo/redo by
  itself.
  * Pros: Will use less memory (e.g. for `delete`, just save the person being deleted).
  * Cons: We must ensure that the implementation of each individual command are correct.

_{more aspects and alternatives to be added}_

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops**

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements**

### Product scope

#### Target user profile

* handle the **planning and administration of deliveries** for tingkat caterers
* has a need to **manage a significant number** of customers
* prefer **desktop apps** over other types
* can **type fast**
* **prefers typing** to mouse interactions
* is **reasonably comfortable using** CLI apps

#### Value proposition
* Provides fast and organised solution for tingkat caterers to manage their customer information for delivery planning



### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​       | I want to …​                                            | So that I can…​                                                                                      |
|----------|---------------|---------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| `* * *`  | beginner user | add a customer                                          | keep track of customers                                                                              |
| `* * *`  | beginner user | view a list of all customers                            | have an overview of my operations                                                                    |
| `* * *`  | beginner user | exit from the app easily                                | avoid cluttering my desktop screen once I have finished using the app                                |
| `* * *`  | beginner user | delete a customer                                       | get rid of customer records that I no longer need to track                                           |
| `* *`    | beginner user | see a message explaining how to access the help page    | learn what each operation does                                                                       |
| `* *`    | user          | import customer data in bulk                            | conveniently transition into the app                                                                 |
| `* *`    | user          | edit customer's data                                    | correct any mistakes or changes to customer data to keep information accuracy                        |
| `* *`    | user          | add an upcoming delivery                                | track deliveries that need to be made                                                                |
| `* *`    | user          | delete an upcoming delivery                             | eliminate cancelled delivery                                                                         |
| `* *`    | familiar user | display all upcoming deliveries                         | inform delivery drivers on their delivery points and plan production                                 |
| `* *`    | familiar user | create a delivery route                                 | inform delivery drivers on their delivery route                                                      |
| `* *`    | familiar user | reorder stops within a delivery route                   | ensures deliveries follow an efficient sequence                                                      |
| `* *`    | familiar user | tag each customer with delivery notes                   | inform drivers about specific instructions with regards to delivery                                  |
| `* *`    | busy user     | search for a customer by name, phone number, or address | quickly locate customer details when handling customer enquiries                                     |
| `*`      | expert user   | set estimated time of delivery for a customer           | ensure all customers have their food delivered on time                                               |
| `*`      | expert user   | set delivery status for a customer                      | keep track of deliveries that have been made and cancelled                                           |
| `*`      | expert user   | track customers' subscription expiry date               | check how many customers have their subscription close to the expiration date and gently remind them |
| `*`      | expert user   | track customers' subscription payment                   | know when I received their payements                                                                 |
| `*`      | expert user   | tag each customer by their food preference              | inform the cooks to prepare food that aligns with the customers' food preference                     |
| `*`      | expert user   | mass copy emails and contact numbers to clipboard       | mass email and message customer about upcoming promotions                                            |
| `*`      | expert user   | view free time slots                                    | schedule new deliveries for new customers                                                            |
| `*`      | expert user   | track the total revenue from a customer                 | know how much I have earned from a customer                                                          |
| `*`      | expert user   | track number of days subscribed by a customer so far    | know who are my loyal customers                                                                      |
| `*`      | expert user   | back up customer and route data                         | ensure that delivery operations are not disrupted by data loss                                       |
| `*`      | expert user   | archive customers data                                  | see only the relevant data for currently subscribed customers                                        |




### Use cases

(For all use cases below, the **System** is the `ServeMate` and the **Actor** is the `user`, unless specified otherwise)

**Use case 1: Add a customer**

**MSS**

1. User requests to add a customer contact with required fields 
2. ServeMate adds the customer into the customer list 
3. ServeMate shows a success message with the added customer’s details

    Use case ends.

**Extensions**

* 1a. Any required field is missing.

    * 1a1. ServeMate shows an error message describing the correct command format.

      Use case ends.

* 1b. Any parameter value is invalid.

    * 1b1. ServeMate shows an error message describing the violated constraint.

      Use case ends.

* 1c. A customer with the same name already exists.

    * 1c1. ServeMate shows an error message "This person already exists in the address book".

      Use case ends.

**Use case 2: Delete a customer**

**MSS**

1. User requests to list customers 
2. ServeMate shows a list of customers 
3. User requests to delete a customer in the list 
4. ServeMate deletes the customer
5. ServeMate shows a confirmation message with the deleted customer’s details

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is not a positive integer.

    * 3a1. ServeMate shows an error message describing the correct command format.

      Use case resumes at step 2.

* 3b. The given index is out of range.

    * 3b1. ServeMate shows an error message "The person index provided is invalid".

      Use case resumes at step 2.

**Use case 3: Edit customer record**

**MSS**

1. User requests to list customers
2. ServeMate shows a list of customers
3. User requests to edit a customer in the list
4. ServeMate updates the customer record
5. ServeMate shows a success message with the updated customer’s details

   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is not a positive integer.

    * 3a1. ServeMate shows an error message describing the correct command format.

      Use case resumes at step 2.

* 3b. The given index is out of range.

    * 3b1. ServeMate shows an error message "The person index provided is invalid".

      Use case resumes at step 2.

* 3c. No fields are specified for editing.

    * 3c1. ServeMate shows an error message "At least one field to edit must be provided".

      Use case resumes at step 2.

* 3d. Any provided field value is invalid.

    * 3d1. ServeMate shows an error message describing the violated constraint.

      Use case resumes at step 2.

* 3e. Editing the name causes a duplicate with an existing customer.

    * 3e1. ServeMate shows an error message "This person already exists in the address book".

      Use case resumes at step 2.

**Use case 4: Filter customer records**

**MSS**

1. User requests to find customers by providing one or more attributes
2. ServeMate displays the list of customers that match the given criteria

   Use case ends.

**Extensions**

* 1a. The command format is invalid.

    * 1a1. ServeMate shows an error message describing the correct command format.

      Use case ends.

* 1b. Any provided attribute value violates format constraints.

    * 1b1. ServeMate shows an error message describing the violated constraint.

      Use case ends.

* 2a. No customers match the specified criteria.

    * 2a1. ServeMate shows an empty result list.

      Use case ends.

### Non-Functional Requirements

#### 💻 Portability
1. The system should support **any mainstream OS** with Java `17` or higher.
2. The system should deliver the product as a **single, executable JAR file**.
3. The system should function as a **standalone product** that does not require additional user installations.

#### ⌨️ Usability
1. A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.

#### 🚀 Performance
1. The system should remain **responsive** even when managing 1000 customer records.

#### 💾 Data Persistence
1. The system should store customer data locally in a **human editable JSON file**.
2. The system should **automatically load the stored customer records** upon every system launch.

#### 📝 Additional Requirements
1. The system should use the standardized **Singapore address and phone number format**.

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Tingkat Delivery**: Subscription-based home-cooked meal delivery service commonly found in Singapore.
* **Tingkat Package**: The food catering package, ordered for a set number of days, usually 5, 10, or 20 days.
* **CLI (Command Line Interface)**: A text-based user interface used to interact with software by typing commands.
* **GUI (Graphical User Interface)**: A visual interface that allows users to interact with the application through graphical elements like buttons, windows, and icons.
* **Customer**: A person who subscribes to the Tingkat delivery service.
* **Delivery Rider/Driver**: A person who delivers meals to customers.
* **Tingkat Administrative Staff**: A person who manages the Tingkat delivery service.
* **Delivery Route**: A sequence of stops planned for delivering meals to customers.
* **Subscription**: A predefined plan for meal delivery over a specific period (e.g., 5, 10, or 20 days).
* **Command**: A user input that triggers a specific action in the application (e.g., `add`, `delete`, `list`).
* **Singapore Address Format:** A sequence of elements used to physically locate buildings, in the form `<Block Number>, <Street Name>, <Unit Number (if any)>, Singapore <Postal Code>` (e.g., 757 Woodlands Ave 4, #04-27, Singapore 730757).
* **Singapore Phone Number Format:** A 8-digit number containing no spaces used for telecommunications within Singapore (e.g., `81234567`).

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<box type="info" seamless>

**Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</box>

### Launch and shutdown

1. Initial launch

   1. Download the jar file and copy into an empty folder

   1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be optimum.

1. Saving window preferences

   1. Resize the window to an optimum size. Move the window to a different location. Close the window.

   1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a person

1. Deleting a person while all persons are being shown

   1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

   1. Test case: `delete 1`<br>
      Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message. Timestamp in the status bar is updated.

   1. Test case: `delete 0`<br>
      Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

   1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
      Expected: Similar to previous.

1. _{ more test cases …​ }_

### Saving data

1. Dealing with missing/corrupted data files

   1. _{explain how to simulate a missing/corrupted file, and the expected behavior}_

1. _{ more test cases …​ }_
