# lerna-tutorial README.md

lerna는 Git과 npm으로 monorepos 를 관리하는 도구이다.

## lerna 의 필요성

(예제: lerna 가 없는 경우, 여러 프로젝트에서 의존성 업데이트를 여러번 수행해야 하는 문제가 발생) 예를 들면, 공통의 의존성들을 가지고, 여러 repositories 에서 여러 TypeScript 프로젝트를 한다고 가정하자.

* 그런데 전체 프로젝트에서 하나의 의존성을 업데이트하려는 경우, 업데이트되어져야 하는 동일한 의존성들이 있는 프로젝트들이 있기 때문에, 당신은 동일한 commands를 여러번 해야 할 것이다.

(lerna의 필요성 및 역할) 그러나, lerna 를 사용하면, 전체 프로젝트에서 동일한 command를 실행할 수 있다. 

* 즉, 모든 프로젝트들이 동일한 build, test, release process 를 공유하여, 당신의 repository 관리를 더 쉽게 한다.
* updates, tests, deployments 는 전체 프로젝트에서 실행되어, 모든 repositories  도 적절하게 작동하는 것을 확인할 수 있다.

(lerna 의 의존성 **link**) 추가적으로, lerna를 사용하면, 프로젝트들 간에 dependencies 를 link 할 수 있다.

* 따라서, 만약 프로젝트 A가 프로젝트 B 에 의존하고 있다면, 직접적인 의존성들이 없는 다른 프로젝트들에 영향을 주지 않으면서, 두 프로젝트 간에 의존성들을 공유할 수 있고, 테스트할 수 있다.

(lerna 는 개별 프로젝트에 있는 **package.json** 으로 의존성을 관리) lerna 를 사용할 때, 당신의 monorepo 안에 있는 개별 프로젝트는 자신의 package.json을 가지고, 자신의 의존성들을 관리한다.

* lerna가 자동으로 하나의 프로젝트를 또 다른 프로젝트에 link 하는 옵션을 주지 않기 때문에, **[yarn workspace](https://classic.yarnpkg.com/en/docs/workspaces/)** 를 사용할 것이다.
* 일단, 의존성들을 설치하면, 프로젝트들 간의 의존성들이 자동으로 link 되고, 개별 프로젝트들로 linked 의존성들을 import 하는 것이 쉬워진다.

# [lerna  사용, fake dice roll 앱 만들기](https://www.baltuta.eu/posts/typescript-lerna-monorepo-the-setup)

| no   | 구분     | 설명       |
| ---- | -------- | ---------- |
| 1    | diceroll | main logic |
| 2    | api      |            |
| 3    | ui       |            |

lerna 와  yarn workspace 를 사용하여, 전체 packages 에서 commands 를 create/bootstrap/run 할 것이다.

## 설치 및 monorepo 셋업

| no   | 명령어                                         | 설명                                                         |
| ---- | ---------------------------------------------- | ------------------------------------------------------------ |
| 1    | $ git init lerna-tutorial && cd lerna-tutorial | 프로젝트 폴더                                                |
| 2    | $ echo “node_modules” >> .gitignore            | .gitignore 파일 생성하고, node_modules 입력                  |
| 3    | $ yarn -D -W add lerna                         | lerna 설치 및 lerna \--version                               |
|      | -W                                             | 이 workspace root 에 설치를 강제하기 위함                    |
| 4    | $ lerna init                                   | lerna repo 생성                                              |
|      |                                                | packages 폴더, lerna.json, package.json 자동생성             |
| 5    | $ touch lerna.json                             | 업데이트: [참조](https://github.com/lerna/lerna#how-it-works) |
|      | independent mode                               | 개별 패키지에 대하여 특정 versions 를 publish 한다.          |
| 6    | $ touch package.json                           | 업데이트                                                     |
|      | workspaces                                     | workspaces 에서 사용되는 폴더들 지정한다.                    |
| 7    | $ yarn add typescript -D -W                    | typescript 설치                                              |
| 8    | $ touch tsconfig.json                          | 생성                                                         |

lerna.json

```json
{
  "packages": [
    "packages/*"
  ],
  "version": "independent",
  "npmClient": "yarn",
  "useWorkspaces": true
}
```

packages.json

```json
{
  "name": "root",
  "private": true,
  "workspaces": [
    "packages/*"
  ],
  "devDependencies": {
    "lerna": "^4.0.0"
  }
}
```

tsconfig.json

```tsx
{
  "compilerOptions": {
    "baseUrl": ".",
      
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    
    "strict": true,
    
    "skipLibCheck": true,
    
    "esModuleInterop": true,
    "module": "commonjs",
    "moduleResolution": "node",
    "target": "es2015",
    
    "composite": true
  }
}
```

##  프로젝트(package) 셋업

lerna create 명령을 사용할 수 있으나, 쓸데 없는 것들을 방지하기 위해 이 명령을 사용하지 않을 것이고, 목표는 다음과 같은 구조를 만드는 것이다.

![image-20210812133031893](/Users/catchhub/Library/Application Support/typora-user-images/image-20210812133031893.png)

**1) diceroll 패키지**

| no   | 명령어                  | 설명                                                         |
| ---- | ----------------------- | ------------------------------------------------------------ |
| 1    | $ cd packages           | 우리 프로젝트를 실행하는데 필요한 모든 typescript 의존성들을 추가한다. |
| 2    | $ mkdir -p diceroll/src | diceroll package 에는 코어 비지니스 로직이 있고, 어떠한 내부적 의존성들이 없다. |
| 3    | $ cd diceroll           |                                                              |
| 4    | $ touch package.json    |                                                              |
| 5    | $ touch tsconfig.json   |                                                              |
| 6    | $ cd src                |                                                              |
| 7    | $ touch index.ts        |                                                              |

packages/diceroll/package.json

```json
{
  "name": "@monorepo/diceroll",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": [
    "dist"
  ],
  "private": true,
  "scripts": {
    "build": "tsc -b"
  }
}
```

packages/diceroll/tsconfig.json

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src"
  },
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

