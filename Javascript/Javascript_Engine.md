# Javascript_Engine

회사에서 `SWT` 에 `CEF` 를 넣어서 통신시키도록 하는 작업을 했다. 그러다 보니 `JS엔진`에 대해서 알고 싶어졌다. 
<br/>

그러다 찾게된 곳을 토대로 엔진들의 공통적인 파이프라인에 대해서 알아보려고 한다.
<br/>

## JS엔진이란?

JS코드를 실행하는 **프로그램(가령 브라우저) 또는 인터프리터** 를 말한다.
<br/>

### V8

제일 유명하고 사람들이 많이 사용하는 크롬에 들어가있는 `V8` 이 있다. 현재 `Electron, Nodejs` 에서도 사용이 되고 있고 `CEF` 의 안에도 들어있다.
<br/>

### SpiderMonkey

**최초의 자바스크립트 엔진** 으로, JS의 창시자인 브랜던 아이크가 넷스케이프 브라우저를 위해 개발이 되었다. 지금은 `Mozilla` 재단에서 관리하며, `FireFox` 에서 사용되는 엔진이다.
<br/>

### Chakra

**마이크로소프트가 개발한 엔진** 이며, `Edge` 브라우저에 사용되고 있고 앞으로는 **V8** 로 바꾼다는 말이 있다. (현재 시점으로 앞으로는 V8엔진을 사용한다고 올라왔다.)
<br/>

`Chakra` 엔진의 중요 부분은 `Chakra Core` 라는 오픈 소스로 구성되어있다.
<br/>

### Javascript Core

애플에서 개발한 JavaScript Core는 처음에 **WebKit 프레임워크** 를 위해 개발. 최근에는 `Safari`와 `React Native App`에서 사용된다고 한다. (RN에서는 웹뷰에서 사용한다는 것 같다.)
<br/>

## 자바스크립트 엔진 파이프라인

소스코드를 기계어로 만드는 과정에 대해서 알아보려고 한다. 
<br/>

1.  자바스크립트 **소스를 파싱해서 AST(추상 구문 트리)로 만든다.**
2. **AST를 토대로 인터프리터는 바이트코드로 만들어준다.**

코드를 더 빨리 실행하기 위해서, 바이트코드는 프로파일링 된 데이터와 함께 `optimizing compiler` 로 보내지고 여기서는 **프로파일된 데이터를 기반으로 하여 최적의 기계어를 생성** 하게 된다.

> 바이트 코드  + 프로파일된 데이터 => 최적의 기계어

그러나 정확하지 않은 결과(쉽게 자주 사용하지 않는다고 하자)가 나왔다면 다시 `deoptimizes`하여 바이트코드로 되돌린다.
<br/>

## 인터프리터 / 컴파일러 파이프라인

위에서 말한것이 기본적인 파이프라인이다. **바이트코드로 만들고 자주 사용되고 빨리 실행해야하는 것들은 최적화를 시킨다.**
<br/>

- 인터프리터 : 최적화 되지 않은 바이트코드를 빠르게 생성
- 최적화 컴파일러 : 매우 최적화된 기계어 코드를 시간을 들여서 생성. (인터프리터보다 느리다는 말이된다.)

바이트코드는 **중간 언어** 이다. 인터프리터모드라면 한줄 한줄 읽으면서 해석을 할 것이며, JIT모드라면 컴파일을 하고 실행할 것이다.
<br/>

> 바이트코드(Bytecode, portable code, p-code)는 특정 하드웨어가 아닌 가상 컴퓨터에서 돌아가는 실행 프로그램을 위한 이진 표현법이다. 하드웨어가 아닌 소프트웨어에 의해 처리되기 때문에, 보통 기계어보다 더 추상적이다.
<br/>

> JIT 컴파일(just-in-time compilation) 또는 동적 번역(dynamic translation)은 프로그램을 실제 실행하는 시점에 기계어로 번역하는 컴파일 기법이다. 이 기법은 프로그램의 실행 속도를 빠르게 하기 위해 사용된다.
<br/>

