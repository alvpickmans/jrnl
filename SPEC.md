# jrnl - Simple Journal Command

## Overview

A bash command-line tool for quick note-taking with date-stamped entries.

## Command Interface

```bash
jrnl "note text"               # Quick add entry
jrnl "note text" -i            # Interactive add entry
jrnl "note text" -i -e <editor> Interactive with editor
jrnl read                      # Show entire journal (pipeable)
jrnl open [-e <editor>]        # Open journal in editor
jrnl -h, jrnl --help           # Show usage
jrnl -v, jrnl --version        # Show jrnl version
```

**Important**: All commands require arguments. Running `jrnl` without arguments shows help.

## File Path Resolution

Priority order:
2. `$JRNL_FILE` environment variable
3. Default: `~/main.jrnl`

## Commands

### Quick Add Mode
Appends a date-stamped entry to the journal.

- Ensures journal file exists
- Appends entry as `YYYY-MM-DD title`
- Adds blank line separator if file is not empty

### Interactive Add Mode (`-i`)
Opens an editor for additional entry context with LSP support.

- Creates a temporary file with `.jrnl` extension for editor detection
- Pre-populates temp file with `YYYY-MM-DD title` header
- Opens `$EDITOR` (or `vi` as fallback) with temp file
- Supports `-e <editor>` flag to specify alternative editor
- Validates first line must be in `YYYY-MM-DD title` format
- Writes header and tabbed content to journal if content exists
- Removes temp file

**Error**: If first line is not in `YYYY-MM-DD title` format, prints error to stderr and exits.

**Error**: If `jrnl` is called without any arguments, print help.

### Open Mode
Opens the journal file in an editor.

- Ensures journal file exists
- Uses `$EDITOR` environment variable (defaults to `vi`)
- Supports `-e <editor>` flag to specify alternative editor

### Show Mode
Outputs the entire journal file to stdout.

- Ensures journal file exists
- Outputs file content (pipeable)

### Help Mode
Displays usage information.

## File Format

### Entry Format
```
YYYY-MM-DD Entry title
	tabbed continuation line
	tabbed continuation line

YYYY-MM-DD Another entry
```

### Entry Separation Rules
- Each entry starts with date and title on one line
- Multi-line entries use tab indentation for continuation lines
- Entries are separated by blank lines
- Multiple consecutive entries can have the same date

### Example
```
2025-01-12 Buy milk
2025-01-12 Call mom about dinner
2025-01-13 Project planning
	First task
	Second task

2025-01-13 Another note separate entry
```

## Implementation Details

### Helper Functions

**`get_today_date()`**
- Returns `$(date +%Y-%m-%d)`

**`ensure_journal_file()`**
- Creates file with parent directories if missing

**`add_entry(text)`**
- Checks if file is empty
- If empty: appends `YYYY-MM-DD ${text}\n`
- If not empty: appends `\nYYYY-MM-DD ${text}\n`

**`add_interactive_entry(title)`**
- Creates temporary file with `mktemp`
- Appends title to journal: `YYYY-MM-DD ${title}\n`
- Opens editor with temp file
- Reads temp file line by line
- Appends each line to journal with tab prefix
- Removes temp file

**`show_journal()`**
- Ensures journal file exists
- Outputs file content via `cat`

### Error Handling

| Condition | Behavior |
|-----------|----------|
| File creation failure | Print error to stderr, exit 1 |
| Editor command not found | Fallback to `vi` |

## Usage Examples

```bash
# Quick notes
jrnl Important meeting at 3pm
jrnl "TODO: review pull request"

# Interactive notes with context
jrnl -i "Startup meeting"
jrnl -i -e nano "Startup meeting"

# Read and pipe
jrnl read | grep "meeting"
jrnl read | less
jrnl read > backup.txt
```

## Future Features (v2+)

### Typed Entry System

Allow structured entry types with custom templates and arguments.

#### Command Syntax
```bash
jrnl --type <type> <text> [--arg value] [--arg value]
jrnl +<type> <text> [--arg value]
jrnl read --raw
jrnl read --type <type>
```

#### Config File: `~/.jrnl.toml`
```toml
[task]
template = "[TODO] $0 (due: $due)"
required = ["due"]

[meeting]
template = "[MEETING] $0 at $time ($location)"
required = ["time", "location"]
optional = ["notes"]
```

#### Storage Format
```
2025-01-13 task Deliver project presentation 
	due = "tomorrow"

2025-01-13 Buy milk

2025-01-13 meeting Team sync 
	time = "3pm"
	location = "Room 4B"

```

#### Template Variables
- `$0` - text after type declaration
- `$<arg>` - named arguments from block

#### Behavior
- Config file optional - if missing, no formatted string shown
- Missing required args → interactive prompt: `Enter due date: `
- Multi-line title → error
- Flags must come before title
- `+type` expands to `--type type`

#### Display Modes
- `jrnl show` - formatted output (templates applied)
- `jrnl show --raw` - raw blocks with arguments
- `jrnl show --type <type>` - filter by type

#### Example Usage
```bash
jrnl --type task "Deliver project" --due tomorrow
jrnl +task "Buy milk" --due today
jrnl +meeting --time 3pm --location "Room 4B" "Team sync" 
```

