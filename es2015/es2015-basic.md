## ES2015 기본 코드스쿨 ##
### let ###

var 는 hoisting 이라는 것 때문에 변수 선언이 if/else 블록 바깥으로 자동(그 스코프의 Top)으로 빠지게 된다. 그래서 let 을 써줘야한다. let을 써주면 scope 내에서 선언하는 변수 처럼 써줄수 있다.
```javascript
let flashMesage = "Hello";
```

### const ###
상수 선언을 할때 쓰임  
const 로 선언한 변수는 바로 위 블록이 스코프이다. 밖에서는 레퍼런싱 할 수 없다.
```javascript
const MAX_USERS = 10;
```

### Function Defaults ###
첫 줄의 타입 체크 코드가 없고 그냥 userNames.length 라고 입력하면 undefined 에러가 난다. 기존 자바스크립트 함수 인자는 유연하지 못하다.  
``` javascript
function loadProfiles(userNames) {
  let names = typeof userNames !== 'undefined' ? userNames : []; // Issue Too Verbose
  let namesLength = names.length;
}
```


타입 체크 코드가 워낙 길기도 하고 verbose 하기 때문에 기본 값을 인자에 넘겨주어 해결할 수 있다.
```javascript
function loadProfiles(userNames = []) { // use Default Value
  let namesLength = userNames.length;
  console.log(namesLength);
}

loadProfiles(); // 0

loadProfiles(undefined); // 0
```

options 객체 같은 경우 함수가 어떤 옵션을 받는지 알기 어렵다.
```javascript
// 다음과 같이 사용하는 함수가 있을때
setPageThread("New Version out Soon!", {
  popular: true,
  expires: 10000,
  activeClass: "is-page-thread"
});

// 아래와 같이 정의하고 쓴다.
function setPageThread(name, options = {}){ // unclear
  let popular = options.popular;
  let expires = options.expires;
  let activeClass = options.activeClass;
}
```
options 객체가 어떤 정보를 가지고 있는지 알수가 없다. options = {} 부분이 엄청 불명확.  
아래와 같이 명시적으로 받을 옵션 이름을 정해서 쓰는게 좋다.
```javascript
function setPageThread(name, { popular, expires, activeClass }){
  let popular = popular;
  let expires = expires;
  let activeClass = activeClass;
}
```
위와 같이 정의하면 인자들을 좀 더 명확하게 정의할 수 있고 함수 호출할때도 그대로 사용할 수 있다. 몇몇 옵션을 생략하고 사용할 수도 있다.
```javascript
setPageThread("New Version out Soon!", {
  popular: true // expires, Active 는 undefiend
});

// 하지만 아래처럼 모든 인자를 생략할 수는 없다.
setPageThread("New Version out Soon!"); // Error

// 위처럼 사용하려면 Default Value 를 세팅해주면 된다.
function setPageThread(name, { popular, expires, activeClass } = {}){
  let popular = popular;
  let expires = expires;
  let activeClass = activeClass;
}

setPageThread("New Version out Soon!"); // New Version out Soon! (No Error And All Option Argument undefined)
```

Varioudic Function.. 인자를 유동적으로 호출할 수 있는 함수  
아래와 같이 ...tags 로 선언하면 tags 를 배열(array)로 받을 수 있다. 항상 인자 위치는 마지막에 위치해야한다.
```javascript
function displayTags(...tags){
  for(let i in tags){
    let tag = tags[i];
    _addToTopic(tag);
  }
}

// Must always go last
function displayTags(targetElement, ...tags){
  let target = _findElement(targetElement);

  for(let i in tags){
    let tag = tags[i];
    _addToTopic(tag);
  }
}
```

만약에 함수 인자를 유동적으로 받을 수 있는 displayTags 함수에 배열을 넘겨주면 에러가 날 것이다. 다음과 같다.
```javascript
getRequest("/topics/17/tags", function(data){
  let tags = data.tags;
  displayTags(tags); // tags 는 배열이기 떄문에 호출 에러
});

// 아래와 같이 쓰면 된다.
getRequest("/topics/17/tags", function(data){
  let tags = data.tags;
  displayTags(...tags);
});
```
위와 같이 쓰면 displayTags(tag,tag,tag) 처럼 Array 를 split 해서 들어간다.  