> src 를 lib 로 바꿀 수도 있는데, 바꿀 경우에는 tsconfig.json 에서 rootDire 를 변경하는 것을 잊지 않아야 한다.

packages/diceroll/src/index.ts

```tsx
export function roll(roll: string): string {
  return `I rolled a dice: ${roll}. Outcome grim`;
}
```

**2) UI  패키지**

이 패키지에는 main, types, files 가 필요하지 않다.

| no   | 구분                 | 설명                       |
| ---- | -------------------- | -------------------------- |
| 1    | $ cd packages        |                            |
| 2    | $ mkdir -p ui/src    | ui 패키지 및 src 폴더 생성 |
| 3    | $ cd ui              |                            |
| 4    | $ touch package.json |                            |
|      |                      |                            |

packages/ui/package.json

```json
{
  "name": "@monorepo/ui",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "build": "tsc -b"
  },
  "dependencies": {
    "@monorepo/diceroll": "^1.0.0"
  }
}
```

packages/ui/tsconfig.json

* target으로 ES2019를 지정하였고, dom 정의를 추가하였다.
* references 섹션에서, Typescript 에게 이 경로에 있는 파일들을 reference 하고 있다고 알리고, 이러한 파일들에 변경이 발생하면, 우리의 code 를 check/comile 하라고 지시한다. – 이렇게 하면 breaking changes 를 빠르게 catch 할 수 있다.

> 이 파일은 import 와는 아무런 상관이 없다. lerna 가 할 파트이다.

```json
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src",
    "target": "ES2019",
    "lib": [ "dom", "ES2019"]
  },
  "references": [
    { "path":  "../diceroll" }
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

packages/ui/src/index.ts

```tsx
import { roll } from '@monorepo/diceroll';

