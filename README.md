TFLP : Functional List Processor (TFLP)
이명) The Functional List Processor

# 기계어

## 명령 분석

제 1 비트 : 코드 주소 인지
제 2 비트 : 함수 시작 인지

## 제 2 비트 거짓일시

"stack인척하는 list의".pop(argv) -> "stack인척하는 list의".push(in a top)

NOTE : 단순히 메모리 편집이다, "평범하지 않지만 합리적인 명령이지" 할정도다.

## 제 2 비트 참일시

"stack인척하는 list의".pop(argv) -> 함수호출 스택으로 ㄱㄱ

NOTE : 단순히 POP이다, "평범하지 않지만 합리적인 명령이지" 할정도다.

## 함수 호출 기전

함수 호출 스택에 주소가 푸쉬되는 즉시 다음 동작이 일어남 : 
top of stack과 program counter를 swap

NOTE : 단순히 MOV다.
NOTE : 콜스텍 복귀는 알아서 최적화되어야만 한다.
NOTE : 사실 제 2 비트 참으로 명령한 순간 모든부분이 기계수준에서 자동처리되는 구상이다 (가상머신으로 돌아간다면 가상머신이 알아서 하갰지.)

## 언어 작동 원리

베이스 포인터 기준 주소를 주소로 작동한다.

## 함수 호출 스택

제 0 깊이 : 함수 호출 레지스터
제 n 깊이 : 스택 깊이 값

## 베이스 포인터 스택

제 0 깊이 : 레지스터
제 n 깊이 : 스택

근데 사실상 -n으로 표기해서 작동함,
-2ⁿ는 예약이고, 그게 레지스터임.

## 실제 -2ⁿ에 있는 값

중간에 11이나 00으로 나오면 그러기 전의 비트를 비트수로 하여 처리하게 만듬.

## 리스트 구조

[베이스 포인터 기준 음수부 : 스택][베이스 포인터 기준 양수부 : 코드]

# 어셈블리

## 기계어 영역에서 힙과 스택영역이 아닌 const, bss, static영역 (코드 마지막에 써놓아야만 한다. 심지어 순서도 정해져있다. 아래처럼.)

주의 : 값 지정은 하드코딩이다. 하드코딩 disable옵션이 있어야 disable된다.

### `#const:` 레이블

기계어 수준에서 아주 잘 작동 (명령어 이전 체크)

### `#data:` 레이블

static 영역이다.
메모리 아낄려고 만든거라,
어셈블리보다 높은 프로그래밍용 언어에서는,
함수형 처리 최적화용으로 권장된다.

### `#bss:`

bss 영역이다.
메모리 아낄려고 만든거라,
어셈블리보다 높은 프로그래밍용 언어에서는,
함수형 처리 최적화용으로 권장된다.

## 하드코딩 비활성화

 - const & static : `변수명: 값`
 - bss : `변수명: 메모리 길이`

## 기본값

탭 두번치고 값 적기.......
음수는 마이너스 치고 꼭 적어야한다...

## 레이블 기능 활성화시 (이제 스택 주소는 양수처리)

"\t\t" 이면 레이블 없음.
"레이블명:\t" 이면 레이블 있음.

# 비고
사실 코드영역은 리스트 제어에서 예외다 (건드리면 segfault라는거)

즉, 이 CPU를 진짜 만들더라도 메모리 구조는 충분히 기존 CPU에서 로우레벨 메모리 영역 권한 편집으로 충분히 아주 충분히 만들수 있다는것.

[call stack]-> <-[리스트][상수적 영역][정적 영역]

참고로 가상머신 동작은 개인적으로는 다음처럼 만들고싶다.

`[백그라운드 JIT코드][그걸 위한 정적 영역]`
`[call stack]-> <-[리스트][상수적 영역][정적 영역]` => 정적 영역인데 프로세스 생성/삭제시 영역 크기를 제어하는 구상이라서 이따구다, 스텍영역으로 쓸수도 있음.
`[JIT 코드의 스택]-> <-[JIT 코드의 힙]`

기타 시스템 콜 등 프로세스 관리나 커널시스템은 CPU 부가기능이거나 가상머신 부가기능으로, 유저프로세스에 접근하는것.

기본적으로 이 체계는 CPU를 가상머신처런 생각한것이다, 걍 HW에서 빠르게 만들고, 가상머신화가 용이함이 목적이지, 가상머신 목적은 아니라는거.

## 잠재적 타입 시스템.

어떤 지네릭 (C++ 지네릭마냥 과하지 않은 Java지네릭 서술이라는게 자명히 쉽게 보일것이다.) 변수 T에 대해서,
```
() := T //empty tuple
(x, ) := (x, x) //special notation of one value tuple
(x, y) := Function<x, y> //pair
(,) := () or (x, ) or (y, z) type
ruleG := x, y, z must (,) type
```
ruleG를 적용하고, `(,)`타입이 모든 타입이 된다면,
해당 파일 체계 안에서는 동적으로 가능하고,
근데 사실상 처리는 정적인 지네릭이다.

1. sizeof(T)가 사실상 T의 핵심일수밖에 없고
2. 비트단위에서 sizeof(void*)를 결정해서, sizeof가 비트인데
3. 이 sizeof(void*)비트가 사실상 CPU요구 단일타입이고,
4. 그 비트를 정할수 있으므로
5. 실행파일 하나가 매우 단순한 지네릭의 구현인 셈이다.

따라서, 타입시스템은 이러한 방식으로 구체화된다고 하겠다.

## 잠재적 프로그래밍용 언어 설계 지향점 철학

함수형 프로그래밍인데 정적 영역에 대한 최적화가 된 함수형 프로그래밍.

+ 구조적으로 어떤 기종에 대해서 시스탬 관리는 "컴퓨터님"이 우리를 따라오는게 아닌, 우리가 "컴퓨터님"을 그렇게 만드는것.

## 메모

def) SIIO = Syscall-Introduce-IO
SIIO에 람다대수를 써야 아름다운 프로그래밍언어라고 생각함.

esolang에 뇌가 절여진 미친새끼여서 그럼 반박시 말 안밪<<퍽퍽 (농담)

SIIO 람다대수는
```
λx. syscall 1 x
```
이런식으로 쓰는 중간언어로 쓸 목적인ㄷ...

하스켈을 요기로 컴파일을 막 으헤ㅔㅎ (정신나감)

#### 홧김에 말하는데

어떤 명령어셋에서 이거 실행 가능한 환경 만들어놓고, 명령 4개를 요걸로 정의하면 이거 돌릴수 있도록 만들어도 되도록
eXpeneded tFLP라고 하겠음

크아아아아아악 tlqkf 바로 이소랭 정상ㅎ<<퍽퍽

[너무 오만해서 검열됨]

# 추가 아이디어

Super TFLP : 함수형 프로그래밍에서의 병렬처리용 연산자 도입.