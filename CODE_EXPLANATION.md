#include <stdlib.h>     // For malloc, free, exit, etc.
#include <string.h>     // For string operations (strcmp, strcpy, strtok, etc.)
#include <unistd.h>     // For chdir, execvp, fork, etc.
#include <stdio.h>      // For standard input/output functions
#include <sys/wait.h>   // For wait() function to wait for child processes
#include <sys/types.h>  // For pid_t type definition

#define max_history 20  // Maximum number of stored commands in history

//-------------------- VARIABLE STRUCTURE --------------------//

// Structure to store a variable (name=value pair)
struct var {
    char name[50];
    char value[100];
};

struct var vars[50];    // Array of 50 variables max
int varcount = 0;       // Counter to track how many variables are stored

//-------------------- EXTERNAL COMMANDS --------------------//

// Executes external Linux commands like ls, mkdir, rm, etc.
int excute_externals(char *x) {
    char *args[10];     // Arguments array (max 10 arguments)
    int i = 0;

    // Split the command string into arguments by spaces
    char *token = strtok(x, " ");
    while (token != NULL && i < 9) {
        args[i++] = token;
        token = strtok(NULL, " ");
    }
    args[i] = NULL; // Last element must be NULL for execvp()

    if (args[0] == NULL) return 1; // Empty command, return error

    pid_t pid = fork();  // Create a child process
    if (pid < 0) {       // Fork failed
        perror("fork failed");
        return 1;
    } 
    else if (pid == 0) { // Child process
        execvp(args[0], args); // Execute external command
        perror("execvp failed"); // If exec fails
        return 1;
    } 
    else {               // Parent process
        wait(NULL);      // Wait until the child finishes
    }

    return 0;
}

//-------------------- VARIABLE UNSET FUNCTION --------------------//

// Removes a variable from the variable list
int unsetvariable(char *p) {
    int idx = -1, i;

    // Error if no variable name is provided
    if (p == NULL || *p == '\0') {
        printf("Error: No variable name provided.\n");
        return 1;
    }

    // Search for the variable name in the list
    for (i = 0; i < varcount; i++) {
        if (strcmp(vars[i].name, p) == 0) {
            idx = i;
            break;
        }
    }

    // Variable not found
    if (idx == -1) {
        printf("Variable '%s' not found, use 'varlist' to check defined variables.\n", p);
        return 1;
    }

    // Shift variables to remove the deleted one
    for (i = idx; i < varcount - 1; i++) {
        vars[i] = vars[i + 1];
    }

    varcount--; // Decrease variable count
    printf("Variable '%s' removed.\n", p);
    return 0;
}

//-------------------- VARIABLE SET FUNCTION --------------------//

// Adds or updates a variable in the list
int setvariable(char *p) {
    int i = 0, j = 0, x, idx;

    // Error if no input
    if (p == NULL || *p == '\0') {
        printf("Error: No variable name provided.\n");
        return 1;
    }

    char n[50] = {0}, v[100] = {0}; // Temporary storage for name and value

    // Extract variable name until '='
    while (p[i] != '=' && p[i] != '\0' && p[i] != '\n') {
        n[i] = p[i];
        i++;
    }
    n[i] = '\0';

    if (p[i] == '=') i++; // Skip '=' sign

    // Extract variable value
    while (p[i] != '\0' && p[i] != '\n') {
        v[j++] = p[i++];
    }
    v[j] = '\0';

    // If variable already exists, update its value
    for (int k = 0; k < varcount; k++) {
        if (strcmp(vars[k].name, n) == 0) {
            strcpy(vars[k].value, v);
            return 0;
        }
    }

    // Add new variable if list not full
    if (varcount < 50) {
        strcpy(vars[varcount].name, n);
        strcpy(vars[varcount].value, v);
        varcount++;
    } 
    else { // If full, ask user to replace an existing variable
        printf("The variable list is full. Choose one to replace:\n");
        for (x = 0; x < 50; x++) {
            printf("%d: %s = %s\n", x, vars[x].name, vars[x].value);
        }
        printf("Enter your index = ");
        scanf("%d", &idx);
        while (getchar() != '\n'); // Clear input buffer
        if (idx >= 0 && idx < 50) {
            strcpy(vars[idx].name, n);
            strcpy(vars[idx].value, v);
        }
    }
    return 0;
}

