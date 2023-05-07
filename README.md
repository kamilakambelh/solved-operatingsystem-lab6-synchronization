Download Link: https://assignmentchef.com/product/solved-operatingsystem-lab6-synchronization
<br>
Goal: This lab helps student to practice with the synchronization in OS, and understand the reason why we need the synchronization.

Content In detail, this lab requires student practice with examples using synchronization techniques to solve the problem called race condition. The synchronization is performed on Thread, including the following techniques:

<ul>

 <li>Mutex</li>

 <li>Condition variable</li>

 <li>Semaphore</li>

</ul>

Result After doing this lab, student can understand the definition of synchronization and write a program which allows to solve the race condition using the techniques above.

Requirement           Student need to review the theory of synchronization.

<h1>1.      Problem</h1>

Let us imagine a simple example where two threads wish to update a global shared variable.

<table width="564">

 <tbody>

  <tr>

   <td width="564">#include &lt;stdio .h&gt;#include &lt;pthread.h&gt; static volatile int counter = 0;void ∗mythread(void ∗arg){printf (“%s : begin
” , (char ∗) arg ); int i ;for ( i = 0; i &lt; 1e7; i++) {counter = counter + 1;} printf (“%s : done
” , (char ∗) arg );return NULL;}int main(int argc , char ∗argv []){ pthread_t p1, p2;printf (“main: begin (counter = %d)
” , counter );pthread_create(&amp;p1, NULL, mythread, “A”); pthread_create(&amp;p2, NULL, mythread, “B”);<em>// join waits for the threads to finish </em>pthread_join(p1, NULL); pthread_join(p2, NULL);printf (“main: done with both (counter = %d)
” , counter );return 0;}</td>

  </tr>

 </tbody>

</table>

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

Finally, and most importantly, we can now look at what each worker is trying to do: add a number to the shared variable counter, and do so 10 million times (1e7) in a loop.

Thus, the desired final result is: 20,000,000.

We now compile and run the program, to see how it behaves.

<h1>2.      Synchronization</h1>

What we have demonstrated above (in section Problem) is called a race condition: the results depend on the timing execution of the code. With some bad luck (i.e., context switches that occur at untimely points in the execution), we get the wrong result. In fact, we may get a different result each time; thus, instead of a nice deterministic computation (which we are used to from computers), we call this result indeterminate, where it is not known what the output will be and it is indeed likely to be different across runs.

Because multiple threads executing this code can result in a race condition, we call this code a critical section. A critical section is a piece of code that accesses a shared variable (or more generally, a shared resource) and must not be concurrently executed by more than one thread.

What we really want for this code is what we call mutual exclusion. This property guarantees that if one thread is executing within the critical section, the others will be prevented from doing so.

<h1>3.      Synchronization with Thread</h1>

<h2>3.1.       Thread API</h2>

In the previous lab, we have studied the functions that allow to work with Thread.

<ul>

 <li>Thread creation</li>

</ul>

#include &lt;pthread.h&gt; int pthread_create( pthread_t ∗ thread , const pthread_attr_t ∗ attr ,

void ∗ (∗start_routine)(void∗), void ∗ arg );

<ul>

 <li>Thread completion</li>

</ul>

int pthread_join(pthread_t thread , void ∗∗value_ptr);

<h2>3.2.       Locks</h2>

The POSIX threads library provides mutual exclusion to protect the critical section via locks. The most basic pair of routines to use for this purpose is provided by this pair of routines:

int pthread_mutex_lock(pthread_mutex_t ∗mutex);

int pthread_mutex_unlock(pthread_mutex_t ∗mutex);

Practice: with the example in Section Problem, we can use this method to solve the problem named race condition.

<table width="564">

 <tbody>

  <tr>

   <td width="564">#include &lt;stdio .h&gt;#include &lt;stdlib .h&gt; #include &lt;assert .h&gt;</td>

  </tr>

 </tbody>

</table>

1

2

3

<table width="564">

 <tbody>

  <tr>

   <td width="564">#include &lt;pthread.h&gt;static volatile int counter = 0;pthread_mutex_t lock ;void ∗mythread(void ∗arg){ printf (“%s
