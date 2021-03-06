# Starve-free-Readers---Writers-Problem
The third readers-writers problem
We will build the solution using C code. We will use semaphores for mutual exclusion (mutex). Those semaphores, being used as locks, are all initialized in the released state (1 available place); those initializations will not appear in code snippets below but will be shown in comments.

First of all, we said earlier that we want fair-queuing between readers and writers in order to prevent starvation. To achieve that, we will use a semaphore named orderMutex that will materialize the order of arrival. This semaphore will be taken by any entity that requests access to the resource, and released as soon as this entity gains access to the resource:


semaphore orderMutex;      // Initialized to 1

void reader()
{
  P(orderMutex);           // Remember our order of arrival
  ...
  V(orderMutex);           // Released when the reader can access the resource
  ...
}

void writer()
{
  P(orderMutex);           // Remember our orderof arrival
  ...
  V(orderMutex);           // Released when the writer can access the resource
}
Now, we can write the writer code as it is the most straightforward of both. The writer wants an exclusive access to the resource. We will create a new semaphore named accessMutex that the writer will request before modifying the resource:
semaphore accessMutex;     // Initialized to 1
semaphore orderMutex;      // Initialized to 1

void reader()
{
  P(orderMutex);           // Remember our order of arrival
  ...
  V(orderMutex);           // Released when the reader can access the resource
  ...
}

void writer()
{
  P(orderMutex);           // Remember our order of arrival
  P(accessMutex);          // Request exclusive access to the resource
  V(orderMutex);           // Release order of arrival semaphore (we have been served)

  WriteResource();         // Here the writer can modify the resource at will

  V(accessMutex);          // Release exclusive access to the resource
}
The reader code is a bit more complicated as multiple readers can simultaneously access the resource. We want t first reader to get access to the resource to lock it so that no writer can access it at the same time. Similarly, when a reader is done with the resource, it needs to release the lock on the resource if there are no more readers currently accessing it.

This is similar to a light switch in a dark room: the first person entering the room will turn the lights on, while the last person leaving the room will turn the lights off. However, it is much more simple if the room has only one door and only one person can enter or leave the room at the same time, to prevent someone from switching the lights off when someone enters at the same time using another door. This is why we will use a counter named readers representing the number of readers currently accessing the resource, as well as a semaphore named readersMutex to protect the counter against conflicting accesses.


semaphore accessMutex;     // Initialized to 1
semaphore readersMutex;    // Initialized to 1
semaphore orderMutex;      // Initialized to 1

unsigned int readers = 0;  // Number of readers accessing the resource

void reader()
{
  P(orderMutex);           // Remember our order of arrival

  P(readersMutex);         // We will manipulate the readers counter
  if (readers == 0)        // If there are currently no readers (we came first)...
    P(accessMutex);        // ...requests exclusive access to the resource for readers
  readers++;               // Note that there is now one more reader
  V(orderMutex);           // Release order of arrival semaphore (we have been served)
  V(readersMutex);         // We are done accessing the number of readers for now

  ReadResource();          // Here the reader can read the resource at will

  P(readersMutex);         // We will manipulate the readers counter
  readers--;               // We are leaving, there is one less reader
  if (readers == 0)        // If there are no more readers currently reading...
    V(accessMutex);        // ...release exclusive access to the resource
  V(readersMutex);         // We are done accessing the number of readers for now
}

void writer()
{
  P(orderMutex);           // Remember our order of arrival
  P(accessMutex);          // Request exclusive access to the resource
  V(orderMutex);           // Release order of arrival semaphore (we have been served)

  WriteResource();         // Here the writer can modify the resource at will

  V(accessMutex);          // Release exclusive access to the resource
