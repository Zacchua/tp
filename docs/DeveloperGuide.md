---
layout: page
title: Developer Guide
---
* Table of Contents
{:toc}
1. [Acknowledgements](#acknowledgements)
2. [Setting Up, Getting Started](#setting-up-getting-started)
3. [Design](#design)
    1. [Architecture](#architecture)
    2. [UI Component](#ui)
    3. [Logic Component](#logic)
    4. [Model Component](#model)
    5. [Storage Component](#storage)
    6. [Common Classes](#common)
4. [Implementation](#implementation)
5. [Documentation](#docs)
6. [Appendix: Requirements](#requirements)
    1. [Product Scope](#scope)
    2. [User Stories](#user-stories)
    3. [Use Cases](#use-cases)
    4. [Non-Functional Requirements](#nfr)
    5. [Glossary](#glossary)
    

--------------------------------------------------------------------------------------------------------------------

## **Acknowledgements** <a name="acknowledgements"></a>

* {list here sources of all reused/adapted ideas, code, documentation, and third-party libraries -- include links to the original source as well}
* Adapted from [_SE-Education_](https://se-education.org/addressbook-level3/DeveloperGuide.html) 's original *AddressBook*

--------------------------------------------------------------------------------------------------------------------

## **Setting Up, Getting Started** <a name="setting-up-getting-started"></a>

Refer to the guide [_Setting Up and Getting Started_](SettingUp.md).

--------------------------------------------------------------------------------------------------------------------

## **Design** <a name="design"></a>

<div markdown="span" class="alert alert-primary">

:bulb: **Tip:** The `.puml` files used to create diagrams in this document can be found in the [diagrams](https://github.com/se-edu/addressbook-level3/tree/master/docs/diagrams/) folder. Refer to the [_PlantUML Tutorial_ at se-edu/guides](https://se-education.org/guides/tutorials/plantUml.html) to learn how to create and edit diagrams.
</div>

### Architecture <a name="architecture"></a>

<img src="images/ArchitectureDiagram.png" width="280" />

The ***Architecture Diagram*** given above explains the high-level design of the App.

Given below is a quick overview of main components and how they interact with each other.

**Main components of the architecture**

**`Main`** has two classes called [`Main`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/Main.java) and [`MainApp`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/MainApp.java). It is responsible for,
* At app launch: Initializes the components in the correct sequence, and connects them up with each other.
* At shut down: Shuts down the components and invokes cleanup methods where necessary.

[**`Commons`**](#common-classes) represents a collection of classes used by multiple other components.

The rest of the App consists of four components.

* [**`UI`**](#ui-component): The UI of the App.
* [**`Logic`**](#logic-component): The command executor.
* [**`Model`**](#model-component): Holds the data of the App in memory.
* [**`Storage`**](#storage-component): Reads data from, and writes data to, the hard disk.


**How the architecture components interact with each other**

The *Sequence Diagram* below shows how the components interact with each other for the scenario where the user issues the command `delete 1`.

<img src="images/ArchitectureSequenceDiagram.png" width="574" />

Each of the four main components (also shown in the diagram above),

* defines its *API* in an `interface` with the same name as the Component.
* implements its functionality using a concrete `{Component Name}Manager` class (which follows the corresponding API `interface` mentioned in the previous point.

For example, the `Logic` component defines its API in the `Logic.java` interface and implements its functionality using the `LogicManager.java` class which follows the `Logic` interface. Other components interact with a given component through its interface rather than the concrete class (reason: to prevent outside component's being coupled to the implementation of a component), as illustrated in the (partial) class diagram below.

<img src="images/ComponentManagers.png" width="300" />

The sections below give more details of each component.

### UI component <a name="ui"></a>

The **API** of this component is specified in [`Ui.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/Ui.java)

![Structure of the UI Component](images/UiClassDiagram.png)

The UI consists of a `MainWindow` that is made up of parts e.g.`CommandBox`, `ResultDisplay`, `PersonListPanel`, `StatusBarFooter` etc. All these, including the `MainWindow`, inherit from the abstract `UiPart` class which captures the commonalities between classes that represent parts of the visible GUI.

The `UI` component uses the JavaFx UI framework. The layout of these UI parts are defined in matching `.fxml` files that are in the `src/main/resources/view` folder. For example, the layout of the [`MainWindow`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/ui/MainWindow.java) is specified in [`MainWindow.fxml`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/resources/view/MainWindow.fxml)

The `UI` component,

* executes user commands using the `Logic` component.
* listens for changes to `Model` data so that the UI can be updated with the modified data.
* keeps a reference to the `Logic` component, because the `UI` relies on the `Logic` to execute commands.
* depends on some classes in the `Model` component, as it displays `Person` object residing in the `Model`.

### Logic component <a name="logic"></a>

**API** : [`Logic.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/logic/Logic.java)

Here's a (partial) class diagram of the `Logic` component:

<img src="images/LogicClassDiagram.png" width="550"/>

How the `Logic` component works:
1. When `Logic` is called upon to execute a command, it uses the `AddressBookParser` class to parse the user command.
1. This results in a `Command` object (more precisely, an object of one of its subclasses e.g., `AddCommand`) which is executed by the `LogicManager`.
1. The command can communicate with the `Model` when it is executed (e.g. to add a person).
1. The result of the command execution is encapsulated as a `CommandResult` object which is returned back from `Logic`.

The Sequence Diagram below illustrates the interactions within the `Logic` component for the `execute("delete 1")` API call.

![Interactions Inside the Logic Component for the `delete 1` Command](images/DeleteSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `DeleteCommandParser` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.
</div>

Here are the other classes in `Logic` (omitted from the class diagram above) that are used for parsing a user command:

<img src="images/ParserClasses.png" width="600"/>

How the parsing works:
* When called upon to parse a user command, the `AddressBookParser` class creates an `XYZCommandParser` (`XYZ` is a placeholder for the specific command name e.g., `AddCommandParser`) which uses the other classes shown above to parse the user command and create a `XYZCommand` object (e.g., `AddCommand`) which the `AddressBookParser` returns back as a `Command` object.
* All `XYZCommandParser` classes (e.g., `AddCommandParser`, `DeleteCommandParser`, ...) inherit from the `Parser` interface so that they can be treated similarly where possible e.g, during testing.

### Model component <a name="model"></a>
**API** : [`Model.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/model/Model.java)

<img src="images/ModelClassDiagram.png" width="450" />


The `Model` component,

* stores the address book data i.e., all `Person` objects (which are contained in a `UniquePersonList` object).
* stores the currently 'selected' `Person` objects (e.g., results of a search query) as a separate _filtered_ list which is exposed to outsiders as an unmodifiable `ObservableList<Person>` that can be 'observed' e.g. the UI can be bound to this list so that the UI automatically updates when the data in the list change.
* stores a `UserPref` object that represents the user’s preferences. This is exposed to the outside as a `ReadOnlyUserPref` objects.
* does not depend on any of the other three components (as the `Model` represents data entities of the domain, they should make sense on their own without depending on other components)

<div markdown="span" class="alert alert-info">:information_source: **Note:** An alternative (arguably, a more OOP) model is given below. It has a `Tag` list in the `AddressBook`, which `Person` references. This allows `AddressBook` to only require one `Tag` object per unique tag, instead of each `Person` needing their own `Tag` objects.<br>

<img src="images/BetterModelClassDiagram.png" width="450" />

</div>


### Storage component <a name="storage"></a>

**API** : [`Storage.java`](https://github.com/se-edu/addressbook-level3/tree/master/src/main/java/seedu/address/storage/Storage.java)

<img src="images/StorageClassDiagram.png" width="550" />

The `Storage` component,
* can save both address book data and user preference data in json format, and read them back into corresponding objects.
* inherits from both `AddressBookStorage` and `UserPrefStorage`, which means it can be treated as either one (if only the functionality of only one is needed).
* depends on some classes in the `Model` component (because the `Storage` component's job is to save/retrieve objects that belong to the `Model`)

### Common classes <a name="common"></a>

Classes used by multiple components are in the `seedu.addressbook.commons` package.

--------------------------------------------------------------------------------------------------------------------

## **Implementation** <a name="implementation"></a>

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

![UndoRedoState0](images/UndoRedoState0.png)

Step 2. The user executes `delete 5` command to delete the 5th person in the address book. The `delete` command calls `Model#commitAddressBook()`, causing the modified state of the address book after the `delete 5` command executes to be saved in the `addressBookStateList`, and the `currentStatePointer` is shifted to the newly inserted address book state.

![UndoRedoState1](images/UndoRedoState1.png)

Step 3. The user executes `add n/David …​` to add a new person. The `add` command also calls `Model#commitAddressBook()`, causing another modified address book state to be saved into the `addressBookStateList`.

![UndoRedoState2](images/UndoRedoState2.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If a command fails its execution, it will not call `Model#commitAddressBook()`, so the address book state will not be saved into the `addressBookStateList`.

</div>

Step 4. The user now decides that adding the person was a mistake, and decides to undo that action by executing the `undo` command. The `undo` command will call `Model#undoAddressBook()`, which will shift the `currentStatePointer` once to the left, pointing it to the previous address book state, and restores the address book to that state.

![UndoRedoState3](images/UndoRedoState3.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index 0, pointing to the initial AddressBook state, then there are no previous AddressBook states to restore. The `undo` command uses `Model#canUndoAddressBook()` to check if this is the case. If so, it will return an error to the user rather
than attempting to perform the undo.

</div>

The following sequence diagram shows how the undo operation works:

![UndoSequenceDiagram](images/UndoSequenceDiagram.png)

<div markdown="span" class="alert alert-info">:information_source: **Note:** The lifeline for `UndoCommand` should end at the destroy marker (X) but due to a limitation of PlantUML, the lifeline reaches the end of diagram.

</div>

The `redo` command does the opposite — it calls `Model#redoAddressBook()`, which shifts the `currentStatePointer` once to the right, pointing to the previously undone state, and restores the address book to that state.

<div markdown="span" class="alert alert-info">:information_source: **Note:** If the `currentStatePointer` is at index `addressBookStateList.size() - 1`, pointing to the latest address book state, then there are no undone AddressBook states to restore. The `redo` command uses `Model#canRedoAddressBook()` to check if this is the case. If so, it will return an error to the user rather than attempting to perform the redo.

</div>

Step 5. The user then decides to execute the command `list`. Commands that do not modify the address book, such as `list`, will usually not call `Model#commitAddressBook()`, `Model#undoAddressBook()` or `Model#redoAddressBook()`. Thus, the `addressBookStateList` remains unchanged.

![UndoRedoState4](images/UndoRedoState4.png)

Step 6. The user executes `clear`, which calls `Model#commitAddressBook()`. Since the `currentStatePointer` is not pointing at the end of the `addressBookStateList`, all address book states after the `currentStatePointer` will be purged. Reason: It no longer makes sense to redo the `add n/David …​` command. This is the behavior that most modern desktop applications follow.

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

### \[Proposed\] Data archiving

_{Explain here how the data archiving feature will be implemented}_


--------------------------------------------------------------------------------------------------------------------

## **Documentation, logging, testing, configuration, dev-ops** <a name="docs"></a>

* [Documentation guide](Documentation.md)
* [Testing guide](Testing.md)
* [Logging guide](Logging.md)
* [Configuration guide](Configuration.md)
* [DevOps guide](DevOps.md)

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Requirements** <a name="requirements"></a>

### Product scope <a name="scope"></a>

**Target user profile**:

* avid social media user
* prefers desktop apps over other types
* prefers typing to mouse interactions
* needs to manage a significant number of contacts
* wants to catch up with their contacts’ activities quickly 
* wants to connect with their followers/friends on various social media platforms through an all-in-one dashboard

**Value proposition**: 

Our product serves as an integrated dashboard for a user to retrieve the social media activities and account information of his/her contacts. This makes it seamless for the user to interact with his/her contacts instead of having to access each social media account that the contact owns.

### User stories <a name="user-stories"></a>

Priorities: High (must have) - `* * *`, Medium (nice to have) - `* *`, Low (unlikely to have) - `*`

<br/>

_Core Functionalities_

|Priority| As a / an …​                              | I want to …​                                                                    | So that I can…​                                                                                      |
|--------| -------------------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
|`* * *` | beginner user                                | add contacts                                                                       | access my contacts' details                                                                             |
|`* * *` | beginner user                                | delete contacts                                                                    | remove irrelevant entries                                                                               |
|`* * *` | beginner user                                | access the social media handles of my contacts                                     | have quicker access to my contacts' social media pages                                                  |
|`* * *` | forgetful user                               | save my contacts' social media handles                                             | easily access my contact's social media account without having to recall the exact handle               |
|`* *`   | user                                         | browse a list of all my contacts                                                   | view all my contacts at a glance                                                                        |
|`*`     | beginner user                                | update contacts                                                                    | modify existing social media handles and add new ones when they are created                             |
|`*`     | beginner user                                | view the recent feed of my contacts                                                | have more meaningful and frequent interactions with my contacts on multiple platforms                   |

<br/>

_Guide for New Users_

|Priority| As a / an …​                              | I want to …​                                                                    | So that I can…​                                                                                      |
|--------| -------------------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
|`* *`   | new user                                     | view the User Guide                                                                | learn how to use SociaLite comprehensively                                                              |
|`* *`   | new user                                     | view sample contacts when I initialise the app                                     | try out the features without having to add actual data                                                  |
|`* *`   | new / returning user                         | access in-app guidance for a specific command                                      | (re)learn the syntax of a selected command without having to open the User Guide via a browser          |
|`* *`   | new user adopting the app for my own use     | purge all data                                                                     | delete sample contacts and add real data                                                                |

<br/>

_Organization of Contacts_

|Priority| As a / an …​                              | I want to …​                                                                    | So that I can…​                                                                                      |
|--------| -------------------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
|`* *`   | organized user                               | create categories to group my contacts                                             | organize my list of contacts                                                                            |
|`* *`   | organized user                               | edit categories as and when required                                               | repurpose such pre-existing categories                                                                  |
|`* *`   | organized user                               | query a group of contacts                                                          | have greater ease of access to my frequent contacts and efficiently contact people for similar purposes |
|`*`     | organised user                               | delete categories associated with contacts                                         | declutter my address book when the category is no longer relevant                                       |
|`*`     | intermediate user                            | filter contacts based on social media platform                                     | find out whose social media contacts I have not gotten and request it from them                         |
|`*`     | user                                         | track when I last queried my contact's information                                 | find out who I have not communicated with for an extended period of time                                |

<br/>

_Ease of Accessibility_

|Priority| As a / an …​                              | I want to …​                                                                    | So that I can…​                                                                                      |
|--------| -------------------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
|`* *`   | avid social media user with many connections | be redirected to my chosen contact's social media platform                         | avoid initialising every social media platform and search for his/her account manually                  |
|`*`     | expert user                                  | customise the information presented to me when the app is initialised              | view the social media contacts of my close friends quickly without keying in additional prompts         |
|`*`     | frequent user                                | create keyboard shortcuts/hotkeys                                                  | quickly pull up the social media handles of a contact-of-interest in the least keystrokes possible      |

<br/>

_Customization of Contacts_

|Priority| As a / an …​                              | I want to …​                                                                    | So that I can…​                                                                                      |
|--------| -------------------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
|`*`     | user                                         | add a profile picture for my contacts                                              | better recognize their appearance                                                                       |
|`*`     | user                                         | add notes about contacts                                                           | recall specific items about my contacts                                                                 |
|`*`     | user                                         | add dates of special occasions (birthdays/appointments) associated with my contact | view important information of my contacts                                                               |
|`*`     | user                                         | set reminders for special occasions associated with my contact                     | be alerted of these events                                                                              |
|`*`     | user                                         | view a dashboard of significant events associated with my contact                  | be reminded of these dates                                                                              |
|`*`     | user                                         | forward relevant details of an existing contact                                    | easily share such information upon request                                                              |

<br/>


### Use Cases <a name="use-cases"></a>

(For all use cases below, the **System** is `SociaLite` and the **Actor** is the `user`, unless specified otherwise)

***Basic Functionalities*** 

**Use Case 01: Add a person**

*MSS*

1.  User requests to add a new contact
2.  SociaLite adds the contact

    Use case ends.

*Extensions*

* 1a. User's input does not conform with the specified format.
    * 1a1. SociaLite shows an error message.
        
        Use case resumes at step 1.

<br/>
    
**Use Case 02: List entries in SociaLite**

*MSS*

1.  User requests to list contacts
2.  SociaLite shows a list of contacts

    Use case ends.

*Extensions*

* 2a. The list is empty.

  Use case ends.

<br/>

**Use Case 03: Edit entries in SociaLite**

*MSS*

1.  User requests to list contacts
2.  SociaLite shows a list of contacts
3.  User requests to edit a specific contact in the list
4.  SociaLite updates the contact with the specified input

    Use case ends.

*Extensions*

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. SociaLite shows an error message.

      Use case resumes at step 2.

* 3b. User's input does not conform with the specified format.

    * 3a1. SociaLite shows an error message.

      Use case resumes at step 2.

<br/>

**Use Case 04: Find a contact**

*MSS*

1.  User requests to locate a specific contact through keywords
2.  SociaLite shows a list of contacts that matches the given criteria

    Use case ends.

*Extensions*

* 2a. The list is empty as there are no matches found.

  Use case ends.

<br/>

**Use Case 05: Delete a person**

*MSS*

1.  User requests to list contacts
2.  SociaLite shows a list of contacts
3.  User requests to delete a specific contact in the list
4.  SociaLite deletes the contact

    Use case ends.

*Extensions*

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. SociaLite shows an error message.

      Use case resumes at step 2.


<br/>

***Organisation of Contacts***

**Use Case 06: Create new category**

*MSS*

1.  User types in command to create a new category (with category name)
2.  SociaLite shows a list of contacts
3.  User can select multiple contacts to add to the category
4.  User selects `create` button

    Use case ends.

*Extensions*

* 2a. Category exists.
  
    * 2a1. SociaLite shows an error message.

      Use case ends.

<br/>

**Use Case 07: Query categories**

*MSS*

1.  User types in `query` command (with category name)
2.  SociaLite shows a list of contacts in the category specified

    Use case ends.

*Extensions*

* 2a. Category does not exist.

    * 2a1. SociaLite shows an error message.

      Use case ends.

<br/>

**Use Case 08: Edit categories**

*MSS*

1.  User enters command to add contact to the category
2.  SociaLite shows a list of updated contacts in the category

    Use case ends.

<br/>

**Use Case 09: Delete categories**

*MSS*

1.  User requests to list contacts
2.  SociaLite shows a list of contacts
3.  User requests to delete a category tagged to a specific contact in the list
4.  SociaLite deletes the tag

    Use case ends.

*Extensions*

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. SociaLite shows an error message.

      Use case resumes at step 2.

* 3b. The given category does not exist.

    * 3a1. SociaLite shows an error message.

      Use case resumes at step 2.

<br/>

**Use Case 10: Retrieve last queried contact**

*MSS*

1.  User requests to retrieve the information of the last queried contact
2.  SociaLite returns the entry that fits this criteria

    Use case ends.


<br/>

***Customization Tools***

**Use Case 11: Add remark for a specific contact**

*MSS*

1.  User requests to **list contacts (UC02)** or **find contact (UC04)**
2.  SociaLite returns a list of contacts according to the UC called
3.  User specifies a remark to be added to the specified contact's entry
4.  SociaLite adds a remark to the specified contact's entry

    Use case ends.

*Extensions*

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. SociaLite shows an error message.

      Use case resumes at step 2.

<br/>

**Use Case 12: View contact card**

*MSS*

1.  User requests to **list contacts (UC02)** or **find contact (UC04)**
2.  SociaLite returns a list of contacts according to the UC called
3.  User specifies the index of his desired contact
4.  SociaLite opens a contact card which presents all previously stored details associated with the contact

    Use case ends.

*Extensions*

* 2a. The list is empty.

  Use case ends.

* 3a. The given index is invalid.

    * 3a1. SociaLite shows an error message.

      Use case resumes at step 2.

<br/>

***Help Guide & Exiting***

**Use Case 13: View User Guide**

*MSS*

1.  User requests to view User Guide
2.  SociaLite displays a link to User Guide and instructions to obtain in-app guidance for five selected commands

    Use case ends.

<br/>

**Use Case 14: View in-app guidance for selected commands**

*MSS*

1.  User requests to view in-app guidance for one out of five selected commands
2.  SociaLite returns an overview and quick guide of the command given as input

    Use case ends.

*Extensions* 

* 1a. The keyword given as input is invalid.

    * 1a1. SociaLite launches HelpWindow for **User Guide (UC13)** by default
    
        Use case ends.

<br/>

**Use Case 15: Purge contacts stored in SociaLite**

*MSS*

1.  User requests to clear all existing contacts (from demo version)
2.  SociaLite clears specimen data from memory
    
    Use case ends.

<br/>

**Use Case 16: Exit application**

*MSS*

1.  User types in command to exit application
2.  SociaLite closes

    Use case ends.



*{More to be added}*

<br/>



### Non-Functional Requirements <a name="nfr"></a>

1.  Should work on any _mainstream OS_ as long as it has Java `11` or above installed.
2.  Should be able to hold up to 1000 persons without a noticeable sluggishness in performance for typical usage.
3.  Should be intuitive enough for users of all technical backgrounds to operate.
4.  A user with above average typing speed for regular English text (i.e. not code, not system admin commands) should be able to accomplish most of the tasks faster using commands than using the mouse.

*{More to be added}*

### Glossary <a name="glossary"></a>

* **Mainstream OS**: Windows, Linux, Unix, OS-X
* **Private contact detail**: A contact detail that is not meant to be shared with others
* **Social Media**:  Various communication platforms such as Instagram, Facebook, Twitter, TikTok, Telegram

--------------------------------------------------------------------------------------------------------------------

## **Appendix: Instructions for manual testing**

Given below are instructions to test the app manually.

<div markdown="span" class="alert alert-info">:information_source: **Note:** These instructions only provide a starting point for testers to work on;
testers are expected to do more *exploratory* testing.

</div>

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