객체를 만들어서 getRequest 함수를 캡슐화(encapsulate)해보자.
```javascript
function TagComponent(target, urlPath){
  this.targetElement = target;
  this.urlPath = urlPath; // 생성자 함수에서 세팅
}

Tag.Component.prototype.render = function(){

  // this.urlPath 다른 인스턴스 메서드에서 접근 가능
  getRequest(this.urlPath, function(data){
    //...
  });
}

// 아래와 같이 쓰기로 한다.
let tagComponent = new TagComponent(targetDiv, "/topics/17/tags");
tagComponent.render();

// scope 가 달라서 this.targetElement 인자를 넘겨줄때 에러가 난다.
Tag.Component.prototype.render = function(){

  getRequest(this.urlPath, function(data){
    let tags = data.tags;
    displayTags(this.targetElement, ...tags); // this.targetElement 는 TagComponent 생성자 함수에서 선언되는 해당 scope 와 이 함수의 scope가 달라서 이렇게 쓰면 에러가 난다.
  });
}
```

아래와 같이 Arrow function 을 이용하면 lexical binding 이 된다.  
정의된 곳의 스코프에 바인딩한다는 의미이다.
```javascript
Tag.Component.prototype.render = function(){

  getRequest(this.urlPath, (data) => {
    let tags = data.tags;
    displayTags(this.targetElement, ...tags); // this.targetElement 는 TagComponent 생성자 함수에서 선언되는 해당 scope 와 이 함수의 scope가 달라서 이렇게 쓰면 에러가 난다.
  });
}
```

### Object and Strings ###

객체를 세팅해줄때 프로퍼티 세팅해주는 거 떄문에 코드가 엄청 길어진다.
```javascript
function buildUser(first, last){
  let fullName = first + " " + last;

  return { first: first, last: last, fullname: fullName}; // long code. bad..
}

// 호출
let user = buildUser("Sam", "Williams");

console.log( user.fisrt ); // Sam
console.log( user.last ); // Williams
console.log( user.fullName ); // Sam Williams

// ES2015 에서 아래와같이 줄일 수 있다.
function buildUser(first, last){
  let fullName = first + " " + last;

  return { first, last, fullName };
}

let name = "Sam";
let age = 45;
let friends = ["Brook", "Tyler"];

let user = { name, age, friends }; // == let user = {name: name, age: age, friends: friends}

console.log( user.name ); // Sam
console.log( user.age ); // 45
console.log( user.friends ); // ["Brook", "Tyler"]
```

object Destructing 을 이용해서 불필요한 선언 반복을 피할 수 있다.
```javascript
let user = buildUser("Sam", "Williams");

// 불필요한 반복
let first = user.first;
let last = user.last;
let fullName = user.fullName;
// 불필요한 반복

let { first, last, fullName } = buildUser("Sam", "Williams");

console.log( fisrt );
console.log( last);
console.log( fullName )

// 모든 프로퍼티를 destructing 할 필요는 없다.
let { fullName } = buildUser("Sam", "Williams");
console.log( fullName );

let {last, fullName } = buildUser("Sam", "Williams");
console.log( last );
console.log( fullName );

```

객체 프로퍼티에 함수를 추가하는 방식도 바뀌었다. 이전 버전 자바스크립트에서는 아래와 같이 익명 함수 모두를 정의해주어야했다.
```javascript
function buildUser(first, last){
  let fullName = first + " " + last;
  const ACTIVE_POST_COUNT = 10;

  return {
    first,
    last,
    fullName,
    isActive: function(){ // full fuction
      return postCount >= ACTIVE_POST_COUNT;
    }
  };

  // 이전 버전에서 full function 을 정의해줘야하는 것과 달리 ES2015 에서는 function 키워드 없이 정의 가능하다.
  return {
    first,
    last,
    fullName,
    isActive(){
      return postCount >= ACTIVE_POST_COUNT;
    }
  };
}
```

