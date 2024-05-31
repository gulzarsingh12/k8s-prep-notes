# Shell

## Type of commands
There are 2 types of commands
* built-in commands
* external commands
To check type of command `type echo`.  it will return the output saying its shell built in.

## Basic Commands
Below are some of the basic useful commands

### mkdir

#### Multiple dirs
`mkdir dir1 dir2`

#### Array
````
arr=("dir1" "dir2")
mkdir $arr
````

#### Recursively
`mkdir -p dir1/dir2`

### cd
There are always 2 additional hidden pointers are created with every `mkdir` call.i.e. `.` and `..`
* `.` points to current dir. `cd .` will change back to current dir
* `..` points to parent dir. `cd ..` will change to parent dir
  
### pushd/popd
`pushd` will work do same as `cd` and also push the dir to stack. `popd' will retrieve it from stack and do `cd`

A good use case would be to push the start/home dir in the beginning to the stack and then do your work in the script by doing `cd` to multiple dirs etc. To come back to the original dir where it started, `popd` can be run again.

### mv
* To rename a file
* To move something from one dir to another dir. To just copy the without moving `cp` can be used.

### cat
* To print the content `cat test.txt`
* To overwrite the text into the file
  ````
   cat > test.txt
   Hello
   cntrl+D
  ````
* To append to the file
  ````
   cat >>test.txt
   Hello again
   cntrl+D
  ````

### ls
* to get the long list `ls -l`.
* to get in the order of file updated (latest on the top). `ls -lt`
* to get in the order of file updated with latest on bottom (revese the order) `ls -ltr`

### echo
to print text without new line `echo -n Hello`

### du
To check the size of a file.

`du -sk test.img` to display the file size in kB
`du -sh test.img` to display the file size in human readable format.e.g. KB,MB,GB etc

**Alternative to check the size**, use `ls -lh` -lh to list the files with human readable size.


## Pager Commands

See the link for comparison.  https://www.baeldung.com/linux/more-less-most-commands.
### more
* To vie w data as a pages `more system.log`
* To scroll one page `space`
* To scroll one page backward `b`
* To scroll one line `enter`
* To search `/`
* Not good for large files

### less
* similar to `more` but use updown arraw to scroll the text by line. `/` to search
* provides vi editor like feature
* better for large files than `more`.
* support bookmarks `m`. this is usefule when you want to come back to certain line after moving up or down during scrolling. use ` to go to bookmark.
* support horizontal scrolling

### most
* Need to install manually. not come by default.
* supports multiple files, horizontal and vertical scrolling.

## Command Help
* To get the help about command.i.e. `whatis date` will print help about date command.
* Can also use `man` command to print help about the command. It will give more details than `whatis`
* `date --help` to print help about command

### apropos
To search the keyword in OS to get help about it
`apropos modpr` it will print what modeprobe and modeprobe.d

## Shell Types
There are multiple types of shell are available. the basic most is bourne shell (sh). Another improved version of sh is borune again shell (bash). It is better in terms of providing the feature not available in sh.

### Bash 
it provides following features

#### tab autocompletion
You dont need to type full command or options/agruments. it will help in auto complete for you. For example `ls Docu` will automatially complete the rest of it and print `ls Documents` for you on pressing `tab`

#### alias
With bash, you can define alias. This is very useful for writing alias for commonly used commands. For example, during CKA,CKAD exam prepation and real exam too. alias was created by myself for many of the commands.i.e. `kga` was alias for `kubectl get all`. This is short and save time.

## Shell env variables
Commonly used env varibles are HOME, SHELL, LOGNAME, USER, PWD, TERM, PATH

### $HOME
To print home dir of user

### $SHELL
To print the current shell `echo $SHELL` will print `/bin/zsh`

### Set env vars
There are 2 ways to set env variables in shell and shell scripts.
* JAVA_HOME=/usr/bin/java
* export JAVA_HOME=/usr/bin/java

if variable set above without export, it's scope is current shell. With export, var is accesible inside current shell and any scripts launched from the shell.

To get all the env type `env`. it will print all the env vars.

To set the environment permanently, it should be set in `~/.profile` or `~/.pam_environment`.
As per google search, you can also set in `~/.bashrc` or `~/.bash_profile`. But `~/.bash_profile` will make this available in bash shell only. To make this available to every process `~/.profile` is best option.

`which` command is used to check the location of the command from where it it executed. It will resolve from `PATH`.
for e.g. `which obs-studio` will print nothing but sfter setting it into `PATH` as `export PATH=$PATH:/opt/obs/bin` will set it and it will start working. Trying to run `which obs-studio` now will print `/opt/obs/bin/obs-studio`

## Customize shell

### shell
As you can check what is current shell using env var `echo $SHELL` but if you want to set the shell you can do so using `chsh` command.
For e.g. `chsh -s /bin/bash bob`. Here bob is user and `-s /bin/bash` to change to the bash. You wont see it changed with echo $SHELL unless you reopen the terminal.

