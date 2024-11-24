-----------------------------------
//matrix multiplication random ip generation 
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define MAX 4

// Matrix dimensions (change as needed)
int A[MAX][MAX], B[MAX][MAX], C[MAX][MAX];

// Number of threads (equal to the number of rows in matrix A)
pthread_t threads[MAX];

// Struct to pass multiple arguments to thread function
typedef struct {
    int row;
} thread_data;

// Thread function to perform matrix multiplication for a single row
void* multiply(void* arg) {
    thread_data* data = (thread_data*)arg;
    int row = data->row;

    for (int i = 0; i < MAX; i++) {
        C[row][i] = 0;
        for (int j = 0; j < MAX; j++) {
            C[row][i] += A[row][j] * B[j][i];
        }
    }

    pthread_exit(NULL);
}

int main() {
    srand(time(0));
    // Initialize matrices A and B
    for (int i = 0; i < MAX; i++) {
        for (int j = 0; j < MAX; j++) {
            A[i][j] = rand() % 10;
            B[i][j] = rand() % 10;
        }
    }

    // Print matrix A
    printf("Matrix A:\n");
    for (int i = 0; i < MAX; i++) {
        for (int j = 0; j < MAX; j++) {
            printf("%d ", A[i][j]);
        }
        printf("\n");
    }

    // Print matrix B
    printf("Matrix B:\n");
    for (int i = 0; i < MAX; i++) {
        for (int j = 0; j < MAX; j++) {
            printf("%d ", B[i][j]);
        }
        printf("\n");
    }

    // Create threads to perform multiplication for each row
    for (int i = 0; i < MAX; i++) {
        thread_data* data = (thread_data*)malloc(sizeof(thread_data));
        data->row = i;
        pthread_create(&threads[i], NULL, multiply, (void*)data);
    }

    // Wait for all threads to finish
    for (int i = 0; i < MAX; i++) {
        pthread_join(threads[i], NULL);
    }

    // Print result matrix C
    printf("Resultant Matrix C (A * B):\n");
    for (int i = 0; i < MAX; i++) {
        for (int j = 0; j < MAX; j++) {
            printf("%d ", C[i][j]);
        }
        printf("\n");
    }

    return 0;
}
-------------------------------
// matrix multi for user input format 
#include <stdio.h>
#include <pthread.h>

#define MAX 3  // Define the maximum size of the matrix

int matA[MAX][MAX]; // First matrix
int matB[MAX][MAX]; // Second matrix
int result[MAX][MAX]; // Result matrix

// Structure for passing data to threads
typedef struct {
    int row;
    int col;
} thread_data_t;

// Thread function to perform matrix multiplication
void* multiply(void* arg) {
    thread_data_t* data = (thread_data_t*)arg;
    int sum = 0;

    // Perform the multiplication for a single element
    for (int i = 0; i < MAX; i++) {
        sum += matA[data->row][i] * matB[i][data->col];
    }
    result[data->row][data->col] = sum;

    pthread_exit(0);
}

int main() {
    pthread_t threads[MAX * MAX];
    thread_data_t thread_data[MAX * MAX];
    int thread_count = 0;

    // Initialize matrices
    printf("Enter elements of matrix A (3x3):\n");
    for (int i = 0; i < MAX; i++) {
        for (int j = 0; j < MAX; j++) {
            scanf("%d", &matA[i][j]);
        }
    }

    printf("Enter elements of matrix B (3x3):\n");
    for (int i = 0; i < MAX; i++) {
        for (int j = 0; j < MAX; j++) {
            scanf("%d", &matB[i][j]);
        }
    }

    // Create threads for each element of the result matrix
    for (int i = 0; i < MAX; i++) {
        for (int j = 0; j < MAX; j++) {
            thread_data[thread_count].row = i;
            thread_data[thread_count].col = j;
            pthread_create(&threads[thread_count], NULL, multiply, &thread_data[thread_count]);
            thread_count++;
        }
    }

    // Wait for all threads to finish
    for (int i = 0; i < thread_count; i++) {
        pthread_join(threads[i], NULL);
    }

    // Print the result matrix
    printf("Resultant matrix after multiplication:\n");
    for (int i = 0; i < MAX; i++) {
        for (int j = 0; j < MAX; j++) {
            printf("%d ", result[i][j]);
        }
        printf("\n");
    }

    return 0;
}
---------------------------------------------
//producer - consumer 
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <semaphore.h>
#include <time.h>  

#define BUFFER_SIZE 5  

int buffer[BUFFER_SIZE];
int in = 0;  //inserting indices
int out = 0;  //removing indices
//semaphor declaration
sem_t empty;  
sem_t full; 
//producer function
void* producer(void* arg) {
    int item;

    for (int i = 0; i < 10; i++) {
        item = rand() % 100;  
        sem_wait(&empty);     
        buffer[in] = item;    
        printf("Producer produced: %d\n", item);
        in = (in + 1) % BUFFER_SIZE;  
        sem_post(&full);  
    }

    pthread_exit(NULL);
}
//consumer function
void* consumer(void* arg) {
    int item;

    for (int i = 0; i < 10; i++) {
        sem_wait(&full);  
        item = buffer[out];  
        printf("Consumer consumed: %d\n", item);
        out = (out + 1) % BUFFER_SIZE;  
        sem_post(&empty);  
    }

    pthread_exit(NULL);
}

