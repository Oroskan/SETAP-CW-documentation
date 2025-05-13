theme.dart
==========

Here we define the app's global theming logic, light/dark themes, using Flutter's `ThemeData` and a `ValueNotifier`which includes a `ThemeHelper` class to simplify accessing and updating the current theme state.

Global Constants
----------------

.. code-block:: dart

   const String keyDarkMode = 'key-dark-mode';

Key used to persist the user's dark mode preference using the `flutter_settings_screens` package.

.. code-block:: dart

   final ValueNotifier<bool> darkModeNotifier = ValueNotifier<bool>(false);

A global notifier that tracks the dark mode state across the app, enabling reactive theme updates.

Theme Configuration
-------------------

.. code-block:: dart

   ThemeData getTheme(bool darkMode)

Returns a configured `ThemeData` object based on the `darkMode` flag. 

Helper Class: ThemeHelper
-------------------------

A utility class to manage and persist the dark mode setting.

.. code-block:: dart

   static bool isDarkMode()

Returns the current theme mode (`true` for dark, `false` for light).

.. code-block:: dart

   static void setDarkMode(bool value)

Sets the dark mode state and persists it using `Settings.setValue`.

.. code-block:: dart

   static void toggleDarkMode()

Toggles between light and dark mode by calling `setDarkMode` with the opposite value.

Usage
-----

To listen and react to theme changes globally, use `darkModeNotifier` in our widget tree:

.. code-block:: dart

   ValueListenableBuilder<bool>(
     valueListenable: darkModeNotifier,
     builder: (context, isDark, _) {
       return MaterialApp(
         theme: getTheme(isDark),
         home: HomePage(),
       );
     },
   )
