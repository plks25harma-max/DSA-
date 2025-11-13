#include <stdio.h>
#include <string.h>

#define MAX 1000

struct Book {
    char title[100];
    char author[100];
    char isbn[20];
    int year;
};

struct Book catalog[MAX];
int countBooks = 0;

// Convert a string to lowercase
void toLowerStr(char *str) {
    for (int i = 0; str[i]; i++) {
        if (str[i] >= 'A' && str[i] <= 'Z')
            str[i] = str[i] + ('a' - 'A');
    }
}

// Compare by title
int compareByTitle(struct Book a, struct Book b) {
    char t1[100], t2[100];
    strcpy(t1, a.title);
    strcpy(t2, b.title);
    toLowerStr(t1);
    toLowerStr(t2);
    return strcmp(t1, t2) < 0;
}

// Compare by author
int compareByAuthor(struct Book a, struct Book b) {
    char t1[100], t2[100];
    strcpy(t1, a.author);
    strcpy(t2, b.author);
    toLowerStr(t1);
    toLowerStr(t2);
    return strcmp(t1, t2) < 0;
}

// Merge Sort
void merge(struct Book arr[], int left, int mid, int right, int sortByTitle) {
    int n1 = mid - left + 1;
    int n2 = right - mid;

    struct Book L[n1];
    struct Book R[n2];

    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];

    int i = 0, j = 0, k = left;

    while (i < n1 && j < n2) {
        int cmp;
        if (sortByTitle)
            cmp = compareByTitle(L[i], R[j]);
        else
            cmp = compareByAuthor(L[i], R[j]);

        if (cmp)
            arr[k++] = L[i++];
        else
            arr[k++] = R[j++];
    }

    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

void mergeSort(struct Book arr[], int left, int right, int sortByTitle) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid, sortByTitle);
    mergeSort(arr, mid + 1, right, sortByTitle);
    merge(arr, left, mid, right, sortByTitle);
}

// Binary Search
int binarySearch(struct Book arr[], int n, char key[], int sortByTitle) {
    char keyLower[100];
    strcpy(keyLower, key);
    toLowerStr(keyLower);

    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = (left + right) / 2;
        char midKey[100];
        if (sortByTitle)
            strcpy(midKey, arr[mid].title);
        else
            strcpy(midKey, arr[mid].author);
        toLowerStr(midKey);

        int cmp = strcmp(midKey, keyLower);
        if (cmp == 0) return mid;
        if (cmp < 0) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

// Add a new book
void addBook() {
    if (countBooks >= MAX) {
        printf("Catalog is full!\n");
        return;
    }

    struct Book b;

    printf("Enter title: ");
    fgets(b.title, sizeof(b.title), stdin);
    b.title[strcspn(b.title, "\n")] = 0;

    printf("Enter author: ");
    fgets(b.author, sizeof(b.author), stdin);
    b.author[strcspn(b.author, "\n")] = 0;

    printf("Enter ISBN: ");
    fgets(b.isbn, sizeof(b.isbn), stdin);
    b.isbn[strcspn(b.isbn, "\n")] = 0;

    printf("Enter year: ");
    scanf("%d", &b.year);
    getchar(); // clear newline

    catalog[countBooks++] = b;
    printf("Book added successfully!\n");
}

// Display all books
void displayAll() {
    if (countBooks == 0) {
        printf("Catalog is empty.\n");
        return;
    }
    for (int i = 0; i < countBooks; i++) {
        printf("[%d] %s by %s (%d) | ISBN: %s\n",
               i + 1, catalog[i].title, catalog[i].author, catalog[i].year, catalog[i].isbn);
    }
}

// Main function
int main() {
    int choice;

    while (1) {
        printf("LIBRARY CATALOG MENU\n");
        printf("1. Add Book\n");
        printf("2. Display All Books\n");
        printf("3. Sort by Title\n");
        printf("4. Sort by Author\n");
        printf("5. Search by Title\n");
        printf("6. Search by Author\n");
        printf("7. Exit\n");
        printf("Enter your choice: ");
        
        if (scanf("%d", &choice) != 1) {
            printf("Invalid input! Exiting.\n");
            break;
        }
        getchar(); // consume newline

        if (choice == 1) addBook();
        else if (choice == 2) displayAll();
        else if (choice == 3) {
            if (countBooks > 0) {
                mergeSort(catalog, 0, countBooks - 1, 1);
                printf("Books sorted by title.\n");
            } else printf("No books to sort.\n");
        }
        else if (choice == 4) {
            if (countBooks > 0) {
                mergeSort(catalog, 0, countBooks - 1, 0);
                printf("Books sorted by author.\n");
            } else printf("No books to sort.\n");
        }
        else if (choice == 5) {
            if (countBooks == 0) { printf("No books in catalog.\n"); continue; }
            mergeSort(catalog, 0, countBooks - 1, 1);
            char query[100];
            printf("Enter exact title: ");
            fgets(query, sizeof(query), stdin);
            query[strcspn(query, "\n")] = 0;
            int idx = binarySearch(catalog, countBooks, query, 1);
            if (idx == -1) printf("Book not found.\n");
            else printf("Found: %s by %s (%d) | ISBN: %s\n",
                        catalog[idx].title, catalog[idx].author, catalog[idx].year, catalog[idx].isbn);
        }
        else if (choice == 6) {
            if (countBooks == 0) { printf("No books in catalog.\n"); continue; }
            mergeSort(catalog, 0, countBooks - 1, 0);
            char query[100];
            printf("Enter exact author: ");
            fgets(query, sizeof(query), stdin);
            query[strcspn(query, "\n")] = 0;
            int idx = binarySearch(catalog, countBooks, query, 0);
            if (idx == -1) printf("Author not found.\n");
            else printf("Found: %s by %s (%d) | ISBN: %s\n",
                        catalog[idx].title, catalog[idx].author, catalog[idx].year, catalog[idx].isbn);
        }
        else if (choice == 7) {
            printf("Exiting program. Goodbye!\n");
            break;
        }
        else printf("Invalid choice. Try again.\n");
    }
    return 0;
}
