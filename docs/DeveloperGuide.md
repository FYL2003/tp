---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# Dream Day Designer Developer Guide

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

1. When `Logic` is called upon to execute a command, it is passed to an `DreamDayDesignerParser` object which in turn creates a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<puml src="diagrams/ParserClasses.puml" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `DreamDayDesignerParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `DreamDayDesignerParser` returns back as a `Command` object.
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

**Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `DreamDayDesigner`, which `Person` references. This allows `DreamDayDesigner` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<puml src="diagrams/BetterModelClassDiagram.puml" width="450" />

</box>


### Storage component

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<puml src="diagrams/StorageClassDiagram.puml" width="550" />

The `Storage` component,
* can save both address book data and user preference data in JSON format, and read them back into corresponding objects.
* inherits from both `DreamDayDesignerStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.ddd.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation**

This section describes some noteworthy details on how certain features are implemented.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedDreamDayDesigner`. It extends `DreamDayDesigner` with an undo/redo history, stored internally as an `dreamDayDesignerStateList` and `currentStatePointer`. Additionally, it implements the following operations:

* `VersionedDreamDayDesigner#commit()` — Saves the current address book state in its history.
* `VersionedDreamDayDesigner#undo()` — Restores the previous address book state from its history.
* `VersionedDreamDayDesigner#redo()` — Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitDreamDayDesigner()`, `Model#undoDreamDayDesigner()` and `Model#redoDreamDayDesigner()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedDreamDayDesigner` will be initialized with the initial address book state, and the `currentStatePointer` pointing to that single address book state.

<puml src="diagrams/UndoRedoState0.puml" alt="UndoRedoState0" />

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitDreamDayDesigner()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `dreamDayDesignerStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

<puml src="diagrams/UndoRedoState1.puml" alt="UndoRedoState1" />

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitDreamDayDesigner()`, causing another modified address book state to be saved into the `dreamDayDesignerStateList`.

<puml src="diagrams/UndoRedoState2.puml" alt="UndoRedoState2" />

<box type="info" seamless>

**Note:** If a command fails its execution, it will not call `Model#commitDreamDayDesigner()`, so the address book state will not be saved into the `dreamDayDesignerStateList`.

</box>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoDreamDayDesigner()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

<puml src="diagrams/UndoRedoState3.puml" alt="UndoRedoState3" />


<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index 0, pointing to the initial DreamDayDesigner state, then there are no previous DreamDayDesigner states to restore. The `undo` command uses `Model#canUndoDreamDayDesigner()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</box>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

<puml src="diagrams/UndoSequenceDiagram-Logic.puml" alt="UndoSequenceDiagram-Logic" />

<box type="info" seamless>

**Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</box>

Similarly, how an undo operation goes through the `Model` component is shown below:

<puml src="diagrams/UndoSequenceDiagram-Model.puml" alt="UndoSequenceDiagram-Model" />

The `redo` command does the opposite — it calls `Model#redoDreamDayDesigner()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<box type="info" seamless>

**Note:** If the `currentStatePointer` is at index `dreamDayDesignerStateList.size() - 1`, pointing to the latest address book state, then there are no undone DreamDayDesigner states to restore. The `redo` command uses `Model#canRedoDreamDayDesigner()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</box>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitDreamDayDesigner()`, `Model#undoDreamDayDesigner()` or `Model#redoDreamDayDesigner()`. Thus, the `dreamDayDesignerStateList` remains unchanged.

<puml src="diagrams/UndoRedoState4.puml" alt="UndoRedoState4" />

