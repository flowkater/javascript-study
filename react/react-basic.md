## React 기본 코드스쿨 ##
#### React는 무엇인가? ####
React는 유저 인터페이스를 만드는 자바스크립트 라이브러리이다. MVC에서 View를 담당한다.  
React는 계속해서 데이터가 변하는 큰 어플리케이션을 만들기위해 사용된다. (by Facebook)  

이 문서에서 다룰 내용
- 리액트 컴포넌트
- 웹 페이지 데이터 렌더링
- 컴포넌트간 통신
- 유저 이벤트 핸들링
- 유저 입력을 잡는법
- 리모트 서버(API 서버)와 통신

React는 기본적으로 컴포넌트 베이스 아키텍처를 가진다.  

#### Virtual DOM ####
페이지에 어떤 변화가 있기 전에 리액트 컴포넌트에 의해 실제 DOM 엘리먼트의 인 메모리 표현.  

> Component -> Virtual DOM -> HTML  

Virtual DOM을 쓰는 이유?  
```html
<!-- 컴포넌트의 첫번째 렌더링 -->
<div>
  <p>Good Morning</p>
  <p>10:45AM</p>
</div>

<!-- 컴포넌트의 두번째 렌더링 -->
<div>
  <p>Good Morning</p>
  <p>10:55AM</p> <!-- 바뀐 부분 -->
</div>
```
두번째 렌더링에서 시간 부분만 바뀌었다. Virtual DOM을 이용하면 실제로 두번째 p 시간 부분만 바뀐다.  
즉, DOM의 차이(diffing)를 통해 DOM의 변화를 최소화하고 그러므로 브라우저 퍼포먼스 향상을 할 수 있다. (그만큼 변하는 부분이 최소화되기 때문)  

#### 첫번째 Component ####
리액트 컴포넌트는 React.Component 클래스를 베이스로 한다. (상속한다.)  
```javascript
// components.js
class StoryBox extends React.Component {
  render() { // 모든 컴포넌트는 render 메서드를 가지고 있다.
    return( <div>Story Box</div> );
  }
}

ReactDOM.render(
  <StoryBox />, document.getElementalById('story-app') // StoryBox를 호출한다.
);
```
기본 파일 구성은 다음과 같다.
```html
<!DOCTYPE html>
<html>
  <body>
    <div id="story-app"></div>
    <script src="vendors/react.js" charset="utf-8"></script>
    <script src="vendors/react-dom.js" charset="utf-8"></script>
    <script src="vendors/babel.js" charset="utf-8"></script> <!-- ES2015 -->
    <script type="text/babel" src="component.js"></script>
  </body>
</html>
```
순서를 살펴 보자.

1. index.html 을 열면 static HTML 페이지가 로드된다.
2. 각 스크립트 태그를 읽으면서 React 라이브러리와 커스텀 컴포넌트도 읽는다.
3. ReactDOM 렌더러는 컴포넌트를 렌더링 한다.
4. Virtual DOM이 생성되고 실제 DOM 엘리먼트로 바뀌어진다.  

#### JSX ####
Javascript XML  
위에서 js 파일 내에 작성한 html 코드는 JSX 코드이다. 해당 JSX 코드는 자바스크립트 코드로 변환된다. 아래와 같다.
```javascript
class StoryBox extends React.Component{
  render(){
    return(
      <div>
        <h3>Stories App</h3>
        <p className="lead">Sample paragraph</p> // JSX에서는 class가 아니라 className으로 사용한다.
      </div>
    );
  }
}

// return 메서드 안의 JSX는 다음과 같이 변환된다.
React.createElement( "div", null,
  React.createElement("h3", null, "Stories App"),
  React.createElement("p", {className: "lead"}, "Sample paragraph")
);
```
자바스크립트 코드로 변환후 브라우저에 렌더링되면 HTML을 생성한다.  

