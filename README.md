# Jobs & Containers

## Initial workflow

1. `test` ➡️ `deploy`

---

## Job에서 컨테이너 실행해보기

1. 컨테이너를 사용해서 워크플로우를 동작시켜봅니다. - [`a6d56a59a`](https://github.com/seongjin2427/08.jobs-container/commit/a6d56a59ab95d13820900fec15a25001e68982ef)

- Process
  - `deploy.yml`
    - ```yml
      ...
      jobs:
        test:
          environment: testing
          runs-on: ubuntu-latest
          # 컨테이너를 사용하기 위해 container키를 지정합니다.
          # 이미지에서 실행되는 Step은 컴퓨터에 직접 접근할 수 없습니다.
          container:
            # ubuntu 러너는 단순히 node 16 버전 도커 이미지를 불러옵니다.
            # 사용하길 원하는 도커 이미지명을 지정합니다.
            image: node:16
            # 이미지에 필요한 환경변수 등도 지정할 수 있습니다.
            # 여기에 지정하는 환경변수는 도커 이미지에서만 사용됩니다.
            # 어떤 환경변수가 필요한지는 도커 이미지의 공식문서에 나와있습니다.
            # env: 
      ...

- Result
  - GitHub 페이지의 Actions 탭에서 `test` Job을 확인해보면, 'Initialize containers' 단계가 진행되어 있는 것을 볼 수 있습니다.
    - 도커 버전을 확인하고
    - 이전 Job들의 리소스를 정리하고
    - 지역 컨테이너 네트워크를 생성하고
    - Job 컨테이너를 시작하고
    - 모든 서비스들이 준비되기를 기다립니다.
  - 기술적으로 `test` Job이 컨테이너 내부에서 실행됩니다.

<br>

2. 별도의 MongoDB 클러스터를 생성하는 것 없이, 격리된 서비스 컨테이너로부터 MongoDB를 사용할 수 있습니다. - [`019ba6ab`](https://github.com/seongjin2427/08.jobs-container/commit/019ba6ab4b2ec81c4bc9219cbe9900474c13fff5)

- Process
  - `deploy.yml`
    - ```yml
      ...
      jobs:
        test:
         environment: testing
         runs-on: ubuntu-latest
         container:
          image: node:16
        env:
          # GitHub Actions가 자동으로 네트워크 환경을 생성하기 때문에
          # services에서 지정한 서비스 컨테이너의 label(여기서는 mongodb)을 지정하여 바로 사용할 수 있습니다.
          MONGODB_CONNECTION_PROTOCOL: mongodb
          MONGODB_CLUSTER_ADDRESS: mongodb
          # 서비스 컨테이너에 사용할 username과 password와 동일하게 지정합니다.
          MONGODB_USERNAME: root
          MONGODB_PASSWORD: example
          PORT: 8080
        # Job 기준으로 서비스 컨테이너를 활용할 수 있습니다.
        services:
          # 서비스 컨테이너를 식별하기 위한 label입니다. 원하는대로 지정할 수 있습니다.
          mongodb:
            # image 키를 반드시 사용해야 하며, 사용할 이미지명을 지정하여 도커 이미지를 사용할 수 있습니다.
            image: mongo
            # 서비스 이미지에 사용할 환경 변수를 지정합니다.
            env:
              # mongo 도커 이미지 공식 문서에서 두 환경 변수를 확인할 수 있습니다.
              # test Job이 실행되는 동안에만 서버가 사용됩니다.
              # 환경 변수의 값은 마음대로 설정할 수 있습니다.
              MONGO_INITDB_ROOT_USERNAME: root
              MONGO_INITDB_ROOT_PASSWORD: example
      steps:
        ...

- Result
  - `test` Job의 서비스 컨테이너에서 필요한 MongoDB를 실행하여 테스트가 정상적으로 통과합니다.

<br>

3. 격리된 컨테이너를 사용하지 않고, GitHub Actions에서 제공하는 기본 러너에서도 서비스 컨테이너 사용이 가능합니다. - [`18ed1f71`](https://github.com/seongjin2427/08.jobs-container/commit/18ed1f7149fd3ffa065a41df442fb0f138f4088d)

- Process
  - `deploy.yml`
    - ```yml
      ...
      jobs:
        test:
          environment: testing
          runs-on: ubuntu-latest
          # 만약 test Job의 이 컨테이너가 동작하지 않는다면
          # 컨테이너 내부에서 Step들을 실행하는 것이 아니라
          # GitHub Actions에서 제공한 기본 러너를 사용할 겁니다.
          # container: 
          #   image: node:16
        env:
          # 하지만 자동으로 생성되는 네트워크는 사용할 수 없기 때문에
          MONGODB_CONNECTION_PROTOCOL: mongodb
          # 로컬 호스트 IP와 연결할 특정 포트를 지정해야 합니다.
          # (mongoDB의 기본 포트는 27017입니다.)
          # mongodb -> 127.0.0.1:27017
          MONGODB_CLUSTER_ADDRESS: 127.0.0.1:27017
          MONGODB_USERNAME: root
          MONGODB_PASSWORD: example
          PORT: 8080
        # 그렇더라도 서비스 컨테이너는 여전히 사용 가능합니다.
        services: 
          mongodb: 
            image: mongo
            # 연결할 포트를 설정해야 합니다.
            # 서비스 컨테이너의 내부 포트번호:러너에 포워딩할 포트번호
            ports:
              - 27017:27017
            env:
              MONGO_INITDB_ROOT_USERNAME: root
              MONGO_INITDB_ROOT_PASSWORD: example
        steps:
          ...

- Result
  - 격리된 컨테이너 환경에서 실행하지 않고도 서비스 컨테이너를 생성하여 서비스 컨테이너 내부에서 생성된 MongoDB 내부 포트 27017과 연결 및 테스트를 통과할 수 있습니다.