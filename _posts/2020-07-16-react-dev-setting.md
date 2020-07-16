---
title: React 개념정리, 개발 환경 구축, 빌드에서 배포까지
tags: [front, react]
comments: true
categories: react
header:
  teaser: "/assets/images/react.png"
---
# 1. React overview

* Virtual DOM 사용으로 인해 속도가 빠릅니다.

* React는 모든 것이 **컴포넌트**입니다. 

* 데이터를 전달할 때 **부모에서 자식에게로만 데이터가 전달이 가능**합니다.

* React 개발에서는 ES6문법을 사용하고있습니다.



<br/>

# 2. React 사용시 알고 있어야 할 ECMAScript 6 요약

#### 2-1. var

```javascript
var a = 100;
if(a > 0){
    var a = 200;
    console.log(a); // 200출력
}
console.log(a); // 200 출력
```

<br/>

#### 2-2. let

```javascript
let a = 100;
if(a > 0){
    let a = 200;
    console.log(a); // 200출력
}
console.log(a); // 100 출력
```

<br/>

#### 2-3. const

```js
const MY_NAME; // 초기값 할당하지 않아 SyntaxError발생
const MY_NAME = 'Kim';
MY_NAME = 'Lee'; // 값을 변경하려 하면 TypeError발생
```

<br/>

#### 2-4. arrow function

```javascript
// Arrow function
// 기존 함수 구문
var add = function(a, b){
    return a + b;
};

// 화살표 함수 구문
let add = (a, b) => {
    return a + b;
}
```

<br/>

블록 구문 `{}`을 생략한 표현식 사용은 가능 하지만 이 경우 `return`은 사용 불가능합니다.

```javascript
// 블록 구문 사용
let add = (a, b) => {
   console.log(a + b);
}

// 블록 구문 생략
let add = (a, b) => console.log(a + b);
```

<br/>

단일 인자만 넘겨받는 경우 `{}`괄호 생략도 가능합니다.

```javascript
// 괄호 사용
let print = (message) => document.write(message);

// 괄호 생략
let print = message => document.write(message);
```



<br/>

#### 2-5. 펼침 연산자(spread operator)

```javascript
let a = [1,2,3]
let b = a
b.push(4);
console.log(a)//[1,2,3,4]
console.log(b)//[1,2,3,4]
```



위 예시처럼 배열의 모든 데이터가 교체되는 일을 리소스를 방지하기 위해 나온 문법이

**펼침 연산자**로, 데이터 불변성과 관련있습니다.



```javascript
let a = [1,2,3]
let b = [...a, 4]
console.log(a) //[1,2,3]
console.log(b) //[1,2,3,4]
```

<br/>

#### 2-6. 클래스 (class)

class를 사용하여 Prototype을 사용하지 않고도 간단하게 상속을 사용할 수 있게 되었습니다.

```javascript
// class키워드 뒤에 클래스명 붙여 선언하고 블록 안쪽에 구문 작성
class Display {
    print() {
        console.log('print');
    }
}

const display = new Display();
display.print();
```

<br/>

#### 2-7. 상속

ES6에서는 extends키워드를 사용하여 보다 쉽게 상속을 구현할 수 있는데, 

말 그대로 부모(상위) 클래스의 속성이나 기능을 자식에게 전달한다는 의미입니다.

```javascript
class Display{
    constructor(x, y){
        this.x = x;
        this.y = y;
    }
}
```

<br/>

선언된 Display 클래스를 React클래스 선언문 뒤에 Extends를 붙여 Display클래스를 상속 받고 있습니다.  

```javascript
class React extends Display {
    constructor(x, y, width, height) {
		// 부모클래스의 생성자로 자식클래스에서 생성자 호출 시
		// 부모 클래스가 초기화 되도록 강제적으로 super를 호출됩니다.
        super(x, y);
        this.width = width;
        this.height = height;
    }
}
```

<br/>

# 3. React JSX

리액트에서는 JSX문법을 사용하는데, 이는 페이스북에서 만든 것으로 HTML과 비슷하게 생기고 

비슷하게 사용하지만 전혀 다른 문법입니다.



JSX에서는 꼭 지켜야할 규칙들이 몇가지 있습니다.



<br/>

#### 3-1. 닫혀야 하는 태그