Date object를 이용해서 현재시간을 JSX에서 사용할 수 있다.  
map function을 사용하는 예도 같이 보자.
```javascript
class StoryBox extends React.Component{
  render(){

    const now = new Date();
    const topicsList = ['HTML', 'Javascript', 'React'];

    return(
      <div>
        <h3>Stories</h3>
        <p className="lead">
          Current time: {now.toTimeString()} // { 여기 안에 자바스크립트 코드를 쓰면 된다. }
        </p>
        <ul>
          {topicsList.map( topic => <li>{topic}</li>)} // 반복생성되는 리스트 태그를 만든다.
        </ul>
      </div>
    );
  }
}
```
- 즉, JSX는 HTML 처럼 생겼으나 Javascript 함수 호출로 변환되고 그 결과가 페이지에 반영이 되는 것이다.
- { 자바스크립트 코드를 쓸 수 있다. } in JSX
- 반복되는 요소에 대해 map을 쓰는것은 일반적인 패턴이다.  

#### props / Component in Component ####
위에서 설명했듯이 리액트는 컴포넌트를 최대한 구성해서 잘게 나누고 구성한다.
```html
<div class="comment-box"> <!-- CommentBox 컴포넌트 -->
  <h3>Comments</h3>
  <h4 class="comment-count">2 comments</h4>
  <div class="comment-list">
    <div class="comment"> <!-- Comment 컴포넌트 -->
      <p class="comment-header">Anne Droid</p>
      <p class="comment-body">
        I wanna know what love is...
      </p>
      <div class="comment-footer">
        <a href="#" class="comment-footer-delete">
          Delete comment
        </a>
      </div>
    </div>
  </div>
</div>
```
위에 html을 리액트 코드, JSX로 옮겨보자. html class는 JSX에서 className 이다.
```javascript
class Comment extends React.Component{
  render(){
    return(
      <div className="comment">
        <p className="comment-header">{this.props.author}</p> // 상위 컴포넌트에 전달받는다. props
        <p className="comment-body">
          {this.props.body} // 상위 컴포넌트에 전달받는다. props
        </p>
        <div className="comment-footer">
          <a href="#" className="comment-footer-delete">
            Delete comment
          </a>
        </div>
      </div>
    );
  }
}

<Comment /> // 이렇게 사용할 수 있다.
```
위에서 구현된 Comment를 CommentBox 안에 구성해야한다.
```javascript
class CommentBox extends React.Component {
  render() {
    return(
      <div className="comment-box">
        <h3>Comments</h3>
        <h4 className="comment-count">2 comments</h4>
        <div className="comment-list">
          <Comment
            author="토니" body="조흔 사진이군하!" /> // 하위 컴포넌트로 props 를 전달한다.
          <Comment
            author="라욘" body="ㅇㅈ? ㅇㅇㅈ" /> // 하위 컴포넌트로 props 를 전달한다.
        </div>
      </div>
    );
  }
}
```
props를 통해서 컴포넌트에 데이터를 인자로 넘겨줄 수 있다.  

하지만 위의 예에선 Comment 컴포넌트가 하드코딩되어서 동적으로 구성해보자.  
```javascript
class CommentBox extends React.Component {
  render() {
    const comments = this._getComments(); // 아래 정의된 메서드에서 컴포넌트 Array를 받는다.
    return(
      <div className="comment-box">
        <h3>Comments</h3>
        <h4 className="comment-count">{this._getcommentsTitle(comments.length)}</h4>
        <div className="comment-list">
          {comments}
        </div>
      </div>
    );
  }

  _getComments() { // React.Component에서 상속받은 메서드 외에 구별하기 위해 Underscore를 사용한다.
    const commentList = [ // API Server 에서 보통 이런식으로 받게 된다.
      { id: 1, author: '토니', body: '조흔 사진이군하!' },
      { id: 2, author: '라욘', body: 'ㅇㅈ? ㅇㅇㅈ' }
    ];

    // 각 코멘트리스트를 Comment 컴포넌트에 할당해서 Object Array로 반환한다.
    return commentList.map( (comment) => {
        return(
          <Comment
            author={comment.author} body={comment.body} key={comment.id}/> //key는 유니크 키, 유니크 키를 명세하면 같은 타입 컴포넌트를 여러개 만들때 성능 개선을 도와준다.
        );
    });
  }

  _getcommentsTitle(commentCount){ // 코멘트 갯수를 예외처리해서 출력해준다.
    if (commentCount === 0) {
      return 'No comment yet';
    } else if (commentCount === 1){
      return '1 comment';
    } else {
      return `${commentCount} comments`;
    }
  }
}
```

