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

**1. Initialize shared preferences** before the first widget is built:

.. code-block:: dart

void main() async {
WidgetsFlutterBinding.ensureInitialized();
await Settings.init(cacheProvider: SharePreferenceCache());
runApp(const MyApp());
}

**2. Wrap MaterialApp** in a ``ValueListenableBuilder`` so the UI rebuilds when the theme changes:

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
|- main.dart            # entry point & theme notifier
|- settings_menu.dart   # everything in this doc
|_ theme.dart           # light & dark ThemeData

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
  UsernameTile(
    currentUsername: currentUsername,
    onUsernameChanged: (name) {
    setState(() => currentUsername = name);
    },
    theme: theme,
  )

**Password**^^^^^^^^^^^^
- Label: Password  ↗  "Change password"
- Interaction: opens `_ChangePasswordDialog`
- Validation: ≥ 6 characters (extend as needed for strength checks)

Preferences Group
**Push Notifications**
^^^^^^^^^^^^^^^^^^^^^^
A simple switch.  Persist the value by writing to ``Settings`` or your own backend:

.. code-block:: dart
   PushNotificationsTile(
     value: pushNotificationsEnabled,
     onChanged: (val) {
       setState(() => pushNotificationsEnabled = val);
       // TODO: subscribe/unsubscribe device token.
     },
     theme: theme,
   )

**Dark Mode**
^^^^^^^^^^^^^

Toggles between light & dark themes in real time and stores the preference:

.. code-block:: dart
   DarkModeTile(
     value: darkModeEnabled,
     onChanged: (val) {
       setState(() => darkModeEnabled = val);
       themeNotifier.value = val ? ThemeMode.dark : ThemeMode.light;
       ThemeHelper.setDarkMode(val); // persists to SharedPreferences
     },
     theme: theme,
   )

Logout Button
~~~~~~~~~~~~~

A centred **elevated** button anchored at the bottom of the list.  Replace the snackbar with your auth sign‑out logic:

   onPressed: () async {
     await AuthService.signOut();
     Navigator.pushReplacementNamed(context, '/login');
   },

Re‑usable Building Blocks
-------------------------

``_PurpleRowItem``

- Reusable list‑tile‑like row with title, subtitle and chevron.
- Uses the theme’s secondary color at 20 % opacity.
- Rounded top corners only so adjacent items create a continuous card.
`_PurpleSwitchRow`

* Row with label + **Material ``Switch``**.
* Shares the same container styling as ``_PurpleRowItem``.

Extending the Module
--------------------

* **Link to system settings:** Use ``AppSettings.openAppSettings()`` for Android/iOS.
* **Biometric lock:** Add a ``_PurpleSwitchRow`` that calls ``local_auth``.
* **Account deletion:** Place a **destructive** button below “Log out”.

Unit Testing
------------

1. Wrap widget under test with ``ValueListenableBuilder`` and inject a fake ``Settings`` cache provider.
2. Verify that toggling **Dark Mode** updates both ``themeNotifier`` and persistent storage.

.. code-block:: dart
   :caption: Example widget test (shortened)

   testWidgets('dark mode switch updates notifier', (tester) async {
     await tester.pumpWidget(const SettingsMenu());
     final switchFinder = find.byType(Switch);
     await tester.tap(switchFinder);
     await tester.pumpAndSettle();
     expect(themeNotifier.value, ThemeMode.dark);
   });
