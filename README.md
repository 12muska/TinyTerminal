# TinyTerminal
c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <unistd.h>
These are the standard C libraries included for input/output, memory allocation, string manipulation, process creation, and inter-process communication.

c


#define MAX_COMMAND_LENGTH 100
#define MAX_ARGUMENTS 10
These are constants defining the maximum length of a command and the maximum number of arguments.

c


void execute_command(char *command, char **arguments, int background) {
    pid_t pid = fork();

    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    } else if (pid == 0) {
        // Child process
        if (background) {
            // If background execution, ignore signals from the terminal
            signal(SIGINT, SIG_IGN);
        }

        // Execute the command
        if (execvp(command, arguments) == -1) {
            perror("execvp");
            exit(EXIT_FAILURE);
        }
    } else {
        // Parent process
        if (!background) {
            // If not background, wait for the child to complete
            int status;
            waitpid(pid, &status, 0);
        } else {
            // If background, print the child's process ID
            printf("Background process started with PID %d\n", pid);
        }
    }
}
This function is responsible for executing a command. It forks a new process, and the child process uses execvp to execute the command with its arguments. The parent process either waits for the child to complete or prints a message for background processes.

c
Copy code
int main() {
    char input[MAX_COMMAND_LENGTH];
    char *arguments[MAX_ARGUMENTS];

    while (1) {
        // Print prompt
        printf("SimpleShell> ");
        fflush(stdout);

        // Read user input
        if (fgets(input, sizeof(input), stdin) == NULL) {
            break;  // Exit on EOF
        }

        // Remove newline character from input
        size_t len = strlen(input);
        if (len > 0 && input[len - 1] == '\n') {
            input[len - 1] = '\0';
        }

        // Tokenize the input into command and arguments
        char *token = strtok(input, " ");
        int i = 0;

        while (token != NULL) {
            arguments[i++] = token;
            token = strtok(NULL, " ");

            // Ensure we don't exceed the maximum number of arguments
            if (i >= MAX_ARGUMENTS) {
                fprintf(stderr, "Too many arguments\n");
                break;
            }
        }

        // Null-terminate the arguments array
        arguments[i] = NULL;

        // Check for built-in commands or exit
        if (i > 0) {
            if (strcmp(arguments[0], "exit") == 0) {
                break;  // Exit the shell
            } else if (strcmp(arguments[i - 1], "&") == 0) {
                // Background execution if the last argument is "&"
                arguments[i - 1] = NULL;
                execute_command(arguments[0], arguments, 1);
            } else {
                // Normal execution
                execute_command(arguments[0], arguments, 0);
            }
        }
    }

    return 0;
}
In the main function, the shell enters a loop where it continuously prompts the user for input. It reads the input, tokenizes it into a command and arguments, and then checks for built-in commands (like "exit") or background execution. It uses the execute_command function to execute the entered command.

This simple shell allows users to run commands, including background commands, and exit the shell by typing "exit." It's a basic starting point, and you can extend its functionality based on your requirements and learning goals.