#### state ####
예를들어 상태에 따라서 댓글들을 보여주거나 숨기거나 해야할때 toggle 상태를 어떻게 컨트롤할 것인가?  

DOM을 조작할 수 있는 방법에는 직접적인 방법과 간접적인 방법이 있는데 React는 간접적으로 조작한다.  
직접적으로 DOM을 수정할 필요없이 컴포넌트의 state 객체를 유저 이벤트 반응과 DOM 업데이트 핸들링에 따라서 조작한다.

> Events --> Update state --> DOM updates

예를들어 다음과 같이 사용한다.
```javascript
render() {
  if (this.state.showComments){
    // code displaying comments
  } else {
    // code displaying comments
  }
}
```
구현된 컴포넌트에서 구체적으로 어떻게 사용되는지 살펴보자. state는 각 컴포넌트 안에 객체로 존재하고 this.state로 접근할 수 있다.
```javascript
class CommentBox extends React.Component {
  //...
  render(){
    const comments = this._getComments();
    if(this.state.showComments){
      // add code for displaying comments
    }
    return(
      <div className="comment-box">
        <h3>Comments</h3>
        <h4 className="comment-count">{this._getcommentsTitle(comments.length)}</h4>
        <div className="comment-list">{comments}</div>
      </div>
    );
  }
  //...
}
```
위 코드를 state가 true일때 comments가 보이게끔 수정해보자. 그리고 state의 초기 상태(Initial State)를 컴포넌트 생성자 함수에 세팅하자.
```javascript
class CommentBox extends React.Component {
  constructor() {
    super();

    this.state = {
      showComments: false // 초기 state는 false이고 comments를 가리게 된다.
    };
  }

  render(){
    const comments = this._getComments();
    let commentNodes;
    if(this.state.showComments){
      commentNodes = <div className="comment-list">{comments}</div>;
    }
    return(
      <div className="comment-box">
        <h3>Comments</h3>
        <h4 className="comment-count">{this._getcommentsTitle(comments.length)}</h4>
        {commentNodes}
      </div>
    );
  }
  //...
}
```
state를 어떻게 업데이트할 수 있을까? 직접적으로 객체에 접근할 수는 없고 setState를 통해서 객체를 넘겨주어야한다.
```javascript
this.state.showComments = true; // XX 잘못된 방법..

// 아래와 같이 해야한다.
this.setState({showComments: true});
```
setState를 통하면 인자를 통해서 state 객체의 프로퍼티를 업데이트하는 것이고 state 객체 전체를 바꾸지 않는다. setState가 호출되면 다시 렌더링(re-render)된다.  

그러면 State는 언제 바뀌는 것일까? 다음과 같다.
- 버튼 클릭
- 링크 클릭
- 폼 제출
- AJAX 요청
- 기타 등등..

클릭 이벤트를 통해서 이벤트가 toggle되는 것을 구현해보자.
```javascript
class CommentBox extends React.Component {
  render() {
    //..
    let buttonText = 'Show comments';

    if (this.state.showComments) {
      buttonText = 'Hide comments';
      //..
    }
    //..
    return (
      //...
        <button onClick={this._handleClick.bind(this)}>{buttonText}</button>
      //...
    );
  }

  _handleClick(){
    this.setState({
      showComments: !this.state.showComments // state를 toggle 시킨다.
    });
  }
}
```