루비 #사인처럼 달러사인으로 String 을 만들수있다. 다음과 같다.

```javascript
function buildUser(first, last){
  let fullName = `${first} ${last}` // first + " " + last;
  const ACTIVE_POST_COUNT = 10;
  //..

}
```

멀티라인 String 도 지원해준다. 이전 자바스크립트 버전에서는 토큰 에러가 난다.
```javascript
let userName = "Sam";
let admin = { fullName: "Alex Williams" };

// backticks 써야된다. ``
let veryLongText = `Hi ${userName},

this is a very
very

veeeeery
long text.

Regards,
  ${admin.fullName}
`;
```

### Object.assign ###
다음과 같은 countdownTimer 함수가 있다.
```javascript
countdownTimer($('.btn-undo'), 60);

countdownTimer($('.btn-undo'), 60, { container: '.new-post-options'});
countdownTimer($('.btn-undo'), 3, { container: '.new-post-opions', timeUnit: 'minutes', timeOutClass: '.time-is-up'});

// 앞에서는 named arguments 를 추천했지만 이번 함수같은 경우는 인자가 엄청 많아서 options = {} 로 쓰는게 훨씬 낫다.
function countdownTimer(target, timeLeft, options = {}){
  //...
}

// default value 를 세팅해줄때 || 연산자를 쓴다.
function countdownTimer(target, timeLeft, options = {}){
  let container = options.container || ".timer-display";
  let timeUnit = options.timeUnit || "seconds";
  let clonedDataAttribute = options.clonedDataAttribute || "cloned";
  //...
}

// || 연산자로 default value 를 세팅해주면 너무 코드가 길어진다.
// 아래와 같이 defaults 객체를 선언함으로써 기본값 세팅을 해줄 수 있다.
function countdownTimer(target, timeLeft, options = {}){
  let defaults = {
    container: ".timer-display",
    timeUnit: "seconds",
    clonedDataAttribute: "cloned",
    //....
  };
}

// 하지만 options 로 넘어온 값들을 무시하고 기본값이 세팅되어 버린다. 그때 Object.assign 을 이용해서 오버라이드 한다.
function countdownTimer(target, timeLeft, options = {}){
  let defaults = {
    container: ".timer-display",
    timeUnit: "seconds",
    clonedDataAttribute: "cloned",
    //....
  };

  let settings = Object.assign({}, defaults, options); // option 의 값을 기본으로 defaults 에 오버라이드한다. 그러면 option 값은 사용하고 defaults 기본 세팅 값도 사용할 수 있다.
  //...
}

// 아래와 같이 쓰면 options3 를 베이스로 -> options2 -> options -> defaults 순으로 오버라이드된다.
let settings = Object.assign({}, defaults, options, options2, options3);

// 다음과 같이 쓸수는 없다.
Object.assign(defaults, options); // defaults is mutated.. 이렇게 쓸수 없다.
let settings = {};
Object.assign(settings, defaults, options); // X 기본 값은 바뀌지 않는다. 여기서 setting 은 값이 아니라 레퍼런스를 넘겨주기 때문에 쓸수없다.
console.log( settings.user );

```

### Arrays ###

반복적으로 변수 할당을 막기 위해 아래와 같이 변수를 할당해줄 수 있다.
```javascript
let users = ["Sam", "Tyler", "Brook"];
let [a, b, c] = users; // 자동으로 할당
console.log(a,b,c); // Sam Tyler Brook

let [a, ,b] = users; // blank 를 둘수있다.
console.log(a,b); // Sam Brook

let [ first, ...rest ] = users; // destructing(변수 할당) + rest params(배열 인자)
console.log(a,b); // Sam ["Tyler", "Brook"]

function activeUsers(){
  let users = ["Sam", "Ales", "Brook"];
  return users;
}

let active = activerUsers();
console.log(active); // ["Sam", "Alex", "Brook"]

let [a, b, c] = activeUsers();
console.log( a,b,c); // Sam Alex Brook
```

