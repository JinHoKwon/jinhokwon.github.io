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



#### 2-2. let

```javascript
let a = 100;
if(a > 0){
    let a = 200;
    console.log(a); // 200출력
}
console.log(a); // 100 출력
```



#### 2-3. const

```js
const MY_NAME; // 초기값 할당하지 않아 SyntaxError발생
const MY_NAME = 'Kim';
MY_NAME = 'Lee'; // 값을 변경하려 하면 TypeError발생
```



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



블록 구문 `{}`을 생략한 표현식 사용은 가능 하지만 이 경우 `return`은 사용 불가능합니다.

```javascript
// 블록 구문 사용
let add = (a, b) => {
   console.log(a + b);
}

// 블록 구문 생략
let add = (a, b) => console.log(a + b);
```



단일 인자만 넘겨받는 경우 `{}`괄호 생략도 가능합니다.

```javascript
// 괄호 사용
let print = (message) => document.write(message);

// 괄호 생략
let print = message => document.write(message);
```





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



# 3. React JSX

리액트에서는 JSX문법을 사용하는데, 이는 페이스북에서 만든 것으로 HTML과 비슷하게 생기고 

비슷하게 사용하지만 전혀 다른 문법입니다.



JSX에서는 꼭 지켜야할 규칙들이 몇가지 있습니다.





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



#### 3-2. 감싸져 있는 엘리먼트

두개 이상의 엘리먼트는 무조건 하나의 엘리먼트로 감싸져있어야 합니다.



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



#### 3-4. CSS 작성 : 인라인 스타일

**리액트는 자바스크립트로 작성하기 때문에 아래처럼 style속성값에 일반 문자열이 아닌 **

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



#### 3-5. CSS 작성 : 외부파일 불러오기

별도의 파일에 스타일을 정의해놓고, 리액트 컴포넌트 파일에서 정의한 css파일을 import합니다.

```javascript
import './App.css' // 불러오기
```





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



#### 4-2. nodejs 설치

```sh
# yum -y install nodejs
```



#### 4-3. npm ssl 비활성화

호스트에 정상적으로 인증서가 설정되어 있지 않는 경우에만 진행합니다.

```sh
# npm config set strict-ssl false
```



#### 4-4.  create-react-app 설치

```sh
# npm install -g create-react-app
```



#### 4-5. node, npm, create-react-app 버전 확인

```sh
# node -v
# npm -v
# create-react-app --version
```



# 5. React render

ReactDOM.render(element, container) 함수를 사용하여, 

container에 element를 출력할 수 있습니다.





#### 5-1. React render 프로젝트 생성

```sh
# cd /tmp
# create-react-app render
# cd render
```



![react_dev_env_setting](/assets/images/react_dev_flow_div.png)





#### 5-2. /render/public/index.html 

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





#### 5-3. /render/src/index.js

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





#### 5-4. /render/src/App.js

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



#### 5-5. /render/src/Footer.js

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





#### 5-6. 결과확인

```sh
# npm start
```

![react_dev_env_setting](/assets/images/react_dev_env_setting.png)





# 6. React deploy

#### 6-1. React render build

```sh
# cd /tmp/render/
# npm run build
```



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



#### 6-3. nginx reload

```sh
# nginx.exe -s stop -c conf/nginx.conf
# nginx.exe -s reload -c conf/nging.conf
# nginx.exe -c conf/nginx.conf
```
