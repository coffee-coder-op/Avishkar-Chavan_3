# Secure Random Password Generator

## Description

**Secure Password Generator** is a desktop GUI application built with Python
and Tkinter that generates cryptographically secure, customizable passwords.
Users can choose the password length, which character types to include
(uppercase, lowercase, numbers, symbols), and whether to exclude visually
ambiguous characters. Every generated password is guaranteed to contain at
least one character from each selected type, is rated for strength, is
automatically copied to the clipboard, and is kept in a temporary,
session-only history of the last 5 passwords.

## Features

- Clean, modern Tkinter GUI with a title, password display field, and
  organized sections
- Password length control from **8 to 64 characters** via a Scale (slider)
  synced with a Spinbox
- Checkboxes for Uppercase, Lowercase, Numbers, and Symbols
- Enforces a minimum of **2 selected character types**, with a clear error
  message if not satisfied
- "Exclude ambiguous characters" checkbox that removes `0`, `O`, `l`, `1`
  from the character pools
- Cryptographically secure password generation using Python's `secrets`
  module (never `random`)
- Guarantees at least one character from **every selected type**, then
  fills the rest of the password securely and shuffles the result using
  `secrets.SystemRandom().shuffle()`
- Password strength indicator (Weak / Medium / Strong) with a visual
  progress bar, based on length and character diversity
- "Copy to Clipboard" button using `pyperclip`, plus **automatic** clipboard
  copy immediately after generation, with an on-screen confirmation message
- Generation history showing the **last 5 passwords**, kept only in memory
  (never written to a file, database, JSON, or CSV)
- "Clear History" button
- Graceful error handling for invalid lengths, insufficient character
  types, missing passwords, and clipboard failures — the app never crashes
  on normal invalid input

## Technologies Used

- **Python 3** — core programming language
- **Tkinter** — Python's standard GUI toolkit, used to build the entire
  interface (windows, buttons, checkboxes, scales, listboxes, etc.)
- **secrets** — Python's standard library module for generating
  cryptographically strong random numbers, suitable for managing secrets
  such as passwords and security tokens
- **string** — provides ready-made character sets
  (`ascii_uppercase`, `ascii_lowercase`, `digits`, `punctuation`)
- **pyperclip** — a small cross-platform library used to copy text to the
  system clipboard

## Installation

Open a terminal (Command Prompt, PowerShell, or the VS Code integrated
terminal) in the project folder.

1. Confirm Python is installed (Python 3.8+ recommended):

```bash
python --version
```

2. (Optional but recommended) create and activate a virtual environment:

```bash
python -m venv venv
venv\Scripts\activate
```

3. Install the required dependency:

```bash
pip install -r requirements.txt
```

> Tkinter ships with the standard Windows Python installer, so it does
> **not** need to be installed separately and is intentionally left out of
> `requirements.txt`.

## Running the Application

From the project folder, run:

```bash
python password_generator.py
```

### Running from VS Code

1. Open the `random_password_generator` folder in VS Code
   (`File > Open Folder...`).
2. Open `password_generator.py`.
3. Make sure the correct Python interpreter is selected in the bottom-right
   corner of VS Code.
4. Press **Run > Run Without Debugging** (or click the ▶ button in the top
   right corner) — this launches the GUI window.

## How It Works

1. **Collect selections** — When "Generate Password" is clicked, the app
   reads the chosen length and which character-type checkboxes are ticked.
2. **Validate input** — The app checks that the length is between 8 and 64
   and that at least 2 character types are selected. If not, a clear error
   message box is shown and no password is generated.
3. **Build character pools** — For each selected type, the app builds a
   character pool from the `string` module. If "Exclude ambiguous
   characters" is checked, the characters `0`, `O`, `l`, `1` are stripped
   from the relevant pools before anything else happens.
4. **Guarantee diversity** — The app uses `secrets.choice()` to pick **one**
   character from *each* selected pool first. This guarantees that every
   selected type is represented in the final password, rather than relying
   on chance from a single combined pool.