기존에 for..in 은 index 벨류를 사용한다.
```javascript
let names = ["Sam", "Tyler", "Brook"]

for(let index in names){
  console.log( names[index] ); // 실제로 엘리먼트 접근하기 위해 index 를 사용해야한다. 부자연스러움
}

// for..of 를 사용하면 바로 객체를 이용할 수 있다.
for(let name of names){
  console.log( name );
}
```
당연히 for..of 는 object 에서는 사용불가능하고 iterator 가 되는 Array에서만 가능하다.
```javascript
console.log( typeof names[Symbol.iterator] ); // function

let post = {
  title: "New Features in JS",
  replies: 19,
  lastReplyFrom: "Sam"
};

console.log( typeof post[Symbol.iterator] ); // undefined
```

Array.find 는 해당 함수의 조건을 만족하는 첫 요소를 반환한다. 다음과 같다.
```javascript
let users = [
  { login: "Sam", admin: false },
  { login: "Brook", admin: true },
  { login: "Tyler", admin: true }
];

// user.admin 이 true 인 첫 요소를 반환한다.
let admin = users.find( (user) => {
    return user.admin;
});

console.log( admin ); // { "login": "Brook", "admin": true }

// one-liner 로 줄일 수 있다.
let admin = users.find( user => user.admin );
console.log( admin );
```

### Maps ###

자바스크립트 Object를 map 으로 활용하게 되면 keys 모두를 항상 문자열로 변환해버린다.
```javascript
let user1 = { name: "Sam" };
let user2 = { name: "Tyler" };

let totalReplies = {};
totalReplies[user1] = 5; // string 으로 "[object Object]"
totalReplies[user2] = 42; // string 으로 "[object Object]"

// 그래서 console 을 찍으면 같은 key값으로 인식한다.
console.log( totalReplies[user1] ); // 42 X
console.log( totalReplies[user2] ); // 42
```

Map 객체는 key/value 자료구조이다. 어떤 key 나 value 를 사용해도 문자열로 변환하지 않는다.
```javascript
let user1 = { name: "Sam" };
let user2 = { name: "Tyler" };

let totalReplies = new Map();
// get() / set() 메서드를 통해서 Maps 에 접근할 수 있다.
totalReplies.set( user1, 5 );
totalReplies.set( user2, 42);

console.log( totalReplies.get(user1) ); // 5
console.log( totalReplies.get(user2) ); // 42
```

Maps 는 런타임에서 코드가 동작하기 때문에 런타임때 까지 keys 를 알지 못하고 반대로 Object 는 정의해서 쓰기 때문에 런타임 전에 keys 값을 알 수 있다.
```javascript
let recentPosts = new Map();

createPost(newPost, (data) => {
  recentPosts.set( data.author, data.message );
});
```
Map 에서는 모든 keys 는 같은 타입이고 모든 values 도 같은 타입이다. 역시 반대로 Object 는 values 가 타입이 다를 수 있다.  
for..of 를 사용해서 interator 를 돌 수 있다.
```javascript
let mapSettings = new Map();

mapSettings.set( "user", "Sam" );
mapSettings.set( "topic", "ES2015" );
mapSettings.set( "replies", ["Can't wait", "So Cool"] );

// destructing 을 해서 반복문을 돈다.
for(let [key, value] of mapSettings){
  console.log(`${key} = ${value}`);
}
```

WeakMap은 Map의 keys 값에 오직 객체 object만 사용할 수 있다. 프리머티브 타입, strings, numbers, booleans 등은 사용할 수 없다.
```javascript
let user = {};
let comment = {};

let mapSettings = new WeakMap();

mapSettings.set( user, "user" );
mapSettings.set( comment, "comment" );

// 다음과 같은 메서드를 사용할 수 있다.
console.log( mapSettings.get(user) ); // 그 키 값 출력 user
console.log( mapSettings.has(user) ); // 그 키를 가지고 있는 true
console.log( mapSettings.delete(user) ); // 그 키를 삭제 true

// 아래는 불가능
mapSettings.set( "topic", "ES2015" ); // Primitive data types are not allowed X
// for..of 문도 사용할 수 없다. iterator function이 존재하지 않는다.
```