int main() {
    //seeding with time
    //srand(time(NULL));

    sem_init(&empty, 0, BUFFER_SIZE);  
    sem_init(&full, 0, 0);  

    pthread_t producer_thread, consumer_thread;
    pthread_create(&producer_thread, NULL, producer, NULL);
    pthread_create(&consumer_thread, NULL, consumer, NULL);

    pthread_join(producer_thread, NULL);
    pthread_join(consumer_thread, NULL);

    sem_destroy(&empty);
    sem_destroy(&full);

    return 0;
}
-----------------------------------
// reader writer problem 
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

struct monitor {
    int rcount; // Number of readers
    int wcount; // Number of writers
    int waiting_r; // Number of readers waiting
    int waiting_w; // Number of writers waiting
    pthread_cond_t can_read;
    pthread_cond_t can_write;
    pthread_mutex_t condlock;
};

struct monitor M;

void init_monitor() {
    M.rcount = 0;
    M.wcount = 0;
    M.waiting_r = 0;
    M.waiting_w = 0;
    pthread_cond_init(&M.can_read, NULL);
    pthread_cond_init(&M.can_write, NULL);
    pthread_mutex_init(&M.condlock, NULL);
}

void begin_read(int i) {
    pthread_mutex_lock(&M.condlock);

    if (M.wcount == 1 || M.waiting_w > 0) {
        M.waiting_r++;
        printf("Reader %d can't read as writer %d is writing\n", i, i - 1);
        pthread_cond_wait(&M.can_read, &M.condlock);
        M.waiting_r--;
    }

    M.rcount++;
    printf("Reader %d is reading\n", i);
    pthread_mutex_unlock(&M.condlock);
    pthread_cond_broadcast(&M.can_read);
}

void end_read(int i) {
    pthread_mutex_lock(&M.condlock);
    if (--M.rcount == 0)
        pthread_cond_signal(&M.can_write);
    pthread_mutex_unlock(&M.condlock);
}

void begin_write(int i) {
    pthread_mutex_lock(&M.condlock);
    if (M.wcount == 1 || M.rcount > 0) {
        printf("Writer %d can't write as reader %d is reading\n", i, i - 1);
        M.waiting_w++;
        pthread_cond_wait(&M.can_write, &M.condlock);
        M.waiting_w--;
    }

    M.wcount = 1;
    printf("Writer %d is writing\n", i);
    pthread_mutex_unlock(&M.condlock);
}

void end_write(int i) {
    pthread_mutex_lock(&M.condlock);
    M.wcount = 0;
    if (M.waiting_r > 0)
        pthread_cond_signal(&M.can_read);
    else
        pthread_cond_signal(&M.can_write);
    pthread_mutex_unlock(&M.condlock);
}

void* reader(void* id) {
    int c = 0;
    int i = (int)id;
    while (c < 5) {
        usleep(1);
        begin_read(i);
        end_read(i);
        c++;
    }
}

void* writer(void* id) {
    int c = 0;
    int i = (int)id;
    while (c < 5) {
        usleep(1);
        begin_write(i);
        end_write(i);
        c++;
    }
}

int main() {
    pthread_t r[5], w[5];
    int id[5];
    init_monitor();

    for (int i = 0; i < 5; i++) {
        id[i] = i;
        pthread_create(&r[i], NULL, &reader, &id[i]);
        pthread_create(&w[i], NULL, &writer, &id[i]);
    }

    for (int i = 0; i < 5; i++) {
        pthread_join(r[i], NULL);
    }

    for (int i = 0; i < 5; i++) {
        pthread_join(w[i], NULL);
    }

    return 0;
}
-------------------------------------
// reader writer with dynamic mem allocation 
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

struct monitor {
    int rcount; // Number of readers
    int wcount; // Number of writers
    int waiting_r; // Number of readers waiting
    int waiting_w; // Number of writers waiting
    pthread_cond_t can_read;
    pthread_cond_t can_write;
    pthread_mutex_t condlock;
};

struct monitor M;

void init_monitor() {
    M.rcount = 0;
    M.wcount = 0;
    M.waiting_r = 0;
    M.waiting_w = 0;
    pthread_cond_init(&M.can_read, NULL);
    pthread_cond_init(&M.can_write, NULL);
    pthread_mutex_init(&M.condlock, NULL);
}

void begin_read(int i) {
    pthread_mutex_lock(&M.condlock);

    if (M.wcount == 1 || M.waiting_w > 0) {
        M.waiting_r++;
        printf("Reader %d can't read as writer %d is writing\n", i, i - 1);
        pthread_cond_wait(&M.can_read, &M.condlock);
        M.waiting_r--;
    }

    M.rcount++;
    printf("Reader %d is reading\n", i);
    pthread_mutex_unlock(&M.condlock);
    pthread_cond_broadcast(&M.can_read);
}

void end_read(int i) {
    pthread_mutex_lock(&M.condlock);
    if (--M.rcount == 0)
        pthread_cond_signal(&M.can_write);
    pthread_mutex_unlock(&M.condlock);
}

void begin_write(int i) {
    pthread_mutex_lock(&M.condlock);
    if (M.wcount == 1 || M.rcount > 0) {
        printf("Writer %d can't write as reader %d is reading\n", i, i - 1);
        M.waiting_w++;
        pthread_cond_wait(&M.can_write, &M.condlock);
        M.waiting_w--;
    }

    M.wcount = 1;
    printf("Writer %d is writing\n", i);
    pthread_mutex_unlock(&M.condlock);
}

void end_write(int i) {
    pthread_mutex_lock(&M.condlock);
    M.wcount = 0;
    if (M.waiting_r > 0)
        pthread_cond_signal(&M.can_read);
    else
        pthread_cond_signal(&M.can_write);
    pthread_mutex_unlock(&M.condlock);
}

