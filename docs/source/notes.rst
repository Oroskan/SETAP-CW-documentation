**Notes**
=====

The ``Notes`` feature in **Learning App** allows users to create, edit, and manage notes within the app. 
Users can enter a title, subject, and content for each note. 
The notes are stored locally on the user's device and can be accessed at any time.

*Creating a New Note*
-----------------
When the user clicks the floating ``+`` button, the application calls the ``_addNewNote`` method.
This method navigates to the ``CreateNotePage`` widget.

.. code-block:: dart

  void _addNewNote() async {
    final result = await Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => const CreateNotePage()),
    );

    if (result != null && result is Note) {
      setState(() {
        notesBox.add(result);
      });
    }
  }

The ``CreateNotePage`` widget provides a user interface for writing and saving notes. 
It consists of three input fields:

- **Title**: A short, descriptive title for the note
- **Subject**: The subject or category of the note
- **Content**: The main text content of the note

The user can fill out these fields and save the note by tapping the save button. 
The note will then be added to the list of notes in the app.

*Saving a Note*
-----------
When the user taps the save button, the ``_saveNote`` method is called. 
The method stores the title, subject and content into the database.

.. code-block:: dart

  void _saveNote() {
    String title = _titleController.text.trim();
    String subject = _subtitleController.text.trim();
    String content = _contentController.text.trim();

    if (title.isEmpty && subject.isEmpty && content.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('Note cannot be empty')),
      );
      return;
    }

    Note note = Note(
      title: title.isEmpty ? 'Untitled' : title,
      subject: subject,
      content: content,
    );

    Navigator.pop(context, note);
  }

*Deleting a Note*
-------------
When the user swipes left on a note in the list, the ``_deleteNote`` method is called.
This method removes the note from the database and updates the list of notes displayed in the app.

.. code-block:: dart

  void _deleteNote(int index) {
    setState(() {
      final note = filteredNotes[index];
      note.delete();
    });
  }

The ``_confirmDelete`` method is called. A confirmation dialog appears to confirm the deletion action.

.. code-block:: dart

  Future<bool> _confirmDelete() async {
    return await showDialog(
          context: context,
          builder: (BuildContext context) => AlertDialog(
            title: const Text('Delete Note'),
            content: const Text('Are you sure you want to delete this note?'),
            actions: [
              TextButton(
                onPressed: () => Navigator.of(context).pop(false),
                child: const Text('Cancel'),
              ),
              TextButton(
                onPressed: () => Navigator.of(context).pop(true),
                child: const Text('Delete'),
              ),
            ],
          ),
        ) ??
        false;
  }

*Searching for Notes*
----------------
The user can search for notes by entering keywords in the search box.
The search function looks for matches in the title and subject fields.

The ``filteredNotes`` getter manages the note search functionality:

- Returns ``no results were found`` when no search query exists
- Filters notes by title and subject when searching
- Performs case-insensitive matching for better search results

.. code-block:: dart

  List<Note> get filteredNotes {
    if (_searchQuery.isEmpty) {
      return notesBox.values.toList();
    }
    return notesBox.values.where((note) {
      final title = note.title.toLowerCase();
      final subject = note.subject.toLowerCase();
      final query = _searchQuery.toLowerCase();
      return title.contains(query) || subject.contains(query);
    }).toList();
  }
