# 기본

## 선언

```javascript
var // 변수를 선언, 추가로 값을 초기화
let // 블록 범위(scope) 지역 변수를 선언, 추가로 동시에 값을 초기화
const // 블록 범위 읽기 전용 상수를 선언
```

## 변수 선언

변수 선언은 3가지 방법으로 가능하다.

1. `var x = 1;`: 이 방법은 지역 및 전역 변수를 선언하는데 모두 사용될 수 있다.
2. `let x = 1;`: 이 방법은 블록 범위의 지역 변수를 선언하는데에 사용된다.
3. `x = 1;`: 선언되지 않은 전역변수로 간단하게 변수에 값을 할당할 수 있다. 그러나 의도되지 않은 동작을 만들 수 있기 떄문에 사용하면 안된다.

## 변수 할당

지정된 초기값 없이 `var` 또는 `let` 문을 사용해서 선언된 변수는 `undefined` 변수 값을 가진다.  
선언되지 않는 변수에 접근을 시도할 경우에는 `ReferenceError` 에외가 발생한다.

```javascript
var a; //undefined

console.log(b); // undefined
var b;

console.log(c); // ReferenceError
```

`undefined`값은 `boolean`문맥에서 사용될 시 false로 간주된다.

```javascript
var arr = [];
if (!arr[0]) {
  func(); // 이 함수는 실행된다.
}
```

`undefined`값은 수치계산에서 `NaN`으로 리턴된다.

```javascript
var a;
console.log(a + 2); // NaN
```

`null` 값은 수치 문맥에서는 0, `boolean` 문맥에서는 `false`로 간주된다.

## 변수 호이스팅

javascript의 변수 특징 중에 하나는 예외를 받지 않고도, 나중에 선언된 변수를 참조할 수 있다는 것이다. 이 개념은 호이스팅(hoisting)이라고 불린다. 즉 javascript 변수가 어떤 의미에서 끌어올려지거나 함수나 문의 최상단으로 옮겨지는 것을 뜻한다. 그러나, 끌어올려진 변수의 값은 `undefined`값을 리턴한다. 그래서 심지어 변수를 사용하거나 참조한 후에 선언 및 초기화 하더라도 `undefined` 값을 리턴한다.

```javascript
var x;
console.log(x); // undefined
x = 1;

var str = "global value";

(function () {
  console.log(str); //undefined
  var str = "local value";
})();
```

따라서 호이스팅때문에 함수 내애 모든 `var` 문은 가능한 함수 상단에 놓는게 좋다.

ECMA2015의 `let`, `const`는 블록 상단으로 끌어올리지 않는다.
변수가 선언되기 전에 접근하면 `ReferenceError`이 발생한다.

```javascript
console.log(x); //ReferenceError
let x = 1;
```

## 전역 변수

전역 변수는 사실 global객체의 속성(property)이다. 웹페이지의 global 개게는 `window` 이므로, `window.variable`을 통해 전역을 만들고 접근한다.

# 데이터 구조 및 형

## 자료형 변환

javascript는 동적 형지정 언어이다. 이는 변수를 선언할 떄 데이터 형을 지정할 필요가 없음을 말한다. 또한 데이터 형이 스크립트 실행 도중 자동으로 필요에 의해 변환된다.

```javascript
var x = 12;
x = "hello world"; // 동적 형지정이기 때문에 오류가 발생하지 않는다.
```

## 문자열을 숫자로 변환

숫자를 나타내는 값이 문자열로 메모리에 있는 경우에 변환을 위한 메서드가 있다.

- `parseInt();`
- `parseFloat();`

## `parseInt`는 오직 정수만 리턴한다. 잘 사용하기 위해서는 항상 진법(radix) 매개변수를 포함해야 한다.

# 리터럴

javascipt에서 값을 나타내기 위해 리터럴을 사용한다. 이는 말 그대로 javascipt에서 고유하게 지정한 값으로, 변수가 아니다.

## 배열 리터럴

