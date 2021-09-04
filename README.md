# nestJS

## Setting

### start

```
$ npm i -g @nestjs/cli
$ nest new project-name
```

1. npm이 설치된 상태에서 nestJs cli를 설치해 줍니다. -g 옵션은 글로벌 모드입니다. 주로 개발 환경을 설정할때 글로벌 옵션을 많이 설정합니다.

```
$ nest new project-name
```

2. cli가 성공적으로 설치되었다면 cli 명령어로 새 프로젝트를 생성합니다. 저는 nest-js 라는 프로젝트로 명령어를 실행시켰습니다.

성공적으로 실행이 되었다면 프로젝트의 구조는 다음과 같습니다.

![image](https://user-images.githubusercontent.com/40652160/132087706-2e5e8e5e-92f3-4e8a-95ef-ef3c5e83e62a.png)

### Hot Reloading

Hot Reloading이란. nodejs에서는 nodemon 모듈로 프로젝트의 변화가 감지되었을때 서버를 재시작 해주는 기능이 있었습니다.
NestJS에서는 이 기능을 hot Reloading이라고 불리는데 간단한 설정으로 이 기능을 활성화 시킵니다.
HotReloading을 활성화하면 개발할때 암걸릴 확률이 줄어듭니다.

우선 라이브러리 먼저 설치를 하겠습니다.

`yarn add webpack webpack-node-externals run-script-webpack-plugin -D`
-D 옵션은 devDependency 에 의존성을 추가하는 옵션입니다.

패키지를 설치했다면 최 상위 프로젝트에 webpack-hmr.config.js 파일을 생성해 줍니다.

```ts
import nodeExternals from 'webpack-node-externals';
import { RunScriptWebpackPlugin } from 'run-script-webpack-plugin';

module.exports = function (options, webpack) {
  return {
    ...options,
    entry: ['webpack/hot/poll?100', options.entry],
    externals: [
      nodeExternals({
        allowlist: ['webpack/hot/poll?100'],
      }),
    ],
    plugins: [
      ...options.plugins,
      new webpack.HotModuleReplacementPlugin(),
      new webpack.WatchIgnorePlugin({
        paths: [/\.js$/, /\.d\.ts$/],
      }),
      new RunScriptWebpackPlugin({ name: options.output.filename }),
    ],
  };
};
```

그 후 이 프로젝트의 entry point은 src/main.ts에 다음과 같이 코드를 수정해 주세요.

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

declare const module: any;

async function bootstrap() {
  console.log('hotReloading tests');
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
  if (module.hot) {
    module.hot.accept();
    module.hot.dispose(() => app.close());
  }
}
bootstrap();
```

거의 다 왔습니다. package.json파일 Script에 다음 코드를 추가해 주세요.

```json
"start:dev-backup": "nest build --webpack --webpackPath webpack-hmr.config.js --watch",
```

![image](https://user-images.githubusercontent.com/40652160/132088191-100d8101-57d6-4ff3-a921-c3443fb04544.png)

이제 ctrl+c키로 서버를 다운시키고 `yarn start:dev` 명령어를 실행시켜 보세요
이제부터 프로젝트 코드 변경 사항이 있을때마다 서버가 재시동 됩니다.
