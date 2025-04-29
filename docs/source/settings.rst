**Settings Menu**
=============

The ``SettingsMenu`` module provides a ready‑to‑use profile and preferences screen for Flutter apps.  It is built with *Material 3*, relies on the ``flutter_settings_screens`` package for persistent storage, and supports live *light/dark‑mode switching* through a global ``ValueNotifier<ThemeMode>``.

**Prerequisites**
=============

**Flutter 3.19** or later (null‑safety enabled)

``flutter_settings_screens`` ≥ 0.3.0

A custom ``theme.dart`` that exposes a ``getTheme(bool isDark)`` helper and a ``ThemeHelper`` utility.

Add the dependency to ``pubspec.yaml``::

dependencies:
flutter_settings_screens: ^0.3.0

**Getting Started**
===============

**1. Initialize shared preferences** before the first widget is built::

void main() async {
WidgetsFlutterBinding.ensureInitialized();
await Settings.init(cacheProvider: SharePreferenceCache());
runApp(const MyApp());
}

**2. Wrap ``MaterialApp``** in a ``ValueListenableBuilder`` so the UI rebuilds when the theme changes::

.. code-block:: dart

  class MyApp extends StatelessWidget {
    @override
      Widget build(BuildContext context) {
        return ValueListenableBuilder(
        valueListenable: themeNotifier,
        builder: (_, mode, __) => MaterialApp(
        themeMode: mode,
        theme: getTheme(false),
        darkTheme: getTheme(true),
        home: const SettingsMenu(),
        ),
      );
    }
  }

**High‑level Structure**
==================


lib/
├─ main.dart            # entry point & theme notifier
├─ settings_menu.dart   # everything in this doc
└─ theme.dart           # light & dark ThemeData

`SettingsMenu` uses a stateful widget (`_ProfilePageState`) to keep local copies of:
- `pushNotificationsEnabled` – `bool`
- `darkModeEnabled` – `bool`
- `currentUsername` – `String`

and listens to an external `darkModeNotifier` so changes elsewhere stay in sync.

**User Interface Walk‑through**
================

Profile Picture & Name

**Username**^^^^^^^^^^^^
- Label: Username  ↗  "Change username"
- Interaction: opens a modal `**ChangeUsernameDialog**`
- Validation rules:
  - Non‑empty
  - ≤ 15 characters
  - Alphanumeric only (`^[A‑Za‑z0‑9]+$`)
  - Different from current username

.. code-block:: dart
:caption: Open the username dialog

UsernameTile(
currentUsername: currentUsername,
onUsernameChanged: (name) {
setState(() => currentUsername = name);
},
theme: theme,
)

Password
^^^^^^^^^^^^

Label: Password  ↗  "Change password"

Interaction: opens _ChangePasswordDialog

Validation: ≥ 6 characters (extend as needed for strength checks)

Preferences Group