” , (char ∗)arg );pthread_mutex_lock(&amp;lock ); int i ; for( i = 0; i &lt; 1e7; i++)counter = counter + 1;pthread_mutex_unlock(&amp;lock );printf (“%s : done
” , (char ∗)arg );return NULL;}int main(int argc , char ∗∗argv){ pthread_t p1, p2; int rc ; pthread_mutex_init(&amp;lock , NULL);printf (“main: begin (counter − %d)
” , counter );rc = pthread_create(&amp;p1, NULL, mythread, “A”); assert(rc == 0); rc = pthread_create(&amp;p2, NULL, mythread, “B”); assert(rc == 0);rc = pthread_join(p1, NULL); assert(rc == 0); rc = pthread_join(p2, NULL); assert(rc == 0);printf (“main: finish with both (counter − %d)
” , counter );return 0;}</td>

  </tr>

 </tbody>

</table>

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

Student implement the function above, then compile and execute it. Give the discussion about the method using locks.

<h2>3.3.       Semaphore</h2>

Binary semaphore A binary semaphore can only be 0 or 1. Binary semaphores are most often used to implement a lock that allows only a single thread into a critical section. The semaphore is initially given the value 1 and when a thread approaches the critical region, it waits on the semaphore to decrease the value and “take out” the lock, then signals the semaphore at the end of the critical region to release the lock.

Binary Semaphore Example The canonical use of a semaphore is a lock associated with some resource so that only one thread at a time has access to the resource. In the example below, we have one piece of global data, the number of tickets remaining to sell, that we want to coordinate the access by multiple threads. In this case, a binary semaphore serves as a lock to guarantee that at most one thread is examining or changing the value of the variable at any given time.

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

<table width="564">

 <tbody>

  <tr>

   <td width="564">#include &lt;stdio .h&gt;#include &lt;pthread.h&gt;#include &lt;stdlib .h&gt;#include &lt;semaphore.h&gt;#define NUM_TICKETS                    35#define NUM_SELLERS                    4#define true 1#define false 0static int numTickets = NUM_TICKETS; static sem_t ticketLock ; void ∗ sellTicket (void ∗arg );int main(int argc , char ∗∗argv){ int i ; int tid [NUM_SELLERS]; pthread_t sellers [NUM_SELLERS];sem_init(&amp;ticketLock , 0, 1); for ( i = 0; i &lt; NUM_SELLERS; i++) { tid [ i ] = i ;pthread_create(&amp;sellers [ i ] , NULL, sellTicket , (void ∗)}for( i = 0; i &lt; NUM_SELLERS; i++) pthread_join( sellers [ i ] , NULL);sem_destroy(&amp;ticketLock ); pthread_exit(NULL);return 0;}</td>

  </tr>

 </tbody>

</table>

25                                                                                                                                                                                      tid [ i ]);

26

27

28

29

30

31

32

33

34

35

<table width="564">

 <tbody>

  <tr>

   <td width="564">void ∗sellTicket (void ∗arg){int done = false ; int numSoldByThisThread = 0; int tid = (int) arg ; while(!done){<em>//                                          sleep (1);</em>sem_wait(&amp;ticketLock );if (numTickets == 0) done = true ; else{ numTickets−−; numSoldByThisThread++;printf (“Thread %d sold one (%d left )
” , tid , numTickets);} sem_post(&amp;ticketLock );sleep (1);} printf (“Thread %d sold %d tickets
” , tid , numSoldByThisThread);pthread_exit(NULL);}</td>

  </tr>

 </tbody>

</table>

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

General semaphores A general semaphore can take on any non-negative value. General semaphores are used for “counting” tasks such as creating a critical region that allows a specified number of threads to enter. For example, if you want at most four threads to be able to enter a section, you could protect it with a semaphore and initialize that semaphore to four. The first four threads will be able to decrease the semaphore and enter the region, but at that point, the semaphore will be zero and any other threads will block outside the critical region until one of the current threads leaves and signals the semaphore.

Generalized semaphore example The next synchronization problem we will confront in this chapter is known as the producer/consumer problem, or sometimes as the bounded buffer problem, which was first posed by Dijkstra [D72]. Imagine one or more producer threads and one or more consumer threads. Producers produce data items and wish to place them in a buffer; consumers grab data items out of the buffer consume them in some way.

This arrangement occurs in many real systems. For example, in a multi-threaded web server, a producer puts HTTP requests into a work queue (i.e., the bounded buffer); consumer threads take requests out of this queue and process them.

To solve the problem above, we use two semaphores, empty and full, which the threads will use to indicate when a buffer entry has been emptied or filled, respectively. Let’s try with the first attempt below:

