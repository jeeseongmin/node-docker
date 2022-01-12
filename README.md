# Node.js에서 Docker 사용하기

Docker와 관련된 example

## Docker를 사용하는 이유

1. 어떤 컴퓨터에서든 똑같은 개발 환경을 보장해주기 때문.

2. 앱 실행을 위한 과정을 패키징해두면, 대량의 앱을 실행시킬 수도 있다. 서비스 확장 시에, 많은 인스턴스 운영을 위해 반드시 도커를 사용해야할 때가 온다.

3. 서버의 안정성

## Node.js 서버 구성

### 1. 일반 프로젝트 폴더 구성

```
$ mkdir node-docker
$ cd node-docker

$ yarn init -y -> package.json 만들기
```

### 2. Express 패키지 설치

```
$ yarn add express
```

### 3. typescript 및 nodemon 설치

```
$ yarn add -D typescript ts-node nodemon $types/node $types/express
$ touch index.ts
$ npx tsc --init -> tsconfig.json 만들기
```

### 4. index.ts 작성

```index.ts
import express from 'express';
const app = express();

app.get('/', (req, res) => {
    res.json({ message : 'Hello, Docker!' })
})

app.listen(3000);
console.log('http://localhost:3000..');
```

### 5. package.json 파일 수정

```package.json
{
    "scripts" : {
        "start" : "node index.js",
        "dev" : "nodemon -L --exec ts-node index.ts",
        "build" : "tsc"
    }
}

```

`yarn dev` 명령어를 입력하고, localhost:3000으로 접속하게 되면 메세지를 응답받을 수 있다.

## Dockerfile 구성

1. Docker 설치

- Docker를 사용하기 위해서는 컴퓨터에 Docker를 설치해야 한다. 따라서 Docker 홈페이지에서 Docker Desktop을 다운로드한다.

2. Docker 설치 확인

```
$ docker -v
-> 제대로 설치되었다면 결과는 다음과 같다.
Docker version 20.10.11, build asdfas
```

3. Dockerfile 작성하기

- Dockerfile은 어떤 순서로 앱을 패키징할 지 나열해준다.

```
$ touch Dockerfile
```

```Dockerfile
# 어떤 환경에서 도커 이미지를 만들지 결정하기.
FROM node:14-slim

# 도커 컨테이너 내부의 작업 디렉토리 결정하기. 원하는 대로 정하면 된다.
WORKDIR /usr/src/app

# 외부 패키지 설치를 위해 package.json과 yarn.lock 파일 복사
COPY package.json .
COPY yarn.lock .

# 패키지 설치
RUN yarn

# 나머지 모두 복사
COPY . .

# 도커 컨테이너에 접근할 수 있게 포트 열어주기
EXPOSE 3000

# 앱 실행시키기
CMD [ "yarn", "dev" ]
```

4. 이미지 생성

```
$ docker build . -t node_app
```

- -t ~ 로 이름을 지정한다는 의미
- 만약 `error during connect: This error may indicate that the docker daemon is not running.` 와 같은 에러가 나타났을 경우

## Reference

- [정말 너무 쉬운 Docker](https://www.peterkimzz.com/super-easy-docker/)