void* reader(void* id) {
    int c = 0;
    //int i = (int)id;
    int i = (int)id;
    while (c < 5) {
        usleep(1);
        begin_read(i);
        end_read(i);
        c++;
    }
}

void* writer(void* id) {
    int c = 0;
    //int i = (int)id;
    int i = (int)id;
    while (c < 5) {
        usleep(1);
        begin_write(i);
        end_write(i);
        c++;
    }
}

int main() {
    pthread_t r[5], w[5];
    int* id[5]; // Use pointers for thread-specific IDs
    init_monitor();

    for (int i = 0; i < 5; i++) {
        id[i] = malloc(sizeof(int)); // Dynamically allocate memory for each ID
        *id[i] = i; // Assign the thread's ID
        pthread_create(&r[i], NULL, &reader, id[i]);
        pthread_create(&w[i], NULL, &writer, id[i]);
    }

    for (int i = 0; i < 5; i++) {
        pthread_join(r[i], NULL);
        pthread_join(w[i], NULL);
        free(id[i]); // Free the allocated memory for IDs
    }

    return 0;
}
-------------------------------------------
//bankers algorithm user input
import java.util.Scanner;

public class BankersAlgorithm {
    private int[][] need, allocate, max;
    private int[] available; // available is a 1D array, not a 2D array
    private int numProcesses, numResources;

    public BankersAlgorithm(int numProcesses, int numResources) {
        this.numProcesses = numProcesses;
        this.numResources = numResources;
        need = new int[numProcesses][numResources];
        allocate = new int[numProcesses][numResources];
        max = new int[numProcesses][numResources];
        available = new int[numResources]; // 1D array
    }

    public void inputData() {
        Scanner sc = new Scanner(System.in);

        System.out.println("Enter Allocation Matrix:");
        for (int i = 0; i < numProcesses; i++) {
            for (int j = 0; j < numResources; j++) {
                allocate[i][j] = sc.nextInt();
            }
        }

        System.out.println("Enter Max Matrix:");
        for (int i = 0; i < numProcesses; i++) {
            for (int j = 0; j < numResources; j++) {
                max[i][j] = sc.nextInt();
            }
        }

        System.out.println("Enter Available Resources:");
        for (int i = 0; i < numResources; i++) {
            available[i] = sc.nextInt(); // Assigning to a 1D array correctly
        }

        // Calculate Need Matrix
        for (int i = 0; i < numProcesses; i++) {
            for (int j = 0; j < numResources; j++) {
                need[i][j] = max[i][j] - allocate[i][j];
            }
        }

        sc.close();
    }

    public boolean checkSafeState() {
        boolean[] finished = new boolean[numProcesses];
        int[] work = available.clone(); // Work array is a 1D array, so this is correct
        int[] safeSequence = new int[numProcesses];
        int count = 0;

        while (count < numProcesses) {
            boolean foundProcess = false;

            for (int i = 0; i < numProcesses; i++) {
                if (!finished[i]) {
                    boolean canExecute = true;

                    for (int j = 0; j < numResources; j++) {
                        if (need[i][j] > work[j]) {
                            canExecute = false;
                            break;
                        }
                    }

                    if (canExecute) {
                        for (int j = 0; j < numResources; j++) {
                            work[j] += allocate[i][j];
                        }
                        safeSequence[count++] = i;
                        finished[i] = true;
                        foundProcess = true;
                    }
                }
            }

            if (!foundProcess) {
                System.out.println("The system is not in a safe state.");
                return false;
            }
        }

        System.out.println("The system is in a safe state.");
        System.out.println("Safe Sequence: ");
        for (int i = 0; i < numProcesses; i++) {
            System.out.print("P" + safeSequence[i] + " ");
        }
        System.out.println();
        return true;
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        System.out.print("Enter number of processes: ");
        int numProcesses = sc.nextInt();

        System.out.print("Enter number of resources: ");
        int numResources = sc.nextInt();

        BankersAlgorithm bankersAlgorithm = new BankersAlgorithm(numProcesses, numResources);

        bankersAlgorithm.inputData();

        bankersAlgorithm.checkSafeState();

        sc.close();
    }
}

Enter number of processes: 3
Enter number of resources: 2
Enter Allocation Matrix:
1 0
0 1
2 1
Enter Max Matrix:
3 2
2 2
4 3
Enter Available Resources:
3 2
----------------------------------------
// fcfs
// first come first search 
//to arrival time, burst time, completion time, turnaround time, waiting time