Step 6. The user executes `clear`, which calls `Model#commitDreamDayDesigner()`. Since the `currentStatePointer` is not pointing at the end of the `dreamDayDesignerStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

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

**Target user profile**:

* Busy wedding planners with many client and vendor contacts
* Is reasonably comfortable using CLI apps
* Works alone
* Likes flexibility in scheduling

**Value proposition**: Provide a way to easily select suitable vendors and freelancers for a wedding event given specific parameters such as budget, time, commission, client needs (culture, style), location. 


### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​                            | I want to …​                                                | So that I can…​                                                                       |
|----------|------------------------------------|-------------------------------------------------------------|---------------------------------------------------------------------------------------|
| `***`    | forgetful user                     | see a list of all upcoming events                           | plan ahead and allocate my time effectively                                           |
| `***`    | normal user                        | create a new event                                          | keep track of which vendors and clients are associated with an event                  |
| `***`    | normal user                        | view all current events                                     | quickly see what events are coming up soon                                            |
| `***`    | normal user                        | search for a specific vendor                                | quickly access vendor details without manually looking through the whole address book |
| `***`    | normal user                        | tag vendors by categories                                   | quickly filter clients who require a specific service                                 |
| `***`    | normal user                        | save my work at any time using a save button                | avoid losing important information when I take a break                                |
| `***`    | user with erratic work schedule    | close the app and return later to continue where I left off | resume planning without disruption                                                    |
| `***`    | user with many clients             | view all client details from a single dashboard             | get a quick summary of my clients' needs                                              |
| `***`    | normal user                        | view contacts by their tags                                 | quickly access client or vendor details related to a particular category              |
| `***`    | user with many vendors             | view all vendor details from a single dashboard             | have a comprehensive overview of all available services                               |
| `**`     | seasoned user                      | use keyboard shortcuts                                      | work more quickly                                                                     |
| `**`     | forgetful user                     | assign due dates for tasks related to an event              | manage timelines more effectively                                                     |
| `**`     | meticulous user                    | add notes to each vendor                                    | track special considerations or preferences for future references                     |
| `**`     | normal user                        | delete an existing event                                    | remove events when my clients cancel on me                                            |
| `**`     | normal user                        | tag clients by event type or size                           | quickly filter clients who require similar services                                   |
| `**`     | normal user                        | add new clients/vendors on the go                           | input information immediately after meeting them                                      |
| `**`     | normal user                        | mark tasks as completed                                     | track progress of the event and ensure no missed steps                                |
| `**`     | user with many clients             | edit existing client details                                | accommodate any changes to their event preferences                                    |
| `**`     | user with many clients             | assign multiple clients to the same event type              | group similar wedding themes or sizes together                                        |
| `**`     | user with many clients and vendors | add notes to each client                                    | remember specific requirements for their event                                        |
| `**`     | user with many vendors             | edit existing vendor details                                | update contact information or service offerings as needed                             |
| `**`     | user with many vendors             | assign multiple vendors to the same category                | compare and choose vendors more easily                                                |
| `**`     | user with many vendors             | filter my vendors by availability                           | choose the ones who are available for the event date                                  |
| `*`      | wannabe multitask user             | switch between different events quickly                     | easily manage multiple events at once                                                 |
| `*`      | busy user                          | see an overview of my workload for the week/month           | better manage my time and commitments                                                 |
| `*`      | normal user                        | update vendors of an existing event                         | change the vendors when there are changes in requirements or circumstances            |
| `*`      | normal user                        | attach files (contracts, proposals) to vendors/clients      | have all necessary documents in one place                                             |
| `*`      | normal user                        | customize how data is displayed (list view, card view)      | organize information in a way that suits my preferred workflow                        |
| `*`      | user with many events              | archive old events                                          | keep my dashboard uncluttered with only active events displayed                       |
| `*`      | user with many events              | quickly restore archived events                             | revisit previous event details if needed for reference                                |
| `*`      | user with many vendors/client data | quickly access old data for vendors and clients             | avoid re-entering details when planning similar events                                |

### Use cases

(For all use cases below, the **System** is `Dream Day Designer` and the **Actor** is the `Wedding planner`, unless specified otherwise)

**Name: UC01 - Create Contact (Vendor/Client)**

**Main Success Scenario (MSS):**
1. Wedding planner selects the option to create a new contact.
2. System requests for the details of the contact (client or vendor).
3. Wedding planner enters the required details (name, phone number, etc.).
4. System validates the entered details (check for format, duplication, etc.).
5. System creates the contact and assigns a unique ID to the contact.
6. System displays a success message confirming the creation of the contact.

**Extensions:**

* 3a. System detects an error in the entered data (e.g., invalid phone number). 
  
  * 3a1. System requests the correct data.
  * 3a2. Wedding planner enters new data. 
    
    Use case resumes from step 4.

* 5a. The contact already exists in the system (duplicate contact).
  
  * 5a1. System displays a message that the contact already exists.
    
    Use case ends.

**Guarantees:**

The contact is successfully created and stored in the system if all input data is valid.
Duplicate contacts will not be created.

___
**Name: UC02 - Delete Contact (Vendor/Client)**

**Main Success Scenario (MSS):**
1. Wedding planner requests to list all contacts. 
2. System displays a list of contacts. 
3. Wedding planner specifies which contact he wishes to delete. 
4. System requests for confirmation. 
5. Wedding planner confirms the deletion of contact. 
6. System displays a message for the successful deletion of contact.
   
   Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. System is unable to locate the contact to be deleted.

  * 3a1. System displays an error message.
    
    Use case ends.
  
* 4a. The wedding planner chooses to abort the deletion of contact.
  
  * 4a1. System displays a message for aborting the deletion.
  
    Use case ends.

**Guarantees**
* The contact is successfully deleted from the system, and any persistent storage.

**Name: UC03 - View all Contacts**

**Preconditions:**
1. Contacts are saved properly.

**Guarantees:**
1. The user's previously saved contacts will be listed with their details.
2. Contacts are sorted alphabetically.

**Main Success Scenario (MSS):**
1.  User requests for previously saved contacts.
2.  System displays the saved contacts to the user.

    Use case ends.

**Extensions**

* 2a. System is unable to get saved contacts. 
  
  * 2a1. System informs the user that the file is corrupted.
    
    Use case ends.

* 2b. The list is empty.
  
  * 2b1. System informs the user that there are no saved contacts.
    
    Use case ends.
    
**Name: UC04 - Save Contact (Client/Vendor)**

**Preconditions:**
1. The system should be operational.

**Guarantees:**

1. The file will be saved to the directory if the directory exists.
2. If there is a duplicate file (file of the same name), it will be overwritten.
3. If the directory is not valid, there will be no effect with the command.

**Main Success Scenario (MSS):**
1. The user specifies the directory to save data in.
2. System displays a success message.

    Use case ends.

**Extensions**

* 1a. The directory does not exist.
  
  * 1a1. System prompts the user that the target directory does not exist.

  Use case ends. 
 
### Non-Functional Requirements

1. Compatibility: Should work on any _mainstream OS_ (Windows/macOS/Linux) as long as it has Java `17` or above installed.
2. Compatibility: The system should be usable on both desktop (Windows/macOS)
3. Compatibility: The system should work on both 32-bit and 64-bit environments.
4. User Experience: Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
5. User Experience: The system should respond within two seconds.
6. User Experience: A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse. 
8. Privacy: User data must remain on the local machine and not be shared or transmitted to any external services unless explicitly requested by the user (e.g., exporting contacts). 
9. Portability: The system should allow easy exporting of data to be imported into other applications if needed.

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **Vendor**: A person who provides certain kinds of services
* **32-bit and 64-bit environment**: The amount of memory that your computer can address. 64-bit machines are able to access much more memory than 32-bit machines.

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
