## Future Chaining

### Future Chaining using flatMap

As we saw in the last example, future chaining is nothing but invoking multiple future functions in a sequential flow where we use the output of one to the input for other. Lets look at an example,

```
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration._
import scala.concurrent.{Await, Future}

object FutureChaining extends App {

  def cleanString(str: String): Future[String] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing cleanString")
    str.replaceAll("[^a-zA-Z]", "")
  }

  def getLength(str: String): Future[Int] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing getLength")
    str.length
  }

  def isEven(len: Int): Future[Boolean] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing isEven")
    if (len % 2 == 0) true else false
  }

  //chaining using flatMap
  val futureResult = cleanString("abc123").flatMap(getLength(_)).flatMap(isEven(_))
  val result = Await.result(futureResult, 100 millisecond)

  println(s"Cleaned string length is even : ${result}")

}

**Output**
Thread ForkJoinPool-1-worker-29 executing cleanString
Thread ForkJoinPool-1-worker-11 executing getLength
Thread ForkJoinPool-1-worker-11 executing isEven
Cleaned string length is even : false

```
As you can see here, we have defined 3 methods 

- cleanString -> to remove characters other than alphabets
- getLength -> to fetch the length of a given string
- isEven -> to see if the string length is even or odd.

And we use flatMap to chain the futures and put everything into `futureResult` which is of type `Future[Boolean]` and we do wait for this to get the final result. 
We generally avoid using `map` to perform future chaining because it creates nested future which is not we want. For example if we use like this in the above program,

```
val futureResult = cleanString("abc123").map(getLength(_))
```
Can you guess the type of this variable `futureResult` ? Yes it is `Future[Future[Int]]` and we cannot use this directly to invoke the next function `isEven`.

### Future chaining using for comprehension or syntatic sugar way

One more way of achieving future chaining is the below one, just shown the changing snippet alone here.

```
val futureResult = for {
    cleanedStr <- cleanString("abcd123")
    length <- getLength(cleanedStr)
    isEven <- isEven(length)
  } yield isEven
```
Here we are using `for yield` to do future chaining in sequential flow and returning the final value `isEven` via yield. The type of `futureResult` is same as `Future[Boolean]` as used with `flatMap`

### Future chaining with exception

Whenever there is an exception happens, it stops there and skips remaining future functions if any. Lets look at it with a complete example,

```
import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.duration._
import scala.concurrent.{Await, Future}
import scala.util.{Failure, Success, Try}

object FutureChaining extends App {

  def cleanString(str: String): Future[String] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing cleanString")
    str.replaceAll("[^a-zA-Z]", "")
  }

  def getLength(str: String): Future[Int] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing getLength")
    if (str.isEmpty) {
      throw new IllegalArgumentException("Invalid cleaned string")
    } else {
      str.length
    }
  }

  def isEven(len: Int): Future[Boolean] = Future {
    println(s"Thread ${Thread.currentThread().getName} executing isEven")
    if (len % 2 == 0) true else false
  }

  val futureResult2 = for {
    cleanedStr <- cleanString("1234")
    length <- getLength(cleanedStr)
    isEven <- isEven(length)
  } yield isEven

  Try {
    Await.result(futureResult2, 100 millisecond)
  } match {
    case Success(result) => println(s"Cleaned string length is even : ${result}")
    case Failure(ex) => println(s"Exception occurred ${ex.getMessage}")
  }

}

**Output**
Thread ForkJoinPool-1-worker-29 executing cleanString
Thread ForkJoinPool-1-worker-11 executing getLength
Exception occurred Invalid cleaned string

```

This is the same example shown above except here in `getLength` function we are throwing `IllegalArgumentException` when the input string is empty. Because we are passing string `1234` to the `cleanString` function, the output of which is empty. When this empty string is passed to `getLength` function, it throws error and skips calls `isEven` method in this case. We could catch the exception in `Try` block as we discussed before.

## Scala blog home

Visit [home page](https://nvenkatp.github.io/scala)