### prompt
You can check current prompt using `echo $PS1`. it will give you current prompt. For example if it prints `[\W]$` then it will open the prompt like `[~]$`  `\W` is pwd (present working directory). You can set this variable to change current prompt.

There are other vars like PS2, PS3, PS4 for other prompts in terminal too.

## Compression
### tar
To create an archive file. this is to put multiple files in a single file called tar file or tar ball.

* `tar -cf test.tar file1 file2`  -c to create tar file.  -f for to specify the file name
* `tar -tf test.tar` to list the contents of the tar file
* `tar -xf test.tar` to extract the tar file
* `tar -czf test.tar` to create the tar file and compress the content of the tar file
* `tar -xzf test.tar` to uncompress and extract the tar file

### compress
* `bzip2 test.img' will compress it create `test.img.bz2`
* `gzip test.img' will compress it create `test.img.gz`
* `xz test.img' will compress it create `test.img.xz`

bzip2 compress to the smaller size than others.

### uncompress
* `buzip2 test.img.bzip2' will uncompress it create `test.img`.
* `gunzip test.img.gz' will uncompress it create `test.img`
* `unxz test.img.xz' will uncompress it create `test.img`

### read without uncompress
* `zcat file.txt.gz` work like `cat file.txt`
* `bzcat file.txt.bz2` work like `cat file.txt`
* `xzcat file.txt.xz` work like `cat file.txt`

**Note** zcat didnt work for me on mac but gzcat worked. zcat/ghzcat work for all extensions.e.g. gz, bzip2, xz etc. but xzcat and bzcat only work for their respective extenions only.

## Search

### File search

#### find
`find` command is used to search the files.
`find /home/bob -name "*.txt"`

#### locate
`locate` command is used to find the files like find but it faster than `find` because find will looks into the filesystem to search the files whereas `locate` will only looks into its database. locate command has its database which is called mlocate.database or mlocate.db. it will have all the information to search its faster. However, it may not find the file if os is just installed and db is not updated. To manually update the db, run `updatedb` command.

* `locate "*.html" -n 20` to limit the search to 20 entries
* `locate -c [.txt]*` to only get the count
* `locate -i *SAMPLE.txt*` to ignore case of file name.
* `locate -i -0 *sample.txt*` to get the results as no delimiter. normally new line is the delimiter to display results

### Text Search

#### grep
`grep second sample.txt`

* `grep -i second sample.txt`  `-i` to search cas insensitive
* `grep -r "second" /home/bob` to search recursviely in a dir
* `grep -v "second" sample.txt` to search all the lines which doesn't contain the search text
* `grep -w "second" sample.txt` to search the whole word and `-vw` to search all line which doesnt contain the whole word second.

## IO Redirection

* `cat test.txt > sample.txt` to redirect the output to sample.txt to **replace** the contents
* `cat test.txt >> sample.txt` to redirect the output to sample.txt to **append** the contents
* `cat missing_file 2>error.txt` 2 means the standard error to error.txt
* `cat missing_file 2>/dev/null` 2 means the standard error to null device
* `echo "Hello | tee -a text.txt` to append to the test.txt. `tee` display also on the stdout.

## VI Editor
This is most commonly used editor. other option is nano. nowadays, vim is used and is more enhanced and feature rich than vi. In many of the os's, vi is pointing to vim. you can check that using `ls -l /usr/bin/vi`  it might be pointing to vim.

There are 3 modes in vi
* command mode
* insert mode
* last line mode

### modes
* when we open a file in `vi sample.txt` it will open in command mode.
* when we press `i` or o or a the it will enter in insert mode and we can write in it.
* when we press `esc` it will come back in command mode
* to go to last line mode, press `:`
* `:w` to save
* `:wq` save and quit
* `:q` quit without saving
* `:q!` to quite without saving and no confirmation (!)

### commands
* use **arrow keys** for up down and left right or  kj for up down and hl for left right
* `dd` to cut/delete a line. `d3d` to cut/delete 3 lines from the current line
* `yy` to copy a line
* `p` to paste a line
* `ZZ` to save the file
* `x` to delete the letter
* `u` to undo
* `ctrl+r` to redo
* `/` to search forward from current line. `n` to find next, `N` to find previous
* `?` to search backward from the current line. `n` to find next/up, `N` to find previous/down

#### ident
`vjj>` will select 3 lines from current line and then ident them once base on set width size.
`v` will change to the visual mode, `j` will select the line, press j the no of time you want to select the no of lines.
once selected then you can press `>` or `<` to ident right or left. 
`3>` to ident 3 times in right

hence command `vjjjj3>` will select the 5 lines and then ident to right 3 times.

### vimrc
To set the tab space, shift width or expand tab etc in ~/.vimrc

````
set expandtab
set tabstop=2
set shiftwidth=2
````
