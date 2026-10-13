<div align="center">

# GTK Desktop Application Standard

Look, behaviour and structure shared by every LinuxGamerLife GTK application.

<br>

![GTK](https://img.shields.io/badge/GTK-4-blue?logo=gtk&logoColor=white)
![libadwaita](https://img.shields.io/badge/libadwaita-1.6+-blue)
![C++](https://img.shields.io/badge/C++-17-blue?logo=cplusplus&logoColor=white)
![CMake](https://img.shields.io/badge/CMake-build-blue?logo=cmake&logoColor=white)

<br>

A user moving between LGL apps should find the same title bar, logo, menu, shortcuts and theme behaviour in each one.

</div>

---

# Purpose

This document defines the user-facing conventions and the project structure for GTK 4 (gtkmm) applications. It is written so that a new application can be built from it without reading another app's source.

It complements the **C++ / Qt Development Standard**, whose engineering rules (ownership, error handling, input validation, warnings as errors, sanitizers, testing) apply to GTK code as well. Where that standard names a Qt mechanism, use the GTK equivalent.

### How to read this document

- Rules without a qualifier apply to every app.
- Rules marked **where applicable** apply only when the app has that feature (files, a list, a help window, a primary action).
- Placeholders are written in angle brackets, and `xx` stands for the app's two-letter CSS prefix in class and colour names:

| Placeholder | Meaning |
|--------|--------|
| `<app-name>` | package, binary, icon and config name |
| `<app-id>` | application ID |
| `<Display Name>` | name shown to users |
| `<short>` | code namespace and GResource folder |
| `<xx>` | two-letter CSS prefix |
| `<APP>` | upper-case prefix for environment variables and CMake options |

---

# Technology Stack

| Component | Requirement |
|--------|--------|
| Toolkit | gtkmm 4 (`gtkmm-4.0 >= 4.20`) |
| Styling base | libadwaita (`libadwaita-1 >= 1.6`), C API |
| Language | C++17 minimum, C++20 preferred |
| Build | CMake 3.20+, `glib-compile-resources` |
| Tests | GLib test framework (`g_test_*`) through CTest |
| Packaging | RPM spec, built on COPR |

### Rules

- Use gtkmm for widgets. Use libadwaita only for initialisation (`adw_init()`) and `AdwStyleManager`; do not use Adw widgets (`AdwApplicationWindow`, `AdwHeaderBar`, `AdwDialog`, `AdwToast` and so on), so the app looks correct under any GTK theme. libadwaita has no C++ bindings; call its C API directly.
- Bundle `style.css`, the app icon and any other images or text the app displays in a **GResource**; never load them from the source tree or relative paths.
- Build and test inside a Fedora container that matches the target release (LGL uses the `fedora-dev1` distrobox), never against the host's libraries.

---

# Project Layout

```
<repo>/
  CMakeLists.txt          thin wrapper: add_subdirectory(gtk)
  Makefile                source archive and srpm targets
  .copr/Makefile          include Makefile
  .github/workflows/ci.yml
  CHANGELOG.md
  README.md
  packaging/
    <app-id>.desktop
    <app-id>.metainfo.xml
    <app-name>.spec
    icons/<app-name>.png                  master artwork (largest available)
    icons/<size>x<size>/<app-name>.png    48, 64, 128, 256
  gtk/
    CMakeLists.txt
    data/
      <short>.gresource.xml
      style.css
    src/
      main.cpp            application lifecycle only
      core/               no widgets; unit tested
      ui/                 windows, dialogs, ThemeWatcher
    tests/<module>_test.cpp
```

### Rules

- The repository root must build the GTK app: `cmake -B build && cmake --build build`. The root `CMakeLists.txt` contains only:

```cmake
cmake_minimum_required(VERSION 3.20)
project(<app-name> LANGUAGES C CXX)
include(CTest)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})   # build/<app-name>
add_subdirectory(gtk)
```

- `gtk/CMakeLists.txt` declares `project(<app-name> VERSION x.y.z LANGUAGES C CXX)` and owns everything else: options, resources, targets, install rules and tests.
- `src/core` must not include widget headers. Everything that can be tested without a display (parsing, settings, history, file planning, theme file reading) belongs there and is built as a static library linked by both the app and the tests.
- `src/ui` holds the widgets and anything that needs a display.
- All code lives in `namespace <short>`. The main window class is `<short>::Window`, a `Gtk::ApplicationWindow`.
- Legacy sources from an earlier toolkit may stay in the repository for reference, but they must not be built or packaged.

---

# Application Identity

| Item | Convention |
|--------|--------|
| Application ID | `com.linuxgamerlife.<app-name>` |
| Binary, icon name, config folder | `<app-name>`, always prefixed `lgl-` |
| Display name | `LGL <Name>` in title case |
| Wordmark | display name in capitals |
| Desktop and metainfo files | `<app-id>.desktop`, `<app-id>.metainfo.xml` |
| GResource prefix | `/com/linuxgamerlife/<short>/` |
| Code namespace | `<short>` |
| CSS prefix | `<xx>-` for classes (`xx-card`), `<xx>_` for colours (`@xx_surface`) |

### Rules

- The window title (`set_title`) is the display name. The compositor title bar, task switchers and snapshots use it.
- Call `Glib::set_application_name()` and `Gtk::Window::set_default_icon_name()` in the application's `startup` handler.
- If an app's ID changes, list the old ID (and any old desktop file name) under `<provides>` in the metainfo, so software centres treat it as the same app, and tell users in the release notes to re-pin it.
- Install the icon into `hicolor` at 48, 64, 128 and 256 px, generated from the master artwork (for example `magick <master> -filter Lanczos -resize 256x256 -strip <out>`). The 256 px PNG is also bundled in the GResource as `<app-name>.png` for the header logo and the About dialog, so an uninstalled build still shows it.

---

# Application Lifecycle

- One `Gtk::Application`, unique by default: a second launch presents the running window. Apps that open files add `Gio::Application::Flags::HANDLES_OPEN` and pass the files to the running window.
- The main window is owned by `main()` in a `std::unique_ptr`, created on first `activate` (or `open`) and presented on every `activate`.
- `startup` handler, in this order:
  1. `Glib::set_application_name("<Display Name>")`
  2. `Gtk::Window::set_default_icon_name("<app-name>")`
  3. `installApplicationStylesheet()` (initialises libadwaita and adds `style.css`)
  4. `app.quit` action, bound to **Ctrl+Q**
- `app.quit` hides the window before calling `quit()`.
- `main()` resets the window after `run()` returns, so it is destroyed before the application.

### gtkmm window rule

gtkmm removes a window from its application when the window is hidden. If it was the only window, the application quits. Code that hides the main window temporarily (for example the title bar switch) must `hold()` the application, re-attach the window with `set_application()` before showing it, and then `release()`. A re-attached window is exported on D-Bus under a new path (`…/window/<n>`); do not rely on the number.

---

# Theming

Every app follows the desktop theme, including live changes, and never ships its own palette.

## Colour sources

- Colours come only from **libadwaita's named colours**: `@window_bg_color`, `@window_fg_color`, `@accent_bg_color`, `@accent_fg_color`, `@headerbar_bg_color` and the rest.
- libadwaita resolves them from the desktop (dark mode and accent colour through the settings portal). The user's `~/.config/gtk-4.0/gtk.css`, as written by theme generators such as Noctalia, overrides them.
- Derived colours are defined once at the top of `style.css`, mixed from the named colours and prefixed with `<xx>_`. Define only the ones the app uses, with exactly these mixes:

```css
@define-color xx_surface color-mix(in srgb, @window_fg_color 4%, @window_bg_color);
@define-color xx_sunken  color-mix(in srgb, @window_fg_color 2%, @window_bg_color);
@define-color xx_border  color-mix(in srgb, @window_fg_color 16%, @window_bg_color);
@define-color xx_hover   color-mix(in srgb, @window_fg_color 8%, @window_bg_color);
@define-color xx_muted   color-mix(in srgb, @window_fg_color 68%, @window_bg_color);
@define-color xx_subtle  color-mix(in srgb, @window_fg_color 42%, @window_bg_color);
```

- Short aliases of named colours are allowed when they make the stylesheet easier to read (`@define-color xx_fg @window_fg_color;`). They must stay plain aliases, never fixed values.
- **No hardcoded colours**, with two exceptions:
  - **Status colours** (positive, neutral, negative) are fixed, with a light and a dark variant selected by the `.dark` class on the main window:

    | Status | Light | Dark |
    |--------|--------|--------|
    | positive | `#067647` | `#47cd89` |
    | neutral | `#a15c07` | `#fdb022` |
    | negative | `#b42318` | `#f97066` |

  - **Image surfaces** that show user content (a preview canvas) may use `black` with `alpha(white, …)` text, because the content, not the theme, sets their background.

## Stylesheet priorities

| Provider | Priority | Purpose |
|--------|--------|--------|
| `style.css` (GResource) | `APPLICATION` | the app's look |
| Monospace font rules | `APPLICATION + 1` | desktop monospace font for mono classes |
| User `gtk.css` (reloaded) | `USER` | the user's theme always wins |

## ThemeWatcher

Each app has a `ThemeWatcher` (`src/ui/themewatcher.{h,cpp}`, with pure, tested helpers in `src/core/theme.{h,cpp}`). The same file also provides `installApplicationStylesheet()`. It adds what libadwaita does not do:

- **Live reload:** watches `~/.config/gtk-4.0/` and the folder of every local `@import` in `gtk.css`, and reloads `gtk.css` into the `USER` provider when any `.css` file changes. GTK itself loads that file only once.
- **Debounce:** theme generators write several files in quick succession, so changes are applied once after **200 ms**.
- **Desktop changes:** re-applies on `AdwStyleManager` `notify::dark` and `notify::accent-color`, and on `org.gnome.desktop.interface` changes.
- **No settings portal:** when `adw_style_manager_get_system_supports_color_schemes()` is false, takes dark mode from `color-scheme`, `gtk-application-prefer-dark-theme` or a `-dark` theme name, and forces the colour scheme accordingly.
- **Monospace font:** applies the desktop `monospace-font-name` to the mono classes. The rule it generates must list **every** class whose CSS uses the mono font stack, so none of them is left on the fallback font.
- **`.dark` class:** sets it on the main window from the effective background luminance (needed only by status colours).
- **Quiet parsing:** ignores parse errors from the user's `gtk.css`, which commonly imports files that do not exist. Errors in the app's own `style.css` are reported with their line number.
- **Change signal:** emits a signal after each re-theme, so secondary windows that copy theme values (such as a help window's monospace font) can update.

## Window shape

```css
:root { --window-radius: 0px; }
```

Window corners are square. Compositors that draw their own rounded borders clip the window to them, and a second radius from libadwaita leaves a visible gap.

---

# Layout and Components

All apps share these classes and measurements.

### Class naming

- Shared component classes that libadwaita also defines, or could, carry the app prefix: `<xx>-card`, `<xx>-primary`, `<xx>-small`, `<xx>-chip`, `<xx>-notice`.
- Shared utility classes are unprefixed: `body`, `card-head`, `section-label`, `wordmark`, `brand-icon`, `mono`, `muted`, `subtle`, `small`, `dialog-body`, `flush`.
- App-specific classes use plain descriptive names (`queue-list`, `preview-area`, `result-tile`).

### Components

| Element | Class | Style |
|--------|--------|--------|
| Window content | `.body` | `padding: 12px` |
| Card | `.xx-card` | `@xx_surface` background, `1px solid @xx_border`, `border-radius: 10px`, `padding: 14px` |
| Card around a scroller or list | `.xx-card.flush` | `padding: 0`; call `set_overflow(Gtk::Overflow::HIDDEN)` on the card widget so its corners clip the content |
| Card heading row (flush cards) | `.card-head` | `padding: 12px 14px 8px 14px` |
| Section label | `.section-label` | mono, `0.78em`, weight 600, `letter-spacing: 0.12em`, `@xx_muted`, text in capitals; the first child of a card |
| Secondary text | `.muted`, `.subtle`, `.small` | `@xx_muted`, `@xx_subtle`, `0.85em` |
| Numbers and paths | `.mono` | mono, `font-feature-settings: "tnum"` |
| Primary action | `button.xx-primary` + `suggested-action` | `background-image: none`, `@accent_bg_color` background and border, `@accent_fg_color` text, weight 600; hover mixes 15% foreground into the accent; disabled opacity 0.45 |
| Compact button | `button.xx-small` | `min-height: 0; padding: 2px 10px; font-size: 0.85em` |
| Status chip | `.xx-chip.positive/.neutral/.negative` | mono, `0.78em`, weight 600, `letter-spacing: 0.06em`, `padding: 1px 8px`, 6px radius, `currentColor` border, 12% tinted background |
| Notice (toast) | `.xx-notice` + optional status class | see **Notices** |
| Hovered tile or row | | `@xx_hover` background |
| Selected list row | | 22% accent mix over transparent, foreground unchanged |
| Focused tile | | `@accent_bg_color` / `@accent_fg_color` |
| Thumbnail placeholder | | `@xx_sunken` background |
| Split views | `paned > separator` | `min-width: 12px`, no background, border or shadow, so panes are separated by the same 12 px gap as cards |
| Scroller viewport | `.<name>-scroller > viewport` | `background: none`, so the card or window shows through |
| Empty state | | a centred, wrapped `.muted` label naming what to do next ("Drop images here, or use Add Images.") |

Mono classes use:

```css
font-family: "Adwaita Mono", "JetBrains Mono", monospace;
```

The ThemeWatcher replaces this with the desktop monospace font when one is set.

### Spacing

| Between | Spacing |
|--------|--------|
| Cards, sections and window-level blocks | 12 |
| Rows inside a card | 10 |
| Controls in a row, dialog buttons | 8 |
| Closely related items (a heading and its content) | 6 |
| A field label and its field | 4 |

### Corner radii

| Element | Radius |
|--------|--------|
| Window | 0 |
| Cards and large surfaces | 10px |
| List rows and notices | 8px |
| Chips and tiles | 6px |
| Thumbnails | 4px |

Buttons, entries and other stock controls keep libadwaita's own radius.

### Notices

Notices report non-blocking results inside the window instead of a dialog.

- A `Gtk::Revealer` (`SLIDE_UP`), centred at the bottom of a `Gtk::Overlay` over the body, with `set_can_target(false)` so it never blocks clicks.
- One label, wrapped, at most 80 characters wide.
- Style: `margin: 0 0 16px 0; padding: 9px 16px;`, 8px radius, `@xx_border` border, `@xx_surface` background, `color: @window_fg_color`, `box-shadow: 0 2px 10px alpha(black, 0.18)`. A status class tints the text with the status colour, the border with a 45% mix and the background with an 8% mix.
- Timing: **5 s** for routine results, **10 s** for results the user may want to read (a saved file path), and no timeout for progress messages, which stay until the next notice replaces them.

---

# Title Bar

Every main window supports two title bar modes, switched at runtime and remembered.

| Mode | Default | Who draws the title bar | Header bar position | Window buttons |
|--------|--------|--------|--------|--------|
| App title bar | yes | GTK (client-side) | title bar | GTK's, per the desktop button layout |
| System title bar | no | the compositor (server-side) | first row inside the window | the compositor's |

In both modes the **same `Gtk::HeaderBar` instance** holds the logo, the app's actions and the main menu. Switching moves the whole header bar, so nothing the user needs is lost in either mode.

### Switching

- Action: `win.system-title-bar`, a stateful boolean.
- Shortcut: **Ctrl+Shift+M**.
- Menu: **Use System Title Bar** checkbox in the main menu.
- Persisted as `[window] systemTitleBar` in the app's `settings.ini`.

### Implementation rules

- The header bar is a window member (`Gtk::HeaderBar m_header`), not `make_managed`, so it survives being moved.
- The window's child is a vertical box (`m_content`) holding the body (or the overlay around it). In system mode the header bar is prepended to it.
- Apply the stored mode in the constructor, before the window is realized.
- App mode: `set_show_title_buttons(true)` and `set_titlebar(m_header)`.
- System mode: `unset_titlebar()`, `set_show_title_buttons(false)`, then prepend to `m_content`.
- GTK can only change who draws the title bar while the window is unrealized. A runtime switch must, in this order:
  1. `hold()` the application
  2. hide the window
  3. `Gtk::Widget::unrealize()`
  4. move the header bar
  5. `set_application()` again
  6. `present()`
  7. return focus to the primary input
  8. `release()` the application
- The window flickers once during the switch. This is expected.

### Primary input

Each app names one **primary input**: the widget that should have keyboard focus when the user returns to the window (a search field, a list, an editor). Focus returns there after a title bar switch and after the main menu is closed with Alt. If the primary input cannot take focus (an empty list), leave focus where GTK puts it.

---

# Logo and Header Bar Layout

```
[icon] LGL APP NAME  [action] [action] ...              [primary] [☰] [×]
└── pack_start ──────────────────────────┘   empty centre   └── pack_end ──┘
```

### Logo

- A horizontal box with spacing **10**, `margin-end` **12**, packed first at the start of the header bar.
- Icon: `Gtk::Image` of the bundled app icon texture, `set_pixel_size(28)`, class `brand-icon`.
- Wordmark: `Gtk::Label` with the display name in capitals, class `wordmark`:

```css
.wordmark {
  font-family: "Adwaita Mono", "JetBrains Mono", monospace;
  font-weight: 600;
  letter-spacing: 0.08em;
}
```

- The wordmark is the title, so the header bar's title widget is an empty `Gtk::Box`.

### Header bar contents

- **Start**, after the logo (where applicable): the app's frequent actions as text buttons, in workflow order.
- **End** (`pack_end`, so the first packed is rightmost): the main menu button, then the primary action (`xx-primary` + `suggested-action`) where the app has one.
- Every header button is bound to a `win.` action through `set_action_name()`, so its sensitivity follows the action and it can share a shortcut and menu entry.
- Every header button has a tooltip. If the action has a shortcut, the tooltip ends with it in brackets, for example `Add images to the queue (Ctrl+O)`.
- An app with no header actions (a single-purpose tool) shows only the logo and the main menu.

---

# Main Menu

- A `Gtk::MenuButton` with icon `open-menu-symbolic`, tooltip `Main menu`, `set_primary(true)` (opens with **F10**), built from a `Gio::Menu`.
- Sections, in this order, each a separate `Gio::Menu` section:
  1. App actions, where applicable (for example Add Images…, Save As…). Use more than one section when the actions form distinct groups (files, then selection).
  2. **Use System Title Bar**
  3. **Help** (where applicable) and **About <Display Name>**
  4. **Quit**
- Items that open a dialog or a file picker end with an ellipsis (`…`).

### Alt key

- Pressing and releasing **Alt** on its own opens the main menu; doing it again closes it. Alt held with any other key does nothing.
- The open menu is a separate surface with its own key events, so the Alt detector is attached to both the window and `m_menuButton->get_popover()`, in the capture phase. The pressed handler records whether the key was Alt and returns `false`; the released handler acts only if the last key pressed was that Alt.
- Closing the menu with Alt returns focus to the primary input, deferred to idle so it runs after GTK has handed focus back to the window.

---

# Keyboard

### Standard shortcuts

| Shortcut | Action | Scope |
|--------|--------|--------|
| Ctrl+Q | Quit | all apps |
| Ctrl+Shift+M | Toggle system title bar | all apps |
| Alt (tap) | Open or close the main menu | all apps |
| F10 | Open the main menu | all apps |
| F1 | Help | apps with a help window |
| Ctrl+O | Open or add files | where applicable |
| Ctrl+Shift+O | Add a folder | where applicable |
| Ctrl+Shift+S | Save As | where applicable |
| Ctrl+A | Select all | lists, while focused |
| Delete | Remove selected | lists, while focused |
| Menu key | Context menu | lists, while focused |
| Escape | Close a menu or dialog; close the window only in transient tools that are reopened for each use | |

### Rules

- Window-wide shortcuts are registered with `application->set_accel_for_action()` against named actions, not in key handlers, so menus display them.
- Shortcuts that apply only while a widget has focus (Delete in a list) use a `Gtk::ShortcutController` on that widget, bound to the same named action with `Gtk::NamedAction`.
- Key handlers that intercept navigation must return `false` for any key combination with Ctrl or Alt, so accelerators keep working.
- Do not reuse a standard shortcut for a different action.

---

# About Dialog

A single `Gtk::AboutDialog`, created on first use and kept (`set_hide_on_close(true)`), transient for and modal to the main window.

| Field | Value |
|--------|--------|
| Program name | display name |
| Version | `<APP>_VERSION`, from CMake `PROJECT_VERSION` via a compile definition |
| Comments | one or two sentences on what the app does, then a blank line and `Built with GTK 4.`, naming any major library (`Built with GTK 4 and ImageMagick.`) |
| Website | `https://github.com/linuxgamerlife/<app-name>`, label `Project page and issue tracker` |
| Authors | `LinuxGamerLife https://www.youtube.com/@linuxgamerlife` |
| Credits | `Inspired by` section where the app builds on someone else's work; `Support` section: `Support development on Ko-fi https://ko-fi.com/G2G3V70LW` |
| Licence | `Gtk::License::MIT_X11` |
| Logo | the bundled app icon texture |

---

# Dialogs and Secondary Windows

- **Files and folders:** `Gtk::FileDialog`, modal, with a title naming the task (`Add images`, `Choose export folder`), filters for the supported types, and the last used folder as the initial folder. A cancelled dialog is not an error.
- **Messages and confirmations:** `Gtk::AlertDialog`, with a heading stating the question or outcome and a detail line with the specifics. Buttons are verbs (`Move to Trash`, `Replace`), the safe choice is the cancel button, and a destructive choice is never the default.
- **Custom dialogs and help windows:** a `Gtk::Window` subclass, `set_transient_for(parent)`, modal where the user must answer, with its own `Gtk::HeaderBar` whose title widget is a `Gtk::Label` in capitals with class `wordmark` (`HELP`, `CONFIRM SAVE AS`). Content uses `.dialog-body` (`padding: 18px`). Buttons sit at the bottom right, cancel first and the affirmative `xx-primary` + `suggested-action` button last; it is the default widget, and Escape and the close button answer cancel.
- **Help:** where applicable, the help text is a `HELP.md` at the repository root, bundled in the GResource and shown in a help window (F1, main menu). The same file is the packaged documentation, so it is written for users and lists every shortcut.
- **Non-blocking results:** report the outcome of a completed operation with a notice rather than a dialog. Use a dialog only when the user must decide something or must not miss a failure.

---

# Settings and User Data

| Data | Location | Format |
|--------|--------|--------|
| Preferences | `$XDG_CONFIG_HOME/<app-name>/settings.ini` | `Glib::KeyFile` |
| History, caches | `$XDG_CACHE_HOME/<app-name>/` | app-defined |

### Rules

- One `Settings` class in `src/core/settings.{h,cpp}` with a typed getter and setter per preference. A second constructor takes a path for tests.
- Every setter saves immediately.
- A missing, unreadable or malformed file, group or value falls back to the default. Settings are a convenience: a failure to save is not fatal and must not interrupt the user.
- Create the folder with mode `0700`.
- Treat stored data as untrusted on read: cap sizes and counts, validate values, ignore malformed entries.
- Write history and cache files atomically (`Glib::file_set_contents`).
- Shared keys:

| Group | Key | Type | Default |
|--------|--------|--------|--------|
| `window` | `systemTitleBar` | boolean | `false` |

App keys go in groups named for their area (`files`, `display`, `history`).

---

# Background Work and External Tools

- Never block the main loop with file, image or network work. Run it on a worker thread and hand results back with `Glib::Dispatcher`, or use the async GIO APIs.
- Results that arrive after the user has moved on (a preview for an item no longer selected) are discarded; tag requests with a generation counter.
- External programs run through `Gio::Subprocess` with an argument vector, never a shell, with a timeout, and with their exit status and error output treated as part of the result. Resolve the program once and report clearly when it is missing.

---

# Build and Quality

- Compile with `-Wall -Wextra -Wpedantic -Werror` for C++ on GCC and Clang. Generated resource C is built with `-w`.
- Provide `<APP>_ENABLE_SANITIZERS` (ASan + UBSan), off by default.
- Pass the version to the code with `target_compile_definitions(... <APP>_VERSION="${PROJECT_VERSION}")`.
- One test executable per core module (`tests/<module>_test.cpp`) registered with CTest. Every app has at least `settings_test` and `theme_test`. Tests that need a tool the build machine may lack call `g_test_skip()`; mark them with `SKIP_RETURN_CODE 77`.
- Tests use a temporary folder from `g_dir_make_tmp()` and never touch the user's real config or cache.
- CI builds and tests from the repository root in a container of the target Fedora release.

### Snapshot smoke test

Every app supports `<APP>_GTK_SNAPSHOT=<folder>`. When set, the app:

- runs `NON_UNIQUE` so it does not hand off to a running instance,
- renders every visible window to `<folder>/<title>.png` after 3 seconds, with non-alphanumeric characters in the title replaced by `-`, and
- quits.

Use it to check a UI change without driving the app by hand. Set `XDG_CONFIG_HOME` to a temporary folder (with a copy of `gtk-4.0/`, following symlinks) to test settings-dependent states such as the system title bar mode. Runtime actions can be exercised on a normally started app through D-Bus (`gdbus call … org.gtk.Actions.Activate`).

---

# Packaging

- The version appears in four places, all changed together in a release:
  1. `packaging/<app-name>.spec` (`Version:`, `Release:` and a `%changelog` entry); the root `Makefile` reads it to name the source archive
  2. `gtk/CMakeLists.txt` (`project(... VERSION ...)`), which feeds the About dialog
  3. the metainfo `<releases>` list (newest first)
  4. `CHANGELOG.md` and the README's current-release line
- The root `Makefile` builds `<app-name>-<version>.tar.gz` from the checked-out tree (excluding build folders) and the SRPM into `result/`. `.copr/Makefile` only includes it, for COPR's `make_srpm` method. No Git tags are required.
- The spec builds from the repository root with `%cmake -DBUILD_TESTING=ON`, runs `%ctest`, and validates the installed desktop and metainfo files.
- CMake installs the binary, the desktop file, the metainfo file and the hicolor icons.
- The desktop file sets `Icon=<app-name>`, `StartupNotify=true`, relevant `Categories` and `Keywords`, and `MimeType` plus `%F` for apps that open files.
- The metainfo uses `<id>` = `<app-id>`, a `<launchable type="desktop-id">`, `<developer id="com.linuxgamerlife">`, an OARS `<content_rating>`, and a `<release>` entry for every version.
- Before release: `desktop-file-validate`, `appstreamcli validate --no-net`, a build of the SRPM and a binary RPM rebuilt from that SRPM.

---

# New Application Checklist

- [ ] Application ID, binary, icon, config, namespace and CSS prefix follow **Application Identity**
- [ ] Root `CMakeLists.txt` builds `gtk/`; `src/core` has no widget headers and has tests
- [ ] `style.css` uses named colours, prefixed derived colours and the fixed status colours only; square window corners
- [ ] ThemeWatcher with live reload, debounce, no-portal fallback, monospace font for every mono class, and `.dark`
- [ ] Components, spacing and radii from **Layout and Components**
- [ ] Header bar: logo, actions, primary action, main menu, in the standard order; tooltips name shortcuts
- [ ] Both title bar modes, Ctrl+Shift+M, menu checkbox, persisted; a named primary input
- [ ] Main menu sections in the standard order; Alt opens and closes it
- [ ] Standard shortcuts registered as accelerators or focused-widget shortcuts
- [ ] About dialog with the standard fields
- [ ] Dialogs, notices and help as in **Dialogs and Secondary Windows**
- [ ] Settings in `settings.ini` through a tested `Settings` class
- [ ] No blocking work on the main loop; external tools run without a shell
- [ ] Snapshot smoke test
- [ ] Desktop file, metainfo, icons at 48/64/128/256, spec, root and COPR Makefiles, CI

---

# Compliance

Each application records any deviation from this standard, with the reason and the plan to remove it, in its own repository (README or project notes). This document does not track individual apps.

---

# Summary

Every LGL GTK app:

- follows the desktop theme live and never ships its own palette
- shares the same components, spacing and corner radii
- shows the same logo, header bar layout and main menu
- lets the user choose between the app's title bar and the compositor's, with Ctrl+Shift+M
- opens and closes its menu with Alt and shares the same shortcuts
- stores preferences in `~/.config/<app-name>/settings.ini`
- keeps logic in a tested, widget-free core and builds with warnings as errors