<table width="564">

 <tbody>

  <tr>

   <td width="564">#include &lt;stdio .h&gt;#include &lt;stdlib .h&gt;#include &lt;semaphore.h&gt;#include &lt;pthread.h&gt;#define MAX_ITEMS 1#define THREADS 1                                  <em>// 1 producer and 1 consumer</em>#define LOOPS 2∗MAX_ITEMS                              <em>// variable</em><em>// Initiate shared buffer </em>int buffer [MAX_ITEMS]; int f i l l = 0; int use = 0;sem_t empty; sem_t full ;void put(int value );                                  <em>// put data into buffer</em>int get ();                                 <em>// get data from buffer</em>void ∗producer(void ∗arg) {int i ; int tid = (int) arg ; for ( i = 0; i &lt; LOOPS; i++) { sem_wait(&amp;empty); <em>// line P1 </em>put( i );               <em>// line P2</em>printf (“Producer %d put data %d
” , tid , i ); sleep (1);sem_post(&amp;full );                              <em>// line P3</em>} pthread_exit(NULL);}void ∗consumer(void ∗arg) { int i , tmp = 0; int tid = (int) arg ;while (tmp != −1) { sem_wait(&amp;full ); <em>// line C1 </em>tmp = get (); <em>// line C2</em>printf (“Consumer %d get data %d
” , tid , tmp); sleep (1);sem_post(&amp;empty);                          <em>// line C3</em></td>

  </tr>

 </tbody>

</table>

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

53

54

55

56

57

58

<table width="564">

 <tbody>

  <tr>

   <td width="564">} pthread_exit(NULL);}int main(int argc , char ∗∗argv){int i , j ; int tid [THREADS]; pthread_t producers [THREADS]; pthread_t consumers[THREADS];sem_init(&amp;empty, 0, MAX_ITEMS); sem_init(&amp;full , 0, 0);for( i = 0; i &lt; THREADS; i++){ tid [ i ] = i ;<em>// Create producer thread </em>pthread_create(&amp;producers [ i ] , NULL, producer , (void ∗)<em>// Create consumer thread </em>pthread_create(&amp;consumers[ i ] , NULL, consumer, (void ∗)}for( i = 0; i &lt; THREADS; i++){pthread_join(producers [ i ] , NULL); pthread_join(consumers[ i ] , NULL);}sem_destroy(&amp;full ); sem_destroy(&amp;empty);return 0;}void put(int value) { buffer [ f i l l ] = value ; <em>// line f1</em>f i l l = ( f i l l + 1) % MAX_ITEMS; <em>// line f2</em>}int get(){ int tmp = buffer [ use ]; <em>// line g1 </em>use = (use + 1) % MAX_ITEMS; <em>// line g2</em>return tmp;}</td>

  </tr>

 </tbody>

</table>

59                                                                                                                                                                                      tid [ i ]);

60

61

62                                                                                                                                                                                      tid [ i ]);

63

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

Some problems issued:

<ul>

 <li>In this example, the producer first waits for a buffer to become empty in order to put data into it, and the consumer similarly waits for a buffer to become filled before using it. Let us first imagine that MAX_ITEMS=1 (there is only one buffer in the array), and see if this works.</li>

 <li>You can try this same example with more threads (e.g., multiple producers, and multiple consumers). It should still work.</li>

 <li>Let us now imagine that MAX_ITEMS is greater than 1 (say MAX_ITEMS =</li>

</ul>

10). For this example, let us assume that there are multiple producers and multiple consumers. We now have a problem. Do you see where it occurs?

<h3>A Solution: Adding Mutual Exclusion</h3>

The filling of a buffer and incrementing of the index into the buffer is a critical section, and thus must be guarded carefully. Now we have added some locks around the entire put()/get() parts of the code, as indicated by the NEW LINE comments. That seems like the right idea, but it also doesn’t work. Why? Deadlock.

<ul>

 <li>Why does deadlock occur?</li>

</ul>

<h3>Avoiding Deadlock</h3>

