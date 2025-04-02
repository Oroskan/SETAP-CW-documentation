Notes
=====

The Notes feature in "Learning App" allows users to create, edit, and manage notes within the app. 
Users can enter a title, subject, and content for each note. 
The notes are stored locally on the user's device and can be accessed at any time.

Creating a New Note
-----------------
When the user clicks the floating plus button, the application calls the ``_addNewNote`` method.
This method navigates to the ``CreateNotePage`` widget.
The ``CreateNotePage`` widget provides a user interface for writing and saving notes. 
It consists of three input fields:

1. Title: A short, descriptive title for the note
2. Subject: The subject or category of the note
3. Content: The main text content of the note

Users can fill out these fields and save the note by tapping the save button. 
The note will then be added to the list of notes in the app.

Saving a Note
-----------
When the user taps the save button, the ``_saveNote`` method is called. 
The method stores the title, subject and content into the database.

Deleting a Note
-------------
When the user swipes left on a note in the list, the ``_deleteNote`` method is called.
This method removes the note from the database and updates the list of notes displayed in the app.
The ``_confirmDelete`` method is called. A confirmation dialog appears to confirm the deletion action.

Searching for Notes
----------------
Users can search for notes by entering keywords in the search box.
The search function looks for matches in the title and subject fields.

The ``filteredNotes`` getter manages the note search functionality:

- Returns ``no results were found`` when no search query exists
- Filters notes by title and subject when searching
- Performs case-insensitive matching for better search results