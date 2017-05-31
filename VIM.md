# VIM
NB These are tips I've curated from various sources over a period of time that I find myself using fairly regularly, but not regularly enough for them to stick. It is by no means comprehensive. I attempt to credit those sources where I can.

## Comprehensive cheat sheets:

https://vim.rtorr.com/

https://www.cs.swarthmore.edu/help/vim/searching.html

## Navigation

`*` search for current word fwd (`#` bckwd)

Use backtick and double-backtick to jump between last two parts of file being edited.

## Copying, cutting, pasting

#### Copy all

`:%y`

**copy all to clipboard:**

`:%y+`


#### Copy a line without the newline break
`^y$`		

`^` goes to the first character of the line. `y` yanks until	`$` end of line.


#### yank
`yy or Y` – yank the current line, including the newline character at the end of the line
`y$` – yank to the end of the current line (but don't yank the newline character)
`yiw` – yank the current word (excluding surrounding whitespace)
`yaw` – yank the current word (including leading or trailing whitespace)

#### Selecting within/without quotes, braces etc

In visual mode, `i"` selects text inside double-quotes, `a"` selects that text and the double-quotes too. Similarly we can use `i'` and `a'`, `i[` and `a[`.

You can also swap out the "v" (visually select) for "d" (delete) or "y" (yank) or "c" (change).

You can also swap out the "i" (inside) for "t" (up to) or "a" (around).

## Block commenting
First, move the cursor to the first char of the first line in block code you want to comment, then type `CTRL + V`.

Vim will go into VISUAL BLOCK mode. Use `j` to move the cursor down until you reach the last line of your code block. Then type `Shift + I`.

Now vim goes to INSERT mode and the cursor is at the first char of the first line. Finally, type `#` then `ESC` and the code block is now commented.

To decomment, do the same things but instead of typing `Shift + I`, you just type `x` to remove all '#' after highlight them in VISUAL BLOCK mode.



## Find and replace

To find each occurrence of ‘eth0’ in the current line only, and replace it with ‘br0’, enter (first press `Esc` key and type):
`:s/eth0/br0/g`

To find and replace all occurrences of ‘eth1’ with ‘br1’, enter:
`:%s/eth1/br1/g`

To find and replace all occurrences of ‘eth1’ with ‘br1’, but ask for confirmation first, enter:
`:%s/eth1/br1/gc`

To find and replace all occurrences of case insensitive ‘eth1’ with ‘br1’, enter:
`:%s/eth1/br1/gi`


## Windows and splits

#### resize window
Resize window to 30 lines high
`:res 30`

Resize all windows (splits) to equal size
`Ctrl-w =`