WeakMap은 메모리에 더 효율적이기 때문에 사용한다.  
WeakMap 안의 각 요소들은 WeakMap이 존재하는 동안에 가비지 콜렉트 될 수 있다.
```javascript
let user = {}; // 모든 객체는 메모리 영역을 발생

let userStatus = new WeakMap();
userStatus.set( user, "logged" ); // 객체의 reference 를 키로 넘긴다.

//...
someOtherFunction( user ); // 반환되면, user 는 가비지 콜렉팅 될 수 있다.
```
즉, keys로 사용된 객체들이 가비지 콜렉팅이 되버리기 때문에 더이상 시스템에서 레퍼런스하지 않는다. (메모리 효율 업)

### Sets ###

Array는 중복 요소가 들어가지만 Set은 요소가 unique한 value가 들어간다.
```javascript
let tags = new Set();

// add 메서드를 통해서 요소를 더한다.
tags.add("JavaScript");
tags.add("Programming");
tags.add({version: "2015"});
tags.add("Web");
tags.add("Web"); // 중복되는 요소는 무시된다.

console.log("Total items ", tags.size); // 4 not 5
```

마찬가지로 for..of를 통해서 iteration을 돌 수 있다.
```javascript
for(let tag of tags){
  console.log(tag);
}

// destructing
let [a,b,c,d] = tags;
console.log(a,b,c,d);
```

Map과 마찬가지로 WeakSet() 객체를 가진다. Value를 객체만 저장할 수 있다.
```javascript
let weakTags = new WeakSet();

weakTags.add("JavaScript"); // X TypeError.. WeakSet Only objects can be added.
weakTags.add({name:"JavaScript"});
let iOS = {name: "iOS"};
weakTags.add(iOS);

weakTags.has(iOS);
weakTags.delete(iOS);
```

WeakMap과 마찬가지로 for..of iterator를 사용할 수 없고 (undefined) value를 읽을 수 있는 메서드가 존재하지 않는다.  
그럼 언제 쓰는가? 아래와 같은 예가 있다.
```javascript
let post = {  };

postList.addEventListener('click', (event) => {
  //...
  post.isRead = true; // post 객체를 직접 조작하게 된다.
});

for(let post of postArray){
  if(!post.isRead){ // 일일이 모든 객체의 프로퍼티를 체크해야한다.
    _addNewPostClass(post.element);
  }
}

// WeakSet을 이용하면 직접 객체를 조작하지 않고 일일이 프로퍼티를 체크안해도 된다.
let readPosts = new WeakSet();

postList.addEventListener('click', (event) => {
  //...
  readPosts.add( post ) // WeakSet에 저장
});

for(let post of postArray){
  if(!weakSet.has(post)){ // has 메서드를 이용해서 WeakSet에 존재하는지만 체크하면 된다.
    _addNewPostClass(post.element);
  }
}
```

### Classes ###
기존 자바스크립트에서 constructor function을 통해서 캡슐화를 한다.
```javascript
function SponsorWidget(name, description, url){
  this.name        = name;
  this.description = description;
  this.url         = url;
}

SponsorWidget.prototype.render = function(){ // 메서드 선언.. too verbose!
  //...
}

// 다음과 같이 호출할 수 있다.
let sponsorWidget = new SponsorWidget(name, description, url);
sponsorWidget.render();
```

