settings_menu.dart
==================

Here we implemented a dynamic and theme-aware **Settings Menu** page using `flutter_settings_screens`, `ValueNotifier`, and Flutter widgets. It supports the dark mode toggle, user profile settings, and preferences e.g. push notifications.

Initialisation
--------------

.. code-block:: dart

   await Settings.init(cacheProvider: SharePreferenceCache());

Initialises the persistent settings store using shared preferences. Must be called before using `Settings`.

.. code-block:: dart

   final ValueNotifier<ThemeMode> themeNotifier = ValueNotifier(ThemeMode.light);

Tracks the current theme (`light`, `dark`, or `system`) and allows real-time theme switching.

UI Structure
------------

The main entry point is `MyApp`, which wraps `MaterialApp` in a `ValueListenableBuilder` to update the theme when the `themeNotifier` changes.

.. code-block:: dart

   MaterialApp(
     themeMode: mode,
     theme: getTheme(false),
     darkTheme: getTheme(true),
     home: const SettingsMenu(),
   );

Main Screen: SettingsMenu
-------------------------

The `SettingsMenu` is a `StatefulWidget` that loads and syncs dark mode state with the global `darkModeNotifier`. It displays:

1. **Profile Picture + Username Pill**:
   - Built using `ProfilePictureName`, shows a placeholder icon and the current username.

2. **SettingsGroup: Personal Information**
   - `UsernameTile`: Opens a dialog (`ChangeUsernameDialog`) for validating and updating the username.
   - `PasswordTile`: Opens `_ChangePasswordDialog` to change the password with simple validation.

3. **SettingsGroup: Preferences**
   - `PushNotificationsTile`: Toggles notification preference using `_PurpleSwitchRow`.
   - `DarkModeTile`: Toggles dark mode and updates both `ThemeHelper` and `themeNotifier`.

4. **Logout Button**:
   - Displays a `SnackBar` as a placeholder for actual logout logic.


Custom Widgets (other)
----------------------

- **_PurpleRowItem**: A stylised row for list items with title, subtitle, and arrow.
- **_PurpleSwitchRow**: A stylised row for switches with theme-aware background and primary color.
- **ChangeUsernameDialog**:
  - Validates username: length ≤ 15, alphanumeric, non-empty, and different from current.
- **_ChangePasswordDialog**:
  - Validates password length (≥ 6 characters).
  - Simple password field, extensible for further logic.

Dark Mode Sync
--------------

Dark mode changes are propagated in two ways:

1. **Local state sync**: `darkModeEnabled` is initialised and updated via `darkModeNotifier`.
2. **Global update**: Toggling `DarkModeTile` calls:

   .. code-block:: dart

      ThemeHelper.setDarkMode(newVal);
      themeNotifier.value = newVal ? ThemeMode.dark : ThemeMode.light;

   This ensures both persistent storage and runtime UI update.




