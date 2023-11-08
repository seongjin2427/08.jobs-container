# github-actions-practice;

## initial workflow

1. `test` ➡️ `deploy`

---

## Run Jobs in Containers

1. 컨테이너를 사용해서 워크플로우를 동작시켜봅니다. - 

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
