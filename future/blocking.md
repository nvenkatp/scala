## Scala Future

Scala Future is very similar to the one used in Java. This is used primarily for non-blocking use cases where we don't want to block the main thread or thread which invokes future call. A future is nothing but a placeholder where we might expect result would be placed at some point in time in future.

Refer this [link](/nonblocking.md) to go for non-blocking examples

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

### Exception handling using recover

You could handle exceptions in a different way using `recover` block like below,

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
  }.recover {
    case ex: ArithmeticException => 
      println("Thread executing recover block - "+Thread.currentThread().getName)
      0
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

Thread executing future task - ForkJoinPool-1-worker-29
Thread executing recover block - ForkJoinPool-1-worker-29
Main Thread - main
Future completed - 0

```

Here we have added a `recover` block for the future variable `f` and we handled `ArithmeticException` alone and returning 0 when it happens. Because we already recovered from an error, we don't see the `Try - Failure` block being executed in this case. Instead we see it is executing the code block in `Try - Success` case.

**NOTE** 
1. In case if we throw some other exception say for example `IllegalStateException` and we only handled `ArithmeticException` in the recover block, then you would have seen the `Try - Failure` block got executed.
2. If you want to handle all exceptions, you could use generic `Exception` in the recover block.

There is another variant called `recoverWith` and the only difference is, we should return some `Future[S]` from the `recoverWith` block as opposed to returning plain value in `recover` block. So with `recoverWith`, you have to use like this,

```
val f = Future {
    println("Thread executing future task - "+Thread.currentThread().getName)
    Thread.sleep(500) //Assume time consuming task
    1 / 0             //throwing exception
  }.recover {
    case ex: ArithmeticException => 
      println("Thread executing recover block - "+Thread.currentThread().getName)
      Future.successful(0) // or Future { 0 }
  }
```

## Future Nonblocking

Refer this [link](/nonblocking.md) to go for non-blocking examples

## Future home page

Refer this [link](/README.md) to go to Future home page

## Scala blog home

Visit [home page](https://nvenkatp.github.io/scala)