#### Event ####
Form을 통해서 Submission하는 컴포넌트를 리액트로 만들어보자.
```javascript
class CommentForm extends React.Component {
  render() {
    return (
      <form className="comment-form" onSubmit={this._handleSubmit.bind(this)}> // 이벤트 리스터를 submit 이벤트에 추가한다. 이벤트 핸들러는 무조건 이 방식으로만 바인딩해야한다.
        <label>Join the discussion</label>
        <div className="comment-form-fields">
          <input placeholder="Name:" ref={ (input) => this._author = input } /> // React는 render 할때 ref 콜백이 동작한다
          <textarea placeholder="Comment:" ref={ (textarea) => this._body = textarea } > // ref를 사용해서 input 요소에 접근
          </textarea>
        </div>
        <div className="comment-form-actions">
          <button type="submit">
            Post comment
          </button>
        </div>
      </form>
    );
  }

  _handleSubmit(event){
    event.preventDefault(); // 페이지가 리로드되는걸 방지

    let author = this._author; // JSX 에서 전달된 ref로 부터 온다.
    let body = this._body;

    this.props.addComment(author.value, body.value); // 이 메서드로 인자 전달
  }
}

<input placeholder="Name:" ref={ (input) => this._author = input } />
// 위 코드는 아래와 같다.
<input placeholder="Name:" ref={
                                  function(input) {  // DOM 요소가 콜백으로 전달
                                    this._author = input; // _author 이름의 프로퍼티 클래스를 새로 만든다.
                                  }.bind(this) // 여기서 this는 CommentForm을 레퍼런싱한다.
                                } />
// React는 render 할때 ref 콜백이 동작한다.
```
CommentForm 이 새로운 Comment 데이터를 CommentBox 가 가지고 있는 comments array에 업데이트 되어야한다.  
CommentForm은 데이터를 콜백 prop으로 넘겨주고 Commentbox는 array를 state에서 가지고 있다.  

```javascript
class CommentBox extends React.Component {
  constructor() {
    super();

    this.state = {
      showComments: false,
      comments: [
        { id: 1, author: '토니', body: "오 쳅"},
        { id: 2, author: '듀크', body: "체바~"}
      ]
    };
  }

  render() {
    return(
      <div className="comment-box">
        // 자바스크립트 함수는 first-class citizens 로 다른 컴포넌트에 prop 로써 넘길수 있다.
        // 함수를 변수처럼 넘긴다는 얘기
        <CommentForm addComment={this._addComment.bind(this)} /> // 콜백 prop을 전달
        //...
      </div>
    );
  }

  _addComment(author, body){
    const comment = {
      id: this.state.comments.length + 1,
      author,
      body
    };
    this.setState({comments: this.state.comments.concat([comment])}); // comments state를 concat을 이용해서 덧붙여 준다. 여기서 concat이 push 보다 더 좋게 동작한다. 배열 레퍼런스를 유지해서 넘겨줌.
  }

  _getComments() {
    return this.state.comments.map((comment)=> {
      return (
        <Comment
          author={comment.author}
          body={comment.body}
          key={comment.id} />
      );
    });
  }
}
```

onSubmit 이벤트는 Synthetic Event 이다. 브라우저마다 submit 이벤트 명명규칙이 다른데 리액트는 브라우저 동작을 래핑해서 알아서 변환해준다.  
https://facebook.github.io/react/docs/events.html 참고  

#### Lifecycle Methods ####
앞서 CommentBox의 comments는 하드코딩되었었는데 실제 어플리케이션에서는 리모트 API 서버에서 받아오게 된다.  
API 통신을 위해 AJAX를 이용하는데 jquery가 필요하다.  
아래와 같이 구현된다.
```javascript
class CommentBox extends React.Component {
  //...
  _fetchComments() {
    jQuery.ajax(
      method: 'GET',
      url: '/api/comments',
      success: (comments) => { // arrow function을 쓰면 클래스의 this에 바인딩된다. ES2015 참고.
        this.setState({ comments })
      } // 서버에서 받아온 comments 배열을 state에 세팅해준다.
    );
  }
}
```
위와 같이 구성을 하더라도 render() 메서드 이전에 함수가 호출되어야한다. 리액트의 라이프사이클 메서드들을 살펴보자.  
- constructor()
- componentWillMount()
- render()
- componentDidMount()
- componentWillUnMount()

https://facebook.github.io/react/docs/component-specs.html#lifecycle-methods 참고  
라이프사이클 메서드를 이용해서 적절하게 호출해보자.
```javascript
class CommentBox extends React.Component {
  //...
  componentWillMount(){
    _fetchComments(); // 컴포넌트가 렌더링되기 전에 서버로부터 comments를 패치받는다.
  }

  _fetchComments() {
    jQuery.ajax(
      method: 'GET',
      url: '/api/comments',
      success: (comments) => {
        this.setState({ comments })
      }
    );
  }
}
```
업데이트 사항이 있다면 계속해서 업데이트해주어야하는데 주기적으로 서버에 요청을 하는 것을 polling이라고 한다.  
componentDidMount는 render 호출 후에 동작한다.
```javascript
class CommentBox extends React.Component {
  //...
  componentDidMount(){
    setInterval(() => this._fetchComments(), 5000); // 5초마다 서버에게 polling 한다.
  }
}
```
리액트는 변화가 있을때만 DOM을 업데이트한다. 첫번째 AJAX 요청으로 state value가 바뀌면 DOM은 변한다.  
두번째 AJAX 요청으로 state value가 변화가 없으면 DOM도 바뀌지 않는다.  

