🧠 MiniShell — A Simple Command Line Interpreter in C
📖 Overview

MiniShell is a lightweight command-line interpreter written in C, designed to simulate the basic behavior of a Linux terminal.
It supports both built-in commands (like cd, set, unset, echo, etc.) and external system commands (like ls, cat, mkdir, rm, etc.) through process creation using fork() and execvp().

⚙️ Features
🔹 Built-in Commands
Command	Description
exit	Exits the shell
cd [path]	Changes current directory
cwd	Displays current working directory
help	Shows available commands and their usage
clear	Clears the terminal screen
set var=value	Creates or updates a shell variable
unset var	Removes a shell variable
varlist	Displays all defined variables
echo text	Prints text or variable value (echo $var)
history	Displays the last 20 executed commands
🔹 External Commands

Executes any standard Linux command, including:

ls, cat, mkdir, rmdir, rm, cp, mv, touch, pwd, date, whoami, ps, and more.

🔹 Other Features

Maintains a command history of up to 20 entries.

Handles variables and their dynamic replacement with echo.

Supports error handling for missing or invalid commands.

Clean terminal screen using ANSI escape sequences.

🧩 How It Works

The shell continuously waits for user input using fgets().

Each command is parsed and compared with known built-in commands.

If a command matches a built-in, it is executed directly in the same process.

If it’s not a built-in, the shell creates a child process using fork() and executes the external command using execvp().

The parent process waits for the child to finish using wait(NULL).

Variables are stored in a global array of structures and can be dynamically set, unset, or printed.

💻 Installation
🔸 Requirements

Linux or macOS terminal

GCC compiler

🔸 Steps

Clone or download the repository:

git clone https://github.com/DreenDark/minishell.git
cd minishell


Compile the project:

gcc minishell.c -o minishell


Run it:

./minishell

🚀 Usage Example
=> set user=Amirreza
=> echo $user
Amirreza
=> mkdir testdir
=> cd testdir
=> touch test.txt
=> ls
test.txt
=> history
=> exit

👨‍💻 Author

Amirreza
Reverse Engineering Enthusiast
