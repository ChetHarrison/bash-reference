# Bash Scripts

A little reference based of "Great Bash" by Carl Albing available
at O'Reilly.

### Stardard Out

	ls -l here > lsout

### Standard In

	wc < lsout

	wc <<
	foo bar
	EOF

### Stardard Error

	ls -l not.here 2> lserr

### Send Both Out and Error

	ls -l here not.here &> lsboth

### Pipe - Send Stdout to Stdin

	ls here | wc

### Pipe Both Out and Error

	ls here not.here 2>&1 | wc

### Grep Script

	grep -i $* <<EOF
	chet 933-1307
	mark 944-2209
	linda 955-3456
	EOF

[an example](/Users/chetharrison/projects/scripts/ph "ph")

### Fork a Process for Async

	bash some-long-process &

This will start another process and run `bash some-long-process &`
returning controll to the parent process.

### Run Multipule Commands (in order)

	pwd ; ls

### Blocking on a  Fork

	wait 12345

Provide the process id or you can wait on all processes.

	wait

### Turn Scripts into Executables

	chmod 755 myscript
	chmod a+x myscript

Those 2 commands are equivalent.

### Debug

	bash -n myscript
	bash -v myscript
	bash -x myscript # this will print out the commands as they happen

-n  Check Syntax (don't execute)
-v  Echo commands (don't execute)
-x  Echo commands (execute)

### Comments

	# start with the pound sign

### Chaining Commands Safely

The `&&` operator will proceed to the second command if the first
one succeeds. The `||` operator will proceed to the second command
if the first one fails.

	cd /temp && rm *

	cd /temp || echo cd failed

You may want to handel your own errors

	cd /temp 2>/dev/null || { echo cd failed ; exit 1 ; }

this will toss the  stderr echo our message and exit. Note the
2 commands after the `||` are grouped with the curly braces and
delimited with a ` ; `. That white space is required syntax. Note
if we use `()` each command would run in it's own process.

### If Statements

	if cd /tpm 2>/dev/null
	then
		echo the cd worked
	else
		echo cd failed
		exit 1
	fi

	echo continuing on
	exit 0

Note the exit codes are flipped here. In Bash `1` indicates *failure*
and `0` indicates *success*.

### Variables

By *convention* variable names have historicly been written in upper
case. We use the `$` operator to reference the value. For example

	myVar = "foo"
	echo myVar   		# -> myVar
	echo $myVar  		# -> foo
	echo ${myVar}able 	# -> fooable

You can assign commands to variables.

	CMD = ls
	$CMD 		# executes ls

**Watch your whitespace.** Whitespace
is a very important part of parsing bash commands. For example:

	myVar = 42	# -> -bash: myVar: command not found
	myVar=42	# will assign 42 to myVar

Beware that commands often run in a subprocess so we need the
`export` or the `declare -x` keywords to pass variables.

	export myVar=42
	declare -x myVar=42

If we had a script that had one line `echo $myVar` and we we then
did an assignment of the command line to `myVar` then ran our script
the myVar is not scoped to the subprocess runing the echo and nothing
would print. If we preceed the command with and assignment to `myVar`
it will contain that value for the duration of the following command.

	cat showit			# -> echo $myVar
	myVar=42			# Note we did not export or declare this var
	./showit			# ->
	myVar=42 ./showit	# ->

### Return Values

The `?` variable will containt the return value of the preceeding
script. You will need to cache that value if you need it before
running another script. For example:

	cd not-here
	echo $? 		# -> 1

	cd am-here
	echo $?			# -> 0

### Setting the Promt String

Bash has a command to set the prompt string `PS1`

	echo $PS1 		# -> $
	PS1="foo" 		# now my promt string is foo

Run a `man bash` and search for "prompting" for all the special
characters you can use in the promt.

### The Path

	echo $PATH 		# list the paths bash will search for commands on
	type ls 		# -> ls is hashed (/bin/ls) Find a commands location
	PATH="$PATH:~/additional-path"  	# adds a new path

Don't use `.` in your path. It will bite you.

### String Formatting

The `printf` command works a C `printf`.

	myVar=42
	printf "my value is %d\n" $myVar 	# -> my value is 42

`%%` will escape the special meaning of `%`.

* `%x` 		hex
* `%6.2f`	floating point 6 char width and 2 is precision
* `%06d`	Zero padded 6 char width decimal
* `%s` 		String

### Script Arguments

`$1` etc will reference argments within a script in the order they
appeared in the call. `$0` will be the command name.

	# Contence of simple-copy
	SRC="$1"
	DEST="$2"
	cp "$SCR" "$DEST"

	# call script
	./simple-copy path/to/src path/to/dest

`$*` and `$@` will return all the arguments. `$@` is a bit harder to
explain so here is and example. We will run the following
`allstar` script on a directory contining file names with
embedded whitespace.

	# allstar
	echo
	echo invike script with '$*'
	./secondscript $*
	echo
	echo invike script with '"$*"'
	./secondscript "$*"
	echo
	echo invike script with '"$@"'
	./secondscript "$@"
	echo

	# secondscript
	echo "invoked with $# arg(s)"
	echo '1st arg ($1) is "'$1'"'
	echo '1st arg ($2) is "'$2'"'

The `allstar` script calls the second script with 3 different
varations of our arguments as `$*`, then `"$*"` then `"$@"`.

	# let's look at the directory
	echo my*  # -> my car.jpg mycopy mydata myfile mymusic.mp3
	./allstar my*

we will get the following output:

	invoke script with $*
	invoked with 6 arg(s)
	1st arg ($1) is "my"
	2nd arg ($2) is "car.jpg"

	invoke script with "$*"
	invoked with 1 arg(s)
	1st arg ($1) is "my car.jpg mycopy mydata myfile mymusic.mp3"
	2nd arg ($2) is ""

	invoke script with "$@""
	invoked with 5 arg(s)
	1st arg ($1) is "my car.jpg"
	2nd arg ($2) is "mycopy"

In this context `"$@"` is the argument list you are
looking for.

### Shell Parameter Substitution and Pattern Matching

If we start with a variable asignment `PIC=abba` we can use
pattern matching to do some substitutions. For example
`${PIC%a}` will remove the trailing `a`. Here are some examples

	echo ${PIC}  	# -> abba
	echo ${PIC%a}  	# -> abb
	# ? matches any char. * matches 0 or more
	echo ${PIC%?}  	# -> abb
	echo ${PIC%b*}  # -> ab
	# %% largest possible match
	echo ${PIC%%b*} # -> a

Note this is not RegExp. Here is a jpg to png converter

	PIC=${1}
	convert "$PIC" "${PIC%.jpg}.png"  # convert and swap extentions

To remove a prefix with `#`.

	echo${PIC#?}  	# -> bba
	echo${PIC#*}  	# -> abba
	echo${PIC#*b}  	# -> ba
	echo${PIC##*b} 	# -> a

Let's parse a `host:path` argument

	FULL=${1}
	FN=${FULL#*:}
	HOST=${FULL%:*}

Use `/` to remove from the middle of an argument

	echo ${PIC}  # -> abba
	echo ${PIC/bb/foo}  # -> afooa
	# clean up a file name
	FN='01 Track 01.mp3.mp3'
	mv "$FN" "${FN/ Track ??.mp3"  # Note how the ? matches 1 character

### Loops

	for VAR
	do
		echo $VAR
	done

args are `$1`, `$2`, `$3`, ...

A good application would be renaming a bunch of files.
`${VAR/whatever}` returns the value of `VAR` with the
whatever removed. `?` will match any character.

	for VAR
	do
		echo $VAR
		mv "$VAR" "${VAR/ Track ??.mp3}"
	done

### For In

	for FN in *.mp3
	do
		echo $FN
		cp -p "$FN" /video/mp3
	done

This script would list all of the `.mp3` files in the
current working directory and copy them to the `/media/mp3`
relative directory. The quotes insure that whitespace in
the file name does not break the script.

The resurved word `in` separates the loop variable from
the list of values. A newline or semicolon ends the list.

### Numeric Loops

	for (( i=1 ; i<9 ; 1++ ))
	do
		FN="0${i}.mp3""
		echo $FN
		cp "$FN" /media/mp3
	done

### String Replacement

	$  echo $SHELL
	/usr/local/bin/bash

	$  echo ${SHELL:2:6}
	sr/loc

### Reverse a String with a Loop

	VAR=%1      # get the first arg
	ANS-=""     # the ANS var is empty string
	for ((i=${#var}-1 ; 1 >= 0 ; i--))
	do
		echo ${VAR:$1:3}
	done
	echo $ANS

### Convert all jpg to png with Image Magic

[Image Magic](http://www.imagemagick.org/script/index.php)

	for PIC in *.jpg # for all the jpg file in the current dir
	do
		echo convert "$PIC" "${PIC%.jpg}.png"  # log what your doing
		convert $"PIC" "${PIC%.jpg}.png"  # strip the .jpg and strap on .png
	done

### Read will assign input to arguments

	read PERM LN USR GRP SIZ MON DAY YRTM FILENM
	echo "File: '$FILENM'" is $SIZE bytes long.

### While Loops

	let i=0
	while (( i < $1 ))
	do
		echo $1
		let i++
	done

The `(())` indicates an arithmatic expression. The while loop
can operate on a pipeline of commands as a predicate.

	while read PERM LN USR GRP SIZ MON DAY YRTM FILENM
	do
		echo "File: '$FILENM'" is $SIZ bytes long.
	done

### Ctr-D is EOF or "End of File"

### Case Statements

	read -p "Anwer yes or no: " ANS

	case "$ANS" in
		yes) echo "OK"
			;;
		no) echo "negative"
			;;
		maybe) echo "make up your mind"
			;;
		*) "you did not follow the instructions."  # using pattern matcher
			exit 1
			;;
	esac

Using patern matching is convenitent for parsing args

	FN="$1"
	case "$FN" in
		(*.gif) echo "a gif file"  # The leading paren is optional
				;;
		*.[Pp][Nn][Gg]) echo "a png file"
				;;
		*.jpg | *.JPEG) echo "a jpg file"
				;;
		*.tif | *.TIFF) echo "a TIFF file"
				;;&  # New in bash v4. & means continue on.
		*.mp3 | *.wav) echo "a music file"
				;&   # New in bash v4. & means fall through.
		* ) printf "File '%s' not supported.\n" "$FN"
			exit 1;
				;;
	esac

### Math

Bash is a string based language. `i=5` actually asigns the string
value "5" to i. To get an integer we use the `declare` or `typeset`
keywordS.

	declare -i COUNT=0
	while read PERM LN USR GRP SIZ MON DAY YRTM FILENM
	do
		echo "File: '$FILENM'" IS $SIZ bytes long
		COUNT+=1
	done
	echo $COUNT

The keyword `let` or `(())` or `$(())` will evaluate mathmaticaly.

	declare -i COUNT=0
	while read PERM LN USR GRP SIZ MON DAY YRTM FILENM
	do
		COUNT+=1
		echo "File: '$FILENM'" IS $SIZ bytes long
		let TOTAL+= SIZ  # same as next 2 statements
		(( SUM+=SIZ ))
		ALL=$((ALL+SIZ))
	done
	echo $COUNT

We can use integers now in conditionals

	if (( VAR < 2 ))  # evaluates as expected

### Calculatro Example

This a simple reverse Polish notation calculator. Note we can pass
in the operator. However if we use `*` we need to single quote it
to escape the pattern matching default behavior like this `'*'`
or this `\*`.

	# rpn-calc
	# usage: rpn 2 2 '*'

	# check for valid arg count
	if (( $# != 3 ))
	then
		echo usage: "$0 num num op"
		exit 1
	fi

	ANS=$(( $1 $3 $2 ))

	echo $ANS

### Shift to Remove Args

If we wanted calculator that could take a dynamic number of args
we can use the `shift` keyword.

	# rpn-calc

	ANS=$(( $1 $3 $2 ))
	shift 3

	while (( $# > 0 ))
	do
		ANS=$(( ANS $2 $1 ))
		shift 2
	done

	echo $ANS

### Arrays

set with `VAR[INDEX]` and get with `${VAR[INDEX]}`. The curly braces
forces bash to interprate the [] as a dereference rather than a
matching pattern syntax. Note the `declare` keyword to indicate we
would like an array.

	declare -a DIGNAM
	DIGNAM=(zero one two three four five six seven eight nine)

	for anarg
	do
		for((i=0; i<${#anarg} ; i++))
		do
			C=${anarg:i:1}  # grab one char substring
			case "$C" in
			[0-9]) SAY="${DIGNAM[$C]}"
					;;
			*) SAY="$C"
			esac
			printf "%s " "$SAY"
		done
		echo
	done

### Read Arrays

This script will take the output of an `ls -l` command
and construct an array with a count of the files created
on each day of the month then print out the day/count
pairs.

	while read -a LSOUT  # The -a flag expects an array input
	do
		DOM=${LSOUT[6]}  # Get the 7th element of the array
		let CNT[DOM]++
	done

	for ((i=0; i<32; i++))
	{
		printf "%2d	%3d\n" $i ${CNT[i]}  # String formating like C
	}

This is executed `ls -l | tail +2 | ./fc`. The `tail +2` command
returns the the input with starting from the second line.

`${CNT[@]}` will be replaced with all the values of the array as
separate words. `${!CNT[@]}` will get us all of the indexes. We can
modify our script to filter 0 count values.

	while read -a LSOUT  # The -a flag expects an array input
	do
		DOM=${LSOUT[6]}  # Get the 7th element of the array
		let CNT[DOM]++
	done

	for NDX in "${!CNT[@]}"
	{
		printf "%2d	%3d\n" ${NDX} ${CNT[NDX]}
	}

### Associative Arrays (Dictionarys)

This is available in bash v4. We delare the array with a `-A` flag as in
`declare -A ASA`. The we can use string keys/indexes.

	declare -A CNT
	while read -a LSOUT
	do
		if [[ $LSOUT == 'total' ]]; then continue; fi  # filter the count line of ls
		MON=${LSOUT[5]}
		let CNT[$MON]++
	done

	for NDX in "${!CNT[@]}"
	{
		printf "%2s %d\n" ${NDX} ${CNT[$NDX]}
	}
