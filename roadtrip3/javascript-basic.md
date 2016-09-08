## 자바스크립트 기본 ##
### 익명 함수 ###

기본적으로 아래와 같이 함수를 정의할 수 있다.
```javascript
var diff = function diffOfSquares(a, b) {};
```

함수의 이름을 생략할 수 있다.
```javascript
var diff = function (a, b) {};
diff(4,2);
console.log(diff);
```

map 함수를 이용해서 함수를 조작할 수 있다.  
함수를 넘겨서 배열의 값들을 조작할 수 있다.
```javascript
var results = numbers.map(*function*);
```
```javascript
var results = numbers.map(
  function(arrayCell){
    return arrayCell * 2;
  }
);
```
한줄로
```javascript
var results = numbers.map(function(arrayCell){return arrayCell * 2;});
```
함수 자체를 배열로 저장할 수 있다.
```javascript
var puzzlers = [
  function (input) { return 3 * input - 8; },
  function (input) { return Math.pow(input + 2, 3); },
  function (input) { return Math.pow(input,2) - 9; },
  function (input) { return input % 4; },
];
```
배열 조작하는 메서드
```javascript
puzzlers.shift(); // return first index function
puzzlers.push(*something*);
```
```javascript
```
### 클로져 ###
테스트
```javascript
var a = 1;
```
