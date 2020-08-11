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

### Example 2 - Transforming the results using map & flatMap

Like we said, callback `onComplete` method would return only `Unit` and if we need to process and transform the future value, we could use map as below,

```
val f1 : Future[Int] = Future { 5 }
val f2 : Future[Int] = f1.map(i => i * 2)
``` 

Here in this example, we have a `f1` which is a Future placeholder of type `Future[Int]` and we are transforming the result of f1 to another Future variable `f2` of type `Future[Int]` using map.

Not necessary that the return type should be the same as original one, we could map to completely different type as below,

```
val f1 : Future[Int] = Future { 5 }
val f2 : Future[String] = f1.map(i => s"Hello $i")
```

Here we have converted from `Future[Int]` to `Future[String]`. You could do the same using flatMap as well. Before that lets look at the map and flatMap function signature,

```
def map[S](f : scala.Function1[T, S])(implicit executor : scala.concurrent.ExecutionContext) : scala.concurrent.Future[S] = {...}

def flatMap[S](f : scala.Function1[T, scala.concurrent.Future[S]])(implicit executor : scala.concurrent.ExecutionContext) : scala.concurrent.Future[S] = {...}
```

The only difference is flatMap has to return another `Future` so we could do the same using below,

```
val f1 : Future[Int] = Future { 5 }
val f2 : Future[Int] = f1.flatMap(i => Future {i * 2})
val f3 : Future[Int] = f1.flatMap(i => Future.successful(i * 2))  // another way
val f4 : Future[String] = f1.flatMap(i => Future { s"Hello $i"})  // to transform to another type
```

## Future Chaining

This is also called future chaining, where we use the result of one future to be the input for another future in a sequential flow.

Refer this [link](/chaining.md) for more details on it.

## Future home page

Refer this [link](/README.md) to go to Future home page

## Scala blog home

Visit [home page](https://nvenkatp.github.io/scala)