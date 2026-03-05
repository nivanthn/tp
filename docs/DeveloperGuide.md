---
  layout: default.md
  title: "Developer Guide"
  pageNav: 3
---

# HireShell Developer Guide

<!-- * Table of Contents -->
<page-nav-print />

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements**

This project is based on the AddressBook-Level3 project created by the [SE-EDU initiative](https://se-education.org).

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

**Target user profile**:

* Meets a lot of new people daily
* Has a large number of contacts with a short lifecycle
* Enjoys using keyboard shortcuts
* Needs to categorise their contacts 
* Works in an office, uses a desktop 
* Values efficiency and speed over user interface 
* Frequently performs batch operations (e.g. deleting a whole group)

**Value proposition**: 

Our application provides a comprehensive list of potential job candidates. 
It provides the ability to quickly change details (e.g. add, delete) of contacts, including streamlined batch operations. 
It also categorises contacts, with the ability to quickly search, sort, and retrieve contacts. 
It is optimised for fast keyboard navigation.



### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …         | I want to …                                                                                                     | So that I can…                                                                          |
| -------- | -------------- | --------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `* *`    | new user       | batch-import a CSV of candidate data                                                                            | populate the app quickly without manual entry                                           |
| `* * *`  | recruiter      | add a new candidate contact with key fields (name, role, source, stage)                                         | capture leads immediately after meeting them                                            |
| `* *`    | mass recruiter | categorise people into large groups                                                                             | do operations on them                                                                   |
| `*`      | mass recruiter | run a cleanup command that groups candidates by inactivity window                                               | batch-archive/delete in one pass                                                        |
| `* * *`  | recruiter      | edit a candidate’s details                                                                                      | update their information when it changes                                                |
| `* *`    | recruiter      | see if a candidate is already in the database                                                                   | not waste time managing the same person twice                                           |
| `* *`    | recruiter      | record last-contacted date                                                                                      | avoid letting candidates go cold                                                        |
| `* *`    | recruiter      | assign custom tags to candidates                                                                                | know more about this candidate without reading a long file                              |
| `* *`    | recruiter      | use boolean-style search (AND/OR/NOT)                                                                           | build precise candidate lists without clicking filters                                  |
| `* *`    | recruiter      | attach a link to a portfolio/LinkedIn/resume location                                                           | jump to supporting material quickly                                                     |
| `* *`    | recruiter      | record the source of a candidate (referral, LinkedIn, event, inbound)                                           | evaluate sourcing channels                                                              |
| `* *`    | recruiter      | assign a simple rating (e.g., 1–5) to a candidate                                                               | compare candidates objectively                                                          |
| `* *`    | recruiter      | sort contacts by "Date Added"                                                                                   | instantly see the most recent people I met today                                        |
| `* *`    | recruiter      | sort candidates by rating                                                                                       | prioritise stronger applicants                                                          |
| `* *`    | recruiter      | export a filtered list (e.g., CSV)                                                                              | share shortlists with hiring managers or other systems                                  |
| `*`      | recruiter      | clear all candidates                                                                                            | reset the system for a new hiring cycle                                                 |
| `* *`    | recruiter      | see a help command listing all available commands                                                               | learn how to use the system if I were to forget                                         |
| `* * *`  | recruiter      | see warnings before any candidates are deleted                                                                  | not delete a bunch of candidates by accident                                            |
| `*`      | expert user    | execute a "Mass Categorize" command                                                                             | move an entire group of candidates from one recruitment stage to another simultaneously |
| `* *`    | recruiter      | be able to delete a group with specific tags (e.g. rejected)                                                    | not have to delete their contacts one by one                                            |
| `* * *`  | recruiter      | receive clear error messages when I enter invalid commands                                                      | correct mistakes easily                                                                 |
| `* *`    | recruiter      | undo the previous command                                                                                       | reverse accidental actions                                                              |
| `* *`    | recruiter      | add/navigate through the list using only the keyboard                                                           | keep my hands on my keyboard and save time                                              |
| `* *`    | recruiter      | use the up/down arrow keys to browse past CLI commands                                                          | reuse or edit past inputs quickly                                                       |
| `* * *`  | recruiter      | search for a candidate by name (supporting partial input like "joh")                                            | locate a contact quickly without needing to type the exact full name                    |
| `*`      | recruiter      | lock certain fields (e.g., ID, email) from accidental edits                                                     | keep critical identifiers stable                                                        |
| `* *`    | recruiter      | add structured fields like location, work authorization, and salary expectations                                | screen faster and avoid mismatches                                                      |
| `*`      | recruiter      | define a contact “lifecycle” rule (e.g., archive after 90 days inactive)                                        | keep my database current with minimal effort                                            |
| `* *`    | recruiter      | use command aliases (e.g., a for add, d for delete, e for edit)                                                 | type commands faster                                                                    |
| `*`      | recruiter      | use tab-based auto-completion for commands and prefixes                                                         | not need to memorise full syntax                                                        |
| `* *`    | recruiter      | find contacts using other fields (phone, email, tag, address)                                                   | retrieve someone even if I forget their name                                            |
| `* * *`  | recruiter      | filter contacts by tag                                                                                          | group similar contacts easily                                                           |
| `*`      | recruiter      | paste a messy block of text (email signature / LinkedIn snippet) and have fields auto-suggested                 | avoid manual retyping                                                                   |
| `* *`    | recruiter      | use positional arguments for fast entry (e.g., add "John Doe" 91234567 [john@email.com](mailto:john@email.com)) | input data quickly                                                                      |
| `*`      | recruiter      | view “recently modified” candidates                                                                             | resume work where I left off without remembering names                                  |
| `* *`    | recruiter      | view “recently contacted” and “overdue follow-ups” lists                                                        | not lose candidates due to slow response                                                |
| `*`      | recruiter      | view a history of recent actions                                                                                | track what changes were made                                                            |
| `* *`    | recruiter      | click on a contact to view detailed information                                                                 | keep the main list clean and readable                                                   |
| `*`      | recruiter      | see a preview of the contact being added before confirmation                                                    | verify details before committing                                                        |
| `* * *`  | recruiter      | save the list of contacts automatically                                                                         | not lose progress if something unexpected happens (power outage)                        |
| `*`      | recruiter      | bulk-apply a “cooldown” status after rejection                                                                  | not accidentally re-contact too soon                                                    |
| `* *`    | recruiter      | add or remove tags from multiple contacts at once                                                               | categorise people faster                                                                |
| `*`      | recruiter      | record time zone and preferred contact hours                                                                    | make outreach more effective and less intrusive                                         |
| `*`      | recruiter      | add custom CLI commands                                                                                         | type commands more efficiently                                                          |
| `* *`    | recruiter      | mark a contact as favourite or important                                                                        | quickly find high-priority candidates                                                   |
| `* * *`  | recruiter      | see a confirmation message after adding, editing, or deleting a contact                                         | know my command worked                                                                  |
| `* *`    | recruiter      | archive contacts instead of deleting                                                                            | keep historical records                                                                 |
| `*`      | recruiter      | copy a candidate's email address to my clipboard with a single command                                          | switch to my email app and paste it                                                     |
| `* *`    | recruiter      | see a count of how many candidates match my current filter                                                      | know if my search pool is too broad                                                     |
| `* * *`  | recruiter      | delete a specific contact                                                                                       | keep my contact list from getting cluttered                                             |
| `* * *`  | recruiter      | navigate through the list                                                                                       | view my contacts                                                                        |


### Use cases

(For all use cases below, the **System** is the `AddressBook` and the **Actor** is the `user`, unless specified otherwise)

**Use case: Delete a person**

**MSS**

1.  User requests to list persons
2.  AddressBook shows a list of persons
3.  User requests to delete a specific person in the list
4.  AddressBook deletes the person

    Use case ends.

**Extensions**

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. AddressBook shows an error message.

      Use case resumes at step 2.

*{More to be added}*

### Non-Functional Requirements

1.  Should work on any _mainstream OS_ as long as it has Java `17` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.
4.  Data should be automatically saved to the local hard disk after every command that modifies data to prevent loss during a crash.
5.  Candidate phone numbers and personal data must be stored in a human-readable but localized format, ensuring no data is transmitted to external servers without consent.
6.  100% of the application’s features must be accessible via keyboard shortcuts, catering to high-speed "power users" in recruiting
7.  The system must provide immediate visual confirmation (e.g., a status message or color change) within 50ms of a command being executed to confirm the action was successful.
8.  The "Delete" function must ensure that all traces of a candidate’s phone number are purged from the local state, helping the manager remain compliant with data privacy laws like GDPR or PDPA.

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **Candidate**: An individual whose contact details and hiring status are being tracked in the system.
* **CLI (Command Line Interface)**: The primary text-based input method used by the Talent Scout to interact with the application without using a mouse.
* **Auto-completion**: A productivity feature where the user presses the Tab key to have the system suggest or finish a command or parameter, reducing typing effort.
* **Batch-import**: The process of pulling multiple candidate records into the system at once from an external file (like a university portal list) rather than adding them individually.
* **Duplicate Entry**: A record that matches an existing contact based on the Primary Key (the unique combination of Name + Phone Number).
* **Source**: The origin of the candidate's data (e.g., LinkedIn, GitHub, or a specific University Career Portal).

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
