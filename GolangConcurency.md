### We will deep dive into Golang Concurrency and Channels in details with all the necessary things to learn

__Process__ is an instance of program that is currently executed by a computer.

__Thread__ represents the sequential execution of a set of instructions. A single process can create multiple threads

__Concurrency__ is the ability of different parts or units of a program, algorithm, or problem to be executed out-of-order or in partial order.

A better sense to explain concurrency
_Concurrency_ can be thought of as a programs ability to execute multiple tasks in overlapping time frames, without guarenteeing that they will execute simultaneously.

> Real-world analogy: Imagine a resturant kitchen with multiple chefs. Each chef is working on their assigned tasks, like chopping vegetables, grilling meat, and preparing sauces. While these chefs work independently on their tasks, the restaurant manager coordiantes their work, ensuring that dishes are prepared efficiently and ready to be served. This is analogous is concurrent programming where tasks are managed efficiently, and the cpu switches between them to ensure progress. `



__Parallelism__, on the other hand, is the simultaneous execution of multiple tasks or processes, typically to improve performance by taking advantage of multiple CPU cores or processors. It’s about running multiple threads or processes in parallel to achieve a speedup in execution time.

> Real-world analogy: Think of an assembly line in a car manufacturing plant. Each worker is responsible for a specific part of the car assembly, and these workers are all working simultaneously. This is analogous to parallel programming, where multiple threads or processes are actively executing at the same time to achieve faster computation.

##### Differences:
1. _Concurrency_ is about managing and organizing tasks, while _parallelism_ is about executing tasks simultaneously to improve performance.

2. _Concurrency_ doesn’t guarantee that tasks will run in true parallel; they may share the same resources or take turns using them. _Parallelism_, on the other hand, ensures tasks are actively running at the same time on separate resources.

