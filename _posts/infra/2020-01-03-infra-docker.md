---
layout: post
title:  "[Infra] Docker"
categories: [Infra]
tags: [docker]
---

### 도커(Docker)의 기본 개념과 간단한 실습

> 이미지와 컨테이너
>
> * 이미지 : 미리 구성된 환경을 저장해 놓은 파일들의 집합
>   * Immutable : 불변한 저장 매체
> * 컨테이너: 이러한 이미지를 기반으로 실행된 격리된 프로세스
>   * Mutable : 특정 이미지를 통해 무언가를 더해서 새로운 이미지를 만들어내는 것이 가능. 즉, 변경 가능함
>
> 즉, 컨테이너는 **가상머신**이라기 보다는 **프로세스**에 가까움






* docker 명령어

  * docker ps

    * 실행되어 있는 container 리스트 조회

      ```bash
      $ docker ps                                                                     CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
      ```

  * docker ps -a

    * 종료된 container를 포함하여 리스트 조회

      > 컨테이너는 SSH 서버가 아니라 배시 쉘 프로세스이기 때문에 쉘을 종료하면 컨테이너도 종료됨
      >
      > docker run 할 때 --name을 통해 이름을 지정하지 않으면 임의의 이름이 자동적으로 부여됨

      ```bash
      $ docker ps -a
      CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
      e6db8abd6be9        centos:latest       "bash"              3 minutes ago       Exited (0) 11 seconds ago                       elastic_curie
      7c2206703f0f        hello-world         "/hello"            17 hours ago        Exited (0) 17 hours ago                         ecstatic_tu
      ```

  * docker restart <CONTAINER_ID>

    * exit된 container 재시작

      ```bash
      $ docker restart e6db8abd6be9
      e6db8abd6be9
      ```

    

  * docker attach <CONTAINER_ID>

    * 컨테이너 restart 후 실행된 프로세스와 터미널 상에서 입출력을 주고받기 위한 명령어

      ```bash 
      $ docker attach e6db8abd6be9
      [root@e6db8abd6be9 /]#
      ```

  * docker images

    * 이미지화가 되어있는 목록 조회

      > git/svn과 같은 명령어(pull, push, commit 등)을 사용하므로 도커에서는 하나의 이미지를 저장소(repository)라고 부르기도 함
      >
      > IMAGE ID는 이미지를 가리키는 고유한 해시값
      >
      > CREATED는 이미지가 생성된 시간
      >
      > SIE는 이미지의 전체 용량을 나타냄

      ```bash
      $ docker images
      REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
      centos              latest              0f3e07c0138f        3 months ago        220MB
      hello-world         latest              fce289e99eb9        12 months ago       1.84kB
      ```

  * docker pull <IMAGE_NAME>:\<TAG\>

    * IMAGE_NAME 뒤에 TAG를 생략할 경우 latest를 태그를 사용함

      > 일반적으로 TAG는 이미지의 versioning을 위해 사용됨

    * git/svn 처럼 pull을 이용해 image를 땡겨받고 push, commit 명령어를 통해 [docker-hub](https://hub.docker.com/)에 push 할 수 있음

      ```bash
      $ docker pull centos
      Using default tag: latest
      latest: Pulling from library/centos
      729ec3a6ada3: Pull complete                                                    Digest: sha256:f94c1d992c193b3dc09e297ffd54d8a4f1dc946c37cbeceb26d35ce1647f88d9
      Status: Downloaded newer image for centos:latest
      docker.io/library/centos:latest
      ```

      > 하기 5가지 이미지 주소는 모두 같은 이미지를 가리킨다.
      >
      > * centos:latest
      > * centos@sha256:f94c1d992c193b3dc09e297ffd54d8a4f1dc946c37cbeceb26d35ce1647f88d9
      >
      > * library/centos:latest
      > * docker.io/library/centos:latest
      > * index.docker.io/library/centos:latest

  * docker run -it 

    * bash shell을 통해 container 실행

      ```bash
      $ docker run -it centos:latest bash
      [root@e6db8abd6be9 /]#
      ```

  * 이외의 명령어

    * stop : 컨테이너 강제 종료

    * rm : 종료된 컨테이너를 삭제

      * docker rm : 컨테이너를 삭제하는 명령어

      * docker rmi : 이미지를 삭제하는 명령어

        > 이미지에서 파생된 (종료상태를 포함하여) 컨테이너가 하나라도 있으면 이미지 삭제가 불가능함

    * run 명령어와 함께 --rm 플래그 : 컨테이너가 종료 상태로 들어갈 때 자동으로 삭제해주는 옵션

* bash 명령어

  * apt-get update

    * 설치 가능한 패키지들 update하는 정도로 생각하면 될 것으로 보임

      ```bash
      root@6a001ef51fce:/# apt-get update
      ...
      Reading package lists... Done
      ```

      

  * apt-get install -y git

    * git 설치

      ```bash
      root@6a001ef51fce:/# apt-get install -y git
      ...
      root@6a001ef51fce:/# git --version
      git version 2.7.4
      ```

    * 다른 터미널에서 부모 이미지와 파생된 컨테이너의 파일 시스템 간의 변경 사항 확인 "docker diff \<CONTAINER ID\>"

      ```bash
      $ docker diff 6a001ef51fce
      ...
      A /etc/init.d/networking
      C /etc/init.d/.depend.stop
      C /etc/init.d/.depend.start
      C /bin
      A /bin/lesskey
      ...
      ```

      >  A - ADD, C - Change, D- Delete의 줄임말

* Dockerfile

  > Dockerfile은 특정한 이미지로부터 새로운 이미지 구성에 필요한 일련의 명령어들을 저장해놓는 파일이다.
  >
  > 장점 : Dockerfile 하나로 어플리케이션 배포 환경을 구출할 수 있으며, 또한 쉽게 유연하게 사용할 수 있음



###### 출처

[도커(Docker) 튜토리얼 : 깐 김에 배포까지](https://www.44bits.io/ko/post/easy-deploy-with-docker)

[만들면서 이해하는 도커(Docker) 이미지의 구조](https://www.44bits.io/ko/post/how-docker-image-work)