배열 리터럴은 0개 이상의 식 목록이다. 각 식은 배열 요소를 나타내고 대괄호[] 로 묶인다. 배열 리터럴을 사용하여 배열을 만들 때, 그 요소로 지정한 값으로 초기화 된다. 배열 리터럴은 _배열 객체_ 이다

## 불리언 리터럴

불리언 형은 `false` `true`의 리터럴 값을 가진다. 이 원시 불리언과 Boolean 객체 값을 혼동하면 안된다. 이 객체는 원시 불린 데이터 형을 감싸는 래퍼이다.

## 객체 리터럴

객체 리터럴은 중활호{}로 묶인 0개 이상인 객체의 key,value 쌍 목록이다.

```javascript
var car = { Cars: { a: "Hyundae", b: "Honda" }, 7: "Porche" };

console.log(car.Cars.b); // Honda
console.log(car[7]); // Porche
```

# 예외 처리문

## throw 문법

예외를 사용할 때 throw문을 사용한다. 예외를 사용할 때, 사용되는 값을 포함하는 표현을 명시해야 한다.

```javascript
function UserException(message) {
  this.message = message;
  this.name = "UserException";
}

UserException.prototype.toString = function () {
  return this.name + ': "' + this.message + '"';
};

throw new UserException("Value too high");
```

## try catch 문법

try..catch 문법은 시도할 블록을 표시하고, 예외가 발생했을 때, 하나 이상의 반응을 명시한다. 만약 예외가 발생한다면 try..catch문법이 잡아낸다.

### `catch` 블록

tyr 블록에서의 모든 예외를 처리하기 위해 사용된다.

```javascript
try {
  throw "myException";
} catch (e) {
  //statements..
  logMyErrors(e); // error handler에 string이 아규먼트로 넘어간다.
}
```

### `finally` 블록

finally 블록은 try 블록과 catch 블록이 시행되고, 그다음에 시행되는 문장들은 포함하고 있다. finally 블록은 예외가 있건 없건 무조건 수행한다.

```javascript
openMyFile();
try {
  writeMyFile(theData); //여기서 에러가 발생한다면
} catch (e) {
  handleError(e); // 예외를 처리하고
} finally {
  closeMyFile(); //  무조건 파일을 닫는다.
}
```

# Promises

ECMA2015를 시작하면서, 자바스크립트는 지연된 흐름과 비동기식 연산을 제어할 수 있는 `Promise` 객체에 대한 지원을 받는다.
`Promise` 객체의 상태는 4가지가 있다.

1. pending: 초기상태 fulfilled되거나 reject되지 않음
2. fulfilled: 연산 수행 성공
3. reject: 연산 수행 실패
4. settled: Promise가 수행 성공 또는 실패한 상태이다. pending은 아니다.