<table width="564">

 <tbody>

  <tr>

   <td colspan="2" width="452">. . .#define#define MAX_ITEMS 10#define THREADS 2                                           <em>// 2 producers and 2 consumers</em>#define LOOPS 2∗MAX_ITEMS                              <em>// variable</em></td>

   <td width="112"> </td>

  </tr>

  <tr>

   <td width="323">. . .sem_t empty; sem_t full ; sem_t lock ;. . .void ∗producer(void ∗arg) {int i ; int tid = (int) arg ; for ( i = 0; i &lt; LOOPS; i++) {sem_wait(&amp;lock );</td>

   <td width="128"><em>// line P0</em></td>

   <td width="112"><em>(NEW LINE)</em></td>

  </tr>

  <tr>

   <td width="323">sem_wait(&amp;empty);</td>

   <td width="128"><em>// line P1</em></td>

   <td width="112"> </td>

  </tr>

 </tbody>

</table>

1

2

3

4

5

6

7

8

9

10

11

12

13

14

15

16

17

18

19

20

<table width="564">

 <tbody>

  <tr>

   <td width="564">                                            put( i );                                                                         <em>// line P2</em>printf (“Producer %d put data %d
” , tid , i ); sleep (1);sem_post(&amp;full );                              <em>// line P3</em>sem_post(&amp;lock ); <em>// line P4 (NEW LINE)</em>} pthread_exit(NULL);}void ∗consumer(void ∗arg) { int i , tmp = 0; int tid = (int) arg ;while (tmp != −1) { sem_wait(&amp;lock ); <em>// line C0 (NEW LINE) </em>sem_wait(&amp;full ); <em>// line C1 </em>tmp = get (); <em>// line C2</em>printf (“Consumer %d get data %d
” , tid , tmp); sleep (2);sem_post(&amp;empty);                          <em>// line C3</em>sem_post(&amp;lock );                                        <em>// line C4 (NEW LINE)</em>} pthread_exit(NULL);} int main(int argc , char ∗∗argv){. . .sem_init(&amp;empty, 0, MAX_ITEMS); sem_init(&amp;full , 0, 0); sem_init(&amp;lock , 0, 1);. . .}</td>

  </tr>

 </tbody>

</table>

21

22

23

24

25

26

27

28

29

30

31

32

33

34

35

36

37

38

39

40

41

42

43

44

45

46

47

48

49

50

51

52

Imagine two threads, one producer and one consumer. The consumer gets to run first. It acquires the lock (line C0), and then calls sem_wait() on the full semaphore (line C1); because there is no data yet, this call causes the consumer to block and thus yield the CPU; importantly, though, the consumer still holds the lock.

A producer then runs. It has data to produce and if it were able to run, it would be able to wake the consumer thread and all would be good. Unfortunately, the first thing it does is call sem_wait() on the binary lock semaphore (line P0). The lock is already held. Hence, the producer is now stuck waiting too.

<h3>A Working Solution</h3>

To solve this problem, we simply must reduce the scope of the lock. We simply move the lock acquire and release to be just around the critical section; the full and empty wait and signal code is left outside. Let’s try it.

<h1>4.      Exercise (Required)</h1>

Problem 1 (5 points): Race conditions are possible in many computer systems. Consider a banking system that maintains an account balance with two functions: deposit (amount) and withdraw (amount). These two functions are passed the amount that is to be deposited or withdrawn from the bank account balance. Assume that a husband and wife share a bank account. Concurrently, the husband calls the withdraw() function and the wife calls deposit(). Write a short essay listing possible outcomes we could get and pointing out in which situations those outcomes are produced. Also, propose methods that the bank could apply to avoid unexpected results.

Problem 2 (5 points): In the Exercise 1 of Lab 5, we wrote a simple multi-thread program for calculating the value of pi using Monte-Carlo method. In this exercise, we also calculate pi using the same method but with a different implementation. We create a shared (global) count variable and let worker threads update on this variable in each of their iteration instead of on their own local count variable. To make sure the result is correct, remember to avoid race conditions on updates to the shared global variable by using mutex locks. Compare the performance of this approach with the previous one in Lab 5.

Put your answer to the questions in both exercise to a single PDF file name ex.pdf. In the problem 2, reuse the file you have written for Lab 5 and only modify parts you think it need to be. You must submit your ZIP file to Sakai.

<h1>A. Condition variables</h1>

The other major component of any threads library, and certainly the case with POSIX threads, is the presence of a condition variable. Condition variables are useful when some kind of signaling must take place between threads, if one thread is waiting for another to do something before it can continue. Two primary routines are used by programs wishing to interact in this way:

int pthread_cond_wait(pthread_cond_t ∗cond, pthread_mutex_t ∗mutex);

int pthread_cond_signal(pthread_cond_t ∗cond);

To use a condition variable, one has to in addition have a lock that is associated with this condition. When calling either of the above routines, this lock should be held.