console.log(roll('1d20'));
```

## lerna bootstrap & build

이제  VSCode 에디터에서 여러 파일들에서 에러 표시가 있지만, 현재로서는 정상이다.

| no   | 구분                       | 설명                                                         |
| ---- | -------------------------- | ------------------------------------------------------------ |
| 1    | $ cd lerna-tutorial        |                                                              |
| 2    | $ lerna bootstrap          | 우리의 의존성들이 설치되고, link 된다.                       |
| 3    | $ lerna run build\--stream | 지정되었던 개별 패키지에 대하여 build 를 실행하고, output 을 stream 한다. |
|      |                            | 개별 패키지에 tsconfig.buildinfo 파일과 dist 폴더/관련 파일들이 자동생성된다. |
|      |                            | (참조) incremental 옵션을 사용하면, 사전에 볼 수 있다.       |

## clean 스크립트 추가

모든 개별 패키지와 root 의 package.json 에  clean 스크립트를 추가한다.

| no   | 구분                                   | 설명                                        |
| ---- | -------------------------------------- | ------------------------------------------- |
| 1    | $ touch packages/diceroll/package.json | diceroll 패키지에 clean 스크립트 추가       |
| 2    | $ touch packages/ui/package.json       | ui 패키지에 clean 스크립트 추가             |
| 3    | $ touch pakage.json                    | root 폴더에 “scripts”와 build 및 clean 추가 |

packages/diceroll/package.json

```json
...
	"scripts": {
     "build": "tsc -b",
     "clean": "rm -rf ./dist && rm tsconfig.tsbuildinfo"
   },
...
```

packages/ui/package.json

```json
...	
	"scripts": {
    "build": "tsc -b",
    "clean": "rm -rf ./dist && rm tsconfig.tsbuildinfo"
  },
...
```

package.json

```json
{
  "name": "root",
  "private": true,
  "files": [
    "dist"
  ],
  "scripts": {
    "build": "lerna run build --stream",
    "clean": "lerna run clean --parallel"
  },
  "devDependencies": {
    "lerna": "^4.0.0",
    "typescript": "^4.3.5"
  }
}

```

## Watch 모드 만들기

| no   | 구분                                   | 설명        |
| ---- | -------------------------------------- | ----------- |
| 1    | $ touch packages/diceroll/package.json | watch  추가 |

 packages/diceroll/package.json

```json
...
	"scripts": {
    "build": "tsc -b",
    "clean": "rm -rf ./dist && rm tsconfig.tsbuildinfo",
    "watch": "tsc -b -w --preserveWatchOutput"
  }
....
```

## UI 패키지 만들기

지금까지 여러 패키지들을 함께 작동하도록 만들어서, Typescript 가 가능한 한 빨리 compile 할 것이고, watch 모드까지 가지고 있으며, cleanup 스크립트도 가지고 있다.

그러나, UI 를 만들어야 한다.

| no   | 구분                                                         | 설명                                                    |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------- |
| 1    | $ cd ui                                                      |                                                         |
| 2    | $ yarn add --dev html-webpack-plugin webpack webpack-cli webpack-dev-server |                                                         |
|      | $ lerna add \--scope=@monorepo/ui -D webpack                 | alternatives                                            |
|      | $ lerna add \--scope=@monorepo/ui -D webpack-cli             | “                                                       |
|      | $ lerna add \--scope=@monorepo/ui -D webpack-dev-server      | “                                                       |
|      | $ lerna add \--scope=@monorepo/ui -D html-webpack-plugin     | “                                                       |
| 3    | $ touch packages/ui/src/index.html                           | 생성                                                    |
| 4    | $ touch packages/ui/webpack.config.js                        | 생성                                                    |
| 5    | $ touch packages/ui/tsconfig.json                            | 업데이트                                                |
|      | > compilerOptions의 outDir 에서 dist 를 build 로 교체하고,   | 이렇게 하는 이유는 우리의 마지막 build 결과는           |
|      | > exclude 에서 build 를 추가한다.                            | TypeScript 가 아니라 Webpack 에 의해 번들되기 때문이다. |
| 6    | $ touch packages/ui/package.json                             | 업데이트                                                |
|      | > awesome-typescript-loader 또는 ts-loader 와 같은 typescript loader 또는 Babel을 사용하지 않을 것이다. |                                                         |
|      | > 이렇게 하면, 각자의 role 이 좀 더 명확해진다.              | 원하면 이러한 방식을 채택한다.                          |

packages/ui/src/index.html

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <meta http-equiv="X-UA-Compatible" content="ie=edge" />

  <title>DiceRoll</title>
</head>

<body>
  <div id="result"></div>
</body>

</html>

```

