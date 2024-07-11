---
layout: post
title:  "Mac 오라클 설치, HR 샘플 스키마 사용하기"
author: "minseo"
categories: [blog]
tags: [Database, Oracle, Mac]
comments: true
hide_image: true
---
# [Oracle] Mac 오라클 설치, HR 샘플 스키마 사용하기

<center><img src="https://allvectorlogo.com/img/2017/02/oracle-database-logo.png" width="50%"></center>

<br>

오라클은 공식적으로 맥을 지원하지 않는다. 따라서, 도커를 이용하여 오라클 데이터베이스를 사용할 수 있다.

## ❗️ To do

- Oracle XE 설치하기
- SQL Developer와 연결하여 Oracle XE 사용하기
- HR 샘플 스키마 사용하기

<br>

이 세 가지 과제를 수행하기 위해 macOS에서 **어떤** 작업들이 **왜** 필요한지 알아보자.

<br>

---

# Oracle XE 설치하기

Oracle XE (Oracle Database Express Edition)란 오라클 데이터베이스의 기능을 무료로 사용할 수 있도록 제공하는 에디션이다. 공식 홈페이지에 접속하면 `간편한 다운로드, 완전한 기능, 손쉬운 사용`이라는 문구로 XE를 자랑하고 있다.

<br><center><img src="../../../static/img/240623/unhappyMac.png" width="60%"></center>
<div class="figcaption"> 그러네 리눅스 환경에서만 작동하는 프로그램 때문에 도커가 필요하네 </div>

그러나 다운로드 페이지에 접속한 macOS 사용자는 오라클을 간편하게 다운로드할 자격을 박탈당한다.

하지만 우리는 튼튼한 맥북 위에 원하는 운영체제를 올려 사용할 수 있는 시대에 살고있기 때문에 이 문제를 해결할 수 있다. 무엇보다 손쉽게 가상화된 Linux OS를 사용할 수 있는 Docker Engine이라는 기술을 활용할 수 있다.

**도커 엔진**을 이용하여 **Oracle XE가 설치된 컨테이너**를 만들고, **컨테이너 내부에서 오라클 데이터베이스를 사용**해보자.

## Colima

도커 이미지로 컨테이너를 생성하기 전에 필요한 작업이 있다.  
Oracle XE가 설치된 컨테이너는 `x86_64` 아키텍처를 요구한다. 그렇기 때문에 사용하는 맥북의 CPU 아키텍처를 확인해야 한다.

```bash

    # 터미널에서 CPU 아키텍처를 확인하는 명령어
    uname -m

```

내가 사용하는 맥북은 M 시리즈 M3 pro로 `arm64` 아키텍처를 쓰는 모델이다. 따라서 지금 상태로는 컨테이너를 실행할 수 없기 때문에 Colima로 `x86_64` 가상 머신을 생성하여 그 위에 컨테이너를 띄울 것이다. **Colima**는 경량화된 리눅스 가상 머신을 통해 도커 엔진을 사용할 수 있는 컨테이너 런타임 환경이다.

<br>

```bash

    # Colima 설치
    brew install Colima

```

참고로 Colima가 인텔 기반 프로그램이기 때문에 애플 실리콘 맥에서 실행하기 위해 Rosetta를 설치하겠냐는 권유 팝업이 떠서 Rosetta도 설치해줬다.

<br>

```bash

    # Colima 실행
    colima start --arch x86_64 --memory 4

```
- `--arch`: x86_64 CPU architecture
- `--memory`: 4GB memory

<br>

이제 원하는 조건을 갖춘 리눅스 환경에서 오라클을 위한 컨테이너를 실행할 수 있게 되었다.

## Docker Container 생성 및 실행

도커 허브에 oracle-xe를 검색했을 때 스타가 가장 많은 `gvenzl/oracle-xe` 이미지를 사용하였다. 최신 버전인 21c를 포함하고 있다. Overview를 읽어보고 필요에 따라 더 낮은 버전의 이미지를 사용할 수도 있다. 

```bash

    # Pull an image from Docker Hub
    docker pull gvenzl/oracle-xe

    # Run (create + start) Docker Container
    docker run --name {$containerName} -d -p 1521:1521 -e ORACLE_PASSWORD={$yourOwnPassword} gvenzl/oracle-xe

```

- 오라클은 기본적으로 1521 포트를 사용한다.
- `-e`: 컨테이너에 ORACLE_PASSWORD라는 환경변수를 주입한다.  
    오라클 데이터베이스에 접속할 때 사용할 비밀번호를 설정해준다.

<br>

```bash

    # 실행중인 Container 확인
    docker container ls -a

```

컨테이너가 정상 실행 중이라면 해당 컨테이너 내부로 들어간 뒤 bash로 oracle 데이터베이스에 접속한다.

<br>

```bash

    # 컨테이너 접속 후 bash 실행
    docker exec -it {$containerName} bash

    # oracle 접속
    sqlplus
    [Enter user-name]: system
    [Enter password]: {$yourOwnPassword}

```

<center><img src="../../../static/img/240623/oracleContainer.png" width="70%"></center>