//-------------------- MAIN FUNCTION --------------------//

int main() {
    printf("Commandline Interpreter\n");

    char *history[max_history]; // Command history storage
    char x[50];                 // Input buffer
    int n = 0;                  // Command counter

    while (1) {
        printf("=> ");          // Shell prompt
        fflush(stdout);         // Ensure prompt is printed

        if (fgets(x, sizeof(x), stdin) == NULL) break; // Read input

        // Store command in history
        if (n < max_history) {
            history[n++] = strdup(x);
        } else {
            // Shift history when full
            free(history[0]);
            for (int i = 1; i <= max_history; i++) {
                history[i - 1] = history[i];
            }
            history[max_history - 1] = strdup(x);
        }

        x[strcspn(x, "\n")] = '\0'; // Remove newline character

        //-------------------- BUILT-IN COMMANDS --------------------//

        if (strcmp(x, "exit") == 0) break; // Exit shell

        else if (strncmp(x, "cd", 2) == 0) { // Change directory
            char *path = x + 2;
            while (*path == ' ') path++;
            if (chdir(path) != 0)
                printf("Error: Failed to change directory.\n");
        } 

        else if (strcmp(x, "cwd") == 0) { // Print current directory
            char cwd[1024];
            if (getcwd(cwd, sizeof(cwd)) != NULL)
                printf("You are in: %s\n", cwd);
        }

        else if (strcmp(x, "help") == 0) { // Help menu
            printf("| Command | Description |\n");
            printf("| -------- | ------------ |\n");
            printf("| exit     | Exit the shell |\n");
            printf("| cd       | Change directory |\n");
            printf("| cwd      | Show current directory |\n");
            printf("| set/unset/varlist | Manage variables |\n");
            printf("| echo     | Print text or variable |\n");
            printf("| history  | Show last 20 commands |\n");
            printf("| clear    | Clear screen |\n");
            printf("| External commands (ls, rm, mv, etc.) are also supported. |\n");
        }

        else if (strcmp(x, "clear") == 0) {
            printf("\033[H\033[J"); // ANSI escape sequence to clear screen
        }

        else if (strcmp(x, "history") == 0) { // Print command history
            if (n != 0) {
                for (int i = 0; i < n; i++) {
                    printf("%d: %s", i, history[i]);
                }
            } else printf("No input yet.\n");
        }

        else if (strncmp(x, "echo", 4) == 0) { // Echo or variable print
            char *sentence = x + 4;
            while (*sentence == ' ') sentence++;
            if (sentence[0] == '$') {
                sentence++;
                if (varcount == 0) printf("No variable set\n");
                else {
                    int found = 0;
                    for (int i = 0; i < varcount; i++) {
                        if (strcmp(vars[i].name, sentence) == 0) {
                            printf("%s\n", vars[i].value);
                            found = 1;
                            break;
                        }
                    }
                    if (!found)
                        printf("No variable set with this name: %s\n", sentence);
                }
            } else {
                printf("%s\n", sentence);
            }
        }

        else if (strncmp(x, "set", 3) == 0) { // Set variable
            char *p = x + 3;
            while (*p == ' ') p++;
            if (setvariable(p) == 0)
                printf("Successful\n");
            else
                printf("Unsuccessful\n");
        }

        else if (strncmp(x, "unset", 5) == 0) { // Unset variable
            char *p = x + 5;
            while (*p == ' ') p++;
            if (unsetvariable(p) == 0)
                printf("Successful\n");
            else
                printf("Unsuccessful\n");
        }

        else if (strcmp(x, "varlist") == 0) { // Print variable list
            if (varcount == 0)
                printf("No variable defined yet.\n");
            else {
                for (int i = 0; i < varcount; i++) {
                    printf("%d: %s=%s\n", i + 1, vars[i].name, vars[i].value);
                }
            }
        }

        //-------------------- EXTERNAL COMMANDS --------------------//

        else { // Execute external command (ls, mkdir, rm, etc.)
            if (excute_externals(x) != 0) {
                printf("Unknown command: %s\n", x);
                printf("Use 'help' to see available commands.\n");
            }
        }
    }
}
