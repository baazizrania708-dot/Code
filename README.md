
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <sys/resource.h>

#define RUNS 5
#define QUADRATIC_LIMIT 10000

void copyArray(int source[], int dest[], int n) {
    for (int i = 0; i < n; i++) dest[i] = source[i];
}

void printArray(int a[], int n) {
    int limit = n < 30 ? n : 30;
    for (int i = 0; i < limit; i++) printf("%d ", a[i]);
    if (n > limit) printf("...");
    printf("\n");
}

void generateArray(int a[], int n) {
    for (int i = 0; i < n; i++) a[i] = rand() % 100000;
}

void bubbleSort(int a[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int swapped = 0;
        for (int j = 0; j < n - i - 1; j++) {
            if (a[j] > a[j + 1]) {
                int temp = a[j];
                a[j] = a[j + 1];
                a[j + 1] = temp;
                swapped = 1;
            }
        }
        if (!swapped) break;
    }
}

void selectionSort(int a[], int n) {
    for (int i = 0; i < n - 1; i++) {
        int min = i;
        for (int j = i + 1; j < n; j++)
            if (a[j] < a[min]) min = j;

        int temp = a[i];
        a[i] = a[min];
        a[min] = temp;
    }
}

void insertionSort(int a[], int n) {
    for (int i = 1; i < n; i++) {
        int key = a[i];
        int j = i - 1;
        while (j >= 0 && a[j] > key) {
            a[j + 1] = a[j];
            j--;
        }
        a[j + 1] = key;
    }
}

void merge(int a[], int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    int *L = malloc(n1 * sizeof(int));
    int *R = malloc(n2 * sizeof(int));

    if (L == NULL || R == NULL) {
        printf("Memory allocation failed.\n");
        free(L);
        free(R);
        exit(1);
    }

    for (int i = 0; i < n1; i++) L[i] = a[left + i];
    for (int j = 0; j < n2; j++) R[j] = a[mid + 1 + j];

    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) a[k++] = L[i++];
        else a[k++] = R[j++];
    }
    while (i < n1) a[k++] = L[i++];
    while (j < n2) a[k++] = R[j++];

    free(L);
    free(R);
}

void mergeSort(int a[], int left, int right) {
    if (left < right) {
        int mid = left + (right - left) / 2;
        mergeSort(a, left, mid);
        mergeSort(a, mid + 1, right);
        merge(a, left, mid, right);
    }
}
int partitionArray(int a[], int low, int high) {
    int pivot = a[high];
    int i = low - 1;

    for (int j = low; j < high; j++) {
        if (a[j] <= pivot) {
            i++;
            int temp = a[i];
            a[i] = a[j];
            a[j] = temp;
        }
    }

    int temp = a[i + 1];
    a[i + 1] = a[high];
    a[high] = temp;
    return i + 1;
}

void quickSort(int a[], int low, int high) {
    if (low < high) {
        int p = partitionArray(a, low, high);
        quickSort(a, low, p - 1);
        quickSort(a, p + 1, high);
    }
}

double memoryUsageKB(void) {
    struct rusage usage;
    if (getrusage(RUSAGE_SELF, &usage) != 0) return -1.0;
#ifdef __APPLE__
    return (double)usage.ru_maxrss / 1024.0;
#else
    return (double)usage.ru_maxrss; /* Linux reports KB */
#endif
}

typedef void (*SortFunction)(int[], int);

void runAndMeasure(char *name, SortFunction sortFunction,
                   int original[], int n) {
    int *copy = malloc(n * sizeof(int));
    if (copy == NULL) {
        printf("Not enough memory.\n");
        return;
    }

    double totalTime = 0.0;
    double peakMemory = 0.0;

    for (int run = 0; run < RUNS; run++) {
        copyArray(original, copy, n);

        clock_t start = clock();
        sortFunction(copy, n);
        clock_t end = clock();

        totalTime += 1000.0 * (double)(end - start) / CLOCKS_PER_SEC;

        double currentMemory = memoryUsageKB();
        if (currentMemory > peakMemory) peakMemory = currentMemory;
    }

    printf("\n%s\n", name);
    printf("Average execution time: %.3f ms (%d runs)\n",
           totalTime / RUNS, RUNS);
    if (peakMemory >= 0)
        printf("Process peak memory (RSS): %.2f KB\n", peakMemory);
    else
        printf("Memory measurement is not available.\n");

    printf("Sorted correctly (first values): ");
    printArray(copy, n);

    free(copy);
}

