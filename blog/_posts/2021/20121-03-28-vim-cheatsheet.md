---
layout: post
title: VIM cheatsheet
description: "My personal vi cheatsheet"
tag: vim vi linux
category: Linux
date: 2021-03-28 10:12:23
---
Listen my favorite Vim shortcuts and commands.

## Global commands

### File handling
gf - edit file under cursor
gx - open file under cursor

### Insert and jump
gi - jump to last edit and switch to insert mode 
gI - insert at the beginning of the line
gv - go to visual mode and use last selection you made

### My .vimrc

```vimrc
" enable filetype detection:
filetype on
filetype plugin on
filetype indent on " file type based indentation

set encoding=utf-8
set expandtab
set tabstop=2

" Status bar
set laststatus=2

" Last line
set showmode
set showcmd

 " netrw
 let g:netrw_banner = 0
 let g:netrw_liststyle = 3
 let g:netrw_browse_split = 4
 let g:netrw_winsize = 25
 let g:netrw_altv = 1
" augroup ProjectDrawer
"   autocmd!
"   autocmd VimEnter * :Vexplore
" augroup END

" in makefiles, don't expand tabs to spaces, since actual tab characters are
" needed, and have indentation at 8 chars to be sure that all indents are tabs
" (despite the mappings later):
autocmd FileType make set noexpandtab shiftwidth=8 softtabstop=0
```

