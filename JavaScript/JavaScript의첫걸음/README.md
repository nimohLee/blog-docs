### JavaScript
JavaScript는 웹(앱)에 상태를 부여시켜주는 언어를 말한다. 즉, HTML CSS로 구성된 정적인 UI를 조작(?)할 수 있는 언어이다. HTML과 CSS로만 이루어진 것은 단순히 파일, 문서에 불과할 뿐 웹 혹은 웹앱이라 할 수없다. 즉 웹(앱)으로 작동하기위해서는 JavaScript는 필수적인 구성요소이다.

 
### 변천사
지금의 JavaScript는 

LiveScript -> JavaScript -> ECMAScript5.0 -> ES2015~ES2016,2017... 로 이어지고 있다.

보통 Modern JavaScript라고 하면 ES(ECMAScript)2015 버전부터를 칭한다. 

ES 버전은 해마다 해당연도에 따라 버전이 업데이트 되지만, ECMAScript5.0이 가장 주도적인 버전이다. 

이유는 최신버전일수록 브라우저의 호환이 떨어질 수 있기 때문이다. ECMAScript5.0이 최적화버전이라 할 수 있겠다.

 

### CSR과 SSR
CSR(Client Side Rendering) : JavaScript가 주도적으로 UI를 만드는 방법을 말한다.

SSR(Server Side Rendering) : WebServer에서 주도적으로 UI를 만들어 HTML을 브라우저에 전송하는 방법을 말한다.

 

이해한 바로는 전자는 프론트엔드관련, 후자는 백엔드관련으로 느껴진다.

 

### JavaScript와 브라우저
JavaScript와 웹은 원래 브라우저의 런타임환경에서만 실행되었다. 하지만 Node.js와 npm이 개발되면서 브라우저가 아닌 환경에서도 웹앱을 실행할 수 있게되었다. 따라서 프론트엔드는 굉장히 급변하게 되었다.

### Transpiler
트랜스파일러는 위에서 언급한 ES2015~2022 등의 버전을 최적화된 ECMAScript 5.0으로 변환해주거나, 개발자가 자신있는 혹은 편한 언어를 사용하여 코드를 작성하고 JavaScript로 변환시켜주는 것을 말한다. 

이것이 가능한 이유는 컴퓨터는 결국 Java, C, C#, JavaScript 등의 언어를 해석하는 것이 아니라 기계어를 해석할 뿐이고, 각각의 언어는 결국 기계어가 되어 실행되는 단계를 거치므로 어떤 언어가 됐든 기계어가 되기 전에 JavaScript로 변환시켜주면 컴퓨터입장에서는 다 똑같은 언어라는 것이다. 트랜스파일러에는 BABEL 등이 있다.

 

### Bundler
여러 JavaScript 안에서 사용되는 모듈들을 모두 해석한 뒤 하나의 파일로 만들어주는 역할을 한다.

Bundler로는 Webpack 등이 있으며 앞에서 얘기한 기능 뿐만 아니라 다양한 기능을 지원해준다.