import java.util.Scanner;
public class Assignment_01{
    public static void main(String[] args) {
        try (Scanner s = new Scanner(System.in)) {
            System.out.print("Number of processes: ");
            int n = s.nextInt();
            
            int[] p_id = new int[n]; 
            int[] at = new int[n];  
            int[] bt = new int[n];  
            int[] ct = new int[n];  
            int[] wt = new int[n];  
            int[] tat = new int[n]; 
            
            System.out.println("Arrival Times:");
            for (int i = 0; i < n; i++) {
                System.out.print("P" + (i + 1) + " arrival time: ");
                at[i] = s.nextInt();
                p_id[i] = i + 1;
            }
            
            System.out.println("Burst Time:");
            for (int i = 0; i < n; i++) {
                System.out.print("P" + (i + 1) + " burst time: ");
                bt[i] = s.nextInt();
            }
            
            
            int currentTime = 0;
            for (int i = 0; i < n; i++) {
                if (currentTime < at[i]) {
                    currentTime = at[i];
                }
                ct[i] = currentTime + bt[i];
                currentTime = ct[i];
            }
            
            float total_WT = 0, total_TAT = 0;
      
            for (int i = 0; i < n; i++) {
                tat[i] = ct[i] - at[i];
                wt[i] = tat[i] - bt[i];
                total_WT += wt[i];
                total_TAT += tat[i];
            }
            
            float avg_WT = total_WT / n;
            float avg_TAT = total_TAT / n;
            
      
            System.out.println("\nP_num\tArrival_Time\tBurst_Time\tCompletion_Time\tTurnaround_Time\tWaiting_Time");
            for (int i = 0; i < n; i++) {
                System.out.println("P" + p_id[i] + "\t\t" + at[i] + "\t\t" + bt[i] + "\t\t" + ct[i] + "\t\t" + tat[i] + "\t\t" + wt[i]);
            }
            System.out.println("\nAvg Waiting_T = " + avg_WT);
            System.out.println("Avg Turnaround_T = " + avg_TAT);
        }
    }
}
---------------------------
//sjf (non p)
//sjf
//non-primitive
import java.util.Scanner;
public class Assignment_02_A{
    public static void main(String[] args) {
        try (Scanner sc = new Scanner(System.in)) {
            System.out.println("Enter number of processes:");
            int n = sc.nextInt();
            int[] pid = new int[n];
            int[] at = new int[n];
            int[] bt = new int[n];
            int[] ct = new int[n];
            int[] ta = new int[n];
            int[] wt = new int[n];
            int[] f = new int[n];

            int currentTime = 0, totalCompProcess = 0;
            float avgwt = 0, avgta = 0;
            
            for (int i = 0; i < n; i++) {
                System.out.println("Enter process " + (i + 1) + " arrival time:");
                at[i] = sc.nextInt();
                System.out.println("Enter process " + (i + 1) + " burst time:");
                bt[i] = sc.nextInt();
                pid[i] = i + 1;
                f[i] = 0;
            }   
            while (true) {
                // n is number of processes
                int c = n, min = 999;
                if (totalCompProcess == n)
                    break;
                
                for (int i = 0; i < n; i++) {
                    if ((at[i] <=currentTime) && (f[i] == 0) && (bt[i] < min)) {
                        min = bt[i];
                        c = i;
                    }
                }
                // if no process is ready
                if (c == n)
                    currentTime++;
                else {
                    ct[c] = currentTime + bt[c];
                    currentTime += bt[c];
                    ta[c] = ct[c] - at[c];
                    wt[c] = ta[c] - bt[c];
                    f[c] = 1;
                    totalCompProcess++;
                }
            }   System.out.println("\npid  arrival  burst  complete  turn  waiting");
            for (int i = 0; i < n; i++) {
                avgwt += wt[i];
                avgta += ta[i];
                System.out.println(pid[i] + "\t" + at[i] + "\t" + bt[i] + "\t" + ct[i] + "\t" + ta[i] + "\t" + wt[i]);
            }   System.out.println("\nAverage Turnaround Time is " + (avgta / n));
            System.out.println("Average Waiting Time is " + (avgwt / n));
        }
    }
}
-------------------------------
// sjf (p)
//sjf - preemptive
import java.util.Scanner;

public class Assignment_02_B {
    public static void main(String[] args) {
        try (Scanner sc = new Scanner(System.in)) {
            System.out.println("Enter number of processes:");
            int n = sc.nextInt();
            int[] pid = new int[n];
            int[] at = new int[n];
            int[] bt = new int[n];
            int[] rt = new int[n]; 
            int[] ct = new int[n];
            int[] ta = new int[n];
            int[] wt = new int[n];
            int[] f = new int[n];
            int currentTime = 0, totalCompProcess = 0;
            float avgwt = 0, avgta = 0;

            for (int i = 0; i < n; i++) {
                System.out.println("Enter process " + (i + 1) + " arrival time:");
                at[i] = sc.nextInt();
                System.out.println("Enter process " + (i + 1) + " burst time:");
                bt[i] = sc.nextInt();
                rt[i] = bt[i]; 
                pid[i] = i + 1;
                f[i] = 0; 
            }

            while (totalCompProcess < n) {
                int c = n, min = 999;
                for (int i = 0; i < n; i++) {
                    if ((at[i] <= currentTime) && (f[i] == 0) && (rt[i] < min)) {
                        min = rt[i];
                        c = i;
                    }
                }
                // if  no process is ready
                if (c == n) {
                    currentTime++;
                } else {
                    rt[c]--; 
                    currentTime++;
                    
                    if (rt[c] == 0) {
                        ct[c] = currentTime;
                        ta[c] = ct[c] - at[c];
                        wt[c] = ta[c] - bt[c];
                        f[c] = 1; 
                        totalCompProcess++;
                    }
                }
            }

            System.out.println("\npid  arrival  burst  complete  turn  waiting");
            for (int i = 0; i < n; i++) {
                avgwt += wt[i];
                avgta += ta[i];
                System.out.println(pid[i] + "\t" + at[i] + "\t" + bt[i] + "\t" + ct[i] + "\t" + ta[i] + "\t" + wt[i]);
            }

            System.out.println("\nAverage Turnaround Time is " + (avgta / n));
            System.out.println("Average Waiting Time is " + (avgwt / n));
        }
    }
}
-------------------------------
// priority (non p)
import java.util.*;

public class Assignment_04_A {

    static void findWaitingTime(int n, int[] at, int[] bt, int[] wt) {
        int[] service_time = new int[n];
        service_time[0] = at[0];
        wt[0] = 0;

        for (int i = 1; i < n; i++) {
            service_time[i] = service_time[i - 1] + bt[i - 1];
            wt[i] = service_time[i] - at[i];
            if (wt[i] < 0) {
                wt[i] = 0;
            }
        }
    }

