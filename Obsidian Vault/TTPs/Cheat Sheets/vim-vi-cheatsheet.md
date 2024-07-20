# Vi/Vim Cheat Sheet

## Modes
- Normal mode: Press `Esc` (default mode for navigation and commands)
- Insert mode: Press `i` (for inserting text)
- Visual mode: Press `v` (for selecting text)
- Command mode: Press `:` (for entering commands)

## Navigation

### Basic Movement
- `h`: Left
- `j`: Down
- `k`: Up
- `l`: Right
- `w`: Next word
- `b`: Previous word
- `e`: End of word

### Line Navigation
- `0`: Start of line
- `$`: End of line
- `^`: First non-blank character

### Screen Navigation
- `H`: Top of screen
- `M`: Middle of screen
- `L`: Bottom of screen
- `gg`: First line of file
- `G`: Last line of file
- `:n`: Go to line n

## Editing

### Inserting
- `i`: Insert before cursor
- `a`: Append after cursor
- `I`: Insert at beginning of line
- `A`: Append at end of line
- `o`: Open new line below
- `O`: Open new line above

### Deleting
- `x`: Delete character
- `dw`: Delete word
- `dd`: Delete line
- `D`: Delete from cursor to end of line

### Changing
- `r`: Replace single character
- `cw`: Change word
- `cc`: Change entire line
- `C`: Change from cursor to end of line

### Copying and Pasting
- `yy`: Yank (copy) line
- `yw`: Yank word
- `p`: Paste after cursor
- `P`: Paste before cursor

## Search and Replace

- `/pattern`: Search forward
- `?pattern`: Search backward
- `n`: Next occurrence
- `N`: Previous occurrence
- `:%s/old/new/g`: Replace all occurrences in file

## File Operations

- `:w`: Save
- `:q`: Quit
- `:wq` or `ZZ`: Save and quit
- `:q!`: Quit without saving

## Multiple Files

- `:e filename`: Edit a file
- `:bn`: Next buffer
- `:bp`: Previous buffer
- `:ls`: List buffers

## Window Management

- `:sp filename`: Split window horizontally
- `:vsp filename`: Split window vertically
- `Ctrl+w` then `h, j, k, l`: Navigate between windows

## Visual Mode

- `v`: Start visual mode
- `V`: Start linewise visual mode
- `Ctrl+v`: Start visual block mode

## Macros

- `q{letter}`: Start recording macro
- `q`: Stop recording macro
- `@{letter}`: Play macro

## Miscellaneous

- `.`: Repeat last command
- `u`: Undo
- `Ctrl+r`: Redo
- `:set number`: Show line numbers
- `:help keyword`: Get help

Remember, Vi/Vim has a steep learning curve, but these commands will help you get started and become more efficient over time.
