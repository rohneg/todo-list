#include <stdio.h>
#include <string.h>
#include <stdlib.h>

struct Task 
{
    char *task;
    int completed;
};

struct Task *tasks = NULL;
int length = 0;


void saveTasksToFile() 
{
    FILE *file = fopen("tasks.txt", "w");
    if (file == NULL) {
        printf("Error .\n");
        return;
    }

    for (int i = 0; i < length; i++) 
    {
        fprintf(file, "%d|%s\n", tasks[i].completed, tasks[i].task);
    }

    fclose(file);
}


void loadTasksFromFile() 
{
    FILE *file = fopen("tasks.txt", "r");
    if (file == NULL) 
    return;

    char line[256];
    while (fgets(line, sizeof(line), file)) 
    {
        char *sep = strchr(line, '|');
        if (sep) {
            *sep = '\0';
            int completed = atoi(line);
            char *taskText = sep + 1;
            taskText[strcspn(taskText, "\n")] = '\0';

            tasks = (struct Task *)realloc(tasks, (length + 1) * sizeof(struct Task));
            tasks[length].task = (char *)malloc(strlen(taskText) + 1);
            strcpy(tasks[length].task, taskText);
            tasks[length].completed = completed;
            length++;
        }
    }

    fclose(file);
}

void addTask(const char *task) 
{
    tasks = (struct Task *)realloc(tasks, (length + 1) * sizeof(struct Task));
    tasks[length].task = (char *)malloc(strlen(task) + 1);
    strcpy(tasks[length].task, task);
    tasks[length].completed = 0;
    length++;
    printf("Task added.\n");
    saveTasksToFile();
}

void listTasks() {
    if (length == 0) {
        printf("No tasks available.\n");
        return;
    }

    for (int i = 0; i < length; i++) 
    {
        printf("%d. %s [%c]\n", i + 1, tasks[i].task, tasks[i].completed ? 'd' : 'n');
    }
}

void markCompleted(int index) 
{
    if (index > 0 && index <= length) 
    {
        tasks[index - 1].completed = 1;
        printf("Task marked as completed.\n");
        saveTasksToFile();
    } 
    else 
    printf("Invalid index.\n");
    
}

void editTask(int index, const char *task) 
{
    if (index > 0 && index <= length) 
    {
        index--;
        char *newTask = (char *)realloc(tasks[index].task, strlen(task) + 1);
        if (newTask != NULL) {
            tasks[index].task = newTask;
            strcpy(tasks[index].task, task);
            printf("Task updated.\n");
            saveTasksToFile();
        } else {
            printf("Memory error.\n");
        }
    } 
    else 
    printf("Invalid index.\n");
    
}

void deleteTask(int index) 
{
    if (index > 0 && index <= length) 
    {
        index--;
        free(tasks[index].task);
        for (int i = index; i < length - 1; i++) 
        {
            tasks[i] = tasks[i + 1];
        }
        length--;
        tasks = (struct Task *)realloc(tasks, length * sizeof(struct Task));
        printf("Task deleted.\n");
        saveTasksToFile();
    } 
    else 
        printf("Invalid index.\n");
    
}

int main() 
{
    int choice, running = 1, indexInput;
    char taskInput[100];

    loadTasksFromFile();

    printf("\nOptions\n");
    printf("1. Add a task\n");
    printf("2. Display all tasks\n");
    printf("3. Mark as complete\n");
    printf("4. Edit task\n");
    printf("5. Delete task\n");
    printf("6. Exit\n");

    while (running) 
    {
        printf("\nEnter choice (1-6): ");
        scanf("%d", &choice);
        getchar(); 

        switch (choice) {
            case 1:
                printf("Enter task: ");
                fgets(taskInput, sizeof(taskInput), stdin);
                taskInput[strcspn(taskInput, "\n")] = '\0';
                addTask(taskInput);
                break;

            case 2:
                listTasks();
                break;

            case 3:
                printf("Enter task number to mark as completed: ");
                scanf("%d", &indexInput);
                markCompleted(indexInput);
                break;

            case 4:
                printf("Enter task number to edit: ");
                scanf("%d", &indexInput);
                getchar();
                printf("Enter new task: ");
                fgets(taskInput, sizeof(taskInput), stdin);
                taskInput[strcspn(taskInput, "\n")] = '\0';
                editTask(indexInput, taskInput);
                break;

            case 5:
                printf("Enter task number to delete: ");
                scanf("%d", &indexInput);
                deleteTask(indexInput);
                break;

            case 6:
                running = 0;
                break;

            default:
                printf("Invalid choice.\n");
                break;
        }
    }

    for (int i = 0; i < length; i++) 
    {
        free(tasks[i].task);
    }
    

    return 0;
}
