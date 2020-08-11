## Future Sequence

When we have multiple futures, instead of waiting for all of them to complete separately, we could convert those into a Future Sequence and wait for this sequence to complete,

```
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.util.{Failure, Success}

object FutureSequence extends App {

  /*
  Future sequence is useful when you have to reduce number of futures into a single future.
  Moreover these futures will be non-blocking and run in parallel which also imply that
  the order of the futures is not guaranteed as they can be interleaved.
   */

  def fInt: Future[Int] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing fInt method")
    Thread.sleep(2000)
    2
  }

  def fBool: Future[Boolean] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing fBool method")
    Thread.sleep(3000)
    false
  }

  def fVoid: Future[Unit] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing fVoid method")
    Thread.sleep(1000)
  }

  println("\nCombine future operations into a List")
  val futureList: List[Future[Any]] = List(fInt, fBool, fVoid)

  println(s"\nCall Future.sequence to run the future operations in parallel")
  val futureSequenceResults = Future.sequence(futureList)
  futureSequenceResults.onComplete {
    case Success(results) => println(s"Results $results")
    case Failure(e)       => println(s"Error processing future operations, error = ${e.getMessage}")
  }

  Thread.sleep(4000)
}

**Output**

Combine future operations into a List
Thread ForkJoinPool-1-worker-29 executing fInt method

Call Future.sequence to run the future operations in parallel
Thread ForkJoinPool-1-worker-11 executing fBool method
Thread ForkJoinPool-1-worker-25 executing fVoid method
Results List(2, false, ())

```

We have 3 future methods here each of which returns different type. We then create a `Future.sequence` with the list of individual futures. We then wait for this `futureSequenceResults` to complete as like we do for individual future.

**NOTE**

Whenever there is any exception to any of the futures, it would finally execute the `onComplete` callback `Failure` block of code. 

## Future Traverse

Future traverse is similar to Future sequence with added benefit of applying a function over the future results. Refer the below example,

```
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.Future
import scala.util.{Failure, Success}

object FutureTraverse extends App {

  def fOptInt(opt: String): Future[Option[Int]] = Future {
    if (opt == "valid") Some(10) else None
  }

  println("Combine future operations into a List")
  val futureList = List(
          fOptInt("valid"),
          fOptInt("valid"),
          fOptInt("invalid")
  )

  println(s"Call Future.sequence to run the future operations in parallel")
  val futureTraverseResults = Future.traverse(futureList) { futureSomeQty =>
    futureSomeQty.map(someQty => someQty.getOrElse(0))
  }
  futureTraverseResults.onComplete {
    case Success(results) => println(s"Results $results") //List(10, 10, 0)
    case Failure(e)       => println(s"Error processing future operations, error = ${e.getMessage}")
  }
  //NOTE in case if we use sequence, the result would be List(Some(10), Some(10), None)

  Thread.sleep(2000)
}

**Output**
Combine future operations into a List
Call Future.sequence to run the future operations in parallel
Results List(10, 10, 0)

```

### Future firstCompletedOf

In case we want to perform multiple parallel tasks and we are interested in any of the tasks that got completed first, we could use this `Future.firstCompletedOf`.

## Future home page

Refer this [link](/README.md) to go to Future home page

## Scala blog home

Visit [home page](https://nvenkatp.github.io/scala)