ES2015에서는 class 키워드를 통해서 다른 언어들처럼 좀 더 직관적이고 쉽게 class를 선언할 수 있다.
```javascript
class SponsorWidget{
  constructor(name, description, url){
    this.name        = name;
    this.description = description;
    this.url         = url;
  }

  render(){ // 간단하게 메서드 선언
    let link = this._buildLink(this.url); // this 키워드를 통해서 인스턴스 변수에 접근할 수 있다.
    //...
  }

  _buildLink(url){  // 접두에 underscore를 붙이는 메서드는 public API 에서 절대 호출하지 않는다. 즉, private 처럼 쓰겠다는 관용적인 표현.
    //...
  }
}

// 호출은 기존과 완전히 동일하게 할 수 있다.
let sponsorWidget = new SponsorWidget(name, description, url);
sponsorWidget.render();
```
class의 장점은 역시 상속을 통한 코드 재사용에 있다.

```javascript
// 부모 클래스
class Widget {

  constructor(){
    this.baseCSS = "site-widget";

    parse(value){
      //...
    }
  }
}

// 자식, 상속 클래스
class SponsorWidget extends Widget {
  constructor(name, description, url){
    super();

    //...
  }

  render(){
    let parsedName = this.parse(this.name); // 상송받은 메서드
    let css = this._buildCSS(this.baseCSS); // 상속받은 프로퍼티
  }

  // 메서드를 오버라이딩할 수 있다.
  parse(){
    let parsedName = super.parse(this.name);
    return `Sponsor: ${parsedName}`;
  }
}
```

### Modules ###
Global Namespace 영역에서 모든 라이브러리가 global하게 공유될때 이름이 충돌하는 등 사이드 이펙트가 존재한다.
```html
<!DOCTYPE html>
<body>
  <script src="./jquery.js"></script>
  <script src="./underscore.js"></script>
  <script src="./flash-message.js"></script>
</body>
```
위처럼 쓰면 모두 global 영역에서 공유되게 된다. 이름 충돌이 일어날 수 있다.  

module을 사용해보자.
```javascript
// flash-message.js
// export: 이 함수가 모듈 시스템에 노출
// export를 default type으로 (함수를 export하는 가장 간단한 방법)
export default function(message){
  alert(message);
}

// import 키워드를 사용해서 쓸 수 있다.
// app.js
import flashMessage from './flash-message'; // default 키워드를 사용한 기본 함수가 매핑이 된다.
flashMessage("Hello");
```
위와 같이 export 해서 쓰면 global 영역을 공유하지 않게 된다.  
여러 메서드를 export해서 쓰려면?

```javascript
export function alertMessage(message){
  alert(message);
}

export function logMessage(message){
  console.log(message);
}
// 더 이상 default 키워드를 사용하지 않고 export로 명시적으로 쓴다.
// 아래와 같이 import 할 수 있다.
import { alertMessage, logMessage } from './flash-message';

alertMessage('Hello from alert');
logMessage('Hello from log');

// 모든 메서드를 써줄수는 없으니 다음과 같이 쓰고 호출할 수 있다.
import * as flash from './flash-message';

flash.alertMessage('Hello from alert');
flash.logMessage('Hello from log');
```

각각 함수 앞에 export 키워드를 사용할 수 있으나 일괄적으로 여러 함수를 export 해줄 수 있다. 다음과 같다.
```javascript
function alertMessage(message){
  alert(message);
}

function logMessage(message){
  console.log(message);
}

export { alertMessage, logMessage }; // 이렇게 export
```

const 상수를 함수내부에 반복적으로 정의하게 되는 경우가 생기는데,  
module을 이용하면 상수를 재사용하면서 구현 상황 캡슐화할 수 있다.  
상수도 사용법은 같다.
```javascript
export const MAX_USERS = 3;
export const MAX_REPLIES = 3;
// 위와 아래는 똑같다.
const MAX_USERS = 3;
const MAX_REPLIES = 3;
export { MAX_USERS, MAX_REPLIES };

// 다른 함수에서 아래와 같이 import 해서 쓰면 된다.
import { MAX_REPLIES } from './constants';
```
class도 export해서 쓸 수 있지 않을까?  
function과 마찬가지로 쓸 수 있다.
```javascript
export default class FlashMessage(){
  constructor(message){
    this.message = message;
  }

  renderAlert(){
    alert(`${this.message} from alert`);
  }

  renderLog(){
    console.log(`${this.message} from log`);
  }
}
// 아래와 같이 import 해서 쓸 수 있다.
import FlashMessage from './flash-message';

let flash = new FlashMessage("Hello");
flash.renderAlert();
flash.renderLog();
```
```javascript
class FlashMessage(){
  constructor(message){
    this.message = message;
  }

  renderAlert(){
    alert(`${this.message} from alert`);
  }

  renderLog(){
    console.log(`${this.message} from log`);
  }
}

// class 도 똑같이 선언 후에 export 해줄 수 있다.
export { FlashMessage };

// name으로 import 할 때는 이름이 반드시 일치해야한다.
import { FlashMessage } from './flash-message';

let flash = new FlashMessage("Hello");
flash.renderAlert();
flash.renderLog();
```