The first routine, pthread_cond_wait(), puts the calling thread to sleep, and thus waits for some other thread to signal it, usually when something in the program has changed that the now-sleeping thread might care about.

Practice: to illustrate the situation using condition variables with Thread. We write a program which creates 3 threads, where 2 threads is used to increase a global variable named count, the last thread is used to cause the condition, it has to wait the signal from other process to continue.

<table width="97">

 <tbody>

  <tr>

   <td width="48">


    <table width="46">

     <tbody>

      <tr>

       <td width="46"><strong>Thread 2</strong></td>

      </tr>

     </tbody>

    </table></td>

   <td width="48">


    <table width="46">

     <tbody>

      <tr>

       <td width="46"><strong>Thread 3</strong></td>

      </tr>

     </tbody>

    </table></td>

  </tr>

 </tbody>

</table>

Figure A.1: Condition variable.

<table width="564">

 <tbody>

  <tr>

   <td width="564">#include &lt;stdio .h&gt;#include &lt;stdlib .h&gt; #include &lt;assert .h&gt;#include &lt;pthread.h&gt;</td>

  </tr>

 </tbody>

</table>

1

2

3

4

5

6#define NUM_THREADS 3

7#define TCOUNT 100

8#define COUNT_LIMIT 20

9

10int count = 10;

11pthread_mutex_t count_mutex;

12pthread_cond_t count_threshold_cv;

13

14void ∗inc_count(void ∗tid){

15int i ;

16long my_id = (long) tid ;

17for( i = 0; i &lt; TCOUNT; i++){

18pthread_mutex_lock(&amp;count_mutex);

19count++;

20if (count == COUNT_LIMIT){

21printf (“inc_count(): thread %ld , count = %d,

22threshold reached.
” , my_id, count);

23pthread_cond_signal(&amp;count_threshold_cv);

24printf (“Just sent signal 
”);

25}

26

27printf (“inc_count(): thread %ld , count = %d,

28uncloking mutex
” , my_id, count);

29pthread_mutex_unlock(&amp;count_mutex);

30sleep (1);

31}

32pthread_exit(NULL);

33}

34

35void ∗watch_count(void ∗tid){

36long my_id = (long) tid ;

37printf (“Starting watch_count(): thread %ld
” , my_id);

38

39pthread_mutex_lock(&amp;count_mutex);

40while(count &lt; COUNT_LIMIT){

41printf (“watch_count(): thread %ld , count = %d,

42waiting …
” , my_id, count);

43pthread_cond_wait(&amp;count_threshold_cv ,

44&amp;count_mutex);

45printf (“watch_count(): thread %ld . Condition signal received .

46Count = %d
” , my_id, count);

47printf (“watch_count(): thread %ld

48Updating the count value …
” , my_id);

49count += 80;

<table width="564">

 <tbody>

  <tr>

   <td width="564">printf (“watch_count(): thread %ld count now = %d
” , my_id, count);} printf (“watch_count(): thread %ld . Unlocking mutex. 
” , my_id);pthread_mutex_unlock(&amp;count_mutex);pthread_exit(NULL);}int main(int argc , char ∗∗argv){ int i , rc ; pthread_t p1, p2, p3;long t1 = 1, t2 = 2, t3 = 3; pthread_attr_t attr ;printf (“main: begin
”);pthread_mutex_init(&amp;count_mutex, NULL); pthread_cond_init(&amp;count_threshold_cv , NULL);thread_attr_init(&amp;attr ); pthread_attr_setdetachstate(&amp;attr , PTHREAD_CREATE_JOINABLE);pthread_create(&amp;p1, &amp;attr , watch_count, (void ∗)t1 ); pthread_create(&amp;p2, &amp;attr , inc_count , (void ∗)t2 ); pthread_create(&amp;p3, &amp;attr , inc_count , (void ∗)t3 );rc = pthread_join(p1, NULL); assert(rc == 0); rc = pthread_join(p2, NULL); assert(rc == 0); rc = pthread_join(p3, NULL); assert(rc == 0); printf (“main: finish , final count = %d
” , count);pthread_attr_destroy(&amp;attr ); pthread_mutex_destroy(&amp;count_mutex); pthread_cond_destroy(&amp;count_threshold_cv); pthread_exit(NULL);return 0;}</td>

  </tr>

 </tbody>

</table>

50

51

52

53

54

55

56

57

58

59

60

61

62

63

64

65

66

67

68

69

70

71

72

73

74

75

76

77

78

79

80

81

82

83

84

85

86

87

88