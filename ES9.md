# ES2018

AsyncIterator 프로토콜 및 비동기 생성기를 통한 비동기 반복 지원 도입 for-await-of
객체 Rest 및 Spread 속성이 포함
dotAll또한 플래그, 명명된 캡처 그룹, 유니코드 속성 이스케이프 및 look-behind 어설션 의 4가지 새로운 정규식 기능이 포함

``for-await-for``
> 반복문 내에서 일어나는 모든 비동기 구문을 기다려주는 구문.


## Asynchronous iteration(비동기 루프)
```
Asynchronous iteration
async function process(array) { 
  for await (let i of array) { 
    // doSomething(i); 
  }
}

```
## Promise.finally()
Promise가 처리되면 충족되거나 거부되는지 여부에 관계없이 지정된 콜백 함수가 실행됨.
Promise.resolve().then().catch(e => e).finally();

## Rest/Spread
const myObj = { a: 1, b: 3, c: 'cc', d: 100 };

const {a, b, ...z} = myObj;
console.log(z); // { "c": "cc", "d": 100 }

const spread = {  ...myObj,  a: 10,  e: 30,};
console.log(spread); // { "a": 10, "b": 3, "c": "cc", "d": 100, "e": 0 }

## 정규식 기능 개선 및 추가

### 1) RegExp lookbehind assertions

앞에 오는 항목에 따라 문자열 일치
// ?= 특정 하위 문자열이 뒤에 오는 문자열을 일치시키는데 사용
/Roger(?=Waters)/
/Roger(?= Waters)/.test('Roger is my dog') //false
/Roger(?= Waters)/.test('Roger is my dog and Roger Waters is a famous musician') //true

// ?! 문자열 뒤에 특정 하위 문자열이 오지 않는 경우 일치하는 역 연산을 수행
/Roger(?!Waters)/
/Roger(?! Waters)/.test('Roger is my dog') //true
/Roger(?! Waters)/.test('Roger Waters is a famous musician') //false

// ?<= 새로 추가된 표현식
/(?<=Roger) Waters/
/(?<=Roger) Waters/.test('Pink Waters is my dog') //false
/(?<=Roger) Waters/.test('Roger is my dog and Roger Waters is a famous musician') //true

// ?<! 새로 추가된 표현식
/(?<!Roger) Waters/
/(?<!Roger) Waters/.test('Pink Waters is my dog') //true
/(?<!Roger) Waters/.test('Roger is my dog and Roger Waters is a famous musician') //false
### 2) 유니코드 속성 이스케이프 \p{…} 및 \P{…}

// ASCII
/^\p{ASCII}+$/u.test('abc')   //✅
/^\p{ASCII}+$/u.test('ABC@')  //✅
/^\p{ASCII}+$/u.test('ABC🙃') //❌

// HEX
/^\p{ASCII_Hex_Digit}+$/u.test('0123456789ABCDEF') //✅
/^\p{ASCII_Hex_Digit}+$/u.test('h')                //❌

// Emoji, ppercase, Lowercase, White_Space, Alphabetic
/^\p{Lowercase}$/u.test('h') //✅
/^\p{Uppercase}$/u.test('H') //✅

/^\p{Emoji}+$/u.test('H')   //❌
/^\p{Emoji}+$/u.test('🙃🙃') //✅

// Script
/^\p{Script=Greek}+$/u.test('ελληνικά') //✅
/^\p{Script=Latin}+$/u.test('hey') //✅
### 3) 명명된 캡처 그룹(named capture group)

ES2018에서는 결과 배열의 슬롯을 할당하는 대신 캡처 그룹을 이름에 할당할 수 있음
const re = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/
const result = re.exec('2015-01-02')

// result.groups.year === '2015';
// result.groups.month === '01';
// result.groups.day === '02';
### 4) s 플래그(dotAll)

.표현식은 개행문자를 제외한 모든 문자였으나, s플래그를 달면 개행식도 포함하게 된다.
/hi.welcome/.test('hi\nwelcome') // false
/hi.welcome/s.test('hi\nwelcome') // true