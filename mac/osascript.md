# osascript

_Source: https://ss64.com/osx/osascript.html_

Execute AppleScripts and other OSA language scripts. Executes the given script file, or standard input if none is given. Scripts can be plain text or compiled scripts. osascript was designed for use with AppleScript, but will work with any Open Scripting Architecture (OSA) language.

Syntax

     osascript [-l language] [-e command] [-s flags] [programfile]

Options
   
     -e command
     Enter one line of a script.  If -e is given, osascript will not
     look for a filename in the argument list.  Multiple -e commands can 
     be given to build up a multi-line script.  Because most scripts use
     characters that are special to many shell programs (e.g., Apple-
     Script uses single and double quote marks, `(', `)', and `*'),
     the command will have to be correctly quoted and escaped to
     get it past the shell intact.

     -l language
     Override the language for any plain text files.  Normally, plain
     text files are compiled as AppleScript.

     -s flags
     Modify the output style.  The flags argument is a string consisting
     of any of the modifier characters e, h, o, and s.  Multiple modi-
     fiers can be concatenated in the same string, and multiple -s

     options can be specified.  The modifiers come in exclusive pairs;
     if conflicting modifiers are specified, the last one takes prece-
     dence.  The meanings of the modifier characters are as follows:

     h  Print values in human-readable form (default).
     s  Print values in recompilable source form.

        osascript normally prints its results in human-readable form:
        strings do not have quotes around them, characters are not
        escaped, braces for lists and records are omitted, etc.  This is
        generally more useful, but can introduce ambiguities.  For exam-
        ple, the lists `{"foo", "bar"}' and `{{"foo", {"bar"}}}' would
        both be displayed as `foo, bar'.  To see the results in an unam-
        biguous form that could be recompiled into the same value, use
        the s modifier.

     e  Print script errors to stderr (default).
     o  Print script errors to stdout.

        osascript normally prints script errors to stderr, so downstream
        clients only see valid results.  When running automated tests,
        however, using the o modifier lets you distinguish script
        errors, which you care about matching, from other diagnostic
        output, which you don't.

`osascript` in macOS 10.0 would translate `\r` characters in the output to `\n` and provided `c` and `r` modifiers for the `-s` option to change this. osascript now always leaves the output alone; pipe through tr if necessary.

For multi-line AppleScript you can use ¬ as a new line continuation character, press Option-L to enter this into your text editor.

Examples

Open or switch to Safari:

`$ osascript -e 'tell app "Safari" to activate'`

Close safari:

`$ osascript -e 'quit app "safari.app"'`

Empty the trash:

`$ osascript -e 'tell application "Finder" to empty trash'`

Set the output volume to 50%

`$ sudo osascript -e 'set volume output volume 50'`

Input volume and Alert volume can also be set from 0 to 100%:

`$ sudo osascript -e 'set volume input volume 40'`

`$ sudo osascript -e 'set volume alert volume 75'`

Mute the output volume (True/False):

`$ osascript -e 'set volume output muted TRUE'`

Shut down without asking for confirmation:

`$ osascript -e 'tell app "System Events" to shut down'`

Restart without asking for confirmation:

`$ osascript -e 'tell app "System Events" to restart'`