    static void findTurnAroundTime(int n, int[] bt, int[] wt, int[] tat, int[] ct, int[] at) {
        for (int i = 0; i < n; i++) {
            tat[i] = bt[i] + wt[i];      // Turnaround Time = Burst Time + Waiting Time
            ct[i] = tat[i] + at[i];       // Completion Time = Turnaround Time + Arrival Time
        }
    }

    static void findavgTime(int n, int[] at, int[] bt, int[] priority) {
        int[] wt = new int[n], tat = new int[n], ct = new int[n];
        int total_wt = 0, total_tat = 0;

        // Sort based on arrival time and priority
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (at[i] > at[j] || (at[i] == at[j] && priority[i] > priority[j])) {
                    // Swap arrival time
                    int temp = at[i];
                    at[i] = at[j];
                    at[j] = temp;

                    // Swap burst time
                    temp = bt[i];
                    bt[i] = bt[j];
                    bt[j] = temp;

                    // Swap priority
                    temp = priority[i];
                    priority[i] = priority[j];
                    priority[j] = temp;
                }
            }
        }

        findWaitingTime(n, at, bt, wt);
        findTurnAroundTime(n, bt, wt, tat, ct, at);

        // Updated column headers, placing Completion Time after Burst Time
        System.out.println("Processes " + "   Arrival time " + "   Burst time " + "   Completion time " + "   Priority " + "     Waiting time " + "   Turnaround time");

        for (int i = 0; i < n; i++) {
            total_wt += wt[i];
            total_tat += tat[i];
            // Updated print statement with Completion Time after Burst Time
            System.out.println(" P" + (i + 1) + "\t\t" + at[i] + "\t\t" + bt[i] + "\t\t " + ct[i] + "\t\t " + priority[i] + "\t\t " + wt[i] + "\t\t " + tat[i]);
        }

        System.out.println("Average waiting time = " + (float) total_wt / n);
        System.out.println("Average turn around time = " + (float) total_tat / n);
    }

    public static void main(String[] args) {
        try (Scanner sc = new Scanner(System.in)) {
            System.out.print("Enter number of processes: ");
            int n = sc.nextInt();
            
            int[] at = new int[n];
            int[] bt = new int[n];
            int[] priority = new int[n];
            
            for (int i = 0; i < n; i++) {
                System.out.println("Enter arrival time for P" + (i + 1) + ":");
                at[i] = sc.nextInt();
                System.out.println("Enter burst time for P" + (i + 1) + ":");
                bt[i] = sc.nextInt();
                System.out.println("Enter priority for P" + (i + 1) + ":");
                priority[i] = sc.nextInt();
            }
            
            findavgTime(n, at, bt, priority);
        }
    }
}
----------------------------------------
// priority (p)
import java.util.*;

class Process {
    String pid; // Process ID
    int bt;     // Burst Time
    int at;     // Arrival Time
    int priority; // Priority
    int rt;     // Remaining Time
    int ct;     // Completion Time

    public Process(String pid, int bt, int at, int priority) {
        this.pid = pid;
        this.bt = bt;
        this.at = at;
        this.priority = priority;
        this.rt = bt;
        this.ct = 0; // Initialize completion time as 0
    }
}

public class Assignment_04_B {

    static void findWaitingTime(Process[] proc, int n, int[] wt) {
        int t = 0; // Current time
        int complete = 0; // Number of completed processes
        int minPriority = Integer.MAX_VALUE;
        int shortest = -1;
        boolean check = false;

        while (complete != n) {
            for (int j = 0; j < n; j++) {
                if (proc[j].at <= t && proc[j].rt > 0 && proc[j].priority < minPriority) {
                    minPriority = proc[j].priority;
                    shortest = j;
                    check = true;
                }
            }

            if (!check) {
                t++;
                continue;
            }

            proc[shortest].rt--; // Decrement remaining time

            if (proc[shortest].rt == 0) {
                complete++;
                check = false;
                int finish_time = t + 1;
                proc[shortest].ct = finish_time; // Set the completion time for the process
                wt[shortest] = finish_time - proc[shortest].bt - proc[shortest].at;
                if (wt[shortest] < 0) {
                    wt[shortest] = 0;
                }
                minPriority = Integer.MAX_VALUE; // Reset minPriority for next process
            }
            t++;
        }
    }

    static void findTurnAroundTime(Process[] proc, int n, int[] wt, int[] tat) {
        for (int i = 0; i < n; i++) {
            tat[i] = proc[i].bt + wt[i];
        }
    }

    static void findavgTime(Process[] proc, int n) {
        int[] wt = new int[n], tat = new int[n];
        int total_wt = 0, total_tat = 0;

        findWaitingTime(proc, n, wt);
        findTurnAroundTime(proc, n, wt, tat);

        // Printing the table with the added Completion Time
        System.out.println("Processes " + " Arrival time " + " Burst time " + " Completion time " + " Priority " + " Waiting time " + " Turnaround time");

        for (int i = 0; i < n; i++) {
            total_wt += wt[i];
            total_tat += tat[i];
            System.out.println(" " + proc[i].pid + "\t\t" + proc[i].at + "\t\t" + proc[i].bt + "\t\t" + proc[i].ct + "\t\t" + proc[i].priority + "\t\t" + wt[i] + "\t\t" + tat[i]);
        }

        System.out.println("Average waiting time = " + (float) total_wt / n);
        System.out.println("Average turnaround time = " + (float) total_tat / n);
    }