### Promise ###
Javascript는 싱글 스레드 모델이다. 서버로부터 응답을 기다릴때 앱 전체가 멈출수도 있다.
```javascript
// 메인 쓰레드에서 호출하면 freeze 될 수 있다.
let results = getPollResultsFromServer("Sass vs. LESS");
ui.renderSidebar(results);

// Javascript에서는 다음과 같이 callback으로 freeze를 막는다.
getPollResultsFromServer("Sass vs. LESS", function(results){
  ui.renderSidebar(results);
});
```

하지만 콜백을 실행할때 연속된 함수가 많으면 function 중첩이 계속 생기고 코드가 복잡해진다.
```javascript
getPollResultsFromServer("Sass vs. LESS", function(error, results){
  if(error){  } //.. handle error
  //...
  ui.renderSidebar(results, function(error){
    if(error){  } //.. handle error
    //...
    ~(, function(error, response){
      if(error){  } //.. handle error
      //...
      ~(, function(error){
        if(error){  } //.. handle error
        //...
      });
    });
  });
});
```
이해하기도 코드구현도 힘들고 복잡하다.  
Promise를 이용하면 다음과 같이 쓸 수 있다.
```javascript
getPollResultsFromServer("Sass vs. LESS")
  .then(ui.renderSidebar)
  .then(sendNotificationToServer)
  .then(doSomethingElseNonBlocking)
  .catch(function(error){
    console.log("Error: ", error);
  });
```

Promise는 다음으로 정의할 수 있다.
```javascript
function getPollResultsFromServer(pollName){
  return new Promise(function(resolve, reject){ // Promise의 Handler는 항상 두개의 인자를 가진다.
    //...
    resolve(someValue); // 논블로킹으로 코드 실행

    //...
    reject(someValue); // 에러가 발생했을때
  });
}
```
Promise LifeCycle
```javascript
new Promise(); // 하게 되면 state는 바로 pending 이 된다.

resolve(value); // 하게 되면 fulfilled state
rejecte(value); // 하게 되면 rejected state
```
Promise는 미래의 값(future value)을 나타낸다. 비동기 연산의 결과값..
```javascript
let fetchingResults = getPollResultsFromServer("Sass vs. LESS");
// fetchingResults는 실제 결과가 아니라 Promise Object 이다.
```

