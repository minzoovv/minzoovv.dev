---
title: Python AsyncIO의 역사
date: 2020-02-04 01:02:95
category: python
draft: false
---

#### 부제: 아니 그 전에 Python Asynchronous programming의 역사

<p style="font-size:12px">
  <i>
      본 내용은 <b>Reactive Programming with Python</b> 책을 기반으로 번역, 정리한 글입니다. <br> 오역이 있거나 잘못된 개념이 있으면 지적 부탁드립니다 :pray:
  </i>
</p>

<hr>
<br>

2000년대 초반부터, Python을 비롯한 많은 프로그래밍 언어에서 비동기 프로그래밍을 더욱 쉽게하기 위한 framework의 발전이 계속 이루어져 왔다.

놀랍게도, Python에 대한 비동기 프로그래밍 지원은 1999년 Python 1에서 `Asyncore`라는 라이브러리로 시작되고 있었다. 그 이후에도 비동기와 관련된 다양한 Standard Library와 Asynchronous Programming을 위한 문법들이 계속해서 생겨났다.

지금부터 Python의 비동기 프로그래밍의 역사를 하나하나 살펴보도록 하자.

## 1. Asyncore

**Asyncore**는 Python 1이 release된 지 5년 후에 탄생한 비동기 지원 모듈이다.

이 모듈은 사실 비동기 소켓 핸들러를 디자인하기 위해서 만들어졌으며, callback을 기반으로 하여 핸들러를 만들거나 reactor(OS단에서 event를 기다리고 처리하는 API)에서 제공하는 `select`, `poll`을 구현하였다. Asyncore는 Python 3.6의 `asyncio`가 더 많은 기능과 함께 이를 대체하면서 deprecated 되었다.

## 2. Generator

Python 2.2에서는 **Generator**를 처음으로 지원하기 시작했다. 다들 알다시피, generator는 iterator와 달리 `on-demand`, 값을 실제로 필요로 하는 시점에 해당 값을 evaluate 한다.  
그렇기 때문에 iterator 대신 generator를 사용함으로써 많은 양의 데이터가 필요하지만, 그걸 전부 메모리에 담지 않을 수 있다. 그 때 그 때 필요할 때마다 데이터를 만들어 내기 때문이다.

물론 generator 그 자체는 비동기 프로그래밍과 큰 연관이 없다. 하지만, generator는 비동기 프로그래밍에서 매우 유용하게 쓰일 수 있는 특징을 가지고 있다. 바로 특정 부분에서 `interruption`이 가능하고, 후에 다시 해당 위치의 context에 돌아올 수 있다는 것이다.

Generator의 이런 특징은 **Operating System이 interrupt handler를 통해 context switching을 하는 구조**와 매우 비슷하다.

## 3. Twisted

이 Generator를 사용하여 비동기 프로그래밍을 쉽게 만든 모듈이 바로 **Twisted**이다.

Twisted는 Asyncore처럼 handler를 위해 callback을 사용하였지만, Asyncore와는 대조되는 2개의 `main improvement`가 있었다.

- transport implementation과 protocol implementation의 분리.

  transport layer에서는 네트워크를 통해 message를 전달하는 것에 대해 정의하고, protocol layer에서는 그 전달되는 message들을 어떻게 처리하는지에 대해 정의하는데, 이 두 부분의 구현을 분리하였다.

  ```python

    from twisted.internet import protocol, reactor, endpoints

    class Echo(protocol.Protocol):
        def dataReceived(self, data):
            self.transport.write(data)

    class EchoFactory(protocol.Factory):
        def buildProtocol(self, addr):
            return Echo()

    endpoints.serverFromString(reactor, "tcp:1234").listen(EchoFactory())
    reactor.run()

  ```

  위의 코드는 twisted 공식 사이트에 들어가면 제일 먼저 보이는 custom network application을 구현한 quick start 코드이다.

  보시다시피 데이터에 대한 처리를 구현하는 `protocol` (여기서는 echo)과, 데이터를 주고받는 `transport` (여기서는 tcp) layer가 분리 되어있는 것을 볼 수 있다. 동일한 transport가 여러개의 protocol을 사용할 수 있도록 그 확장성을 높였다.

* generator를 이용하여 async programming을 sync 코드처럼 보이게 한 것.

  하나의 비동기 동작을 많은 callback을 사용하는 대신에 generator function 하나로 끝내면서, 코드에 대한 가독성을 매우 높였다.