    public static void main(String[] args) {
        try (Scanner sc = new Scanner(System.in)) {
            System.out.print("Enter number of processes: ");
            int numProcesses = sc.nextInt();
            
            Process[] proc = new Process[numProcesses];
            
            for (int i = 0; i < numProcesses; i++) {
                String pid = "P" + (i + 1);
                System.out.println("Enter arrival time for " + pid + ":");
                int at = sc.nextInt();
                System.out.println("Enter burst time for " + pid + ":");
                int bt = sc.nextInt();
                System.out.println("Enter priority for " + pid + ":");
                int priority = sc.nextInt();
                
                proc[i] = new Process(pid, bt, at, priority);
            }
            
            findavgTime(proc, numProcesses);
        }
    }
}
------------------------------------
//round robin 
import java.util.Scanner;

public class Assignment_03 {
    public static void main(String[] args) {
        try (Scanner scanner = new Scanner(System.in)) {
            int n = getNumberOfProcesses(scanner);
            int[] bt = new int[n];
            int[] at = new int[n];
            int[] rt = new int[n];
            int[] wt = new int[n];
            int[] ta = new int[n];
            int[] ct = new int[n];

            inputBurstTimes(scanner, n, bt, rt);
            inputArrivalTimes(scanner, n, at);

            int quantum = getTimeQuantum(scanner);
            int time = 0;
            boolean done;

            do {
                done = true;
                for (int i = 0; i < n; i++) {
                    if (rt[i] > 0 && at[i] <= time) {
                        done = false;
                        if (rt[i] > quantum) {
                            time += quantum;
                            rt[i] -= quantum;
                        } else {
                            time += rt[i];
                            wt[i] = time - bt[i] - at[i];
                            ct[i] = time;
                            rt[i] = 0;
                        }
                    }
                }
            } while (!done);

            calculateTurnaroundTimes(n, bt, wt, ta);
            displayResults(n, bt, at, ct, wt, ta);
            displayAverages(n, wt, ta);
        }
    }

    private static int getNumberOfProcesses(Scanner scanner) {
        System.out.print("Enter the number of processes: ");
        return scanner.nextInt();
    }

    private static void inputBurstTimes(Scanner scanner, int n, int[] bt, int[] rt) {
        System.out.println("Enter the burst time for each process:");
        for (int i = 0; i < n; i++) {
            bt[i] = scanner.nextInt();
            rt[i] = bt[i];
        }
    }

    private static void inputArrivalTimes(Scanner scanner, int n, int[] at) {
        System.out.println("Enter the arrival time for each process:");
        for (int i = 0; i < n; i++) {
            at[i] = scanner.nextInt();
        }
    }

    private static int getTimeQuantum(Scanner scanner) {
        System.out.print("Enter the time quantum: ");
        return scanner.nextInt();
    }

    private static void calculateTurnaroundTimes(int n, int[] bt, int[] wt, int[] ta) {
        for (int i = 0; i < n; i++) {
            ta[i] = bt[i] + wt[i];
        }
    }

    private static void displayResults(int n, int[] bt, int[] at, int[] ct, int[] wt, int[] ta) {
        System.out.println("Process\tBurst Time\tArrival Time\tCompletion Time\tWaiting Time\tTurnaround Time");
        for (int i = 0; i < n; i++) {
            System.out.println((i + 1) + "\t" + bt[i] + "\t\t" + at[i] +
                    "\t\t" + ct[i] + "\t\t" + wt[i] + "\t\t" + ta[i]);
        }
    }

    private static void displayAverages(int n, int[] wt, int[] ta) {
        int totalWaitingTime = 0, totalTurnaroundTime = 0;
        for (int i = 0; i < n; i++) {
            totalWaitingTime += wt[i];
            totalTurnaroundTime += ta[i];
        }
        float avgWaitingTime = (float) totalWaitingTime / n;
        float avgTurnaroundTime = (float) totalTurnaroundTime / n;
        System.out.println("\nAverage Waiting Time: " + avgWaitingTime);
        System.out.println("Average Turnaround Time: " + avgTurnaroundTime);
    }
}
------------------------------------
// Address book cpp
#include <iostream>
#include <vector>
#include <string>
using namespace std;

class Contact {
public:
    string name;
    string phone;
    string email;

    // Constructor to initialize a contact
    Contact(string n, string p, string e) {
        name = n;
        phone = p;
        email = e;
    }
};

class AddressBook {
private:
    vector<Contact> contacts;

public:
    // Function to create an address book (already handled by the constructor)
    void createAddressBook() {
        cout << "Address book created successfully." << endl;
    }

    // Function to view all contacts in the address book
    void viewAddressBook() {
        if (contacts.empty()) {
            cout << "Address book is empty." << endl;
        } else {
            cout << "Contacts in the address book: " << endl;
            for (size_t i = 0; i < contacts.size(); ++i) {
                cout << i + 1 << ". Name: " << contacts[i].name << ", Phone: " << contacts[i].phone << ", Email: " << contacts[i].email << endl;
            }
        }
    }

    // Function to insert a new contact
    void insertRecord() {
        string name, phone, email;
        cout << "Enter Name: ";
        cin.ignore(); // To ignore the leftover newline character in the buffer
        getline(cin, name);
        cout << "Enter Phone Number: ";
        getline(cin, phone);
        cout << "Enter Email Address: ";
        getline(cin, email);
        
        Contact newContact(name, phone, email);
        contacts.push_back(newContact);
        cout << "New contact added successfully." << endl;
    }

