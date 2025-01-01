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

###### Goroutines: 
Goroutines are lightweight, independently scheduled functions that can be executed concurrently. They are similar to threads but are managed by Go’s runtime, making them more efficient. Goroutines enable concurrent execution in Go.

###### Channels: 
Channels are a communication mechanism in Go that allows goroutines to communicate and synchronize. They facilitate coordination between concurrent tasks, which is crucial for many concurrent programs.

#### Concurrenty is about dealing with lots of things at once.
#### Parellism is about doing lots of things at once.

### WaitGroups

A WaitGroup blocks a program and waits for a set of goroutines to finish before moving to the next steps of execution.



We can use Waitgroups through the following functiions:
Add(int): This function takes in an integer value which is essentially

```
package main

import (
	"fmt"
	"sync"
	"time"
)

// Because we are going to use a method "func (wg *WaitGroup) Done()",
// so "WaitGroup" must be passed by "pointer"
func worker(id int, wg *sync.WaitGroup) {
	fmt.Printf("Worker : %d -> Starting\n", id)

	// Sleep to simulate an expensive task
	time.Sleep(time.Second)
	fmt.Printf("Worker : %d -> Done\n", id)

	// Notify the "WaitGroup" that this "worker" is "done"
	wg.Done()
}

func main() {

	// This "WaitGroup" is used to wait for all the goroutines launched here to finish
	var wg sync.WaitGroup

	// Launch several goroutines and increase the "WaitGroup" "counter" for each
	for id := 1; id <= 5; id++ {
		wg.Add(1)
		go worker(id, &wg)
	}

	// Block until the "WaitGroup" "counter" goes back to "0",
	// meaning all the workers notified that they are "done"
	wg.Wait()
}
```



