## Scala Future

Scala Future is very similar to the one used in Java. This is used primarily for non-blocking use cases where we don't want to block the main thread or thread which invokes future call. A future is nothing but a placeholder where we might expect result would be placed at some point in time in future.

### Blocking Example 1

```
import scala.concurrent.{Await, Future}
import scala.util.{Failure, Success, Try}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration._

object Future extends App {

  val f = Future {
    println("Thread executing future task - "+Thread.currentThread().getName)
    Thread.sleep(500) //Assume time consuming task
    1 + 1
  }
  // Main Thread
  println("Main Thread - "+Thread.currentThread().getName)

  // Main Thread blocks for 1 second to get the Future task result
  Try {
    Await.result(f, 1 second)
  }
  match {
    case Success(res) => println(s"Future completed - ${res}")
    case Failure(ex) => {
      println("Exception occured :", ex)
      System.exit(1)
    }
  }
}

**Output**

Main Thread - main
Thread executing future task - ForkJoinPool-1-worker-29
Future completed - 2

```

This is a very basic example where the main thread invokes a future task and waits for `1 second` to get response from the future placeholder `f`. When you use `scala.concurrent.ExecutionContext.Implicits.global`, it would use default ForkJoin thread pool. The future task would get executed by a thread in this ForkJoin thread pool. Two threads involved here in this simple program, main thread and another thread from ForkJoinPool.

**NOTE** You won't be seeing same ForkJoinPool thread name as shown in the output above.

If you notice, the main thread after delegating the work to other thread blocks for the latter to complete. Usually we don't follow this approach. This is just for the example purpose to begin with.

### Blocking Example 2 - Future task times out

```
import scala.concurrent.{Await, Future}
import scala.util.{Failure, Success, Try}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration._

object Future extends App {

  val f = Future {
    println("Thread executing future task - "+Thread.currentThread().getName)
    Thread.sleep(1500) //Assume time consuming task
    1 + 1
  }
  // Main Thread
  println("Main Thread - "+Thread.currentThread().getName)

  // Main Thread blocks for 1 second to get the Future task result
  Try {
    Await.result(f, 1 second)
  }
  match {
    case Success(res) => println(s"Future completed - ${res}")
    case Failure(ex) => {
      println("Exception occured :", ex)
      System.exit(1)
    }
  }
}

**Output**

Main Thread - main
Thread executing future task - ForkJoinPool-1-worker-29
(Exception occured :,java.util.concurrent.TimeoutException: Futures timed out after [1 second])

```

This is very similar to the first example except here the future task takes `1.5 seconds` to complete whereas the main thread waits for just `1 second`. This is to show how to handle TimeoutException. The code under `Failure` case for `Try` block would get executed here.

### Blocking Example 3 - Exception with future task

```
import scala.concurrent.{Await, Future}
import scala.util.{Failure, Success, Try}
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration._

object Future extends App {

  val f = Future {
    println("Thread executing future task - "+Thread.currentThread().getName)
    Thread.sleep(500) //Assume time consuming task
    1 / 0             //throwing exception
  }
  // Main Thread
  println("Main Thread - "+Thread.currentThread().getName)

  // Main Thread blocks for 1 second to get the Future task result
  Try {
    Await.result(f, 1 second)
  }
  match {
    case Success(res) => println(s"Future completed - ${res}")
    case Failure(ex) => {
      println("Exception occured :", ex)
      System.exit(1)
    }
  }
}

**Output**

Main Thread - main
Thread executing future task - ForkJoinPool-1-worker-29
(Exception occured :,java.lang.ArithmeticException: / by zero)

```

All exceptions that occured when executing the future task can be caught by this `Try` case `Failure` block. This example is to show how `ArithmeticException`was thrown and caught. We could also use regular `try/catch` block instead of this `Try` case block. Use any one form to capture all exceptions.