**Quizzes**
=============

The ``Quizzes`` feature in **Learning App** allows users to create, edit, and manage quizzes within the app.
Users can enter a title, subject, and questions for each note. 
The quizzes are stored locally on the user's device and can be accessed at any time.

*Creating a New Quiz*
----------------
When the user clicks the floating ``+`` button, the application calls the ``_addNewQuiz`` method.
This method navigates to the ``CreateQuizPage`` widget.

.. code-block:: dart

  void _addNewQuiz() async {
    final result = await Navigator.push(
      context,
      MaterialPageRoute(builder: (context) => const CreateQuizPage()),
    );

    if (result != null && result is Quiz) {
      setState(() {
        quizzesBox.add(result);
      });
    }
  }

The ``CreateQuizPage`` widget provides a user interface for creating new titles of quizzes. 
When a title is created, the user can add questions to the quiz.

*Editing Quizzes and Adding Questions*
----------------
The user can access the edit page for a quiz by swiping the title in the quiz menu to the right.
The ``_editQuiz`` method is called, which navigates to the ``QuizEditorPage`` widget.

.. code-block:: dart

  void _editQuiz(int index) async {
    final quiz = filteredQuizzes[index];
    final result = await Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => QuizEditorPage(
          quiz: quiz,
          quizIndex: index,
        ),
      ),
    );
    if (result != null && result is Quiz) {
      setState(() {
        // Update the quiz in the box
        final boxIndex = quizzesBox.values.toList().indexOf(quiz);
        if (boxIndex >= 0) {
          quizzesBox.putAt(boxIndex, result);
        }
      });
    }
  }

On the edit page, the user can edit the quiz title, add and edit questions.
For a new quiz, there are no questions available.

The user can add questions by clicking the floating ``+`` button.
The application calls the ``_editQuestion`` method, which navigates to the ``QuestionEditorPage`` widget.

.. code-block:: dart

  Future<void> _editQuestion(int index) async {
    final result = await Navigator.push(
      context,
      MaterialPageRoute(
        builder: (context) => QuestionEditorPage(
          question: _editedQuiz.questions[index],
          questionIndex: index,
        ),
      ),
    );

    if (result != null && result is Question) {
      setState(() {
        _editedQuiz.questions[index] = result;
      });
    }
  }

The ``QuestionEditorPage`` widget provides a user interface for creating and editing questions.
The user can enter the question text, select the question type, and enter the correct answer.
The question type can be among the following:
- Multiple Choice
    - The user can enter multiple options for the question.
    - The user can enter two or more options.
    - The user should enter the correct answer which is one of the options.
- Question&Answer
    - The user can enter a question and an answer.
    - The user should enter the correct answer.
- Fill in the Blank
    - The user can enter a question with a blank space.
    - The user can enter the hint for the blank space.
    - The user should enter the correct answer.

When the user taps the check button, the ``_saveQuestion`` method is called.
The method stores the question text, question type, and correct answer into the database.

.. code-block:: dart

  void _saveQuestion() {
    if (_questionController.text.trim().isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: const Text('Question text cannot be empty'),
          backgroundColor: Theme.of(context).colorScheme.error,
        ),
      );
      return;
    }

    if (_answerController.text.trim().isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: const Text('Answer cannot be empty'),
          backgroundColor: Theme.of(context).colorScheme.error,
        ),
      );
      return;
    }

    if (_selectedType == 'multiple_choice') {
      // Check if we have at least 2 non-empty choices
      final nonEmptyChoices = _choiceControllers
          .map((c) => c.text.trim())
          .where((text) => text.isNotEmpty)
          .toList();

      if (nonEmptyChoices.length < 2) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content:
                const Text('Multiple choice questions need at least 2 options'),
            backgroundColor: Theme.of(context).colorScheme.error,
          ),
        );
        return;
      }

      if (!nonEmptyChoices.contains(_answerController.text.trim())) {
        ScaffoldMessenger.of(context).showSnackBar(
          SnackBar(
            content:
                const Text('The answer must be one of the provided choices'),
            backgroundColor: Theme.of(context).colorScheme.error,
          ),
        );
        return;
      }
    }

    Navigator.pop(context, _buildQuestion());
  }

The user can also delete a question by tapping the delete button.

.. code-block:: dart

    trailing: IconButton(
        icon: Icon(
        Icons.delete,
        color: theme.colorScheme.error,
        ),
        onPressed: () {
        setState(() {
            _editedQuiz.questions.removeAt(index);
        });
        },
    ),

The user can save the quiz by tapping the save button.
The ``_saveChanges`` method is called, which stores the quiz title and questions into the database.
The method checks if the quiz title is empty and displays a warning message.

.. code-block:: dart

  void _saveChanges() {
    if (_titleController.text.trim().isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: const Text('Quiz title cannot be empty'),
          backgroundColor: Theme.of(context).colorScheme.error,
        ),
      );
      return;
    }

    if (_editedQuiz.questions.isEmpty) {
      showDialog(
        ...
      );
    } else {
      _editedQuiz.title = _titleController.text.trim();
      Navigator.pop(context, _editedQuiz);
    }
  }

*Deleting a Quiz*
-------------
When the user swipes left on a quiz in the list, the ``_deleteQuiz`` method is called.