5. **Fill remaining length** — The remaining characters needed to reach the
   requested length are chosen with `secrets.choice()` from the combined
   pool of all selected types.
6. **Secure shuffle** — All characters (guaranteed + random fill) are
   shuffled together using `secrets.SystemRandom().shuffle()` so the
   guaranteed characters aren't predictably placed at the start of the
   password.
7. **Display & rate strength** — The password is shown in a read-only entry
   field, and a strength score is calculated from its length and the
   number of distinct character types used.
8. **Clipboard** — The password is copied to the clipboard automatically
   right after generation, and can also be copied manually at any time
   using the "Copy to Clipboard" button.
9. **History** — The password is appended to an in-memory list. If the
   list grows beyond 5 entries, the oldest entry is removed.

## Security Explanation

### Why `secrets` instead of `random`

Python's built-in `random` module uses a **Mersenne Twister** pseudo-random
number generator. It is fast and great for simulations, games, and
statistics, but it is **not cryptographically secure**: its internal state
can, in principle, be reconstructed from observed outputs, which would let
an attacker predict future "random" values — including passwords.

The `secrets` module, added to the standard library specifically for
security-sensitive tasks, draws randomness from the operating system's
cryptographically secure source (e.g., `os.urandom`). The official Python
documentation explicitly recommends `secrets` over `random` for generating
passwords, security tokens, and similar values:

<https://docs.python.org/3/library/secrets.html>

This project uses `secrets.choice()` to pick individual characters and
`secrets.SystemRandom().shuffle()` to shuffle the final character list —
both are secure, unpredictable operations. The `random` module is never
imported or used anywhere in this project.

### Why password history is not saved to disk

Storing generated passwords in a file, database, or any other persistent
location would create a security risk: anyone with access to that file
(now or in the future, including backups) could recover previously
generated passwords. Because this application is a demonstration/utility
tool, history is kept **only in memory** (a plain Python list) for the
current run of the program. As soon as the application is closed, the
history is gone. Passwords are also never printed to the console or logged
anywhere.

## Testing

Use the following checklist to manually verify the application:

1. **Length below 8** — Set the length to a value below 8 (e.g. via the
   Spinbox) and click Generate. Expected: an error message about length
   is displayed and no password is generated.
2. **Length above 64** — Attempt to set the length above 64. Expected:
   the Scale/Spinbox is capped at 64, or an error message appears if
   bypassed.
3. **Only one character type selected** — Uncheck all but one character
   type and click Generate. Expected: an error message stating at least
   2 types are required.
4. **Two character types selected** — Select exactly 2 types (e.g.
   Uppercase + Numbers) and generate. Expected: a valid password
   containing only those two character types.
5. **All four character types selected** — Select Uppercase, Lowercase,
   Numbers, and Symbols, then generate. Expected: the password contains
   at least one character from each of the four types.
6. **Ambiguous characters excluded** — Check "Exclude ambiguous
   characters", generate several passwords, and confirm none contain
   `0`, `O`, `l`, or `1`.
7. **Clipboard copying** — Generate a password, then paste (Ctrl+V) into
   any text field. Expected: the pasted text matches the displayed
   password (this verifies both automatic and manual copy).
8. **Strength indicator** — Generate a short password with few types
   (Weak), a medium-length password with 2 types (Medium), and a long
   password (16+) with 3-4 types (Strong). Expected: the strength label
   and progress bar update accordingly.
9. **History limit of 5** — Generate 6 or more passwords in a row.
   Expected: the history list always shows only the most recent 5,
   with the oldest entry removed each time a new one is added.
10. **Clear History** — Click "Clear History". Expected: the history
    list becomes empty immediately.

## References

- Python official documentation — `secrets` module:
  <https://docs.python.org/3/library/secrets.html>
- Python official documentation — `random` module (for comparison):
  <https://docs.python.org/3/library/random.html>
- Python official documentation — `string` module:
  <https://docs.python.org/3/library/string.html>
- Python official documentation — `tkinter`:
  <https://docs.python.org/3/library/tkinter.html>
- `pyperclip` package documentation:
  <https://pypi.org/project/pyperclip/>
