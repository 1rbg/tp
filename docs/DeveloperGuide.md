---
layout: page
title: Developer Guide
---

* Table of Contents
    * [Acknowledgements](#acknowledgements)
    * [Setting up, getting started](#setting-up-getting-started)
    * [Design](#design)
        * [Architecture](#architecture)
        * [UI component](#ui-component)
        * [Logic component](#logic-component)
        * [Model component](#model-component)
        * [Storage component](#storage-component)
        * [Common classes](#common-classes)
    * [Implementation](#implementation)
        * [\[Implemented\] Schedule feature](#implemented-schedule-feature)
            * [Proposed Implementation](#proposed-implementation)
        * [\[Implemented\] Contact archiving](#implemented-contact-archiving)
            * [Proposed Implementation](#proposed-implementation)
        * [\[Proposed\] Undo/redo feature](#proposed-undoredo-feature)
            * [Proposed Implementation](#proposed-implementation)
            * [Design considerations:](#design-considerations)
    * [Documentation, logging, testing, configuration, dev-ops](#documentation-logging-testing-configuration-dev-ops)
    * [Appendix: Requirements](#appendix-requirements)
        * [Product scope](#product-scope)
        * [User stories](#user-stories)
        * [Use cases](#use-cases)
        * [Non-Functional Requirements](#non-functional-requirements)
        * [Glossary](#glossary)
    * [Appendix: Instructions for manual testing](#appendix-instructions-for-manual-testing)
        * [Launch and shutdown](#launch-and-shutdown)
        * [Deleting a person](#deleting-a-person)

--------------------------------------------------------------------------------------------------------------------

## Acknowledgements

* Calendar for Schedule feature adapted from: https://gist.github.com/Da9el00/f4340927b8ba6941eb7562a3306e93b6

--------------------------------------------------------------------------------------------------------------------

## Setting up, getting started

Refer to the guide [_Setting up and getting started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## Design

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document `docs/diagrams` folder. Refer to the [
_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create
and edit diagrams.
</div>

### Architecture

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** (consisting of
classes [`Main`](https://github.com/AY2324S2-CS2103T-T17-4/tp/tree/master/src/main/java/seedu/address/Main.java)
and [`MainApp`](https://github.com/AY2324S2-CS2103T-T17-4/tp/tree/master/src/main/java/seedu/address/MainApp.java)) is
in charge of the app launch and shut down.

* At app launch, it initializes the other components in the correct sequence, and connects them up with each other.
* At shut down, it shuts down the other components and invokes cleanup methods where necessary.

The bulk of the app's work is done by the following four components:

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues
the command `delete 98765432`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding
  API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using
the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component
through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the
implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component

The **API** of this component is specified
in [`Ui.java`](https://github.com/AY2324S2-CS2103T-T17-4/tp/tree/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts
e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`,
inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the
visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that
are in the `src/main/resources/view` folder. For example, the layout of
the [`MainWindow`](https://github.com/AY2324S2-CS2103T-T17-4/tp/tree/master/src/main/java/seedu/address/ui/MainWindow.java)
is specified
in [`MainWindow.fxml`](https://github.com/AY2324S2-CS2103T-T17-4/tp/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component

The **API** of this component is specified in [`Logic.java`](https://github.com/AY2324S2-CS2103T-T17-4/tp/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

The sequence diagram below illustrates the interactions within the `Logic` component, taking 
`execute("delete 98765432")` API call as an example.

![Interactions Inside the Logic Component for the `delete 98765432` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline continues till the end of diagram.
</div>

How the `Logic` component works:

1. When `Logic` is called upon to execute a command, it is passed to an `AddressBookParser` object which in turn creates
   a parser that matches the command (e.g., `DeleteCommandParser`) and uses it to parse the command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `DeleteCommand`) which
   is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to delete a person).<br>
   Note that although this is shown as a single step in the diagram above (for simplicity), in the code it can take
   several interactions (between the command object and the `Model`) to achieve.
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:

* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a
  placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse
  the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as
  a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser`
  interface so that they can be treated similarly where possible e.g, during testing.

### Model component

**API
** : [`Model.java`](https://github.com/AY2324S2-CS2103T-T17-4/tp/tree/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which
  is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to
  this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as
  a `ReadOnlyUserPref` objects.
* stores a `Schedule` object that represents the employees' work schedule.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they
  should make sense on their own without depending on other components)

<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<img src="images/BetterModelClassDiagram.png" width="450" />

</div>

### Storage component

**API
** : [`Storage.java`](https://github.com/AY2324S2-CS2103T-T17-4/tp/tree/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,

* can save both address book data and user preference data in JSON format, and read them back into corresponding
  objects.
* inherits from all of `AddressBookStorage`, `UserPrefStorage` and `ScheduleStorage`, which means it can be treated as 
  either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects
  that belong to the `Model`)

### Common classes

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## Implementation

This section describes some noteworthy details on how certain features are implemented.

### \[Implemented\] Schedule feature
#### Proposed Implementation

The current implementation of the schedule feature is facilitated by the `Schedule` interface. It contains a 
`Set<ScheduleDate>`, which contains the `ScheduleDate` objects that represent the dates in the schedule and hold a
list of who is working on that particular date.

Given below is an example usage scenario and how the schedule mechanism behaves at each step.

Step 1. The user launches the application for the first time. A concrete class implementing `Schedule` will be 
initialised with an empty set of `ScheduleDate` objects. The `ModelManager` is then initialised with the `Schedule`. 

Step 2. The user executes `schedule 98765432 2024-04-15` command to add a schedule entry for the person with phone 
number `98765432` on the date `2024-04-15`. The old `Schedule` object is then updated in the `Model` through the calling 
of `Model#addPersonToSchedule()`, which creates a new `ScheduleDate` object that corresponds to the date 
`2024-04-15`, and adds the person with phone number `98765432` to the list of people working on that date. The UI 
changes screen to show the updated schedule.

<img src="images/ScheduleSequenceDiagram.png"/>

Step 3. The user decides that he would like to add another person on the same date. The user executes `schedule 
92345678 2024-04-15` command to add a schedule entry for the person with phone number `92345678` on the date 
`2024-04-15`. The old `Schedule` object is then updated in the `Model` through the calling of 
`Model#addPersonToSchedule()`, which uses the existing `ScheduleDate` object associated with `2024-04-15` that 
corresponds to the date `2024-04-15`, and adds the person with phone number `98765432` to the list of people working 
on that date. The UI changes screen to show the updated schedule.

Step 4. The user decides that he would like to remove the schedule entry. The user executes `unschedule 98765432 
2024-04-15` command to remove the schedule entry for the person with phone number `98765432` on the date 
`2024-04-15`. The `unschedule` command removes a person with phone number `98765432` from the existing 
`ScheduleDate` with date `2024-04-15`. The UI changes screen to show the updated schedule.

### \[Implemented\] Contact archiving
#### Proposed Implementation

The current implementation of contact archiving (and by virtue un-archiving) is facilitated by the `ArchiveCommand`
and `UnarchiveCommand`. It extends `Command` and is responsible for updating the `Person` object's `Archive` field.

Given below is an example usage scenario and how the archive/unarchive mechanism behaves at each step.

Step 1. The user launches the application. Existing contacts in storage are loaded into the `AddressBook`, the `ModelManager`
is then initialised with the `AddressBook`. The `Person` objects in the `AddressBook` that have `ArchiveStatus` field set to 'false' 
are filtered by `ModelManager` into the `ObservableList<Person>` to be displayed in the UI.

Step 2. The user executes `archive 98765432` command to archive the person in the address book. The `archive` command 
creates a new `Person` object that corresponds to all of the selected `Person` object's fields, except for the 
`ArchiveStatus` field which it intialises as 'true'. The old `Person` object is then updated in the `AddressBook` through 
the calling of `Model#archivePerson()`. The UI updates the `ObservableList<Person>` to reflect the changes made.

<img src="images/ArchiveSequenceDiagram.png"/>

Step 3. The user decides that he would like to view the archived contact. The user executes `list archive` command to
view all archived contacts. The `list archive` command filters the `Person` objects in the `AddressBook` that have 
`ArchiveStatus` field set to 'true' into the `ObservableList<Person>` to be displayed in the UI.

Step 4. The employee of the archived contact decides to return to the company. The user executes `unarchive 98765432`
command to unarchive the person in the address book. The `unarchive` command creates a new `Person` object that 
corresponds to all of the selected `Person` object's fields, except for the `ArchiveStatus` field which it intialises
as 'false'. The old `Person` object is then updated in the `AddressBook` through the calling of `Model#archivePerson()`.

### \[Proposed\] Undo/redo feature

#### Proposed Implementation

The proposed undo/redo mechanism is facilitated by `VersionedAddressBook`. It extends `AddressBook` with an undo/redo
history, stored internally as an `addressBookStateList` and `currentStatePointer`. Additionally, it implements the
following operations:

* `VersionedAddressBook#commit()`— Saves the current address book state in its history.
* `VersionedAddressBook#undo()`— Restores the previous address book state from its history.
* `VersionedAddressBook#redo()`— Restores a previously undone address book state from its history.

These operations are exposed in the `Model` interface as `Model#commitAddressBook()`, `Model#undoAddressBook()`
and `Model#redoAddressBook()` respectively.

Given below is an example usage scenario and how the undo/redo mechanism behaves at each step.

Step 1. The user launches the application for the first time. The `VersionedAddressBook` will be initialized with the
initial address book state, and the `currentStatePointer` pointing to that single address book state.

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command
calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes
to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book
state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also
calls `Model#commitAddressBook()`, causing another modified address book state to be saved into
the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing
the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer`
once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how an undo operation goes through the `Logic` component:

![UndoSequenceDiagram](images/UndoSequenceDiagram-Logic.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

Similarly, how an undo operation goes through the `Model` component is shown below:

![UndoSequenceDiagram](images/UndoSequenceDiagram-Model.png)

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once
to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such
as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`.
Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not
pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be
purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern
desktop applications follow.

![UndoRedoState5](images/UndoRedoState5.png)

The following activity diagram summarizes what happens when a user executes a new command:

<img src="images/CommitActivityDiagram.png" width="250" />

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

--------------------------------------------------------------------------------------------------------------------

## Documentation, logging, testing, configuration, dev-ops

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## Appendix: Requirements

### Product scope

**Target user profile**:

* has a need to manage a significant number of employee contacts and banking details
* has a need to track employee worked hours
* has a need to tabulate payroll for employees with different pay rates
* prefer desktop apps over other types
* can type fast
* prefers typing to mouse interactions
* is reasonably comfortable using CLI apps

**Value proposition**: manage employee salary disbursement faster than a typical exel sheet with manual calculations.

### User stories

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

| Priority | As a …​                                                        | I want to …​                                       | So that I can…​                                                       |
|----------|----------------------------------------------------------------|----------------------------------------------------|-----------------------------------------------------------------------|
| `* *`    | new user                                                       | see a tutorial and usage instructions              | familiarise with FnBuddy' features                                    |
| `* * *`  | user                                                           | add an employee contact with banking details       | quickly access the employee's banking details for salary disbursement |
| `* * *`  | user                                                           | delete an employee contact                         | remove entries that I no longer need                                  |
| `* * *`  | user                                                           | view all employee contacts                         |                                                                       |
| `* *`    | user                                                           | track an employee's weekly worked hours            | access it for employee salary calculation                             |
| `* *`    | user handling employees with a variety of employment contracts | tag an employee contact with their employment type | retrieve the salary rate of the employee                              |
| `* *`    | user with human error tendencies                               | retrieve an employee's calculated pay              | avoid paying out an incorrect salary amount                           |
| `* *`    | user                                                           | edit an employee contact details                   | keep the employee's details up to date                                |
| `* *`    | user                                                           | schedule my employee shifts                        | plan workload more easily                                             |
| `*`      | user with many employees                                       | sort employees contacts by name                    | locate the employee contact easily                                    |
| `*`      | user with potential returning employees                        | archive an employee contact                        | reopen the employee's details when they return easily                 |
| `*`      | user with forgetfulness                                        | search for contacts by keyword                     | find contacts without needing to provide their full name              |
| `*`      | user with many employees                                       | filter employee contacts by tag(s)                 | identify which employee(s) are of that employment type                |

*{More to be added}*

### Use cases

(For all use cases below, the **System** is the `FnBuddy` and the **Actor** is the `user`, unless specified otherwise)

**Use case: Add an employee**

**MSS**

1.  User requests to add employee contact
2.  FnBuddy requests contact information of employee
3.  User provides required information
4.  FnBuddy adds the employee contact

   Use case ends.

**Extensions**

* 3a. The given contact information is invalid.

    * 3a1. FnBuddy shows an error message.

      Use case resumes at step 2.

**Use case: Delete an employee**

**MSS**

1.  User requests to delete an employee contact
2.  FnBuddy requests employee contact 
3.  User provides required information
4.  FnBuddy deletes the employee contact

    Use case ends.

**Extensions**

* 2a. The contact book is empty.
    * 2a1. FnBuddy shows an error message.

        Use case ends


* 3a. The given contact information is invalid.

    * 3a1. FnBuddy shows an error message.

      Use case resumes at step 2.

**Use case: List all employee contacts**

**MSS**

1.  User requests to list all employee contacts
2.  FnBuddy lists all employee contacts

    Use case ends.

**Extensions**

* 2a. The contact book is empty.
    * 2a1. FnBuddy shows an error message.

      Use case ends

**Use case: Track employee's work hours**

**MSS**

1.  User requests to add employee's work hours
2.  FnBuddy requests employee contact and work hours
3.  User provides required information
4.  FnBuddy tracks the employee's work hours

    Use case ends.

**Extensions**

* 2a. The contact book is empty.
    * 2a1. FnBuddy shows an error message.

      Use case ends


* 3a. The given contact information or working hours is invalid.

    * 3a1. FnBuddy shows an error message.

      Use case resumes at step 2.


*{More to be added}*

### Non-Functional Requirements

1. Should work on any _mainstream OS_ as long as it has Java `11` or above installed.
2. Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3. A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be
   able to accomplish most of the tasks faster using commands than using the mouse.
4. The app should be highly reliable, minimizing downtime and ensuring continuous availability during operational hours.
5. It should have built-in mechanisms for data backup and recovery to prevent loss of employee contact information.

### Glossary

* **Mainstream OS**: Windows, Linux, Unix, MacOS
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **Worked hours**: The number of hours an employee has spent working in a month
* **Bank Details**: Account number for salary disbursement, 7-11 digits in length.

--------------------------------------------------------------------------------------------------------------------

## Appendix: Instructions for manual testing

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

### Launch and shutdown

1. Initial launch

    1. Download the jar file and copy into an empty folder

    1. Double-click the jar file Expected: Shows the GUI with a set of sample contacts. The window size may not be
       optimum.

1. Saving window preferences

    1. Resize the window to an optimum size. Move the window to a different location. Close the window.

    1. Re-launch the app by double-clicking the jar file.<br>
       Expected: The most recent window size and location is retained.

1. _{ more test cases …​ }_

### Deleting a person

1. Deleting a person while all persons are being shown

    1. Prerequisites: List all persons using the `list` command. Multiple persons in the list.

    1. Test case: `delete 1`<br>
       Expected: First contact is deleted from the list. Details of the deleted contact shown in the status message.
       Timestamp in the status bar is updated.

    1. Test case: `delete 0`<br>
       Expected: No person is deleted. Error details shown in the status message. Status bar remains the same.

    1. Other incorrect delete commands to try: `delete`, `delete x`, `...` (where x is larger than the list size)<br>
       Expected: Similar to previous.