페이지가 바뀔때 메모리 누수가 일어난다. 예를들어 commentBox가 화면마다 있을거고 그때마다 componentDidMount 에 의해 타이머가 동작하고 서버에 요청을 한다. 5초마다. 페이지가 계속 바뀔때마다 그게 중첩되서 쌓여서 결국 메모리 누수가 일어날 수 있다.  
그래서 componentWillUnMount()를 통해서 타이머(interval 요청)를 해제해줄 수 있다.
```javascript
class CommentBox extends React.Component {
  //...
  componentDidMount(){
    this._timer = setInterval(
      () => this._fetchComments(), 5000
    );
  }

  componentWillUnMount(){
    clearInterval(this._timer);
  }
}
```
Comment를 불러오는 과정을 다시 보면 아래와 같다.  

1. componentWillMount() 호출  
2. render() 호출 및 commentBox 마운트  
3. Component는 API 응답을 기다리다가 받아서 setState()를 호출하고 render()를 호출  
4. componentDidMount() 호출되고 5초마다 this.fetchComments() 호출된다.  
5. componentWillUnMount() 는 DOM으로부터 막 제거될때 호출되고 fetchComments 인터벌은 클리어(해제)된다.  

Comment를 삭제하는 액션을 살펴보자  
```javascript
class CommentBox extends React.Component{
  //...
  _deleteComment(comment) {
    jQuery.ajax({
      method: 'DELETE',
      url: `/api/comments/${comment.id}`
    });


    // comments를 가져와서 잘라내고 다시 업데이트 해주는 작업
    const comments = [...this.state.comments];
    const commentIndex = comments.indexOf(comment);
    comments.splice(commentIndex, 1); // 해당 comment를 array에서 삭제

    this.setState({ comments });
  }

  _getComments() {
    return this.state.comments.map(comment => {
        return(
          <Comment
            comment={comment}
            key={comment.id}
            onDelete={this._deleteComment.bind(this)} />  // Comment 컴포넌트에 정의된 이벤트 핸들러를 prop 으로 전달
        );
    });
  }
}

class Comment extends React.Component {
  render() {
    return(
      //...
        <a href="#" onClick={this._handleDelete.bind(this)}>
          Delete comment
        </a>
      //...
    );
  }

  _handleDelete(event){
    event.preventDefault();
    if(confirm('Are you sure?')) {
      this.props.onDelete(this.props.comment); // 상위 컴포넌트에 전달받은 이벤트를 호출한다.  
    }
  }
}
```
Delete comment 링크를 클릭하면 handleDelete가 호출되고 그 안에서 CommentBox에서 정의한 onDelete 메서드가 호출되면서 comments를 업데이트한다.  

이제 더하는 액션을 살펴보자.
```javascript
class CommentBox extends React.Component {
  //...
  _addComment(author, body) {
    const comment = { author, body };

    jQuery.post('/api/comments', { comment })
      .success(newComment => {
        this.setState({ comments: this.state.comments.concat([newComment]) });
      }); // post api 요청이 성공하면 state를 업데이트 해준다.
  }
}
```
결국 흐름이 위에서 아래로 한방향으로 흐르는 것을 알 수 있다.   

1. CommentBox가 deleteComment 메서드를 콜백으로 Comment 컴포넌트에 넘겨주고 author, body 같은 각 코멘트의 prop도 넘겨준다.  
2. CommentBox가 addComment 메서드를 콜백으로 CommentForm 컴포넌트에 넘겨주고 CommentForm 에서 submit 이벤트가 발생했을때 해당 콜백을 호출한면 된다.  

[reference](https://www.dropbox.com/s/1zabzrnfqzamcq2/CodeSchool-PoweringUpWithReact.pdf?dl=0)