## 다시 V8

제일 공통의 파이프라인과 비슷하다. `Ignition` 이라고 불리는 인터프리터를 통해서 코드를 **점화** 하여 바이트 코드를 생성 및 실행을 한다. 
<br/>

바이트코드가 실행될 때 인터프리터는 프로파일링 데이터를 수집하여 나중에 실행 속도를 빠르게 한다.(위에서 말한 최적화된 코드를 생성하는데 사용됨)
<br/>

특정 함수를 자주 사용하게 되면 뜨거워지는 이를 바이트코드와 프로파일링된 데이터를 **TurboFan** 이라고 불리는 최적화 컴파일러로 보내서 식혀준다. 최적화 컴파일러는 2개의 데이터를 통해서 최적화된 코드를 뽑아낸다.
<br/>

최근 엔진들은 최적화를 적용하는데 있어서 **JITC이 아닌 Adative Compilation을 적용한다고 한다.** 이는 반복되는 정도에 따라 서로 다른 최적화를 적용한다는 것이다.
<br/>

## SpiderMonkey   

`SpiderMonkey` 는 최적화된 컴파일러가 **2개** 이다. `Baseline Complier` 을 토대로 약간 최적화된 코드를 만들어내고 코드를 실행하는 동안에 수집된 프로파일링 데이터와 합쳐져서 `IonMonkey` 라는 최적화 컴파일러로 보내져서 고도로 최적화 된 코드를 만들어 낸다.
<br/>

만약에 추측성 최적화가 실패하게 되면, **IonMonkey는 이를 Baseline 코드로 되돌리게 된다.** 그렇다면 바이트코드까지를 다시 `decompile` 이 안된다는 말이 되는 듯 하다.
<br/>

## Chakra Core

`Chakra Core` 역시 `spider monkey` 와 동일하게 **2개의 컴파일러를 가진다.** 
<br/>

`SimpleJIT` 를 통해서 약간 최적화된 코드를 만들어내고 `FullJIT` 를 통해서 프로파일링 데이터와 함께 더욱 고도로 최적화 된 코드를 생성한다. 
<br/>

> 어째 얘는 decompile이 이루어지면 바이트코드로 들어간다.(최적화된 것을 풀수도 있다는 말)
<br/>

## Javascript Core

`JSC` 는 **3개의 컴파일러를 가지고 있다.** `LLInt(Low-Level Interpreter)` 라는 인터프리터 `Baseline` 컴파일러로 제일 어림짐작해서 약간 최적화된 코드를 만들어내고, 이후에는` DFG(Data Flow Graph)과 FTL(Faster Than Light)` 라는 최적화 컴파일러를 사용한다.
<br/>

## 처리방법의 차이점과 공통점

왜 많은 최적화 컴파일러를 가지고 있을까?? 
<br/>

바로 `trade-off` 때문이라고 한다.
<br/>

인터프리터는 바이트코드를 빠르게 생성할 수 있지만 효율적인 코드는 아니다. 반대로 최적화 컴파일러는 시간이 조금 더 걸리지만 훨씬 효율적인 기계 코드를 생성한다. 
<br/>

따라서, 어떤 엔진은 여러 개의 최적화 컴파일러를 선택함으로써 복잡해지는 비용을 감수하고 이러한 인터프리터와 컴파일러 사이의 균형을 필요에 따라 세부적으로 제어할 수 있도록 한 것이다. 

결국, 자바스크립트 엔진마다 구체적인 최적화 과정은 차이가 있으나, 파서와 인터프리터/컴파일러가 포함된 동일한 아키텍쳐로 구성이 된 것이다.
<br/>

---

#### Reference 

- [Velog](https://velog.io/@godori/JavaScript-%EC%97%94%EC%A7%84-%ED%86%BA%EC%95%84%EB%B3%B4%EA%B8%B0-mdjowmjlcb)
- [TostMeetup_JITC, Adaptive Compilation](https://meetup.toast.com/posts/77)
- [TostMeetup_Hidden class, Inline Caching](https://meetup.toast.com/posts/78)