참고로 오라클 데이터베이스의 기본 계정은 system이기 때문에 system, 설치 시 입력한 비밀번호<span style="color:#737373; font-size:14px; font-weight:300;"> 주입한 환경변수로 설정되었다 </span>로 접속하면 된다. 나처럼 sys로 접속하면 실패한다 😅

위 사진처럼 `SQL > ` 쉘이 나타나면 오라클 데이터베이스에 성공적으로 접속한 것이다.

<br>

---

# SQL Developer를 통해 Oracle 접속하기

쉘 창에서 작업하는 것보다 좀 더 직관적이고 아름다운 시각 효과를 얻고 싶다면 추가로 애플리케이션 인터페이스를 사용할 수 있다. SQL Developer는 오라클 데이터베이스 개발과 관리를 단순화하는 무료 통합 개발 환경이다. 설치하고 사용해보자. SQL Developer는 맥 유저도 간편하게 다운로드할 수 있다~<br><br>

❗️ <a href="https://www.oracle.com/database/sqldeveloper/technologies/download/" target="_blank">SQL Developer Downloads</a>
<br><br>

설치 후 SQL Developer를 실행하고 왼쪽 위 초록색 + 버튼으로 `새 접속`을 생성해준다. 데이터베이스 이름 설정, 사용자 지정, 호스트와 포트 번호, SID를 확인해준 뒤 테스트 먼저 진행해준다. 테스트가 성공하면 접속하면 된다.<br>

- **cf.** `SID`: 기존에는 하나의 인스턴스에서 하나의 DB만 사용했는데, 오라클이 Multi-tenant DB가 되면서 하나의 인스턴스에서 여러 DB를 사용할 수 있게 되었다. 따라서 이 여러 개의 데이터베이스를 구별해주는 것이 SID이다.

    ```sql

        SELECT instancee from v$thread;

    ```
    위 명령어를 통해 현재 사용하고 있는 DB가 인스턴스 내부에서 어떤 이름으로 식별되는지 확인할 수 있다. 동일한 과정을 진행했다면 `XE`일 것이다.

<br><br>

<center><img src="../../../static/img/240623/sqldeveloperconnect.png" width="70%"></center><br>

이제 SQL Developer로 컨테이너 내부의 오라클 데이터베이스를 사용할 수 있게 되었다.

<br>

---

# HR Sample Database 사용하기

오라클에서 제공하는 샘플 데이터베이스를 가져와 편리하게 SQL문을 연습해보자. 나는 데이터베이스 수업 시간에 자주 사용했던 HR 스키마를 오라클 데이터베이스에 가져올 것이다. 아래의 레포지토리에서 설치 파일과 방법을 확인할 수 있다.<br><br>

❗️ <a href="https://github.com/oracle-samples/db-sample-schemas" target="_blank">oracle-samples/db-sample-schemas repository</a>
<br><br><br>

여기서 두 번째 문제가 생기는데, 설치 파일을 받아서 바로 실행하면 당연히 오라클이 설치되지도 않은 내 맥 컴퓨터에 설치를 진행하려 할 것이다. 따라서 샘플 스키마 설치를 위한 파일들을 오라클이 설치된 컨테이너 <span style="color:#737373; font-size:14px; font-weight:300;">  오라클이 설치된 컴퓨터라고 생각하자 </span> 로 복사하여, 컨테이너 내부에서 하나하나 실행시키는 방법을 사용하기로 했다.

```bash

    # 아래 파일들이 있는 위치에서 실행
    # cd ~/db-sample-schemas-23.3/human_resources

    docker cp hr_install.sql {$containerId}:/home

    docker cp hr_code.sql {$containerId}:/home

    docker cp hr_create.sql {$containerId}:/home

    docker cp hr_populate.sql {$containerId}:/home

    docker cp hr_uninstall.sql {$containerId}:/home

```

HR 스키마를 위한 파일들을 컨테이너의 홈 디렉토리에 전부 복사해줬다. <br><br> 

이제 컨테이너의 홈 디렉토리에서, 오라클 데이터베이스에 접속한 뒤 `hr_install.sql` 스크립트를 실행해주면 된다.

```bash

    # hr_install이 있는 위치에서
    sqlplus
    
    # 설치를 위한 스크립트 실행
    @hr_install.sql
    
```

스크립트를 실행하면 HR 스키마를 사용할 hr 유저의 비밀번호를 포함한 몇 가지 설정 과정이 진행된다. 나중에 여기서 설정한 비밀번호와 사용자명 `hr`으로 데이터베이스에 접속하면 된다.<br>

<center><img src="../../../static/img/240623/hr-install.png" width="70%"></center>

이렇게 설치가 완료되고 데이터베이스 접속이 해제되면 이제 오라클 데이터베이스에서 샘플 스키마를 사용할 수 있게 된다!

<br><center><img src="../../../static/img/240623/sql-hr-ex.png" width="70%"></center><br>
<div class="figcaption"> SQL Developer에서 HR 샘플 데이터로 질의문을 수행한 모습이다. </div>


<br><br>
<details>
<summary> &nbsp; 📁 참고 자료</summary>
    <div>
    ❗️ <a href="https://velog.io/@seok990301/CPU-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98" target="_blank">https://velog.io/@seok990301/CPU-아키텍처</a>
    </div>
    <li>
    <bl>데이터베이스 수업</bl>
    </li>
</details>