    // Function to delete a contact by index
    void deleteRecord() {
        int index;
        cout << "Enter the index of the contact to delete (1-based index): ";
        cin >> index;

        if (index < 1 || index > contacts.size()) {
            cout << "Invalid index!" << endl;
        } else {
            contacts.erase(contacts.begin() + index - 1);
            cout << "Contact deleted successfully." << endl;
        }
    }

    // Function to modify a contact's details
    void modifyRecord() {
        int index;
        cout << "Enter the index of the contact to modify (1-based index): ";
        cin >> index;

        if (index < 1 || index > contacts.size()) {
            cout << "Invalid index!" << endl;
        } else {
            string name, phone, email;
            cout << "Enter new Name: ";
            cin.ignore(); // To ignore the leftover newline character in the buffer
            getline(cin, name);
            cout << "Enter new Phone Number: ";
            getline(cin, phone);
            cout << "Enter new Email Address: ";
            getline(cin, email);

            contacts[index - 1].name = name;
            contacts[index - 1].phone = phone;
            contacts[index - 1].email = email;
            cout << "Contact modified successfully." << endl;
        }
    }

    // Function to exit the program
    void exit() {
        cout << "Exiting address book program." << endl;
    }
};

int main() {
    AddressBook addressBook;
    int choice;

    while (true) {
        cout << "\nMenu:\n";
        cout << "1. Create Address Book\n";
        cout << "2. View Address Book\n";
        cout << "3. Insert Record\n";
        cout << "4. Delete Record\n";
        cout << "5. Modify Record\n";
        cout << "6. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
            case 1:
                addressBook.createAddressBook();
                break;
            case 2:
                addressBook.viewAddressBook();
                break;
            case 3:
                addressBook.insertRecord();
                break;
            case 4:
                addressBook.deleteRecord();
                break;
            case 5:
                addressBook.modifyRecord();
                break;
            case 6:
                addressBook.exit();
                return 0;
            default:
                cout << "Invalid choice. Please try again." << endl;
        }
    }

    return 0;
}
-----------------------------------
//disc scheduling(SSTF/SCAN/CSCAN)
#include <stdio.h>
#include <stdlib.h>

#define MAX 100

int abs_diff(int a, int b) {
    return abs(a - b);
}

//SSTF fucntion
void SSTF(int requests[], int n, int head) {
    int completed[MAX] = {0}; 
    int total_seek = 0;
    int current_position = head;

    printf("\nSSTF Disk Scheduling:\n");
    printf("Seek Sequence: %d", current_position);

    for (int i = 0; i < n; i++) {
        int min_seek = MAX;
        int closest_request = -1;

        //closest unserved request
        for (int j = 0; j < n; j++) {
            if (!completed[j] && abs_diff(current_position, requests[j]) < min_seek) {
                min_seek = abs_diff(current_position, requests[j]);
                closest_request = j;
            }
        }

        total_seek += min_seek;
        current_position = requests[closest_request];
        completed[closest_request] = 1; 
        printf(" -> %d", current_position);
    }

    printf("\nTotal seek time: %d\n", total_seek);
}

// SCAN (Elevator) funtion
void SCAN(int requests[], int n, int head, int disk_size, int direction) {
    int total_seek = 0;
    int current_position = head;
    
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (requests[i] > requests[j]) {
                int temp = requests[i];
                requests[i] = requests[j];
                requests[j] = temp;
            }
        }
    }

    printf("\nSCAN Disk Scheduling:\n");
    printf("Seek Sequence: %d", current_position);

    if (direction == 1) { //move to larger requests
        for (int i = 0; i < n; i++) {
            if (requests[i] >= head) {
                printf(" -> %d", requests[i]);
                total_seek += abs_diff(current_position, requests[i]);
                current_position = requests[i];
            }
        }
        //move to end
        total_seek += abs_diff(current_position, disk_size - 1);
        printf(" -> %d", disk_size - 1);
        current_position = disk_size - 1;

        // back towards smaller request
        for (int i = n - 1; i >= 0; i--) {
            if (requests[i] < head) {
                printf(" -> %d", requests[i]);
                total_seek += abs_diff(current_position, requests[i]);
                current_position = requests[i];
            }
        }
    } else { //move to smaller request
        for (int i = n - 1; i >= 0; i--) {
            if (requests[i] <= head) {
                printf(" -> %d", requests[i]);
                total_seek += abs_diff(current_position, requests[i]);
                current_position = requests[i];
            }
        }
        // move to start 
        total_seek += abs_diff(current_position, 0);
        printf(" -> %d", 0);
        current_position = 0;

        // back towards karger requests
        for (int i = 0; i < n; i++) {
            if (requests[i] > head) {
                printf(" -> %d", requests[i]);
                total_seek += abs_diff(current_position, requests[i]);
                current_position = requests[i];
            }
        }
    }

    printf("\nTotal seek time: %d\n", total_seek);
}

