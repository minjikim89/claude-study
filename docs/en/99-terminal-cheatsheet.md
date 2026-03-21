# Mac Terminal & Claude Code — Essential Command Cheat Sheet

> A complete reference for essential commands and shortcuts for Mac users new to the terminal

---

## 1. Navigation

| Command | Description |
|---|---|
| `pwd` | Print the current directory path (where am I?) |
| `ls` | List files and folders in the current directory |
| `ls -al` | Detailed list including hidden files (permissions, size, date, etc.) |
| `ls -lh` | List with file sizes in human-readable units (KB, MB) |
| `cd [folder]` | Navigate into a specific folder |
| `cd ..` | Go up one level to the parent folder |
| `cd ~` | Jump immediately to the home directory (/Users/username) |
| `cd -` | Go back to the previous directory |
| `cd /` | Go to the root directory |
| `open .` | Open the current folder in Finder (Mac-only, very useful) |
| `open [filename]` | Open a file with its default application |

---

## 2. File and Directory Operations

| Command | Description |
|---|---|
| `mkdir [name]` | Create a new folder |
| `mkdir -p a/b/c` | Create nested folders in one step (including intermediate paths) |
| `touch [name]` | Create a new empty file |
| `cp [source] [dest]` | Copy a file |
| `cp -r [folder] [dest]` | Copy an entire folder (`-r` = recursive) |
| `mv [source] [dest]` | Move or rename a file or folder |
| `rm [filename]` | Delete a file (bypasses Trash — permanent!) |
| `rm -r [folder]` | Delete a folder and all its contents (use with caution!) |
| `rm -i [filename]` | Prompt for confirmation before deleting (safer) |
| `cat [filename]` | Print the full contents of a file to the terminal |
| `less [filename]` | View file contents with scrolling (`q` to quit) |
| `head -n 10 [file]` | Show the first 10 lines of a file |
| `tail -n 10 [file]` | Show the last 10 lines of a file |
| `wc -l [filename]` | Count the total number of lines in a file |
| `diff [fileA] [fileB]` | Compare the differences between two files |
| `which [command]` | Show the path where a command is installed |

---

## 3. Search

| Command | Description |
|---|---|
| `find . -name "*.txt"` | Find all .txt files under the current directory |
| `grep "term" [file]` | Search for specific text inside a file |
| `grep -r "term" .` | Search for text recursively in the current directory |
| `grep -i "term" [file]` | Case-insensitive search |
| `grep -n "term" [file]` | Show line numbers in search results |

---

## 4. Common Terminal Shortcuts

### Execution Control

| Shortcut | Description |
|---|---|
| `Tab` | Auto-complete file/folder/command names (use this constantly!) |
| `Tab Tab` | Show all completion candidates when there are multiple |
| `Ctrl + C` | Force-stop / cancel the running command |
| `Ctrl + Z` | Pause the running process (sends it to background) |
| `Ctrl + D` | Close the terminal session (same as `exit`) |
| `Ctrl + L` | Clear the screen (same as `clear` command) |
| `↑` / `↓` | Recall previous / next commands from history |
| `!!` | Re-run the last command (e.g., `sudo !!` → re-run last command as admin) |
| `history` | Show the full command history |
| `Ctrl + R` | Search command history (type a keyword → shows matching past commands) |

### Text Editing (Command Line)

| Shortcut | Description |
|---|---|
| `Ctrl + A` | Move cursor to the beginning of the line |
| `Ctrl + E` | Move cursor to the end of the line |
| `Ctrl + U` | Delete everything to the left of the cursor |
| `Ctrl + K` | Delete everything to the right of the cursor |
| `Ctrl + W` | Delete the word immediately before the cursor |
| `Ctrl + Y` | Paste the last deleted text (Yank) |
| `Ctrl + T` | Swap the two characters around the cursor (fix typos) |
| `Option + ←` | Move left one word at a time (Mac Terminal / iTerm2) |
| `Option + →` | Move right one word at a time |
| `Option + Delete` | Delete one word (Mac default terminal) |

---

## 5. iTerm2-Specific Shortcuts

> Switching from the default Mac terminal to iTerm2 significantly boosts productivity.

### Split Panes and Tab Management

| Shortcut | Description |
|---|---|
| `Cmd + D` | Split vertically (side by side) |
| `Cmd + Shift + D` | Split horizontally (top and bottom) |
| `Cmd + ]` / `Cmd + [` | Move between split panes |
| `Cmd + Option + arrow key` | Resize split panes |
| `Cmd + W` | Close current pane/tab |
| `Cmd + T` | Open a new tab |
| `Cmd + 1–9` | Jump directly to that tab number |
| `Cmd + ←` / `Cmd + →` | Move to previous/next tab |

### Convenience Features