```javascript
import React, { Component } from 'react';

class App extends Component {
	render() {
   		return (
			<div>
            	<p>Hello world // 에러발생
      			<p>Hello world</p> // 정상작동
      			<input type='text' > 에러발생
      			<input type='text' /> // 정상작동
			</div>
   		);
	}
}

export default App;
```

<br/>

#### 3-2. 감싸져 있는 엘리먼트

두개 이상의 엘리먼트는 무조건 하나의 엘리먼트로 감싸져있어야 합니다.

<br/>

div 로 해결한 경우

```javascript
import React, { Component } from ‘react’;

class App extends Component {
	render() {
    	return (
      		<div>
        		<p> Hello world</p>
        		<p> Have a Nice day :)</p>
      		<div>
    	);
  	}
}

export default App;
```

<br/>

Fragment 로 해결한 경우

```javascript
// 리액트를 불러올 때 Component와 함께 Fragment도 불러와야 함
import React, { Component, Fragment } from ‘react’;

class App extends Component {
	render() {
    	return (
      		<Fragment>
        		<p> Hello world</p>
        		<p> Have a Nice day :)</p>
      		</Fragment>
    	);
  	}
}

export default App;
```

<br/>

#### 3-3. JSX 안에 자바스크립트 값 사용

```javascript
import React, { Component } from 'react';

class App extends Component {
  render() {
    let name = 'react';
    return (
      <div>
        <p>Hi {name}!</p>
      </div>
    );
  }
}

export default App;
```

<br/>

#### 3-4. CSS 작성 : 인라인 스타일

리액트는 자바스크립트로 작성하기 때문에 아래처럼 style속성값에 일반 문자열이 아닌

자바스크립트 객체가 할당되어야 합니다.



그리고 font-size와 같이 중간에 대시 기호(-)가 들어간 속성명은 fontSize와 같이 카멜케이스로 바꿔줘야합니다. 

(유지보수나 성능의 이슈가 있어 권장되지 않는 방법입니다.)



```javascript
import React, { Component } from 'react';

class App extends Component {
    render() {
	    const styles = {
	        backgroundColor: 'black',
	        padding: '16px',
	        color: 'white',
	        fontSize: '12px'
	    };
	
	    return (
	        <div style={styles}>
	        Hello World
	        </div>
	    );
    }
}

export default App;
```

<br/>

#### 3-5. CSS 작성 : 외부파일 불러오기

별도의 파일에 스타일을 정의해놓고, 리액트 컴포넌트 파일에서 정의한 css파일을 import합니다.

```javascript
import './App.css' // 불러오기
```

<br/>



#### 3-6. html에서의 class는 className

```javascript
return (
	<div className="App">
    	Hello World
  	</div>
);
```

React DOM은 HTML 어트리뷰트(attribute) 이름 대신 캐멀케이(camelCase)를 네이밍 컨벤션으로 사용합니다. 

예를 들어, JSX에서 tabindex는 tabIndex로 작성합니다.

**class 어트리뷰트는 JavaScript의 예약어이므로 className으로 작성**합니다.

<br/>

# 4. React 개발환경구축 

#### 4-1. yum ssl verify 비활성화

호스트에 정상적으로 인증서가 설정되어 있지 않는 경우에만 진행합니다.

/etc/yum.conf 파일을 편집하여 `sslverify=false` 항목을 추가함.

```ini
[main]
cachedir=/var/cache/yum/$basearch/$releasever
keepcache=0
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
sslverify=false
installonly_limit=5
```

<br/>

#### 4-2. nodejs 설치

```sh
# yum -y install nodejs
```

<br/>

#### 4-3. npm ssl 비활성화

호스트에 정상적으로 인증서가 설정되어 있지 않는 경우에만 진행합니다.

```sh
# npm config set strict-ssl false
```

<br/>

#### 4-4.  create-react-app 설치

```sh
# npm install -g create-react-app
```

<br/>

#### 4-5. node, npm, create-react-app 버전 확인

```sh
# node -v
v10.21.0

# npm -v
6.14.4

# create-react-app --version
3.4.1
```

<br/>

# 5. React render

ReactDOM.render(element, container) 함수를 사용하여, 

container에 element를 출력할 수 있습니다.



<br/>

#### 5-1. React render 프로젝트 생성

```sh
# cd /tmp

# create-react-app render
[  ................] | fetchMetadata: sill resolveWithNewModule scheduler@0.19.1 checking installable status

# cd render
```