XMLHttpRequest 객체 API를 통해서 Promise를 랩핑하고 resolve() 핸들러를 통해 Promise를 fulfilled state로, reject() 핸들러를 통해 Proimse가 rejected state가 된다.
```javascript
function getPollResultsFromServer(pollName){

  return new Promise(function(resolve, reject){
    let url = `/results/${pollName}`;
    let request = new XMLHttpRequest();
    request.open('GET', url, true);
    request.onload = function(){
      if (request.status >= 200 && request.status < 400){
        resolve(JSON.parse(request.response)); // 성공했을때 resolve
      } else {
        reject( new Error(request.status) ); // reject 핸들러는 Error 객체를 넘긴다.
      }
    };
    request.onerror = function(){
      reject(new Error("Error Fetching Results")); // reject 핸들러는 Error 객체를 넘긴다.
    };
  });
}

// 다음과 같이 사용할 수 있다.
let fetchingResults = getPollResultsFromServer("Sass vs. Less");
fetchingResults.then(function(results){ // 여기 results는 위에서 정의한 resolve(JSON.parse(request.response)); 에서 인자 JSON.parse 부분이다.
  ui.renderSidebar(results);
});

// 임시 변수는 제거하고 체이닝해서 then을 쓸 수 있다.
getPollResultsFromServer("Sass vs. Less")
  .then(function(results){
    return results.filter((result) => result.city === "Orlando");
  })
  .then(function(resultsFromOrlando){
    ui.renderSidebar(resultsFromOrlando);
  })
  .catch(function(error){
    console.log("Error: ", error); // 도중에 에러가 발생하면 catch() 함수로 즉시 이동해서 호출된다.
  });

// 각각의 함수를 변수에 저장하고 호출하면 심플하게 호출된다.
function filterResults(results) {  }

let ui = {
  renderSidebar(filteredResults) {  }
};

getPollResultsFromServer("Sass vs. Less")
  .then(filterResults)
  .then(ui.renderSidebar)
  .catch(function(error){
    console.log("Error: ", error);
  });
```
### Iterators ###
Array는 iterable 객체이다. 앞서 for..of 를 사용할 수 있었다.  
하지만 Object는 iterable 하지 않아서 for..of 를 사용할 수 없다.  

iterator의 내부 코드 동작을 살펴보자.
```javascript
let names = ["Sam", "Tyler", "Brook"];

for(let name of names){ // 여기 동작구조를 아래에서 살펴본다.
  console.log(name);
}

let iterator = names[Symbole.iterator]();

// 각 iterator가 동작할때 next()메서드는 done, value를 가진 객체가 반환
let firstRun = iterator.next(); // { done: false, value: "Sam"}
let name = firstRun.value;

let secondRun = iterator.next(); // { done: false, value: "Tyler"}
let name = secondRun.value;

let thirdRun = iterator.next(); // { done: false, value: "Brook"}
let name = thirdRun.value;

// done 이 true이면 이전에 멈춘다.
let fourthRun = iterator.next(); // { done: true, value: undefined}
let name = fourthRun.value;
```
Object도 Iterator로 만들어보자.
```javascript
let post = {
  title: "New Features in JS",
  replies: 19
};

// 이렇게 정의한다.
post[Symbol.iterator] = function(){

  let properties = Object.keys(this); // property 이름들을 배열로 반환
  let count = 0; // 인덱스로 properties 배열에 엑세스하기 위해
  let isDone = false; // loop가 끝날때는 true로 세팅되어질 것이다.

  let next = () => {
    if(count >= properties.length){ // 마지막까지 루프 돌기
      isDone = true;
    }
    return { done: isDone, value: this[properties[count++]] };
    // this 는 post 객체
  }

  return { next };
};

// 아래와 같이 사용할 수 있다.
for(let p of post){
  console.log(p);
}
// New Features in JS
// 19

// 물론 iterator 이니 ...spread 연산자를 이용해서 할당할 수 있다.
let values = [...post];
console.log( values );

// destructing 도 물론 가능
let [title, replies] = post;
console.log( title ); // New Features in JS
console.log( replies ); // 19
```

### Generators ###
function * 선언은 generator function을 정의한다. iterator 객체들을 반환하기위해 yield 키워드를 사용할 수 있다.
```javascript
function *nameList(){
  yield "Sam"; // => { done: false, value: "Sam" }
  yield "Tyler"; // => { done: false, value: "Tyler" }
}

// *의 위치는 상관없다. 아래 표현은 모두 문제없다.
function *nameList(){}
function* nameList(){}
function * nameList(){}
```
위에서 object에 iterator 하기 위해서 구현했던 것을 yield와 * 를 이용해서 간단히 구현할 수 있다.
```javascript
post[Symbol.iterator] = function *(){ // Generator Function 표시

  let properties = Object.keys(this);

  for(let p of properties){
    yield this[p];
  }
};

// 아래와 같이 사용할 수 있다.
for(let p of post){
  console.log(p);
}
// New Features in JS
// 19
```















































--