| Shortcut | Description |
|---|---|
| `Cmd + F` | Search text in the terminal window |
| `Cmd + Shift + H` | Paste history (list of previously copied items) |
| `Cmd + ;` | Auto-complete popup |
| `Cmd + K` | Clear screen including scroll buffer |
| `Cmd + /` | Highlight cursor location (find your cursor) |
| `Cmd + Option + E` | Search across all tabs/panes (Expose) |
| `Cmd + Shift + I` | Broadcast input to all panes simultaneously |
| `Cmd + Enter` | Toggle full screen |
| `Cmd + +` / `Cmd + -` | Increase / decrease font size |
| `Cmd + 0` | Reset font size to default |

---

## 6. Claude Code Commands and Shortcuts

### Launch and Exit

| Command / Key | Description |
|---|---|
| `claude` | Start Claude Code |
| `claude "question"` | Ask a single question and get an immediate response (no session) |
| `/exit` or `Ctrl + C` × 2 | Exit Claude Code session, return to terminal |
| `Escape` | Stop the current AI response mid-generation |

### Session Management Slash Commands

| Command | Description |
|---|---|
| `/stats` | View token usage and current session cost |
| `/cost` | View cumulative usage cost |
| `/compact` | Summarize conversation and optimize context (essential for long sessions) |
| `/clear` | Reset the current conversation history (start fresh) |
| `/undo` | Revert the last code change made by AI |
| `/files` | View the list of files Claude is currently reading |
| `/config` | View and change settings |
| `/help` | Show help |

### Input Shortcuts

| Shortcut | Description |
|---|---|
| `Enter` | New line (continue typing) |
| `Ctrl + J` | New line (alternative key) |
| `Option + Enter` | Send message (default, configurable) |
| `↑` / `↓` | Recall previously sent prompts |

### Permission Responses (When Claude Modifies Files or Runs Commands)

| Key | Description |
|---|---|
| `y` | Allow this one time |
| `n` | Deny |
| `a` | Always allow for this session |

---

## 7. Essential Git Commands (Version Control)

> The Git commands you'll naturally encounter while using Claude Code

| Command | Description |
|---|---|
| `git status` | Check the status of changed files (most frequently used) |
| `git add [file]` | Stage a specific file for commit |
| `git add .` | Stage all changed files |
| `git commit -m "message"` | Commit staged files |
| `git log --oneline` | View a compact commit history |
| `git diff` | View a diff of changes |
| `git branch` | List current branches |
| `git checkout -b [name]` | Create a new branch and switch to it |
| `git push` | Upload local commits to the remote repository |
| `git pull` | Fetch the latest changes from the remote |
| `git clone [URL]` | Clone a remote repository to your computer |

---

## 8. System Information

| Command | Description |
|---|---|
| `df -h` | Check total available disk space |
| `du -sh .` | Check how much space the current folder uses |
| `du -sh *` | Check space used by each item in the current folder |
| `top` | Real-time CPU and memory usage monitor (`q` to quit) |
| `htop` | Improved version of `top` (install separately: `brew install htop`) |
| `ps aux` | List all currently running processes |
| `kill [PID]` | Kill a specific process (find PID with `ps`) |
| `whoami` | Show the current logged-in username |
| `sw_vers` | Show macOS version information |
| `uname -a` | Show system kernel information |
| `uptime` | Show how long the computer has been running |

---

## 9. Package Management (Homebrew)

> The package manager used to install development tools on Mac

| Command | Description |
|---|---|
| `brew install [package]` | Install a package |
| `brew uninstall [package]` | Uninstall a package |
| `brew list` | List installed packages |
| `brew update` | Update Homebrew itself |
| `brew upgrade` | Upgrade all installed packages to the latest version |
| `brew search [keyword]` | Search for packages |

---

## 10. Permissions and Networking

| Command | Description |
|---|---|
| `sudo [command]` | Run command with admin privileges (password required) |
| `chmod +x [file]` | Grant execute permission to a file |
| `chown [user] [file]` | Change the owner of a file |
| `ping [address]` | Check network connectivity (`Ctrl+C` to stop) |
| `curl [URL]` | Fetch data from a URL (API testing, etc.) |
| `ifconfig` | Show network interface info (check IP address) |

---

## Quick Reference: Top 10 Most-Used

| Rank | Key / Command | One-line summary |
|---|---|---|
| 1 | `Tab` | Auto-complete names |
| 2 | `Ctrl + C` | Force stop |
| 3 | `↑` arrow key | Reuse previous command |
| 4 | `cd` / `ls` / `pwd` | Navigate and check location |
| 5 | `Ctrl + R` | Search command history |
| 6 | `Ctrl + A` / `Ctrl + E` | Jump to beginning/end of line |
| 7 | `Cmd + D` (iTerm2) | Split screen |
| 8 | `open .` | Open current folder in Finder |
| 9 | `/compact` (Claude Code) | Optimize context |
| 10 | `Ctrl + L` | Clear screen |

---

**Tip**: Start by learning the Top 10, then refer back to this sheet as needed. The terminal is all about repetition — after a few days of use, the commands become second nature.
