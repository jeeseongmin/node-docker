# Node.js with Docker

Docker를 다루기에 앞서, Docker와 node.js를 사용해보면서 이해하기 위한 예제

## Docker를 사용하는 이유

1. 어떤 컴퓨터에서든 똑같은 개발 환경을 보장해주기 때문.

2. 앱 실행을 위한 과정을 패키징해두면, 대량의 앱을 실행시킬 수도 있다. 서비스 확장 시에, 많은 인스턴스 운영을 위해 반드시 도커를 사용해야할 때가 온다.

3. 서버의 안정성

---

## Node-Docker 코드 구성

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
- COPY는 `COPY A B`일 때 A(내 컴퓨터 쪽)를 B(도커 컨테이너 쪽)로 복사한다는 의미이다.

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
$ docker images -> 빌드된 이미지 확인
```

- 이미지 생성 시에 문제가 발생하는 경우가 있다.
  `//./pipe/docker_engine: The system cannot find the file specified` 와 같은 메시지가 나타나며 이미지 생성이 안되는 문제이다. 아마도 이미지 생성 뿐 아니라 더 많은 docker 명령어가 동작하지 않을 것이다.
  이러한 경우는 리눅스 커널 업데이트를 진행하지 않았기 때문에 그렇다.

  window에서 Docker Desktop을 설치하는 경우 다음과 같이 업데이트 화면이 나타나게 된다.
  ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FXPVfx%2Fbtq0dnOA0Ir%2F2nkFTT0z6pREgQHUEGeZ11%2Fimg.png)
  그런데 아무리 Restart를 눌러도 잘 되지 않는다. 그럴 때는 다음과 같은 순서로 업데이트를 진행해야 한다.

  (1) Powershell 관리자 권한으로 실행
  (2) 리눅스 서브시스템 활성 명령어 입력

  ```
  dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
  ```

  (3) 가상 머신 플랫폼 기능 활성화 명령어 입력

  ```
  dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
  ```

  (4) x64 머신용 최신 WSL2 Linux 커널 업데이트 패키지 다운로드 및 설치

  - [wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi](wslstorestorage.blob.core.windows.net/wslblob/wsl_update_x64.msi)

  (5) Docker Desktop 실행 후 Restart 선택

  (6) Docker 설치가 완료되었다고 나타나는 것을 볼 수 있다.
  ![image](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FzUnXt%2Fbtq0dngODSP%2FzMQI15EqZiK7o1MDEIq9v1%2Fimg.png)

5. 빌드된 이미지 실행

```
$ docker run -p 3000:3000 node_app

# 컨테이너 여러개 실행 가능
$ docker run -p 3001:3000 node_app
$ docker run -p 3002:3000 node_app
```

- 빌드된 이미지가 실행되면 `컨테이너`라고 부른다.

```
# 실행 중인 모든 컨테이너 목록 확인
$ docker ps -a
```

- `STATUS`가 `Up`이라면 앱이 실행되고 있는 상태이다. 만약 삭제하고 싶다면 'CONTAINER ID'를 이용해 삭제해준다.

```
$ docker rm -f CONTAINER_ID
```

---

## Docker Compose

- Docker CLI를 이용하는 것은 사실 비효율적이다. Docker 특성상 많은 옵션을 주어야 하기 때문에 명령어가 길어지기 때문이다.
  이미지 여러 개를 띄워야 하는 경우에도 이미지 하나씩 올리는 것은 귀찮은 일이 되어버린다.

그래서 여러 개의 이미지를 한 번에 관리할 수 있게 개발된 것이 'Docker Compose'이다.

프로젝트 루트 디렉토리에 `docker-compose.yml` 파일을 만들어주고 터미널에 `docker compose up`을 입력하면 정상적으로 컨테이너가 생성되고, 로컬호스트로 접근이 가능해진다.

- 단일 컨테이너