이렇게 생성된 틀을 살펴보면, 

index.html 의 div 태그 안에 결과가 표시되며,

해당 결과는 각각 분리된 컴퍼넌트로 개발이 진행됩니다.



![react_dev_env_setting](/assets/images/react_dev_flow_div.png)



<br/>

#### 5-2. /render/public/index.html 

index.html 파일에 위치하고 있는 div 태그안에 최종 결과물이 출력됩니다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
    <!--
      manifest.json provides metadata used when your web app is installed on a
      user's mobile device or desktop. See https://developers.google.com/web/fundamentals/web-app-manifest/
    -->
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <!--
      Notice the use of %PUBLIC_URL% in the tags above.
      It will be replaced with the URL of the `public` folder during the build.
      Only files inside the `public` folder can be referenced from the HTML.

      Unlike "/favicon.ico" or "favicon.ico", "%PUBLIC_URL%/favicon.ico" will
      work correctly both with client-side routing and a non-root public URL.
      Learn how to configure a non-root public URL by running `npm run build`.
    -->
    <title>React App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
    <!--
      This HTML file is a template.
      If you open it directly in the browser, you will see an empty page.

      You can add webfonts, meta tags, or analytics to this file.
      The build step will place the bundled scripts into the <body> tag.

      To begin the development, run `npm start` or `yarn start`.
      To create a production bundle, use `npm run build` or `yarn build`.
    -->
  </body>
</html>
```



<br/>

#### 5-3. /render/src/index.js

각각의 컴퍼넌트(예제에서는 App, Footer 컴퍼넌트)가 명시되며,

최종 출력될 위치는 2번째 인자로 설정합니다.

<br/>

App 컴퍼넌트와 Footer 컴퍼넌트를 사용하기 위해서 import 합니다.

```react
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import Footer from './Footer';
import * as serviceWorker from './serviceWorker';

ReactDOM.render(
	<React.StrictMode>
    	<App />
    	<Footer />
  	</React.StrictMode>,
	document.getElementById('root')
);

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();
```



<br/>

#### 5-4. /render/src/App.js

App 컴퍼넌트에 랜더링될 내용을 기록합니다.

```react
import React from 'react';
import logo from './logo.svg';
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <img src={logo} className="App-logo" alt="logo" />
        <p>
          Edit <code>src/App.js</code> and save to reload.
        </p>
        <a
          className="App-link"
          href="https://reactjs.org"
          target="_blank"
          rel="noopener noreferrer"
        >
          Learn React
        </a>
        <h3>이곳에 필요한 부분을 수정합니다.</h3>
      </header>
    </div>
  );
}

export default App;
```

<br/>

#### 5-5. /render/src/Footer.js

Footer 컴퍼넌트에 랜더링될 내용을 기록합니다.

```react
import React from 'react';

function Footer() {
	return (
    	<div className="Footer">
        	<h1> 이곳에 Footer 를 기록합니다. </h1>
		</div>
	);
}

export default Footer;
```

<br/>



#### 5-6. 결과확인

```sh
# npm start
Starting the development server...
Compiled successfully!

You can now view render in the browser.

  Local:            http://localhost:3000
  On Your Network:  http://10.0.2.15:3000

Note that the development build is not optimized.
To create a production build, use npm run build.
```

![react_dev_env_setting](/assets/images/react_dev_env_setting.png)

<br/>



# 6. React deploy

#### 6-1. React render build

```sh
# cd /tmp/render/
# npm run build
```

<br/>

#### 6-2. react app 배포

```sh
# cp -rf build/favicon.ico  	$NGINX_HOME/html/
# cp -rf build/index.html   	$NGINX_HOME/html/
# cp -rf build/logo192.png  	$NGINX_HOME/html/
# cp -rf build/logo512.png   	$NGINX_HOME/html/
# cp -rf build/static/js/*   	$NGINX_HOME/html/js/
# cp -rf build/static/css/*  	$NGINX_HOME/html/css/
# cp -rf build/static/media/*  	$NGINX_HOME/html/media/
```

<br/>

#### 6-3. nginx reload

```sh
# nginx.exe -s stop -c conf/nginx.conf
# nginx.exe -s reload -c conf/nging.conf
# nginx.exe -c conf/nginx.conf
```

<br/>



# 7. React props 

* props는 단방향 데이터 바인딩

* props는 readonly (즉, 불변객체)

* props는 타입지정 가능

* props는 required 지정 가능

* props는 기본값 설정 가능

  

<br/>

#### 7-1. React props 프로젝트 생성

```sh
# create-react-app props
# cd props
```

<br/>

#### 7-2. props/src/index.js

user와 age 변수는 읽기전용으로 App 컴퍼넌트로 전달됩니다.

```react
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import * as serviceWorker from './serviceWorker';