void bubbleWrapper(int a[], int n) { bubbleSort(a, n); }
void selectionWrapper(int a[], int n) { selectionSort(a, n); }
void insertionWrapper(int a[], int n) { insertionSort(a, n); }
void mergeWrapper(int a[], int n) { mergeSort(a, 0, n - 1); }
void quickWrapper(int a[], int n) { quickSort(a, 0, n - 1); }

int main(void) {
    int choice, n;
    int *data = NULL;

    srand(42); /* fixed seed: reproducible random values */

    do {
        printf("\n.     SORTING ALGORITHMS   \n");
        printf("1. Bubble Sort\n");
        printf("2. Selection Sort\n");
        printf("3. Insertion Sort\n");
        printf("4. Merge Sort\n");
        printf("5. Quick Sort\n");
        printf("6. Compare all algorithms\n");
        printf("7. Big array example (100000 values)\n");
        printf("0. Exit\n");
        printf("Your choice: ");

        if (scanf("%d", &choice) != 1) {
            printf("Invalid input. Please enter a number.\n");
            int ch;
            while ((ch = getchar()) != '\n' && ch != EOF) {}
            choice = -1;
            continue;
        }

        if (choice == 0) {
            printf("Goodbye!\n");
            break;
        }

        if (choice < 1 || choice > 7) {
            printf("Invalid choice.\n");
            continue;
        }

        if (choice == 7) n = 100000;
        else {
            printf("Enter array size: ");
            if (scanf("%d", &n) != 1 || n <= 0 || n > 2000000) {
                printf("Invalid size. Use a value from 1 to 2000000.\n");
                int ch;
                while ((ch = getchar()) != '\n' && ch != EOF) {}
                continue;
            }
        }

        data = malloc(n * sizeof(int));
        if (data == NULL) {
            printf("Cannot allocate this array size.\n");
            continue;
        }

        generateArray(data, n);

        if (choice >= 1 && choice <= 5) {
            if (n > QUADRATIC_LIMIT && choice <= 3) {
                printf("This algorithm is slow for large arrays.\n");
                printf("Please use a size <= %d for this algorithm.\n",
                       QUADRATIC_LIMIT);
            } else if (choice == 1)
                runAndMeasure("Bubble Sort", bubbleWrapper, data, n);
            else if (choice == 2)
                runAndMeasure("Selection Sort", selectionWrapper, data, n);
            else if (choice == 3)
                runAndMeasure("Insertion Sort", insertionWrapper, data, n);
            else if (choice == 4)
                runAndMeasure("Merge Sort", mergeWrapper, data, n);
            else
                runAndMeasure("Quick Sort", quickWrapper, data, n);
        } else {
            printf("\nComparison on the same random input (%d values)\n", n);
            printf("Each algorithm sorts its own copy.\n");

            if (n <= QUADRATIC_LIMIT) {
                runAndMeasure("Bubble Sort", bubbleWrapper, data, n);
                runAndMeasure("Selection Sort", selectionWrapper, data, n);
                runAndMeasure("Insertion Sort", insertionWrapper, data, n);
            } else {
                printf("\nBubble, Selection and Insertion Sort skipped: ");
                printf("quadratic algorithms are too slow above %d values.\n",
                       QUADRATIC_LIMIT);
            }

            runAndMeasure("Merge Sort", mergeWrapper, data, n);
            runAndMeasure("Quick Sort", quickWrapper, data, n);
        }

        free(data);
        data = NULL;

    } while (choice != 0);

    return 0;
}
