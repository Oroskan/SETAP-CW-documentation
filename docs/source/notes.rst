Notes
=============

The Notes feature in "Learning App" allows users to create, edit, and manage notes within the app. 
Users can enter a title, subject, and content for each note. 
The notes are stored locally on the user's device and can be accessed at any time.

Creating a New Note
-------------------
The `CreateNotePage` widget provides a user interface for writing and saving notes. 
It consists of three input fields:

1. Title: A short, descriptive title for the note.
2. Subject: The subject or category of the note.
3. Content: The main text content of the note.

.. code-block:: dart
  final TextEditingController _titleController = TextEditingController();
  final TextEditingController _subtitleController = TextEditingController();
  final TextEditingController _contentController = TextEditingController();

Users can fill out these fields and save the note by tapping the save button. 
The note will then be added to the list of notes in the app.

UI Implementation

.. code-block:: dart
  Widget build(BuildContext context) {
    return ValueListenableBuilder<bool>(
        valueListenable: darkModeNotifier,
        builder: (context, isDarkMode, _) {
          final theme = getTheme(isDarkMode);

          return PopScope(
            canPop: false,
            onPopInvokedWithResult: (didPop, [result]) async {
              if (didPop) return;
              Navigator.pop(context, null);
            },
            child: Theme(
              data: theme,
              child: Scaffold(
                appBar: AppBar(
                  title: const Text("New Note"),
                  leading: IconButton(
                    icon: const Icon(Icons.arrow_back),
                    onPressed: () => Navigator.pop(context, null),
                  ),
                  actions: [
                    TextButton(
                      onPressed: _saveNote,
                      child: Text("Done",
                          style: TextStyle(color: theme.colorScheme.onPrimary)),
                    ),
                  ],
                ),
                body: SingleChildScrollView(
                  child: Padding(
                    padding: const EdgeInsets.all(16.0),
                    child: Column(
                      children: [
                        TextField(
                          controller: _titleController,
                          style: const TextStyle(
                            fontSize: 28,
                            fontWeight: FontWeight.bold,
                          ),
                          decoration: const InputDecoration(
                            hintText: 'Title',
                            border: InputBorder.none,
                            contentPadding: EdgeInsets.symmetric(vertical: 8),
                          ),
                        ),
                        TextField(
                          controller: _subtitleController,
                          style: TextStyle(
                            fontSize: 16,
                            color: theme.colorScheme.onSurface.withOpacity(0.6),
                            fontStyle: FontStyle.italic,
                          ),
                          decoration: const InputDecoration(
                            hintText: 'Subject',
                            border: InputBorder.none,
                            contentPadding: EdgeInsets.symmetric(vertical: 8),
                          ),
                        ),
                        Divider(
                          height: 32,
                          color: theme.dividerTheme.color,
                          thickness: theme.dividerTheme.thickness,
                        ),
                        TextField(
                          controller: _contentController,
                          style: const TextStyle(
                            fontSize: 16,
                            height: 1.5,
                          ),
                          decoration: const InputDecoration(
                            hintText: 'Start typing your note...',
                            border: InputBorder.none,
                          ),
                          maxLines: null,
                          keyboardType: TextInputType.multiline,
                        ),
                      ],
                    ),
                  ),
                ),
              ),
            ),
          );
        }
    );
  }



Saving a Note
-------------
When the user taps the save button, the `_saveNote` method is called.

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
