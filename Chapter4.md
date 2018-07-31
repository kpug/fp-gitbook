## 4장 펑터

### 4.1 Functor란 무엇인가?

Functor는 보통 펑터라고 부르며, 프로그래머 세계에서는 함수자로 불린다. 함수자에 대한 위키피디아의 사전적 설명은 아래와 같다.

> 범주론에서 함자(函子, 영어: functor 펑크터[*])는 두 범주 사이의 함수에 해당하는 구조로, 대상을 대상으로, 사상을 사상으로 대응시킨다. 함자는 작은 범주의 범주의 사상으로 볼 수 있다.
좀 더 간결하게 다듬어서 이야기 해보면 "자료 타입에서 map 구현이 가능하고, 특정 법칙을 만족한다면" 이것을 Functor라고 할 수 있다.


함수형 프로그래밍 언어들로 개발 하다가 보면 여러 map함수를 만날 수 있다. map은 사상한다는 의미로 보통 List에서는 아래와 같이 사용할 수 있다.


##### 코드5.1 List의 map 함수

````bash
scala> val list = (1 to 10).toList
list: List[Int] = List(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

scala> list.map(x => x * 3)
res0: List[Int] = List(3, 6, 9, 12, 15, 18, 21, 24, 27, 30)
````

이렇게 자료구조에 사용할 수도 있고, Option이나 Future에서도 map을 사용할 수 있다.

##### 코드5.2 Option의 map 함수

````bash
scala> val option = Option(5)
option: Option[Int] = Some(5)

scala> option.map(x => x * 5)
res4: Option[Int] = Some(25)
````

##### 코드5.3 Future의 map 함수

````bash
scala> import scala.concurrent._
import scala.concurrent._

scala> import scala.concurrent.ExecutionContext.Implicits.global
import scala.concurrent.ExecutionContext.Implicits.global

scala> future.map(x => x * 5)
res5: scala.concurrent.Future[Int] = scala.concurrent.impl.Promise$DefaultPromise@37468787

scala> res5.map(x => println(x))
25
res6: scala.concurrent.Future[Unit] = scala.concurrent.impl.Promise$DefaultPromise@330c1f61
````

위의 예제 소스들을 보면 map은 내부의 요소들만 수정하며, 타입을 변경하지는 않는다. 이를 구조의 보존(Structure-Preserving) 법칙이라 하며, 함수자를 정의하는 중요한 법칙이다.

따라서 항등 함수를 적용한다면 자기 자신이며 아래와 같은 수식이 만족된다.

````scala
x.map( a => a ) == x
````

### 4.2 Functor의 이점

Functor가 가지는 이점은 Functor의 추상을 응용하여 유용한 함수들을 발견할 수 있다. 다시 말해 map함수만으로도 추가적인 유용한 연산들을 정의할 수 있다는 것이다.

예를 들어 List안에 Tuple이 있다고 할 때, 우리는 map 연산을 통해서 이를 2개의 List의 Tuple로 분리할 수가 있다. 아래 코드를 보자.

````bash
scala> val list = List((1,2),(3,4),(4,5))
list: List[(Int, Int)] = List((1,2), (3,4), (4,5))

scala> ( list.map(e => e._1), list.map(e => e._2) )
res0: (List[Int], List[Int]) = (List(1, 3, 4),List(2, 4, 5))
````

이렇게 map함수를 이용해서 첫번째 요소의 리스트와, 두번째 요소의 리스트를 간단하게 분리할 수 있다.

이 함수가 여러분들이 알고 있는 unzip 함수 이다.

````bash
scala> list.unzip
res1: (List[Int], List[Int]) = (List(1, 3, 4),List(2, 4, 5))
````

###4.3 정리

이번 장에서 살펴본 Functor는 모노이드 보다는 훨씬 더 이해하기 쉽고 간단해보인다. 하지만 Functor를 활용하여 유용한 여러 함수를 발견할 수 있다는 사실이 중요하다. Functor에 대해서 더욱 깊게 고찰하면 결국 모나드를 만나게 된다. Functor자체 만으로도 강력하지만 펑터는 사실 빙산에 일각에 불과하다. 그 뒤에 모나드, 어플리커티브, 프리모나드 까지 엄청난 녀석들이 줄줄이 사탕으로 달려있다. 다음 장으로 넘어가서 모나드에 대해서 살펴보자.