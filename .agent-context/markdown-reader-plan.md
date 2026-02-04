## Project overview

Build a **.NET 9 MAUI** application called **"Docs Everywhere"** that runs on **Windows desktop** and **Android** (emulator is fine) and acts as a **markdown (.md) reader and editor** for files stored in a synced Google Drive folder (Windows) or opened from Google Drive (Android).

The app must support:

- Opening `.md` files without locking them.
- Automatically reloading when the underlying file changes on disk (Windows).
- Being associated with `.md` files on both Windows and Android so that users can "open with" this app.

---

## 1. Functional requirements

### 1.1 Platforms and project setup

- Use **.NET 9 MAUI** to create a single cross-platform solution targeting:
  - Windows desktop.
  - Android.
- Use a clean architecture (e.g., **MVVM**) with:
  - Separate projects or folders for UI, view models, and services.
  - Dependency injection for services (file IO, markdown rendering, settings).

### 1.2 Markdown viewing and editing

- The app must be able to open `.md` files and display them with proper markdown rendering:
  - Headings, lists, code blocks, links, tables.
- Provide a **split view** editor page:
  - **Left side:** editable text area for raw markdown.
  - **Right side:** live preview using a markdown rendering control.
  - Preview updates as the user types (with a small debounce).
- Provide an option to toggle between:
  - View-only mode.
  - Edit mode (with preview).
- Support basic markdown editing UX:
  - Monospaced font in the editor.
  - Line numbers optional, but nice to have.
  - Basic keyboard shortcuts (`Ctrl+S` to save, `Ctrl+F` to search inside the document if feasible).

---

## 2. File access and Google Drive usage

### 2.1 Windows – local Google Drive folder

- Assume Google Drive for desktop is installed and syncing a local folder (e.g., under the user's profile).
- The app must allow the user to:
  - Choose a base folder (e.g., the root of the synced Docs folder) on first run.
  - Browse `.md` files under that folder (simple tree or list with search).
  - Open a selected `.md` file into the editor/preview.

### 2.2 Android – open from Google Drive

- The app must handle file open requests from Google Drive and other file managers:
  - Register for `.md` extension and relevant mime types (e.g., `text/markdown`, `text/plain`, `*/*` as needed).
  - When the user taps "Open with" on a `.md` file in Google Drive, the app should start and display that file in the editor/preview.

### 2.3 Launch modes

- Handle both:
  - **Normal launches** (no file passed): show a home screen with:
    - Recent documents list.
    - "Open file" button (file picker).
  - **File-association launches** (file path/URI passed): open directly in the editor.

---

## 3. File locking and reload behavior (Notepad++-style)

### 3.1 Non-locking file reads

- The app must **not lock** files for reading:
  - When loading a file from disk, open it in a way that allows other processes to read/write simultaneously (e.g., shared read).
  - Close file streams immediately after reading; do not keep open file handles for the loaded document.

### 3.2 Saving changes

- The app must be able to save changes to the file:
  - Overwrite the file with the edited content on save.
  - Avoid leaving temporary/lock files in the folder (unless necessary and cleaned up).

### 3.3 Automatic reload on change (Windows)

- Implement a file-change detection mechanism (e.g., directory watcher or polling) so that:
  - If the underlying `.md` file changes on disk and the document in the app has **no unsaved edits**, the app automatically reloads the file and refreshes the view.
  - Show a non-blocking indicator (e.g., toast/banner) saying "File changed on disk and was reloaded."

### 3.4 Conflict handling

- If the file changes on disk while the user has unsaved edits in the app:
  - Do **not** automatically overwrite the editor contents.
  - Instead, show a prompt or banner with options:
    - "Reload from disk (discard my changes)"
    - "Keep my version (ignore external changes)"
  - Make it easy to choose either behavior.

### 3.5 Android behavior

- Android may not support the same continuous watching semantics:
  - At minimum:
    - Use non-locking reads.
    - Provide a manual "Reload from source" button to re-read the file when the user wants to sync with external changes.

---

## 4. App behavior and navigation

### 4.1 Home screen (normal launch)

- Show:
  - "Open file" action.
  - List of recent files opened (with path/location, last opened time).
- Recent list must work for both Windows and Android.

### 4.2 Editor screen

- Title bar displaying current file name and a small indicator if there are unsaved changes.
- Commands:
  - Save.
  - Reload (manual reload from disk/URI).
  - Toggle View/Edit.
  - Optional: light/dark theme toggle.

### 4.3 Navigation

- Easy way to go back to the home screen from the editor.
- Preserve unsaved changes warnings when navigating away.

---

## 5. Cross-platform considerations

- Design the file access abstraction so that:
  - Windows implementation works with local file paths.
  - Android implementation can work with content URIs from intents and file pickers.
- Ensure that markdown rendering and editor behavior is consistent across Windows and Android.
- Provide project configuration and build instructions so that:
  - The app can be built and run on Windows.
  - The Android emulator can be started and the app deployed easily for demo purposes.

---

## 6. Demo-friendly features

- Include a **sample docs set** in a `Docs` folder (e.g., `README.md`, `architecture.md`, `tips.md`) for quick demo use.
- Support deep linking from Windows Explorer:
  - Double-click `.md` file in the synced Google Drive folder → app opens with that file.
- Support deep linking from Google Drive (Android):
  - "Open with Docs Everywhere" → app opens with that file.
- Provide a simple visual indicator that the **same file** is open on both platforms:
  - E.g., display the full relative path within the chosen docs root, or a unique ID.

---

## 7. Non-functional requirements

- Code should be structured and commented enough for other .NET developers to follow.
- Provide a short `README` explaining:
  - How to run on Windows.
  - How to deploy to Android emulator.
  - How file association and "open with" behavior works for both platforms.
  - Any platform-specific permissions or settings needed.
