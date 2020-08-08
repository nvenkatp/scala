## Future without blocking

We could use callback functions to do the task once we get the result from the future placeholder. Doing so, the main thread is not blocked and it could perform some other tasks. At the same time, if we do exit the main program before the other thread completes, you won't see the result from the future variable.

### Example 1 - using callback functions

```
import scala.concurrent.Future
import scala.util.{Failure, Success}
import scala.concurrent.ExecutionContext.Implicits.global

object Future2 extends App {

  val f = Future {
    println("Thread executing future task - "+Thread.currentThread().getName)
    Thread.sleep(200)
    1 + 1
  }

  //Example of non-blocking
  f.onComplete {
    case Success(value) => println(s"Thread - ${Thread.currentThread().getName} - Got Answer = $value")
    case Failure(e) => e.printStackTrace()
  }

  // Main thread doing some other task instead of blocking
  println("Main thread still running !!!")

  // Finally Main thread waits for sometime for the other thread to complete.
  for (i <- 1 to 5) {
    println("Main thread waiting for result...")
    Thread.sleep(100)
  }

  println("Main thread completed")
}

**Output**

Thread executing future task - ForkJoinPool-1-worker-29
Main thread still running !!!
Main thread waiting for result...
Main thread waiting for result...
Main thread waiting for result...
Thread - ForkJoinPool-1-worker-29 - Got Answer = 2
Main thread waiting for result...
Main thread waiting for result...
Main thread completed

```

The `onComplete` method is a callback one which will execute once the future variable `f` completes. Note that the main thread can still perform other tasks after `f.onComplete` method. At last the main thread has to wait for sometime so that the other thread complete and the callback functions execute.

**NOTE** 
1. In case if the main thread doesn't wait for the other thread (assume we don't have that for block waiting for `100 milliseconds` for 5 times), you will get the output as like below,

```
Thread executing future task - ForkJoinPool-1-worker-29
Main thread still running !!!
Main thread completed
```
2. Callback `onComplete` would execute no matter what future completed successfully or errored out. To see if is success or failure, you have to use matching case blocks `Success` or `Failure`.
3. One problem with this callback function is that it won't return anything and so its return type is `Unit`. Hence we should use this callback when we don't have to return anything.