```
version: '3.9'

services:
  app: # 이미지 이름 (마음대로 설정해도 됩니다)
    build: . # Dockerfile이 있는 경로를 넣어주기
    ports:
      - '3000:3000' # docker CLI의 "-p 3000:3000" 과 같은 표현

```

- 여러 개의 컨테이너

```
version: '3.9'

services:
  app1:
    build: .
    ports:
      - '3000:3000'

  app2:
    build: .
    ports:
      - '3001:3000'

  app3:
    build: .
    ports:
      - '3002:3000'

  app4:
    build: .
    ports:
      - '3003:3000'
```

docker compose는 일반 도커 명령어와 다르게, 터미널에서 작업을 종료하면 그대로 컨테이너들이 모두 비활성화된다. 백그라운드에서도 계속 실행시키고 싶다면 `docker compose up -d` 명령어를 사용해야 한다.

그리고 백그라운드에서 실행된 컨테이너들을 한 번에 지우려면 `docker compose down`을 하면 된다.

---

## 프로젝트 구조 개선

지금까지 Docker에 대해서 살펴봤지만, 더 개발 환경을 다듬어야 한다.

1.  핫 리로딩

    Docker 컨테이너로 개발을 하게 되면, 코드를 수정할 때마다 개발 서버가 다시 시작되지는 않는다. 이유는 우리가 수정하는 코드가 로컬 컴퓨터의 코드이지, 컨테이너 안의 코드가 아니기 때문이다.

    그러기 위해서는 로컬 컴퓨터와 컨테이너의 저장 공간을 공유하는 방법을 사용한다.
    로컬 코드를 수정하면 바로 컨테이너 안의 코드도 같이 수정이 된다.

    - docker-compose.yml

    ```
    version: '3.9'

    services:
    app:
        build: .
        ports:
        - '3000:3000'
        volumes:
        - '.:/usr/src/api' # Dockerfile의 WORKDIR와 맞추기
        - '/usr/src/api/node_modules' # 핫 리로드 성능 개선
    ```

    `docker compose up --build` 명령어로 새로 빌드하면서 컨테이너를 띄워주고, 코드를 수정하면 바로 서버가 다시 시작하게 된다.

2.  개발용, 배포용 이미지 분리하기

    기존 도커파일은 개발용이었기 때문에, 파일 이름을 `Dockerfile.dev`로 변경해주고, 배포용 파일인 `Dockerfile`을 새로 만들어준다.

    - 배포용 도커 파일 (Dockerfile)

    ```
    FROM node:14-slim

    WORKDIR /usr/src/app

    COPY package.json .
    COPY yarn.lock .

    RUN yarn

    COPY . .

    RUN yarn build # 빌드하는 부분 추가

    EXPOSE 3000

    CMD [ "yarn", "start" ] # `yarn dev`에서 `yarn start`로 변경
    ```

    기존 `docker-compose.yml`도 이름을 `docker-compose.dev.yml`로 바꾸고, 새로운 컴포즈 파일을 만들고 아래 내용을 작성한다.

    ```
    version: '3.9'

    services:
    app:
    build: .
    ports: - '80:3000'
    ```

    물론 배포용 앱은 웹서버와 함께 올리기 때문에 proxy 설정은 따로 해주어야 하지만, 일단 여기서는 제외하도록 한다.

    - docker-compose.dev.yml은 다음과 같이 설정한다.

    ```

    version: '3.9'

    services:
    app:
    build:
    context: .
    dockerfile: Dockerfile.dev
    ports: - '3000:3000'
    volumes: - '.:/usr/src/app' - '/usr/src/app/node_modules'

    ```

    `build` 부분이바뀌는데, 개발 때는 `Dockerfile.dev`를 읽도록 변경한다.

    마지막으로 CLI로 실행할 때는 `-f` 플래그를 이용하면 된다.

    ```
    $ docker compose -f docker-compose.dev.yml up --build
    ```

## Reference

- [정말 너무 쉬운 Docker](https://www.peterkimzz.com/super-easy-docker/)
- [WSL 2 installation is incomplete](https://blog.nachal.com/1691)
