#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define SIZE 5 // Maximum size of the circular queue waitlist 

// ---------------------------------------------------------
// DATA STRUCTURE DEFINITION
// Combines BST node structure and Circular Queue array 
// ---------------------------------------------------------
struct BookNode {
    int bookID;
    char title[100]; 
    int isAvailable; // 1 if available, 0 if checked out

    // Circular Queue parameters for the waitlist
    char waitlist[SIZE][50];
    int front;
    int rear;

    // Binary Search Tree pointers
    struct BookNode *Lchild; 
    struct BookNode *Rchild; 
};

// ---------------------------------------------------------
// CIRCULAR QUEUE FUNCTIONS (For Waitlist Management)
// ---------------------------------------------------------

// Enqueues a user into the waiting list
void cenqueue(struct BookNode *book, char *userName) {
    if ((book->rear + 1) % SIZE == book->front) { 
        printf("Queue is full. Cannot add more to waitlist!\n");
        return;
    }
    if (book->rear == -1 && book->front == -1) { 
        book->rear = 0; 
        book->front = 0;
        strcpy(book->waitlist[book->rear], userName);
    } else {
        book->rear = (book->rear + 1) % SIZE; 
        strcpy(book->waitlist[book->rear], userName);
    }
    printf("ADDED TO WAITLIST: %s (Queue position %d)\n\n", userName, book->rear + 1);
}

// Dequeues a user from the waiting list when a book is returned
int cdequeue(struct BookNode *book, char *nextUser) {
    if (book->rear == -1 && book->front == -1) { 
        return 0; // Queue is empty
    }
    
    // Copy the front user to the nextUser string
    strcpy(nextUser, book->waitlist[book->front]);
    
    if (book->rear == book->front) { 
        book->front = -1; 
        book->rear = -1;
    } else {
        book->front = (book->front + 1) % SIZE; 
    }
    return 1;
}

// ---------------------------------------------------------
// BINARY SEARCH TREE FUNCTIONS (For Library Management)
// ---------------------------------------------------------

// Inserts a new book into the BST based on bookID
struct BookNode* insert(struct BookNode *root, int id, char *title) {
    if (root == NULL) { 
        root = (struct BookNode*)malloc(sizeof(struct BookNode));
        root->bookID = id;
        strcpy(root->title, title); 
        root->isAvailable = 1;
        
        // Initialize queue pointers for the waitlist
        root->front = -1;
        root->rear = -1;
        
        root->Lchild = NULL; 
        root->Rchild = NULL; 
        return root;
    }
    if (id < root->bookID) { 
        root->Lchild = insert(root->Lchild, id, title);
    }
    else if (id > root->bookID) { 
        root->Rchild = insert(root->Rchild, id, title);
    }
    return root;
}

// Searches for a book by its bookID
struct BookNode* search(struct BookNode *root, int id) {
    if (root == NULL) { 
        return NULL; // Not found
    }
    if (id == root->bookID) { 
        return root; // Match found
    }
    else if (id < root->bookID) { 
        return search(root->Lchild, id);
    }
    else { 
        return search(root->Rchild, id);
    }
}

// Displays all currently available books in sorted order (Inorder Traversal)
void displayAvailable(struct BookNode *root) {
    if (root != NULL) {
        displayAvailable(root->Lchild);
        if (root->isAvailable == 1) {
            printf("[ ID: %d | TITLE: %s ]\n", root->bookID, root->title); 
        }
        displayAvailable(root->Rchild);
    }
}

// ---------------------------------------------------------
// APPLICATION LOGIC
// ---------------------------------------------------------

void checkOutBook(struct BookNode *root, int id, char *userName) {
    struct BookNode *book = search(root, id);
    if (book == NULL) {
        printf("BOOK NOT FOUND!!\n\n"); 
        return;
    }
    if (book->isAvailable == 1) {
        book->isAvailable = 0;
        printf("SUCCESS: Book '%s' checked out by %s.\n\n", book->title, userName);
    } else {
        printf("NOTICE: Book is currently checked out.\n");
        cenqueue(book, userName);
    }
}

void checkInBook(struct BookNode *root, int id) {
    struct BookNode *book = search(root, id);
    if (book == NULL) {
        printf("BOOK NOT FOUND!!\n\n"); 
        return;
    }
    if (book->isAvailable == 1) {
        printf("NOTICE: Book '%s' is already in the library.\n\n", book->title);
        return;
    }
    
    char nextUser[50];
    // Check if anyone is in the waiting list
    if (cdequeue(book, nextUser)) {
        printf("RETURNED: Book '%s' has been automatically checked out to the next person in waitlist: %s.\n\n", book->title, nextUser);
    } else {
        book->isAvailable = 1;
        printf("RETURNED: Book '%s' is now available.\n\n", book->title);
    }
}

int main() {
    int c;
    int id;
    char title[100], userName[50]; 
    struct BookNode *root = NULL; 
    
    // Adding some initial books
    root = insert(root, 105, "The_Great_Gatsby");
    root = insert(root, 102, "1984");
    root = insert(root, 108, "To_Kill_a_Mockingbird");

    do {
        printf("========== LIBRARY MENU ==========\n");
        printf("[1.VIEW AVAILABLE | 2.CHECK OUT | 3.CHECK IN | 4.ADD BOOK | 5.EXIT]\n"); 
        printf("ENTER CHOICE: ");
        scanf("%d", &c);
        switch(c) {
            case 1: {
                printf("\n--- AVAILABLE BOOKS ---\n");
                displayAvailable(root);
                printf("-----------------------\n\n");
                break;
            }
            case 2: {
                printf("ENTER BOOK ID TO CHECK OUT: ");
                scanf("%d", &id);
                printf("ENTER YOUR NAME: ");
                scanf("%s", userName);
                checkOutBook(root, id, userName);
                break;
            }
            case 3: {
                printf("ENTER BOOK ID TO RETURN: ");
                scanf("%d", &id);
                checkInBook(root, id);
                break;
            }
            case 4: {
                printf("ENTER BOOK ID: ");
                scanf("%d", &id);
                printf("ENTER BOOK TITLE: ");
                scanf("%s", title);
                root = insert(root, id, title); 
                printf("BOOK ADDED SUCCESSFULLY!\n\n");
                break;
            }
            case 5: {
                printf("YOU HAVE EXITED!!\n\n"); 
                break;
            }
            default: {
                printf("INVALID INPUT\n\n"); 
            }
        }
    } while(c != 5); 
    
    return 0;
}