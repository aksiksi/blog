---
layout: post
title: "A Vim Cheatsheet"
author: {{ site.author.name }}
tags:
    - vim
    - cheatsheet
    - learning
---

## Basics

* To enter a command: `:`
* To quit Vim: `:q`
* To save current file: `:w`
* To save and quit: `:wq`
* Save current file as: `:saveas <filename>`
* By default, Vim is in **NORMAL** mode. To move, use up/down/left/right: `k/j/h/l`
* To insert text, hit `i` to switch into **INSERT** mode.

### Modes

1. Normal mode (`n`): used for navigation and modification (yank/delete/paste)
2. Insert mode (`i`): used to insert text (e.g., write code)
3. Visual line mode (`V`): used to select multiple lines
	* Visual mode (`v`): select at the word level

## Navigation

* Up 10 lines: `10k`
* Down 10 lines: `10j`
* Move by a word: `w`
* Move backward a word: `b`
* Move to the end of a word: `e`
* Particular line: `:<line_no>`
* Start of line: `0`
	* Start of non-empty line: `^`
* End of line: `$`
* Top of file: `gg`
* Bottom of file: `G`
* Top of window: `H` (high)
* Bottom of window: `L` (low)
* Middle of window: `M` (middle)
* Move to opening brace: `[{`
* Move to closing brace: `]}`
* To skip code blocks: `[[` and `]]` (up and down)
* To move up/down a line to first non-space: `+` and `-`

## Scrolling

* Scroll up a line: `Ctrl-y`
* Scroll down a line: `Ctrl-e`
* Scroll up by half-page: `Ctrl-u`
* Scroll down half-page: `Ctrl-d`
* Scroll up by full page: `Ctrl-f`
* Scroll down by full page: `Ctrl-b`
* Move cursor (and line) to top of screen: `zt`
	* Cursor to bottom: `zc`
	* Cursor to middle: `z.`

## Editing

* Insert blank line: `o` (after current line) or `O` (before)
* Delete a line: `dd`
* Paste a line: `p`
	* Paste line above current line: `P`
* Paste a line from clipboard (global register): `"+p`
* To append to the end of a line: `A`
* To insert at the start of a line: `I`
* Select the whole line: `Shift+v`
* To delete a word: `dw`
* To delete a letter: `dl`
* To change a word (delete, then INSERT): `cw` 
* To repeat any command: `.`
    * For example, to insert 5 blanks line above current: `O<ESC>5.`

## Searching

* To search forward: `/`
    * Type in the pattern, then hit Enter
* To search back: `?`
* To search for current word at cursor: `*` (forward) or `#` (backward)
* After searching, go back to original position: `Ctrl-o`
* Find and replace: `:%s/<find>/<replace>/g`
	* `%` (in all lines), `s` (start), `g` (end)
	* You could limit to a lines 10 to 100 with `10,100` instead of `%`

### Marking

* Mark cursor position: `m<char>`
* Return to cursor position: `char`
* Return to beginning of line of marked position: `'<char>` 

## Splitting

* To split vertically: `:vsp`
* To split horizontally: `:sp`
* To switch between windows: `Ctrl-W, Ctrl-W`
* To switch in a certain direction: `Ctrl-W, [h/j/k/l]`
* To split equally: `Ctrl-W, =`
* To resize a window height (horizontal): `:resize <height>`
* To resize window width: `:vertical resize <width>`
* Max height: `Ctrl-W, _`
* Max width: `Ctrl-W, |`

## Tabs

* Open a file in new tab: `:tabedit <file>`
* Go to next tab: `gt`
* Go to previous tab: `gT`
* Close a tab window: `Ctrl-W c` 

## Buffers

* List current open buffers: `:ls`
* Load a particular buffer into current window: `:b<num>`
* Load buffer into split: `:sb<num>`

## Indenting and Folding

* To indent a block: `>`
* To unindent: `<`
    * Example: use `v` to enter visual mode, then `3j` to select 3 lines, then `>` to indent
* To indent in insert mode: `Ctrl-T` and `Ctrl-D` (d-indent)
* To open/close a fold: `zo, zc`
* To open/close all folds in a file: `zR, zM`

## Cut & Paste

* To enter visual mode: `v` or `V` (select full line)
* To cut a line: `dd`
* To cut a word: `dw`
* To copy a selected line: `y`
* To paste in general: `p`
* To paste from system clipboard: `"+p`
* To copy a word to system clipboard: `"+y`
* To copy entire file: `:%y+`
    * Explanation: `%` means "apply next command to all lines", `y` is yank/copy, and `+` means use system clipboard
* To copy something, delete something, then paste the original copied thing: `yy dd "0p`
* To paste something yanked earlier: `"1p` or higher to backwards
	* Use `:registers` to view all yanked data

## Session Management

* Save current session: `:mksession ~/session.vim`
* Load a session: `source ~/session.vim`

## Commands

* To view command history: `q:`
* To view search history: `q/`

## Diffing

* To diff all open splits/windows: `:windo diffthis`
    * `:windo <cmd>` executes the given command on *all* open windows
* To get and put hunks: `do` (get) and `dp` (put)
* To stop the diff: `:diffoff`

## Tags

* To create a tags file (in shell): `find . -name "*.[ch]" | ctags -L -`
	* Or use `:!` to execute from within Vim (blocking!)
* To search for a tag: `:tag <function_name>`
* To jump into function: `Ctrl+]`
* To jump back out: `Ctrl+t`
* To view tag stack: `:tags`
* To auto-complete using tags: `Ctrl-n` (next) and `Ctrl-p` (prev)

## Cscope

* To create a cscope database: `find . -name "*.[ch]" > cscope.files && cscope -q -b`
* To list all symbols (under cursor): `Ctrl-\ s`
* To go to definition: `Ctrl-\ g`
* Find functions calling this function: `Ctrl-\ c`
* Find files including this file: `Ctrl-\ i`
* Find this text string: `Ctrl-\ t`

## Other Stuff

* Open cursor in relevant application (e.g., URL in browser): `gx`
* Record a macro: `q<letter><commands>q`
    * Execute the macro: `@<letter>`
* To execute a single command while in INSERT mode: `Ctrl-o <command>`
    * After the command executes -- e.g., `10j` -- you will be put back into INSERT mode

## Useful Plugins

### CtrlP

Fuzzy file finder and search.

Shortcuts:

* Search: `Ctrl-p`
* Open file in new tab: `Ctrl-t`
* Open file in vertical/horizontal split: `Ctrl-v/x`
