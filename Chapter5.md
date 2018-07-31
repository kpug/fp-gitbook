## 5장 모나드

### 5.1 Monad란 무엇인가?

Monad도 Functor와 마찬가지로 몇가지의 구현과 법칙을 통해 정의해 볼 수 있다. 하지만 Functor 보다 훨씬 더 강력한 이점들을 얻을 수 있다. 일반적으로 자료 타입에서 unit, flatMap 구현이 가능하고, 몇가지 법칙(Law)을 만족한다면 우리는 이것을 Monad라 할 수 있다. Monad란 “flatMap을 구현하는 자료형식”이라는 관점에서 볼때, flatMap의 추상화라고 할 수 있다. Monad 역시 여러 함수형 라이브러리드에서 추출할 수 있는 여러 추상들 중 하나이다.

#### 5.1.1  flatMap 추상화

Functor에서와 마찬가지로, 우리가 흔히 사용하는 Option 과 List를 통해 flatMap을 구현할 수 있다.

````scala
def flatMap[A, B](oa: Option[A])(fb: A => Option[B]): Option[B] = oa flatMap fb
def flatMap[A, B](la: List[A])(fb: A => List[B]): List[B] = la flatMap fb
````

컨테이너 타입(Option, List)을 M으로 일반화 하면, 다음과 같다.

````scala
def flatMap[A, B](ma: M[A])(fb: A => M[B]): M[B]
````

즉, flatMap은 컨테이너 타입 M의 내부 값을 꺼내서 fa함수를 적용해서 반환값을 돌려준다.

얼핏 보면 map과 동일해 보이지만, flatMap은 두번째 파라미터가 값을 받아서 컨테이너 타입을 반환하는 함수이다.M[M[A]]이 반환 될 것으로 예상되지만, 실제로 평평(flat)하게 처리되어 M[A]으로 반환된다는 차이점이 있다.

````scala
def oneToSizeList(i: Int) = (1 to i toList) map (_.toString) 
defined function oneToSizeList

List(1,2,3).flatMap(oneToSizeList) 
res1: List[String] = List("1", "1", "2", "1", "2", "3")

// List[Int] => List[String] (O)
// List[Int] => List[List[String] (X) 
// result: List[List[Int]] = List(List("1"), List("1", "2"), List("1, "2", "3")) 
````

#### 5.1.2 unit

Monad에서는 값을 받아서 해당 컨테이너 타입에 그 값을 담아서 돌려 주는 unit 함수가 필요하다. 스칼라에서는 일반적으로 생성자나 case class 의 apply 메소드로 구현한다.

````scala
def apply[A](x: A): Option[A] // Option의 apply
def apply[A](xs: A*): List[A] // List의 apply
````

컨테이너 타입(Option, List)을 M로 일반화 하고, 이름을 unit으로 변경하면, 다음과 같다.

````scala
def unit[A](a: => A): M[A]
````

unit은 인자로 받은 값을 감싼 컨테이너 타입으로 반환한다.



#### 5.1.3  법칙(Law)

* 결합법칙

Monad에서 결합법칙이란, 연속된 두개의 flatMap 연산에서  첫번째 연산을 먼저 평가하고, 그 결과를 두번째 연산에 적용한 것과 두번째 연산을 평가하는 함수를 첫번째 연산에 적용한 결과는 동일해야 한다.

````scala
flatMap(flatMap(m)(f))(g) == flatMap(m)(x => flatMap(f(x))(g))
````

flatMap을 이항연산으로 표현할 경우 다음과 같이 표시 할 수 있고,  

````scala
(m flatMap f) flatMap g = m flatMap (x => f(x) flatMap g))
````

이것은 덧셈의 결합법칙과 동일하다.

````scala
(a + b) + c = a + (b + c)
````

*  항등법칙

첫번째 파라미터에 unit 적용(왼쪽 항등)

````scala
flatMap(unit(x))(f) == f(x) 
````

두번째 파라미터에 unit 적용(오른쪽 항등)

````scala
flatMap(m)(unit) == m
````

첫번째 파라미터에 컨테이너 타입 m을 적용하고,  두번째 파라미터에 unit을 적용했을 경우, m과 동일해야 한다. 

### 5.2. Monad의 이점

#### 5.2.1 Monad는 Functor의 확장

unit과 flatMap을 이용해 다음과 같이 Functor의 map을 구현할 수 있다.

````scala
def map[A, B] (ma: F[A])(f: A => B): F[B] = flatMap(ma)(a => unit(f(a)))
````

Moand는 Functor의 map 구현을 제공할 수 있으므로 Functor를 확장할 수 있다. 그러므로 모든 Monad는 Functor라고 할 수 있지만, 모든 Functor가 Monad인 것은 아니다.

#### 5.2.2 unit과 flatMap을 용해 이미 구현된 유용한 함수들

Monad가 가지는 이점 중 하나는 Functor와 마찬가지로, unit과 flatMap의 추상을 응용하여 유용한 함수들을 발견할 수 있다. 다시 말해 unit과 flatMap 함수만으로도 추가적인 유용한 연산들을 정의할 수 있다는 것이다.

````scala
def map2[A,B,C](fa: F[A], fb: F[B])(f: (A,B) => C): F[C] =
  flatMap(fa)(a => map(fb)(b => f(a,b)))

def sequence[A](lma: List[F[A]]): F[List[A]] =
  lma.foldRight(unit(List[A]()))( (fa, fb) => map2(fa, fb)( (a, b) => a :: b))

def traverse[A, B](la: List[A])(f: A => F[B]): F[List[B]] =
  la.foldRight(unit(List[B]()))( (a, fb) => map2(f(a), fb)( (a, b) => a :: b))

def replicateM[A](n: Int, ma: F[A]): F[List[A]] = sequence(List.fill(n)(ma))
````


일단 한번만 정의해 두면 코드의 중복을 크게 줄일 수 있는 유용한 연산을 많이 만들어 낼 수 있다. 더 다양한 함수들은 스칼라 함수형 프로그래밍을 위한 라이브러리인 cats scala library(Monad trait)에서 확인 할 수 있다.

#### 5.2.3 for comprehension 

* map과 flatMap을 이용한 구구단

````scala
(2 to 9).flatMap( i => (1 to 9).map( j => s"$i*$j=${i*j}"))
````

* for comprehension을 이용한 구구단

````scala
for {
  i <- 2 to 9
  j <- 1 to 9
} yield s"$i*$j=${i*j}"
````

for comprehension는 sugar syntax로써 복잡한 코드를 개발자 쉽게 사용할 수 있도록, 그리고 가독성을 높여 주기 위한 문법이다. 여기서는 flatMap과 map의 코드를 변환하는 예제를 보여줬지만, 실제로는 좀 더 다양한 방법으로 for comprehension 사용할 수 있다.

### 5.3 정리

이번 장을 통해서 함수형 프로그래밍에서 가장 무시무시한 존재 모나드에 대해서 살펴보았다. 모나드 자체는 너무 추상적인데다가 감에 의존하여 판단하는 경우가 많아서 "이게 모나드 인가" 하는 의심만 드는 경우가 대부분이다. 그리고 너무나도 다행인 것은 모나드를 모른다고 해도 살아가는데 아무런 어려움이 없다는 사실이다. 이 사실에 안도하면서 이 글을 마친다.