// C-LOOK function
void CLOOK(int requests[], int n, int head, int direction) {
    int total_seek = 0;
    int current_position = head;
    
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (requests[i] > requests[j]) {
                int temp = requests[i];
                requests[i] = requests[j];
                requests[j] = temp;
            }
        }
    }

    printf("\nC-LOOK Disk Scheduling:\n");
    printf("Seek Sequence: %d", current_position);

    if (direction == 1) { // Move towards larger requests
        for (int i = 0; i < n; i++) {
            if (requests[i] >= head) {
                printf(" -> %d", requests[i]);
                total_seek += abs_diff(current_position, requests[i]);
                current_position = requests[i];
            }
        }
        // Jump to the smallest request
        total_seek += abs_diff(current_position, requests[0]);
        current_position = requests[0];
        printf(" -> %d", requests[0]);

        // Serve the remaining smaller requests
        for (int i = 1; i < n; i++) {
            if (requests[i] < head) {
                printf(" -> %d", requests[i]);
                total_seek += abs_diff(current_position, requests[i]);
                current_position = requests[i];
            }
        }
    } else { // Move towards smaller requests
        for (int i = n - 1; i >= 0; i--) {
            if (requests[i] <= head) {
                printf(" -> %d", requests[i]);
                total_seek += abs_diff(current_position, requests[i]);
                current_position = requests[i];
            }
        }
        // Jump to the largest request
        total_seek += abs_diff(current_position, requests[n - 1]);
        current_position = requests[n - 1];
        printf(" -> %d", requests[n - 1]);

        // Serve the remaining larger requests
        for (int i = n - 2; i >= 0; i--) {
            if (requests[i] > head) {
                printf(" -> %d", requests[i]);
                total_seek += abs_diff(current_position, requests[i]);
                current_position = requests[i];
            }
        }
    }

    printf("\nTotal seek time: %d\n", total_seek);
}

int main() {
    int n, head, disk_size, direction;
    int requests[MAX];

    // Input
    printf("Number of requests: ");
    scanf("%d", &n);

    printf("Enter requests:\n");
    for (int i = 0; i < n; i++) {
        scanf("%d", &requests[i]);
    }

    printf("Head: ");
    scanf("%d", &head);

    printf("Disk size: ");
    scanf("%d", &disk_size);

    printf("Direction(1-right/0-left): ");
    scanf("%d", &direction);
    
    SSTF(requests, n, head);
    SCAN(requests, n, head, disk_size, direction);
    CLOOK(requests, n, head, direction);

    return 0;
}
--------------------------------------
//Page replacement (FIFO/LRU/OPTIMAL)
#include <stdio.h>
#include <stdbool.h>

// FIFO Page Replacement Algorithm
int fifo_page_faults(int reference_string[], int n, int frame_size) {
    int page_table[frame_size];
    int page_faults = 0;
    int front = 0;

    for (int i = 0; i < frame_size; i++) {
        page_table[i] = -1;
    }

    for (int i = 0; i < n; i++) {
        bool page_found = false;
        int page = reference_string[i];

        for (int j = 0; j < frame_size; j++) {
            if (page_table[j] == page) {
                page_found = true;
                break;
            }
        }

        if (!page_found) {
            page_table[front] = page;
            front = (front + 1) % frame_size;
            page_faults++;
        }
    }

    return page_faults;
}

// LRU Page Replacement Algorithm
int lru_page_faults(int reference_string[], int n, int frame_size) {
    int page_table[frame_size];
    int page_faults = 0;

    for (int i = 0; i < frame_size; i++) {
        page_table[i] = -1;
    }

    for (int i = 0; i < n; i++) {
        bool page_found = false;
        int page = reference_string[i];

        for (int j = 0; j < frame_size; j++) {
            if (page_table[j] == page) {
                page_found = true;
                // Move the accessed page to the end of the table (most recently used)
                for (int k = j; k < frame_size - 1; k++) {
                    page_table[k] = page_table[k + 1];
                }
                page_table[frame_size - 1] = page;
                break;
            }
        }

        if (!page_found) {
            // If a page fault occurs, replace the first page (least recently used)
            for (int j = 0; j < frame_size - 1; j++) {
                page_table[j] = page_table[j + 1];
            }
            page_table[frame_size - 1] = page;
            page_faults++;
        }
    }

    return page_faults;
}

// Optimal Page Replacement Algorithm
int optimal_page_faults(int reference_string[], int n, int frame_size) {
    int page_table[frame_size];
    int page_faults = 0;

    for (int i = 0; i < frame_size; i++) {
        page_table[i] = -1;
    }

    for (int i = 0; i < n; i++) {
        bool page_found = false;
        int page = reference_string[i];

        for (int j = 0; j < frame_size; j++) {
            if (page_table[j] == page) {
                page_found = true;
                break;
            }
        }

        if (!page_found) {
            int replace_index = -1;
            int furthest_used = -1;

            for (int j = 0; j < frame_size; j++) {
                int page_in_memory = page_table[j];
                int next_use = n;  // Default to the last occurrence

                for (int k = i + 1; k < n; k++) {
                    if (reference_string[k] == page_in_memory) {
                        next_use = k;
                        break;
                    }
                }

                if (next_use > furthest_used) {
                    furthest_used = next_use;
                    replace_index = j;
                }
            }

            page_table[replace_index] = page;
            page_faults++;
        }
    }

    return page_faults;
}

int main() {
    int n, frame_size;
    
    printf("Enter the number of pages: ");
    scanf("%d", &n);
    
    int reference_string[n];
    
    printf("Enter the reference string:\n");
    for (int i = 0; i < n; i++) {
        scanf("%d", &reference_string[i]);
    }
    
    printf("Enter the frame size: ");
    scanf("%d", &frame_size);

    int fifo_faults = fifo_page_faults(reference_string, n, frame_size);
    int lru_faults = lru_page_faults(reference_string, n, frame_size);
    int optimal_faults = optimal_page_faults(reference_string, n, frame_size);

    printf("FIFO Page Faults: %d\n", fifo_faults);
    printf("LRU Page Faults: %d\n", lru_faults);
    printf("Optimal Page Faults: %d\n", optimal_faults);

    return 0;
}
--------------------------------------
