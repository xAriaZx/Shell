Project: Develop Simple Shell 
Introduction
This document describes how to develop a shell on Linux/macOS that supports:
Builtin commands (e.g., pwd, cd, exit, set, unset, echo, help, history).
Shell variables for tasks like set, unset, echo, and variable expansion ($VAR).
External command execution via process creation, searching directories in PATH or using direct pathnames.
A history feature that stores and can display the last 500 commands the user entered.

1. Shell Workflow
In an interactive loop, the shell will:
Display a prompt (e.g., mysh> ).
Read a command line from the user.
Parse that line into command + arguments.
Check if the first token is a builtin:
If builtin, run it with internal logic.
Otherwise, attempt to run as an external command by creating a child process (fork) and executing the command in that child.
Before executing, store the user’s typed line into a history buffer for recall.
Repeat until user issues exit or until an end-of-file condition is reached.

2. Shell Variables
2.1. Internal Key–Value Pairs
Maintain a dictionary of variables for your shell. Each entry has a unique name (key) and a value.
When the user types set NAME=VALUE, you parse out NAME and VALUE and store them in your dictionary.
When the user types unset NAME, you remove that NAME from the dictionary if it exists.
When any command line token starts with $, you look up the substring after $ in this dictionary and replace it with the stored value (if any).
2.2. Builtin Variables (e.g., PWD)
If the user runs cd, you update PWD (current directory path) if the directory change succeeds.
If the user runs pwd, you retrieve PWD from your dictionary and print it.
This approach keeps track of the shell’s notion of the current working directory.

3. Builtin Commands
3.1. set
Allows creating or updating variables. If the user types set NAME=VALUE, you add or modify that variable in your dictionary. If the user types set with no arguments, you can list all defined variables.
3.2. unset
Removes a named variable from your dictionary, so future expansions of $NAME produce empty or an error.
3.3. echo
Prints text, performing variable expansion on tokens that start with $. For example, echo hello $USER would replace $USER with that dictionary entry’s value.
3.4. pwd
Retrieves the shell’s known path for PWD and prints it. If you keep PWD in the dictionary, this is just looking up PWD and showing it.
3.5. cd
Changes the shell’s idea of the current directory. You can check if the path is absolute or relative, then apply logic to update PWD on success.
3.6. help
Lists your shell’s available builtin commands and simple usage notes.
3.7. history
Displays previously entered commands. The shell internally stores up to 500 lines in a circular or ring buffer. By default, history can print all stored lines, or if history N is entered, print only the last N.
3.8. exit
Ends the main shell loop, typically returning control to the operating system.

4. History Feature
You store the last 500 commands typed by the user in a data structure that overwrites the oldest entry once it reaches 500. When the user types history, you can display all or part of these lines.
Each time the user enters a command, you add it to the buffer.
If you reach 500 entries, you overwrite the oldest.
history builtin can list them in chronological order (from oldest to newest) or any approach you choose.

5. External Commands
When the first token is not recognized as a builtin, you run it externally:
Check if the token has a slash (./).
If yes, assume it is a direct path and attempt to run it.
If no, search for it in the directories specified by the user’s path list or PATH variable.
Create a child process (fork).
In the child, call an execution function to run the command. If it fails, show an error message.
In the parent, wait for the child unless you allow background jobs.

6. Step-by-Step Shell Cycle
Prompt the user (mysh> ).
Read a line.
Store the line in the history data structure.
Parse it into tokens (command + arguments).
Perform variable expansion, replacing $VAR tokens with their dictionary values.
Check if the command is a builtin:
If builtin, run it with the logic described above.
Otherwise, treat it as external (search in path or direct path), then fork and execute.
Return to the prompt for the next command line.

7. Suggestions for Implementation
Memory Management: Each new command line stored in the history needs to be allocated, and if you overwrite an older command line, you should free it to prevent memory leaks.
Circular Buffer for History: Keep an index that you increment on each new command, wrapping around at 500. Also track how many commands have been stored if the user hasn’t typed 500 lines yet.
Error Handling: Print errors if cd fails or if external commands are not found.
Optional Features: Redirection (> <), piping (|), background jobs (&), persistent history saving, or advanced variable expansions can be added for a more advanced project.
