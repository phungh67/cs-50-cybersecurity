
#mapreduce #intermediate 

This lab focuses on recreating the famous **MapReduce** (source on this paper: [What is a MapReduce](https://static.googleusercontent.com/media/research.google.com/en//archive/mapreduce-osdi04.pdf)). Materials are also borrowed (and of course inspired from the lab of MIT [GoLang lab of MIT](https://g.csail.mit.edu/6.5840-golabs-2023)). There are some components that we don't have to implement:
- A map function: to explode the data from a huge body of a book. This is done by the code, take a look in the word-count file, or something with similar name for understand how this program detects and put lines of words into map of words.
- A reduce function: in the last step where we have something like `{"apple", "1 1 1 1 1"}` this will help us to "reduce" the data to `{"apple", "5"}`.

The tasks are:
- Implement a "coordinator":
	- This is a program that distributes task to the worker. 
	- It also has knowledge about how many reduce workers should be called.
	- It just handle the task for whatever worker came to it and asked: hey, any task now?
- Implement a "Worker" skeletal:
	- There are two main tasks: Map and Reduce, worker is just simply a process that executes what ever type of task it was given.
	- While there are unbounded number of map worker (as many as the task queue for map still countable), there is a limited number of reduce worker. This defined in the `mrcoordinator.go` file (hardcoded).

The map phase should divide the intermediate keys into buckets for `nReduce` reduce tasks, where `nReduce` is the number of reduce tasks -- argument that `main/mrcoordinator.go` passes to `MakeCoordinator()`. So, each mapper needs to create `nReduce` intermediate files for consumption by the reduce tasks.

The worker implementation should put the output of the X'th reduce task in the file `mr-out-X`.

A `mr-out-X` file should contain one line per Reduce function output. The line should be generated with the Go `"%v %v"` format, called with the key and value. Have a look in `main/mrsequential.go` for the line commented "this is the correct format". The test script will fail if your implementation deviates too much from this format.

You can modify `mr/worker.go`, `mr/coordinator.go`, and `mr/rpc.go`. You can temporarily modify other files for testing, but make sure your code works with the original versions; we'll test with the original versions.

The worker should put intermediate Map output in files in the current directory, where your worker can later read them as input to Reduce tasks.

`main/mrcoordinator.go` expects `mr/coordinator.go` to implement a `Done()` method that returns true when the MapReduce job is completely finished; at that point, `mrcoordinator.go` will exit.



![[MapReduce architecture.png]]


When the job is completely finished, the worker processes should exit. A simple way to implement this is to use the return value from `call()`: if the worker fails to contact the coordinator, it can assume that the coordinator has exited because the job is done, and so the worker can terminate too. Depending on your design, you might also find it helpful to have a "please exit" pseudo-task that the coordinator can give to workers.