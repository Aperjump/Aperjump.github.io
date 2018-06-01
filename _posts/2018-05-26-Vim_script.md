---
layout: post
title: "Vim Script"
description: "Vim Script"
categories: [Vim]
tags: [Vim]
redirect_from:
  - /2018/05/26/
---

Vim has its own scripting language, most standard dynamic-language features
```
:vmap <expr> aa Select_Entire_File()
:nmap x :call Comment_To_EOF()
:imap <expr> \p Insert_Balanced_Tags('<p>')
:nmap	;y :call Taggle_Syntax()
```

Variables, expression, control structures, build-in functions ... 
For full details:
```
:help vim-scrpt-intro
```
For endless examples:
```
http://vim.sourceforge.net/scripts/
```

For detailed work examples:
```
http://www.ibm.com/developerworks/library/l-vim-script-1/
```
Five ways to execute vim scripts
(1) simplest: put them in `.vim` file
and then execute it with `:source`
`:source ./scriptfile.vim`
(2) `:source` the file you are editing
`:source %`
(3) If file somewhere in `$VIMRUNTIME` path
`runtime scriptfile.vim`
`:source` and `:runtime` are handy ways of factoring out parts of your `.vimrc`
More useful in a live session:
Can type commands directly on command-line:
`:call MyBackupFunc(expand('%'), {'all':1, 'save':'recent'})`
```
:command SF source scriptfile.vim
:command -nargs=1 MBF call MyBackupFunc(expand(<q-args>))
```
Even more useful is triggering a command from Normal mode...
Create `map` to type it for you:
```
:nmap ;s :source scriptfile.vim<CR>
:nmap \b :call MyBackupFunc(expand('%'), {'all' : 1})<CR>
```
For the very least effort
```
:autocmd BufReadPost *.txt source scriptfile.vim
:autocmd WinLeave * call MyBackupFunc(expand('%'))
```

### Statement
all statements terminated by newline:
```
echo "Starting"
call Phase(1)
echo "Done"
```
To continue across multiple lines, start next line with backslash:
```
let full_name = 
\ first_name . ' ' . middle_initial . ' ' . family_name
```
To put two statements on one line, separate with vertical bar:
`echo "Starting" | call Phase(1) | call Phase(2) | echo "Done"`
 Leading colon is also allowed in a Vim script, but not required:
```
 function! Tidy_and_save()
	set expandtabs
	%retab!
	write
```
### Values
Four build-in types: `String, Number, List, and Dictionary`

![image](/assets/images/1527350515440.png)

### commands
Start with double quote`"`
It run to end-of-line:
```
" This is a comment
```
So maybe add a more familiar marker
```
call Setup(0) " // comment
```
### Variables
All assignments require `let` keyword
Vars can store numbers, strings, lists, dictionaries, or just function refs:
```
let name = "Damian"
let height = 170
let haha = {"Vim", "asdjl", "key"}
let duration = {"vim":32, "pasd":32}
let task = function('teach')
```
The variable type initialized with determinte the type of variable for the remaining time. 
if you `let duration = 90` will cause a type error. 
`unlet duration`
![image](/assets/images/1527353138645.png)

each scope name also a symbol table
```
g:varname
g:['varname']
```
```
for varname in keys(b:)
	unlet b:[varname]
endfor
```
![image](/assets/images/1527353463462.png)

Give direct access to vim environment

### Expressions
![image](/assets/images/1527353726859.png)

`.` is string concatenation
All comparators do both string and numeric comparison
string comparisons honour `ignorecase`
but can be explicitly marked case-sensitive(`#`)
case-insensitive(`?`)
```
if name ==? 'Batman'
	echo "I'm a Batman"
else if name ==# 'ee cummings'
	echo "asjdlj"
endif
```
Most important limitation is on arithmetic
Numeric expressions are **integer only**
Some floating point support in Vim 7.2
Common mistake:
```
for filenum in range(filecount)
	"Show progress...
	echo(file / filecount * 100).'% done'
	"Make progress...
	call progress_file(filenum)
endfor
```
This will always echo `Now 0% done`
have to use `filenum * 100 / filecount`
Another notice point:
```
for result in result_list
	let sum += result
endfor
```
### ControlFlow
```
if condition
	statements()
elseif other_condition
	other_statements
else
	still()
endif
```
```
while condition
	statements()
endwhile
```
```
for varname in list
	statements()
endfor
```
Parallel iteration
```
for [var1, var2, etc] in listoflists
	statements()
endfor
```
```
let g:words = {1:"one", 2:"two", 3:"many", 4:"lots"}
for [key, val] in items(g:words)
	echo key. ": ".val
endfor
```
```
for number in list_of_numbers
	if number < 0
		continue "go to next loop
	elseif number > 9
		break "Exit loop completely
	endif 
	echo number
endfor
```