![promises](https://mdn.mozillademos.org/files/8633/promises.png)

# 클로저

클로저는 자바스크립트의 강력한 기능중 하나이다. 자바스크립트는 함수의 중첩을 허용하고, 내부함수가 외부 함수 안에서 정의된 모든 변수와 함수들을 완전하게 접근 할 수 있도록 해준다. 그러나 외부 함수는 내부 함수안에서 정의된 변수와 함수들에 접근을 할 수 없다. 이는 내부 함수의 변수에 대한 일종의 캡슐화를 제공한다.

```javascript
var createDog = function (name) {
  var sex;

  return {
    setName: function (newName) {
      name = newName;
    },

    getName: function () {
      return name;
    },

    getSex: function () {
      return sex;
    },

    setSex: function (newSex) {
      if (
        typeof newSex == "string" &&
        (newSex.toLowerCase() == "male" || newSex.toLowerCase() == "female")
      ) {
        sex = newSex;
      }
    },
  };
};
var dog = createDog("Ggomi");
dog.getName(); // Ggomi;
dog.setName("Cookie");
dog.setSex("female");
dog.getName(); // Cookie
dog.getSex(); // female
```

# 인수(arguments)객체 사용하기

함수의 인수는 배열과 비슷한 객체로 처리가 된다. 함수 내에서는, 전달 된 인수를 다음과 같이 다룰 수 있다.

> arguments[i]

# 함수의 매개변수

## 디폴트 매개변수

자바스크립트에서 함수의 매개변수는 `undefined`가 기본이다.

```javascript
function multiply(a, b = 1) {
  return a * b;
}
multiply(5); // 5
```

## 나머지 매개변수

나머지 매개변수 구문을 사용하면 배열로 불확실한 개수의 인수를 나타낼 수 있다.

```javascript
function multiply(multiplier, ...theArgs) {
  return theArgs.map((x) => multiplier * x);
}
var arr = multiply(2, 1, 2, 3);
console.log(arr); // [2, 4, 6];
```

# 화살표 함수

화살표 함수 표현은 짧은 문법을 가지고 있는 사전적으로 this를 묶는다. 화살표 함수는 언제나 익명이다.

```javascript
var a = ["CSS", "HTML", "Javascript", "Python"];

var a2 = a.map(function (x) {
  return x.length;
});

var a3 = a.map((x) => s.length);
```

## 사전적 this

화살표 함수에서, 모든 new함수들은 그들의 this 값을 정의한다.(생성자로서의 새로운 객체, 정의 되지 않은 stric mode의 함수 호출, 함수가 'object method'로 호출했을때의 context object 등등) 이런것은 oop스타일에서 짜증을 부른다.

```javascript
function Person() {
  this.age = 0;
  setInterval(() => {
    this.age++;
  }, 1000);
}
var p = new Person();
```

# 정규 표현식

## 방법

1. `var re = "/ad+c/";` 정규식 리터럴(슬래쉬(/)로 감싸는 패턴)
2. `var re = RegExp("ab+c");` RegExp객체로 생성자 함수 호출

## 정규식에서의 특수문자

### 1. `\`

특수문자가 아닌 문자 앞에서 사용된 백슬래시는 '해당 문자는 특별하고, 문자 그대로 해석되면 안된다'는 뜻이다. 예를 들어, `\b` 라는 정규식은 어떤 문자와도 대응되지 않는다. 이 문자는 특별한 단어 경계 문자를 형성한다.

특수 문자 앞에 위치한 백슬래시는 '그 특수문자는 특별하지핞고 문자 그대로 해석되야 한다'는 뜻이다. `/a\*/`라는 정규식은 실제로 a\* 라는 문자열과 대응된다.

### 2. `^`

입력의 시작 부분에 대응된다. 만약 다중행 플래그가 참으로 설정되어 있으면, 줄 바꿈 문자 바로 다음 부분과도 대응된다.
`/^A/` 는 "an A"는 대응되지 않지만 "An E"는 대응된다.

### 3. `$`

입력의 끝 부분과 대응된다. 만약 다중행 플래그가 참으로 설정되어 있으면, 줄 바꿈 문자 바로 다음 부분과도 대응된다.
`/t$/`는 "eater" 와 대응되지 않지만 "eat" 과 대응된다.

### 4. `*`

앞의 표현식이 0회 이상 반복되는 부분과 대응된다.

### 5. `+`

앞의 표현식이 1회 이상 반복되는 부분과 대응된다.

### 6. `?`

앞의 표현식이 0 또는 1회 등장하는 부분과 대응된다.

### 7.`.`

개행 문자를 제외한 모든 단일 문자와 대응된다.

## 정규식에서 쓰이는 메소드

### 1. exec

대응되는 문자열을 찾는 `RegExp` 객체 메소드이다. 정보를 가지고 있는 배열을 반환하고 찾지 못한다면 `null`을 반환한다.

### 2. test

대응되는 문자열이 있는지 검사하는 `RegExp` 객체 메소드이다. `true` or `false`를 리턴한다.

### 3. match

대응되는 문자열을 찾는 `RegExp` 객체 메소드이다. 정보를 가지고 있는 배열을 반환하고 찾지 못한다면 `null`을 반환한다.

### 4. search

대응되는 문자열이 있는지 검사하는 String 객체 메소드이다.

### 5. replace

대응되는 문자열을 찾아 다른 문자열로 치환하는 String 객체 메소드이다.

### 6. split

정규식 또는 문자열로 대상 문자열을 나누어 배열로 반환하는 String 메소드이다.

# WeakMap ojbect

`WeakMap`객체는 object만을 키로 허용하고 값은 임의의 값을 허용하는 키/값 형태 요소의 집합이다. 키가 가지고 있는 객체에 대한 참조는 객체에 대한 참조가 더이상 존재하지 않을 경우 GC의 수거 대상이 되는 약한 참조를 의미한다. `WeakMap` API는 Map API와 동일하다.

`WeakMap`를 사용하는 한가지의 경우는 객체의 사적인 정보를 저장하기 위해서 이거나 상세 구현 내용을 숨기기 위한 것이다. 객체가 가지고 있지만 외부에 공개되지 않는 데이터나 메서드들을 WeakMap 객체인 privates에 저장된다. 인스턴스를 통해 접근 가능한 모든 것들과 prototype은 public이고 다른 것들은 외부에서는 접근이 불가능하다. 그 이유는 privates는 모듈로부터 exports가 되지 않기 떄문이다.

```javascript
const privates = WeakMap();

function Public() {
  const me = {
    // Private data
  };
  privates.set(this, me);
}

Public.prototype.method = function () {
  const me = privates.get(this);
  // Do stuff with privates data in `me`
};

module.exports = Public;
```

# 모든 것이 객체!

자바스크립트 세상에서는 거의 모든 것들이 객체이다. `null`과 `undefined`를 제외한 모든 원시 타입도 객체로 취급된다. 이러한 원시 타입들엗 프로퍼티가 추가될 수 있고, 모두 객체로써 특징을 갖는다.

# 상속

Javascript에서의 모든 객체들은 최소한 하나의 다른 객체로부터 상속을 받는다. 상속을 제공하는 객체를 프로토타입이라고 부르고, 상속되는 속성들은 `prototype`이라는 생성자 객체에서 찾을 수 있다.

# 객체 참조를 위해 this 사용하기

Javascript는 `this`라는 특별한 키워드를 가지고 있다. 메서드 내부에 `this`키워드를 사용하면 해당 메서드를 포함한 객체를 가리키게 된다.

# getters와 setters 정의

getter과 setter는 두개의 방법으로 정의될 수 있다.

1. object initializers 를 사용하여 정의
2. getter과 setter에서도 추가 방법을 사용하여 언제든지 특정 객체에 나중에 추가하여 정의

object initializers를 사용해서 정의할 때에는 get 또는 set 접두사만 추가하면 된다.

```javascript
var o = {
  a: 7,
  get b() {
    return this.a + 1;
  },
  set c(x) {
    this.a = x;
  },
};
```

`Object.defineProperties` 메서드를 사용하면 객체 생성이후에도 getter와 setter를 정의할 수 있다.

# 프로퍼티의 삭제

상속 받지 않은 속성은 `delete` 연산자를 이용하여 제거할 수 있다.

```javascript
var myobj = new Object();
myobj.a = 5;
myobj.b = 12;

delete myobj.b;
console.log("a" in myobj); //false
```

`var` 키워드를 선언하지 않은 전역 변수도 `delete`를 이용하여 삭제할 수 있다.

```javascript
g = 17;
delete g;
```

# 객체 간의 비교

javascript에서 객체들은 레퍼런스 타입이다. 두 개의 별개 객체들은 속성값들이 모두 다 같더라도 절대 동일하다고 비교될 수 없다.

```javascript
var fruit = { name: "apple" };
var fruitbear = { name: "apple" };
fruit == fruitbear; // false
fruit === fruitbear; // false

fruitbear = fruit; // fruit 객체 레퍼런스를 fruitbear에 할당

fruit == fruitbear; // true
fruit === fruitbear; // true
```
