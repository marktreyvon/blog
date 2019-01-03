---

layout: post

title: "some useful UNIX and VIM command"

author: "markt"

---

- exctags -R . 			create the new tags   
  - add root path lib:      :set tags+=/usr/include/tags  (if do not have the tags in this directory, use command to create it yourself)
- Alt + F123456		to another terminal
- lsof (list of open file )           like the task manager
- pkg install commandname         install software    ,like apt install

## vim

- :set tags += (path)			set tags
- press ctrl+z to the terminal   and fg to back
- tselect   list  all the tag you finds
- yy 10    copy the following 10lines
  - p   paste
- ESC :			quit
- i     			insert
- :set ls=2		 show your "pwd"
- Ctrl+]   			jump to tag  
- :ls 				list buffers
- /keyword				search the keyword  , 'n' for  next
- gd			go to definition
- - search the keyword, 'n' for next
- Ctrl + o			go back from the definition
- Ctrl + i
- u				undo
- b4			jump to buffer 4
- e . 			come in present directory
- hjkl			←↓↑→

## 11.30

makefile			you should create the MakeFile yourself

cc yourfile.c -o newname        compile your c file

netstat

whatis command  		explain the command simply

man command		explain the command with usage

tcpdump	software which is used for catching tcp packet

| tee filename    save the output with name

tty  			show which terminal window you are 

| less or | more  use when the output is more than a window

## git

.gitignore

git tags



![](https://img-blog.csdn.net/20160907133419436?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
