---
layout: post
title: "Vim abbreviation and map"
description: "Vim abbreviation and map"
categories: [Vim]
tags: [Vim]
redirect_from:
  - /2018/05/25/
---
## Vim abbreviation and map
abbreviation
```
:abbreviate hdco aperjump.com
:abbreviate ww wei@aperjump.me
:ab ww wei@aperjump.me
```
There are strict limitations on what you can use a an abbreviation
- must be an identifier consisting entirely of keyword characters:
```
ab url http://www.baidu.com
ab krd Kind regards, <CR><CR>Damian Conway
```
- must consist entirely of non-keyword characters, except for the last one
```
ab #1 I'll make that my number one priority
ab --c <CR>----cut------cut-----cut-<C-O>:center<CR><DOWN>
```
- must end in a **non-keyword** character
```
ab ph# +1-802-555-1122
ab ??? //
ab orz! I'm not worthy
```

expand abbreviation
Abbreviations also require some trailing context to know they've been entered
--> you need to type a non-keyword character after the abbreviation before it will be expanded
You can expand an abbreviation without trailing context by typing `CTRL-]`

How to review abbreviation
If you type 
```
:abbreviate
```
without an argumant, you can get a list of the active abbreviations
To remove abs, just type `:unabbreviate bqc` or clear all `:abclear`
To deactivate them type a literal `<CRTL-V>` before the abbreviation
For example, if you frequently need to save to a particular file:
```
:ab bak /usr/local/tmp/backup/damian/checkpoint
```
A better option is `iabbrev` and `cabbrev` versions
With these you can tell Vim exactly where to expand the abbreviation:
Only in insertions:
```
:iabbrev bqc <blockquote><cite><CR><CR></cite></blockquote><UP><TAB>
```
only in command line:
```
:cabbrev bak /usr/local/tmp/backup/damian/checkpoint
```
**You can specify an abbreviation should expand to the result of some expression in the vimmish command language**
`:iabbr <expr> TS strftime("%c")`
`:iabbr <expr> PPP getreg('')`

### 1. Maps
Abbreviations are grreat, but suffer from two major constraints
They're only available in Insert mode and on the command line
They require an extra character typed after them

Mays remedy both those problems
A map creates a sequence that is expanded as soon as typed
A **macro**
We can specify maps for specific modes

#### 1.1 Insert map
They can be used to replace `iabbr`
```
:imap ww http://www.google.com
:imap ee wei@aperjump.me
```
advantage: don't require additional character to be typed

#### 1.2 Normal map
```
:nmap X dip
```
dip: delete the interior of paragraph
Likewise, if visual block mode more useful than visual line mode.
We can use this
```
:nmap v <C-V>
:nmap <Space> <PageDown>
```
Or we can streamline file navigation:
```
:nmap <DOWN> :next<CR>
:nmap <UP> prev<CR>
```
#### 1.3 Command line maps
If you frequently write to a backup file:
```
:w ~/backup/latest<CR>
```
You can add this to `.vimrc`
```
cmap wb w ~/backup/latest<CR>
```
or we can simplify subdirectory tours:
```
:cmap *** **/*
```
which then allows:
```
:next examples/***.c
```