ReactDOM.render(
  <React.StrictMode>
    <App />        
    <App user="james" age="20" />        
  </React.StrictMode>,
  document.getElementById('root')
);

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();
```

<br/>

#### 7-3. props/src/App.js 

앞에서 전달받은 user, age 변수는 this.props.user, this.props.age 와 같은 형태로 접근이 가능하며,

기본값 또한 다음과 같이 설정할 수 있습니다.

```react
import React from 'react';

class App extends React.Component {
    constructor(props) {
        super(props);
    }

    render() {
        return (
            <div>
                <h1>Hello {this.props.user || 'default name'}, {this.props.age || 19} </h1>
            </div>
        );
    }
}

export default App;
```

<br/>



#### 7-4. React props 결과 확인

```sh
# npm start
```

![react_dev_env_setting](/assets/images/react/react_props.png)

<br/>



# 8. React state

#### 8-1. React state 프로젝트 생성

```sh
# cd /tmp
# create-react-app state
# cd state
```

<br/>

#### 8-2. state/src/App.js 

```react
import React from 'react';

class App extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            clickCount: 0
        }
        
        this.myClick = this.myClick.bind(this);
    }
    
    myClick() {
        this.setState({
            clickCount: this.state.clickCount + 1
        });
    }
    
    render() {
        return (
            <div>
				<input type="button" value="Click!" onClick={this.myClick}/><br/>
                <p>{this.state.clickCount}</p>
            </div>
        );
    }
}

export default App;
```



<br/>

#### 8-3. React state 결과 확인

```sh
# npm start
```

![react_dev_env_setting](/assets/images/react/react_state.png)

<br/>







# 9. React button

#### 9-1. React button 프로젝트 생성

```sh
# cd /tmp
# create-react-app button
# cd button
```

<br/>

#### 9-2. button/src/App.js 

react 에서는 로컬 변수를 읽기 전용 변수인 props와 읽기+쓰기가 가능한 state 변수로 

구분하여 사용을 합니다. 

```react
import React from 'react';

class App extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            // 값 변경이 가능하게끔 선언된 defaultAlertMsg 로컬 변수
            defaultAlertMsg: 'default message'
        }
        this.showAlert = this.showAlert.bind(this);
    }

    showAlert(msg) {
        alert(msg);
    }

    render() {
        return (
            <div>
                <button onClick={e => this.showAlert(this.state.defaultAlertMsg)}>click me</button>
            </div>
        );
    }
}

export default App;
```

<br/>



#### 9-3. React button 결과 확인

```sh
# npm start
```

![react_dev_env_setting](/assets/images/react/react_button.png)



<br/>

# 10. React life cycle

컴퍼넌트가 load되거나, unload 되는 시점에 특정 행위가 필요한 경우,

React life cycle를 활용할 수 있습니다.

* componentDidMount : 컴퍼넌트가 load 되었을 때 호출됨.
* componentWillUnmount : 컴퍼넌트가 unload 되었을 때 호출됨.

<br/>

#### 10-1. React life cycle 프로젝트 생성

```sh
# cd /tmp
# create-react-app life
# cd life
```

<br/>

#### 10-2. life/src/App.js

```react
import React from 'react';

class App extends React.Component {
    constructor(props) {
        super(props);
        this.state = {date: new Date()};
    }

    componentDidMount() {
        this.timerID = setInterval(() => this.tick(), 1000);
    }

    componentWillUnmount() {
        clearInterval(this.timerID);
    }

    tick() {
        this.setState({
            date: new Date()
        });
    }

    render() {
        return (
            <div>
                <h1>Hello, world!</h1>
                <h2>It is {this.state.date.toLocaleTimeString()}</h2>
            </div>
        );
    }
}

