## Scala Future

Scala Future is very similar to the one used in Java. This is used primarily for non-blocking use cases where we don't want to block the main thread or thread which invokes future call. A future is a placeholder where we might expect result would be placed at some point in time in future.

### Blocking Example

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

This is a very basic example where the main thread invokes a future task and waits for 1 second to get response from the future placeholder 'f'. When you use `scala.concurrent.ExecutionContext.Implicits.global`, it would use default ForkJoin thread pool. The future task would get executed by a thread in this ForkJoin thread pool. Two threads involved here in this simple program, main thread and another thread from ForkJoinPool.

**NOTE** You won't be seeing same ForkJoinPool thread name as shown in the output above.

If you notice, the main thread after delegating the work to other thread blocks for the latter to complete. Usually we don't follow this approach. This is just for example purpose to begin with.