.. code-block:: dart

  void _deleteQuiz(int index) {
    setState(() {
      final quiz = filteredQuizzes[index];
      quiz.delete();
    });
  }
  onDismissed: (_) => _deleteQuiz(index),

When the user taps the delete button, the ``_confirmDelete`` method is called.
It displays a confirmation dialog to confirm the deletion action.

.. code-block:: dart

    Future<bool> _confirmDelete() async {
        return await showDialog(
            context: context,
            builder: (BuildContext context) => AlertDialog(
                title: const Text('Delete Quiz'),
                content: const Text('Are you sure you want to delete this quiz?'),
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
            ) ?? false;
    }

*Take a Quiz*
-------------
The user can take a quiz by tapping the quiz title in the quiz menu if the quiz is not empty. 
The ``_takeQuiz`` method is called, which navigates to the ``QuizScreen`` widget.
Otherwise, a warning message is displayed.

.. code-block:: dart

  void _takeQuiz(Quiz quiz) {
    if (quiz.questions.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(
          content: const Text(
              'This quiz has no questions yet. Add questions before taking the quiz.'),
          duration: const Duration(seconds: 3),
          backgroundColor: Theme.of(context).colorScheme.error,
        ),
      );
      return;
    }
    Navigator.push<int>(
      context,
      MaterialPageRoute(
        builder: (context) => QuizScreen(
          quiz: quiz,
          completedQuizzes: 0,
        ),
      ),
    );
  }

On the quiz screen, the user can answer or skip the question.
When the user inputs the answer and taps the submit button, 
the correct answer is showen and the continue button is displayed.

.. code-block:: dart

  if (answered)
    Padding(
      padding: const EdgeInsets.only(top: 16.0),
      child: Text(
      userAnswer == question.answer
        ? 'Correct! The answer is: ${question.answer}'
        : 'Incorrect. The correct answer is: ${question.answer}',
      style: TextStyle(
        color: userAnswer == question.answer
          ? theme.colorScheme.tertiary
          : theme.colorScheme.error,
        fontWeight: FontWeight.bold,
      ),
    ),
  )

If the user skips the question, the correct answer is not shown, 
and the user will move to the next question or the summary page.

.. code-block:: dart

  void skipQuestion() {
    setState(() {
      if (!answered) {
        incorrectCount++;
        // Record skipped question
        userAnswers.add({
          'question': widget.quiz.questions[currentQuestionIndex],
          'userAnswer': null,
          'isCorrect': false
        });
      }
      answered = false;
      userAnswer = null;
      textController.clear();

      if (currentQuestionIndex < widget.quiz.questions.length - 1) {
        currentQuestionIndex++;
      } else {
        Navigator.pop(context);
        Navigator.push<int>(
            //summarize the quiz
            ...
        );
      }
    });
  }

*Quiz Summary*
------------
The summary page provides users with a summary of their quiz performance, 
including the number of correct and incorrect answers, time taken, and options to review answers or start a new quiz.
- Displays a message summarizing the user's performance.
- Shows time taken to complete the quiz.
- Presents the number of correct and incorrect answers, along with a visual representation (circle chart).
- **Review Answers**: Reviews the user's answers and the correct answers.
- **Take a New Test**: Navigates to the quiz menu to start a new quiz.

.. code-block:: dart

  class QuizSummary extends StatefulWidget {
    final int correctAnswers;
    final int incorrectAnswers;
    final String message;
    final double timeTaken;
    final List<Map<String, dynamic>>? userAnswers;
    final Quiz? quiz;

    const QuizSummary({
        super.key,
        required this.correctAnswers,
        required this.incorrectAnswers,
        required this.message,
        required this.timeTaken,
        this.userAnswers,
        this.quiz,
    });

    @override
    State<QuizSummary> createState() => _QuizSummary();
  }

**Review Answers**

The review page allows users to review their answers after completing a quiz. 
It provides detailed feedback on each question, including the user's answer, the correct answer, and whether the user's response was correct or incorrect.

.. code-block:: dart

  class ReviewPage extends StatelessWidget {
    final List<Map<String, dynamic>> userAnswers;

    const ReviewPage({super.key, required this.userAnswers});

    @override
    Widget build(BuildContext context) {
      return Scaffold(
        appBar: AppBar(title: const Text('Review Answers')),
        body: ListView.builder(
          itemCount: userAnswers.length,
          itemBuilder: (context, index) {
            final question = userAnswers[index]['question'] as Question;
            final userAnswer = userAnswers[index]['userAnswer'];
            final isCorrect = userAnswers[index]['isCorrect'] as bool;

            return ListTile(
              title: Text('Question ${index + 1}: ${question.question}'),
              subtitle: Text(
                'Correct: ${_getCorrectAnswer(question)}\n'
                'Your Answer: ${userAnswer ?? 'Skipped'}\n'
                'Result: ${isCorrect ? 'Correct' : 'Incorrect'}',
              ),
            );
          },
        ),
      );
    }

    String _getCorrectAnswer(Question question) {
      if (question is QuestionAnswer || question is FillInTheBlank) {
        return question.answer;
      }
      return '';
    }
  }

**Take a New Test**

The user can start a new quiz by tapping the button.
The ``QuizMenu`` widget is called, which displays the list of quizzes.
