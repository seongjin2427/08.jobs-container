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