Twisted에 대해 더 알고 싶다면, 이 [링크](http://wiki.tuestudy.org/aosabook/materials/twisted)
를 참고하면 좋을 것이다.

## 4. Generator expression (), 그리고 send() method

2004, 2005년 Python에서의 위의 두 가지의 지원이 Generator에 엄청난 발전을 가져왔다.

아래 두 링크는 두 개의 문법에 대한 PEP(Python Enhance Proposal)이다.

[PEP 289: Generator Expression](https://www.python.org/dev/peps/pep-0289/)  
[PEP 342: Coroutines via Enhnaced Generators](https://www.python.org/dev/peps/pep-0342/#new-generator-method-send-value)

일단, Generator expression `()`은 list comprehension 처럼 generator를 사용할 수 있도록 해주었다. 또한, `send()` 함수는 Generator가 다시 본래 함수로 돌아올때, caller로 부터 값을 넘겨 받을 수 있도록 만들어 주었다. 즉, generator가 caller와 `의사소통`을 할 수 있게 된 것이다.

## 5. Gevent

2008년 7월, 또 다른 새로운 비동기 프레임워크인 **Gevent**가 탄생한다.

Gevent는 앞서 위에서 언급한 Twisted의 대체재로서, 순수 파이썬으로 구현된 event loop를 사용하는 Twisted와는 다르게 `libev` 또는 `libuv`를 event loop를 사용한다.

`libev`와 `libuv` 모두 Cpython으로 구현되어 있고, 특히 `libuv`는 nodejs에서 사용하기 위해 만들어진 고성능 멀티 플랫폼 비동기 IO 라이브러리로서 그 안정성과 속도가 보장된 이벤트 루프이다. 따라서 기존의 Python으로만 구현된 event loop에 비해 좋은 성능을 가진다.

Gevent 1.0 버전 release가 2013년에 이루어졌다.

## 6. Future

2011년, 드디어 Python의 비동기 프로그래밍의 한 획을 긋고, Python을 비동기 프로그래밍을 고려한 최첨단 언어로 만든 최종적인 진화가 이 때에 일어난다. 바로 **Future**의 등장이다.

Future는 2011년 Python 3.2 release와 함께 처음으로 등장한다. Future는 현재는 접근할 수 없지만, `미래에 어느 시점에는` 해당 값에 접근할 수 있다는 것을 표현하는 방법이며, 그러한 객체이다.  
또한, sub-generator로의 Delegation도 2012년, python 3.3 release와 함께 추가 되었다.

[PEP 3148: futures - execute computations asynchronously](https://www.python.org/dev/peps/pep-3148/)  
[PEP 380: Syntax for Delegating to a Subgenerator](https://www.python.org/dev/peps/pep-0236/)

## 7. 드디어 Asyncio!

2014년, **Asyncio** 모듈이 Python 3.4에 Standard Library로서 등장한다. 이것은 Python을 다른 추가적인 서드파티 라이브러리 없이 비동기에 `준비된` 언어로 만들어 주었다.  
Asyncio는 기존에 존재하는 여러 비동기 프레임워크들의 영감을 그대로 가져왔고, 그 프레임워크들이 가지는 최고의 아이디어들만 Python에서 사용할 수 있도록 하였다.

역시 새로운 라이브러리인 만큼, 기존에 built-in 으로 존재했던 `Asyncore` 와는 비교되는 주요 improvement들이 존재한다.

- 앞서 말했던 Twisted에서 transport와 protocol layer를 분리하는 개념이 여기서 다시 사용되었다.
- Event loop가 OS에 따라서 reactor나 proactor에서 실행될 수 있고, Generator를 기반으로 하는 `Coroutine`이 비동기 코드를 동기적으로 보이고 쓰일 수 있도록 사용되었다.

## 8. async 와 await

Python 비동기 프로그래밍의 최종적인 발전은 2015년에 **async**와 **await** 문법의 추가로 이루어진다.

[PEP 492: Coroutines with async and await syntax](https://www.python.org/dev/peps/pep-0492/)

async와 await은 generator의 yield keyword의 `syntactic sugar`로서, async와 await 키워드를 붙임으로써 비동기 코드와 coroutine들을 더욱 가시적으로 보이게 하고, 비동기 프로그래밍 뿐만 아니라 다른 목적으로도 사용될 수 있는 generator와 구분 될 수 있도록 하였다.

이러한 두 키워드들은 비동기 코드를 작성하고 사용하는 것 자체를 기존에 사용하던 동기 코드를 사용하고 쓰는 것만큼 쉽도록 해주었다.

<br>
<hr>

### 번역을 마치며...

지금까지 Python의 비동기 프로그래밍의 역사에 대해서 간략히 알아보았다.  
Asyncio를 다뤄보면서 한번쯤은 정리 해보고 싶다 생각했던 주제였는데, 이렇게 정리할 수 있어서 좋았던 것 같다. 계속해서 Python 3.6, 3.7 Asyncio에 대해서 Deep Dive해 볼 예정이다.