export default App;
```

<br/>

#### 10-3. React life cycle 실행 결과 확인

```sh
# npm start
```

![react_dev_env_setting](/assets/images/react/react_life.png)

<br/>



# 11. React redux 

redux 에서는 action을 dispatch하여 상태값을 변경할 수 있습니다.

이 때, 다음과 같은 함수들이 사용이 됩니다.

* **mapStateProps** : state를 props로 연결해 주는 함수
* **mapDispatchToProps** : dispatch한 함수를 props로 연결해 주는 함수
* **mergeProps** : state와 dispatch를 파라미터로 가진 함수, 두 개 동시에 사용할 때 사용하는 함수.

<br />

그리고, Reducer는 순수 함수로만 작성되어야 합니다.

- 외부 네트워크 혹은 데이터베이스에 접근하지 않아야 합니다.
- return 값은 오직 parameter 값에만 의존되어야 합니다.
- 인수는 변경되지 않아야 합니다.
- 같은 인수로 실행된 함수는 언제나 같은 결과를 반환해야 합니다.
- 순수하지 않은 API 호출을 하지 말아야 합니다. (Date 및 Math 의 함수 등)



<br/>

React redux의 구성요소

* Action : 사용자와 상호작용
* Dispatcher : 이벤트 발생시 호출될 Callback 등록
* Store : 상태와 로직을 담고 있으며, 상태를 변경하려면, Action -> Dispatch 과정을 통해야 함.



<br/>

#### 11-1. React redux 프로젝트 생성

```sh
# cd /tmp
# create-react-app react-redux-example
# cd react-redux-example
# npm install --save redux react-redux
# rm -f logo.svg index.css App.css App.test.js setupTest.js 
```

<br />



#### 11-2. /src/index.js

```react
import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';
import { Provider  } from 'react-redux';
import App from './components/App';
import counterApp from './reducers';
import * as serviceWorker from './serviceWorker';

// store 를 만들고 counterApp reducer와 연동함.
const store = createStore(counterApp);

ReactDOM.render(
    <Provider store = {store}>
        <App />
    </Provider>,
    document.getElementById('root')
);

// If you want your app to work offline and load faster, you can change
// unregister() to register() below. Note this comes with some pitfalls.
// Learn more about service workers: https://bit.ly/CRA-PWA
serviceWorker.unregister();
```

<br />

#### 11-3. /src/actions/index.js

action은 어떤 변화가 일어나야 할지 알려주는 객체

```react
export const INCREMENT = 'INCREMENT';
export const DECREMENT = 'DECREMENT';

export function increment() {
    return {
        type: INCREMENT
    };
}

export function decrement() {
    return {
        type: DECREMENT
    };
}
```

<br />

#### 11-4. /src/components/App.js

```react
import React from 'react';
import { connect } from 'react-redux';
import { increment, decrement } from '../actions';

class App extends React.Component {
    render(){
        return (
            <div style={ {textAlign: 'center'} }>
                <h1>VALUE: { this.props.value }</h1>
                <button type="button" onClick={ this.props.onIncrement }>+</button>
                <button type="button" onClick={ this.props.onDecrement }>-</button>
            </div>
        );
    }
}

// state를 props에 연결함.
// 접근시에 this.props.value 와 같이 사용함.
let mapStateToProps = (state) => {
    return {
        value: state.counter.value
    };
}

// dispatch 함수를 props에 연결함.
// 접근시에 this.props.onIncrement 와 같이 사용함.
let mapDispatchToProps = (dispatch) => {
    return {
        onIncrement: () => dispatch(increment()),
        onDecrement: () => dispatch(decrement())
    }
}

App = connect(mapStateToProps, mapDispatchToProps)(App);

export default App;
```

<br />

#### 11-5. /src/reducers/index.js

```react
import { INCREMENT, DECREMENT } from '../actions';
import { combineReducers } from 'redux';

const counterInitialState = {
    value: 0
};

const counter = (state = counterInitialState, action) => {
    switch(action.type) {
        case INCREMENT:
            return Object.assign({}, state, {
                value: state.value + 1
            });
        case DECREMENT:
            return Object.assign({}, state, {
                value: state.value - 1
            });
        default:
            return state;
    }
};

// combineReducers는 reducer가 여러개 있다면, 하나로 합쳐주는 메소드
const counterApp = combineReducers({
    counter
});

export default counterApp;
```



<br />

#### 11-7. React redux 실행 결과 확인

```sh
# cd /tmp/react-redux-example/
# npm start
```

![react_dev_env_setting](/assets/images/react/react_redux.png)

<br />





