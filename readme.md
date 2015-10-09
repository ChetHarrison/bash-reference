# Bash Scripts

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
	bash -x myscript

-n  Check Syntax (don't execute)
-v  Echo commands (don't execute)
-x  Echo commands (execute)

### Comments

	# start with the pound sign

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
