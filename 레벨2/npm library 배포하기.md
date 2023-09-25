# npm library 배포하기(feat. react, typescript, webpack)

## 서론

우아한테크코스 페이먼츠 미션에서 사용할 모달을 라이브러리로 배포하게 되었다. 간단한 모달이지만 이렇게 오픈소스 라이브러리를 배포해보는 경험은 처음이라 글로 남겨보기로 했다. 그리고 한 번도 프로젝트 설정을 직접 해본적이 없어 CRA가 아닌 웹팩을 사용하기로 결정했다.

[배포된 라이브러리 보러가기](https://www.npmjs.com/package/tami-modal)

## CRA없이 리액트 프로젝트 만들기

### npm init

```bash
npm init
```

`npm init` 명령어를 통해 npm 라이브러리에 올리기 위한 최소한의 정보들을 입력할 수 있다. (package name, description, git repository 등등) 정보를 입력하게 되면 `package.json` 파일이 생성된다.

### 필요한 dependency 실행

1. React, Typescript 설치

```bash
npm install react react-dom typescript @types/react @types/react-dom ts-loader
```

tsconfig.json

```json
{
  "compilerOptions": {
    "allowSyntheticDefaultImports": true,
    "moduleResolution": "node",
    "target": "es6",
    "module": "ESNext",
    "lib": ["es6", "dom"],
    "declaration": true,
    "outDir": "dist",
    "strict": true,
    "jsx": "react-jsx"
  },
  "include": ["src/index.tsx"]
}
```

- 리액트 프로젝트이기 때문에 `"jsx" : "react-jsx"` 를 설정해줘야 한다.

2. babel 설치

```bash
npm install -D babel-loader @babel/core
npm install -D @babel/preset-env @babel/preset-react @babel/preset-typescript
```

- @babel/core: 바벨을 사용할 때 필수로 설치해야하는 패키지
- @babel/preset-env: ES6이상의 문법을 ES5로 트랜스파일링
- babel-loader: 웹팩으로 번들링할 때 바벨로 트랜스파일링이 되도록 해주는 로더

.babelrc

```bash
{
  "presets": [
    "@babel/preset-env",
    "@babel/preset-react",
    "@babel/preset-typescript"
  ]
}
```

3. webpack 설치

```bash
npm install -D webpack webpack-dev-server webpack-cli
```

```jsx
//webpack.config.js
const path = require('path');
const webpack = require('webpack');
const prod = process.env.NODE_ENV === 'production';

module.exports = {
  mode: prod ? 'production' : 'development',
  devtool: prod ? 'hidden-source-map' : 'eval',
  entry: './src/index.tsx',
  resolve: {
    extensions: ['.js', '.jsx', '.ts', '.tsx'],
  },

  module: {
    rules: [
      {
        test: /\.tsx?$/,
        exclude: /node_modules/,
        use: ['babel-loader', 'ts-loader'],
      },
      {
        test: /\.(js|jsx)$/,
        exclude: /node_modules/,
        use: ['babel-loader'],
      },
    ],
  },

  output: {
    path: path.join(__dirname, '/dist'),
    filename: 'index.js',
  },
};
```

- entry: 번들링을 하기 위한 최초 진입점
- output: 번들링 하고자 하는 파일을 어느 위치에 놓을지 결정 (위의 프로젝트의 경우 /dist/index.js로 생성)
- rules: 해당 확장자를 가진 파일에 대해서 어떤 loader을 사용할지 설정

## dependency 관리하기

지금까지 패키지를 설치할 때 별 생각 없이 받았었는데 라이브러리를 배포하면서 dependency, peer dependency, dev dependency의 차이점을 알게 되었다.

- dependency: 개발, 런타임, 빌드타임에서 모두 필요한 패키지들
- dev dependency: 런타임에선 필요 없고 개발, 빌드타임에서만 필요한 패키지
- peer dependency: 해당 프로젝트를 사용할 때 종속되는 패키지들

내가 만든 라이브러리는 리액트에서만 사용할 수 있기 때문에 peerDependency로 리액트를 관리했다. 그리고 styled-components를 사용했기 때문에 dependency에 styled-components를 추가해줬다. (사실 굳이 styled-components를 쓸 필요는 없지만 시간이 없기 때문에 익숙한 도구를 사용할 수 밖에 없었다..🥲)

## 배포 준비하기

### npm info

```bash
npm info [배포할 패키지명]
```

`npm info` 명령어를 통해 자신이 배포할 라이브러리 이름이 중복인지 확인할 수 있다. 404 에러가 뜬다면 아직 존재 하지 않는 패키지명이기 때문에 사용할 수 있다.

### npm login

```bash
npm login
```

라이브러리에 배포를 하기 위해서는 npm에 로그인을 해야 한다. `npm whoami` 명령어를 쓰면 로그인 되어있는지 확인할 수 있다.

### version

package.json을 보면 라이브러리의 버전을 작성하도록 하고 있다. `npm publish`를 할 때마다 꼭 버전을 올려줘야 반영이 된다. npm에서 사용하는 버전은 semantic versioning으로 `[major].[minor].[patch]` 를 규칙으로 한다.

- major: 기능 변화가 이루어져 하위호환이 되지 않는 경우
- minor: 기능 변화가 이루어졌지만 하위호환이 되는 경우
- patch: 버그같은 작은 변화가 생긴 경우

### script

```json
"scripts":{
	"publish:npm": "rm -rf dist && mkdir dist && tsc"
}
```

배포하기전 `npm publish:npm` 이라는 명령어를 통해 빌드한다. (명령어의 이름은 바꿔도 상관 없다.) 매번 dist 아래 내용을 삭제하고 새롭게 빌드를 한다는 뜻이다. 뒤의 `tsc` 는 타입스크립트를 빌드한다는 뜻이다.

```
⚠️ 이 전에 `babel src -d dist --copy-files` 이라는 사용했었는데 이렇게 하면 추가적인 loader가 필요하다는 에러가 나온다. 정확한 이유는 파악하지 못했지만 `copy-files` 로 인해 파일이 갱신되지 않는다는 추측을 하고 있다.
```

### npm publish

```bash
npm publish
```

`npm publish` 를 통해 npm에 라이브러리를 업로드한다. 이미 업로드 되어있는데 버전만 올려주는 것이라면 npm version patch, minor, major를 사용할 수 있다.

## 고도화

모달을 배포하면서 다양한 라이브러리를 구경해봤다. (chakra-ui, react-modal 등) 문서를 어떻게 작성했는지, 어떤 컴포넌트를 제공해주는지 등을 보고 어떻게 모달을 구성할지 고민해봤다. 하지만 시간이 부족했기에 다음으로 미뤘고 현재는 모달의 위치(정가운데 혹은 아래)만 정해줄 수 있는 상태이다. 고도화를 하게 된다면 chakra-ui처럼 ModalContent, ModalLayer등의 컴포넌트를 제공해서 쉽게 커스터마이징 할 수 있는 모달을 만들고 싶다.

추가로 요즘 재사용성이 높은 컴포넌트를 만드는 것에 대해 고민을 하고 있는데, 모달 말고도 자주 쓰이는 Input이나 Button 같이 자주 쓰는 컴포넌트들을 모아서 나만의 라이브러리를 배포해보고 싶다. 준이 일주일에 한 번씩 버전을 올려보면서 만들어보라고 했는데 뭔가 재미있을 것 같고, 나중에 프로젝트를 하면서 꺼내 쓰기도 편할 것 같다.

## 느낀점

처음이라 막막하기만 했던 라이브러리 배포를 결국엔 성공적으로 끝낼 수 있었다. cra로 했으면 금방 끝낼 수도 있었을 것 같은데 웹팩설정을 하면서 생긴 오류들로 인해 시간이 오래 걸렸다. 하지만 만약 라이브러리를 오래 유지보수 할 것이라면 직접 환경 설정을 해보는 것도 좋을 것 같다. 설정들만 하면 라이브러리를 유지하는 것은 어렵지 않다. 라이브러리를 배포해보는 것은 한 번쯤 해보면 좋은 경험인 것 같다.
