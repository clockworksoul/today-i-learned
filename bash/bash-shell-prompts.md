# Bash Shell: Take Control of PS1, PS2, PS3, PS4 and PROMPT_COMMAND

_Source: https://www.thegeekstuff.com/2008/09/bash-shell-take-control-of-ps1-ps2-ps3-ps4-and-prompt_command/_

Your interaction with Linux Bash shell will become very pleasant if you use PS1, PS2, PS3, PS4, and PROMPT_COMMAND effectively. PS stands for prompt statement. This article will give you a jumpstart on the Linux command prompt environment variables using simple examples.

## PS1 – Default interaction prompt

The default interactive prompt on your Linux can be modified as shown below to something useful and informative. In the following example, the default PS1 was `\s-\v\$`, which displays the shell name and the version number. Let us change this default behavior to display the username, hostname and current working directory name as shown below.

```bash
-bash-3.2$  export PS1="\u@\h \w> "

ramesh@dev-db ~> cd /etc/mail
ramesh@dev-db /etc/mail>
```

Prompt changed to "username@hostname current-dir>" format.

The following PS1 codes are used in this example:

* `\u` – Username
* `\h` – Hostname
* `\w` – Full pathname of current directory. Please note that when you are in the home directory, this will display only `~` as shown above
* Note that there is a space at the end in the value of PS1. Personally, I prefer a space at the end of the prompt for better readability.

Make this setting permanent by adding export `PS1="\u@\h \w> "` to either `.bash_profile` (Mac) or `.bashrc` (Linux/WSL).

## PS2 – Continuation interactive prompt

A very long Unix command can be broken down to multiple line by giving `\` at the end of the line. The default interactive prompt for a multi-line command is "> ".  Let us change this default behavior to display `continue->` by using PS2 environment variable as shown below.

```
ramesh@dev-db ~> myisamchk --silent --force --fast --update-state \
> --key_buffer_size=512M --sort_buffer_size=512M \
> --read_buffer_size=4M --write_buffer_size=4M \
> /var/lib/mysql/bugs/*.MYI
```

This uses the default ">" for continuation prompt.

```
ramesh@dev-db ~> export PS2="continue-> "

ramesh@dev-db ~> myisamchk --silent --force --fast --update-state \
continue-> --key_buffer_size=512M --sort_buffer_size=512M \
continue-> --read_buffer_size=4M --write_buffer_size=4M \
continue-> /var/lib/mysql/bugs/*.MYI
```

This uses the modified "continue-> " for continuation prompt.

I found it very helpful and easy to read, when I break my long commands into multiple lines using \. I have also seen others who don’t like to break-up long commands. What is your preference? Do you like breaking up long commands into multiple lines?

## PS3 – Prompt used by "select" inside shell script

You can define a custom prompt for the select loop inside a shell script, using the PS3 environment variable, as explained below.

### Shell script and output WITHOUT PS3

```
ramesh@dev-db ~> cat ps3.sh

select i in mon tue wed exit
do
  case $i in
    mon) echo "Monday";;
    tue) echo "Tuesday";;
    wed) echo "Wednesday";;
    exit) exit;;
  esac
done

ramesh@dev-db ~> ./ps3.sh

1) mon
2) tue
3) wed
4) exit
#? 1
Monday
#? 4
```

This displays the default "#?" for select command prompt.

### Shell script and output WITH PS3

```
ramesh@dev-db ~> cat ps3.sh

PS3="Select a day (1-4): "
select i in mon tue wed exit
do
  case $i in
    mon) echo "Monday";;
    tue) echo "Tuesday";;
    wed) echo "Wednesday";;
    exit) exit;;
  esac
done

ramesh@dev-db ~> ./ps3.sh
1) mon
2) tue
3) wed
4) exit
Select a day (1-4): 1
Monday
Select a day (1-4): 4
```

This displays the modified "Select a day (1-4): " for select command prompt.

## PS4 – Used by "set -x" to prefix tracing output

The PS4 shell variable defines the prompt that gets displayed, when you execute a shell script in debug mode as shown below.

### Shell script and output WITHOUT PS4

```
ramesh@dev-db ~> cat ps4.sh

set -x
echo "PS4 demo script"
ls -l /etc/ | wc -l
du -sh ~

ramesh@dev-db ~> ./ps4.sh

++ echo 'PS4 demo script'
PS4 demo script
++ ls -l /etc/
++ wc -l
243
++ du -sh /home/ramesh
48K     /home/ramesh
```

This displays the default "++" while tracing the output using `set -x`.

### Shell script and output WITH PS4

The PS4 defined below in the ps4.sh has the following two codes:

* `$0` – indicates the name of script
* `$LINENO` – displays the current line number within the script

```
ramesh@dev-db ~> cat ps4.sh

export PS4='$0.$LINENO+ '
set -x
echo "PS4 demo script"
ls -l /etc/ | wc -l
du -sh ~

ramesh@dev-db ~> ./ps4.sh
../ps4.sh.3+ echo 'PS4 demo script'
PS4 demo script
../ps4.sh.4+ ls -l /etc/
../ps4.sh.4+ wc -l
243
../ps4.sh.5+ du -sh /home/ramesh
48K     /home/ramesh
```

This displays the modified "{script-name}.{line-number}+" while tracing the output using `set -x`.

## PROMPT_COMMAND

Bash shell executes the content of the `PROMPT_COMMAND` just before displaying the PS1 variable.

```
ramesh@dev-db ~> export PROMPT_COMMAND="date +%k:%m:%S"
22:08:42
ramesh@dev-db ~>
```

This displays the `PROMPT_COMMAND` and PS1 output on different lines.

If you want to display the value of `PROMPT_COMMAND` in the same line as the PS1, use the `echo -n` as shown below:

```
ramesh@dev-db ~> export PROMPT_COMMAND="echo -n [$(date +%k:%m:%S)]"
[22:08:51]ramesh@dev-db ~>
```

This displays the PROMPT_COMMAND and PS1 output on the same line.