packages/ui/webpack.config.js

```tsx
const HtmlWebpackPlugin = require('html-webpack-plugin');

module.exports = function(env, argv) {
  return {
    mode: env.production ? 'production' : 'development',
    devtool: env.production ? 'source-map' : 'eval',

    devServer: {
      open: true,
      historyApiFallback: true
    },

    entry: {
      index: './build/index.js',
    },

    plugins: [
      new HtmlWebpackPlugin({
        template: './src/index.html',
      }),
    ]
  }
};
```

packages/ui/package.json

```json
   "scripts": {
-    "build": "tsc -b",
-    "clean": "rm -rf ./dist && rm tsconfig.tsbuildinfo",
-    "watch": "tsc -b -w --preserveWatchOutput"
+    "start": "webpack serve",
+    "build": "yarn compile && webpack --env production",
+    "clean": "rm -rf ./dist && rm -rf ./build && rm tsconfig.tsbuildinfo",
+    "watch": "tsc -b -w --preserveWatchOutput",
+    "compile": "tsc -b"
   },
```

## API 패키지

앞에서 비지니스 로직을 구현한 diceroll 패키지와 UI 를 구현한 ui 패키지를 만들었다.

이제, UI 에 대한 API 패키지를 구현할 것이다.

* API 패키지에서, nodemon 사용, 앱을 restart 할 것이다.
* ts-node는 느려서, ts-node 를 사용하지 않을 것이다.

| no   | 구분                                                 | 설명                                                       |
| ---- | ---------------------------------------------------- | ---------------------------------------------------------- |
| 1    | $ cd packages                                        |                                                            |
| 2    | $ mkdir -p api/src                                   | api 폴더와 src 폴더를 만든다.                              |
| 3    | $ cd api                                             |                                                            |
| 4    | $ touch package.json                                 | diceroll 폴더의 package.json 을 복사한 다음 업데이트한다.  |
|      | > name                                               | @monorepo/api 로 변경한다.                                 |
|      | > scripts                                            | start 를 추가한다.                                         |
| 5    | $ touch tsconfig.json                                | diceroll 폴더의 tsconfig.json 을 복사한 다음 업데이트한다. |
|      | > compilerOptions  에서                              |                                                            |
|      | >> module                                            | commonjs 추가                                              |
|      | >>  target                                           | ES2019 추가                                                |
|      | >> lib                                               | [“ES2019”] 추가                                            |
|      | > references                                         | 추가                                                       |
|      | >> path                                              | 추가                                                       |
| 6    | $ cd api                                             |                                                            |
|      | $ lerna add \--scope=@monorepo/api express           | api 패키지에 express 설치                                  |
|      | $ lerna add \--scope=@monorepo/api -D nodemon        | api 패키지에 nodemon 설치                                  |
|      | $ lerna add \--scope=@monorepo/api -D @types/express | api 패키지에 @types/express 설치                           |
| 7    |                                                      |                                                            |

packages/api/package.json

```json
{
  "name": "@monorepo/api",
  "version": "1.0.0",
  "main": "dist/index.js",
  "types": "dist/index.d.ts",
  "files": [
    "dist"
  ],
  "private": true,
  "scripts": {
    "build": "tsc -b",
    "start": "nodemon --inspect dist/index.js",
    "clean": "rm -rf ./dist && rm tsconfig.tsbuildinfo",
    "watch": "tsc -b -w --preserveWatchOutput"
  }
}
```

packages/api/tsconfig.json

```tsx
{
  "extends": "../../tsconfig.json",
  "compilerOptions": {
    "outDir": "dist",
    "rootDir": "src",
    "module": "commonjs",
    "target": "ES2019",
    "lib": ["ES2019"]
  },
  "references": [
    { "path": "../diceroll" }
  ],
  "exclude": [
    "node_modules",
    "dist"
  ]
}
```

