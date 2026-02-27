# 컴퓨터구조 서버 설정 방법

## 알아두어야 할 것
* 설정에 사용되는 프로젝트
    - scripts: 서버 설정할 때 사용할 각종 스크립트 파일 (BSC 설치, 계정 생성 및 삭제 등)
    - server: 스켈레톤 파일 배부, 과제 파일 등을 관리하는 서버
    - client: 위 서버를 학생/TA/관리자 등이 접근하고 이용할 때 사용하는 클라이언트 프로그램
* 서버 신규 설치에만 수행할 부분은 (* 신규), 전년도 서버 재사용 시에만 수행할 부분은 (* 재사용)으로 표기
* 미리 생각해둘 것
    - 공유 디렉토리: /ca25, /ca26 등과 같이 당해년도를 넣어 작명. 아래에서 공유 디렉토리 란에 `ca25`가 아니라 `/ca25`와 같이 작성해야 함.
    - 테스트용 학생 계정: 계정 이름은 2025-00000과 같이 작명 (학번 형태이나 다른 학생과 겹칠 일 없게)

## 사전 작업
* OS 설치 시 주의사항 (* 신규)
    - 관리자 계정 이름: `ca-admin`
    - OpenSSH 설치 여부 체크 (데스크탑 버전은 추후에 설치)
* 인터넷 설정
    - IP 담당자에게 IP와 맥 주소를 수령하여 진행
    - 완료 여부는 `curl ifconfig.me` 실행
* 인터넷 설정이 완료되면 다음 실행 (* 신규)
    ```sh
    sudo su
    apt update
    apt upgrade -y
    apt install -y git openssh-server vim sshpass
    snap install btop tree
    ```
    - `openssh-server`는 앞에서 OpenSSH 설치하지 않은 경우 진행
    - 이후 재부팅 (권장)
* SSH 포트 220번으로 변경. (* 신규)
    - 교내에 있는 서버는 22번 포트를 열려면 귀찮으므로 220번으로 변경하는 것이 좋음.
* 디스크 마운트 (* 신규, 하단 참고)
* (root 계정이 아닌 상태로) 다음 실행 (* 신규)
    ```sh
    cd ~
    git clone https://github.com/ku-cylee/ca-mgmt-client client
    git clone https://github.com/ku-cylee/ca-mgmt-scripts scripts
    git clone https://github.com/ku-cylee/ca-mgmt-server server
    ```

### Netplan으로 인터넷 설정 방법
* 주의사항
    - Ubuntu 20.04.6 Server 버전에서 되는 것을 확인함 (Desktop 버전이나, 24 버전은 다른 방법 찾아봐야 함.)
    - Netplan 버전이 바뀌어서 세부적인 방법이 바뀌었을 수 있음. 이 부분은 찾아봐야 함.
* netplan으로 설정하는 방법 (Ubuntu 20.04.6 Server 버전 기준)
    - `sudo su`
    - `ip addr`을 실행해서 인터페이스 이름 확인 (예: enp0s3)
    - `cd /etc/netplan`
    - 디렉토리 내 파일들 삭제 (설정 실패에 대비해서 다른 위치에 옮겨 백업해두는 것도 좋음)
    - `nano /etc/netplan/00.yml` (vim이 설치되어 있지 않으므로. vi를 다룰 줄 알면 vi 써도 됨)
    - 다음과 같이 작성
        ```
        network:
            ethernets:
                <interface-name>:
                    dhcp4: false
                    addresses: [<ip-addr>/24]
                    gateway4: <gateway>
                    macaddress: <macaddress>
                    nameservers:
                        addresses: [8.8.8.8, 8.8.4.4]
            version: 2
        ```
        + `<interface-name>`: 위에서 확인한 인터페이스 이름
        + `<ip-addr>`: 할당받은 IP 주소 (예: 147.46.242.252)
        + `<gateway>`: IP 주소에서 마지막 부분만 1로 변경 (예: 147.46.242.1)
        + `<macaddress>`: 할당받은 MAC 주소 (예: 12:34:56:78:9A:BC)
    - Ctrl+X -> Y -> Enter 눌러서 저장
    - `netplan apply -t` 실행해서 00.yml 파일을 잘 작성했는지 확인
    - `netplan apply`해서 적용

### SSH에서 포트 번호 바꾸는 법
* `/etc/ssh/sshd_config` 파일을 열고, `#Port 22`로 된 부분을 `Port 220`으로 변경
* `sudo service ssh restart` 실행

### 디스크 마운트
* 디스크 마운트는 수강생이 많은 경우, 수강생 당 최소 용량을 확보해주기 위해 수강생들의 홈 디렉토리를 고용량 HDD에 할당해주기 위한 작업임.
    - 수강생 인당 35 - 40GiB 정도로 잡고 계산해서 디스크 마운트 여부 결정할 것.
* `sudo fdisk -l` 실행해서 마운트 할 디스크 결정 (/dev/sdX 형태 기억)
* `sudo blkid` 실행해서 /dev/sdX의 UUID 값 확인
* `/etc/fstab` 파일 루트 권한으로 실행하여 맨 밑에 다음 줄 추가
    ```
    UUID=<uuid>     /home/stdnt     ext4    defaults    0   0
    ```
    - 공백 사이는 모두 탭(`\t`)
    - 잘못 작성하면 부팅이 안 될 수 있으므로 주의 (백업 생성하는 것도 좋은 방법)
* `mount -a`로 마운트 (여기서 오류나면 반드시 `fstab` 파일을 수정하여야 함)
* 재부팅 후 `df -h`를 통해 /home/stdnt에 디스크가 잘 마운트 되었는지 확인

## 제출 서버 설정

### 프로그램 설치 (* 신규)
* Node.js
    ```sh
    curl -sL https://deb.nodesource.com/setup_22.x | sudo bash -
    sudo apt install -y nodejs
    ```
    - `node -v`로 설치 확인
* MySQL
    ```sh
    sudo apt install -y mariadb-server
    ```
    - `mysql -V`로 설치 확인
* yarn, forever: `sudo npm i -g forever yarn`

### 서버 설정
* server/ 디렉토리로 이동
* 실행: `yarn install`
* `.env` 파일을 다음과 같이 작성
    ```
    PORT=10000

    ADMIN_USERNAME=<rand8>
    ADMIN_SECRETKEY=<rand64>

    DB_USER=<db-user>
    DB_PASS=<rand32>
    DB_NAME=<db-name>
    DB_SYNC=true
    DB_LOGGING=false
    DB_ENTITIES=dist/src/models/**/*.entity.js

    BOMB_SERVER_PORT=10002
    BOMB_SERVER_SECRET=<rand64>
    ```
    + `<randN>`은 N자리 random hex string으로 작성하라는 뜻임. 온라인 SHA256 생성기 등으로 생성할 것.
    + (* 재사용) `<db-user>`과 `<db-name>`은 재사용이 불가능함. 해가 바뀌면 반드시 변경. 두 값은 같아도 상관없음.

### DB 설정
* `sudo mysql`로 MariaDB 접속
* 다음 세 명령어를 실행하여 계정 및 DB 생성
    ```
    create user '<db-user>'@'%' identified by '<db-pass>';
    create database <db-name>;
    grant all privileges on <db-name>.* to '<db-user>'@'%';
    ```
    + db-user, db-pass, db-name은 위의 .env 파일의 `DB_USER`, `DB_PASS`, `DB_NAME` 값임.
* 설정이 끝나면 Ctrl+D를 눌러 빠져나옴.
* 다음 명령어로 설정이 잘 되었는지 확인: `mysql -u<db-user> -p<db-pass> -D<db-name>`

### 서버 실행 및 끄기
* server/ 디렉토리에서
* 서버 빌드 (코드 변경 시에만): `yarn build`
    - 패키지가 추가되면 `yarn install`도 해야 함
* 실행: `yarn start:prod`
* 백그라운드에서 실행 유지: `forever start -c "yarn start:prod" .`
* 백그라운드 실행 중인 서버 끄기
    - `forever list`에서 서버의 uid 확인
    - `forever stop <uid>`
    - 서버의 자식 process를 죽지 않는 문제가 있음. 서버를 재실행했는데 해당 포트가 이미 사용 중이라는 오류가 뜨면 99% 기존 프로세스가 살아있어서 그럼. `pkill -f node`로 죽여야 함.

### 환경 변수 설정 (* 신규)
* /etc/profile을 루트 권한으로 열고
* `export CA_SERVER_URL=http://localhost:10000`

## BSC (BlueSpec Compiler) 설치
* scripts 프로젝트의 ./envs/setup-bsc 파일에서 `SHARED_DIR`, `TARBALL_URL` 값 업데이트
    - `SHARED_DIR`은 앞에서 설정함.
    - `TARBALL_URL`은 [bsc 프로젝트 레포](https://github.com/B-Lang-org/bsc/)의 릴리즈 목록에서 버전과 우분투 버전을 잘 선택하여 설정
* `sudo ./envs/setup-bsc` 실행
* 설치 확인
    - 서버에 재로그인 후, `bsc` 커맨드를 실행하여 설치가 완료됨을 확인
    - 과제 프로젝트 등이 잘 실행되는지 확인


## 제출 클라이언트 설정
* client/ 디렉토리로 이동

### 설치 (* 신규)
```sh
sudo apt install -y python3-pip
pip3 install -r requirements.txt
```

### 수정할 부분
* src/student.py에서 `SHARED_DIR` 값 업데이트 (앞에서 설정한 공유 디렉토리)
* src/ta.py에서 `SUBMISSION_EXCLUDE_USERS` 값을 테스트용 계정 포함하게끔 수정 (예: `['2026-00000']`)

### Deploy
* 빌드 (코드 수정 시)
    - `./build`
    - dist/ 디렉토리 내에 caadmin, cata, calab 세 파일이 생성되어 있어야 함
* 복사
    - `cp dist/* <shared-dir>/scripts` 실행
    - `<shared-dir>`은 앞에서 설정한 공유 디렉토리
    - 권한 변경 (중요)
        - `sudo ./authority <shared-dir>`
        - `ls -al <shared-dir>/scripts/`를 했을 때 cata의 그룹이 ta, caadmin의 모드가 770, cata의 모드가 750으로 바뀌어있어야 함
* 환경변수 설정
    - `/etc/profile`을 루트 권한으로 열고
    - (* 신규) 아래와 같이 추가
    ```
    if [ -d <shared-dir>/scripts ]; then
        PATH="<shared-dir>/scripts:$PATH"
    fi
    ```
    - (* 재사용) scripts가 들어간 부분에서 공유 디렉토리 이름만 변경
* 확인
    - `caadmin`, `cata`, `calab` 등을 실행하여 확인
    - TA는 `caadmin`이, 학생은 `caadmin`과 `cata`를 실행할 수 없어야 함.

## 계정 관리
* scripts 프로젝트의 ./accounts 디렉토리의 파일들 활용
* `sudo apt install -y sshpass` 실행하여 설치
* 전년도 공유 디렉토리 삭제 (* 재사용)

### 계정 생성 과정
* 계정을 생성할 때는 계정 이름과 secret key를 제공하여야 함.
    - secret key != 계정 비밀번호. 서버에 접속할 때 사용하는 비밀번호임.
* 계정 이름은 다음과 같이 작명
    - TA: 이메일 아이디 (e.g. cylee@davinci.snu.ac.kr -> cylee)
    - 학생: 학번 (e.g. 2025-12345)
* 먼저 각 스크립트에 대해 이해하고, 하단의 관리하는 방법을 따라 설정할 것을 추천.

#### Secret Key 생성
* scripts 디렉토리에서, tmp 디렉토리 생성 (BSC 설정을 했다면 이미 있을 것)
* tmp 디렉토리 밑에 파일을 생성하고 (예: tmp/tas.txt) 계정 이름 목록을 각 줄에 작성. 예:
```
cylee
ihchoi
hejung
```
* `./accounts/generate-secrets tmp/tas.txt` 실행하면, tmp/tas.secret.txt에 계정 이름과 secret key가 기록된 계정 데이터 파일이 저장되어 있음. 예:
```
cylee,125d2c11976b460bb4c48736e8cac8a6
ihchoi,0fff1f1b0ffb2f1880f647bb778cd6fa
hejung,0b3e31e4bf952002ed637ea52693d20b
```

#### 계정 생성
* accounts/의 `setup-ta`, `setup-stdnt`는 각각 TA, 학생들을 관리하기 위한 그룹, 디렉토리 등을 생성하고, 앞에서 생성한 계정 데이터 파일에 있는 계정들을 생성함.
    * TA 계정의 홈 디렉토리는 `/home/ta/<ta-username>`에 생성
    * 학생 계정의 홈 디렉토리는 `/home/stdnt/<stdnt-name>`에 생성
    * 비밀번호는 자동으로 초기화 됨
* 사용
    - 예: `./accounts/setup-ta tas.secret.txt`
    - 최초 1번은 반드시 실행해야 함.
    - 최초 1번 실행 이후에도 또 실행해도 상관없음.
* accounts/의 `single-ta`, `single-stdnt`는 계정 이름과 secret key로 계정을 생성함.
    - 예: `./accounts/single-ta cylee 125d2c11976b460bb4c48736e8cac8a6`
    - 앞의 setup 스크립트가 최소 1회 실행된 상태라야 함.

#### 계정 등록
* 앞의 서버, 클라이언트 설정이 완료되고, 서버가 켜져있어야 함.
* TA 계정은 `caadmin ta add <data-file-path>`로 추가
* 학생 계정은 `caadmin stdnt add <data-file-path>`로 추가
* 예: `caadmin ta add ./tmp/tas.secret.txt`

### 계정 삭제
* `./accounts/del-stdnts`는 모든 학생 계정을 삭제함 (종강 시 사용)
* `./accounts/del-tas`는 모든 조교 계정을 삭제함
* 개개인 계정 삭제 스크립트는 없음 (리눅스 명령어로 삭제하고, 해당 계정의 홈 디렉토리를 지워주면 됨.)
* 제출 서버에서 계정 삭제
    - `caadmin ta delete <username>`
    - `caadmin stdnt delete <username>`
    - 종강 및 서버 폐쇄 시에는 굳이 하지 않아도 됨. 수강취소생에 한하여 진행
    - 주의: 이미 삭제한 사용자는 재등록이 불가능함. 재등록이 필요하다면 DB를 직접 열어 해당 row를 찾아 직접 삭제.

### 계정 비밀번호 초기화
* TA나 학생으로부터 계정 비밀번호 초기화 요청을 받았을 때 실행.
* TA는 `./accounts/reset-ta-pass`, 학생은 `./accounts/reset-stdnt-pass`

#### 관리하는 방법
* 계정 생성 절차는 항상 사용자 이름 목록 생성 -> secret key 포함된 데이터 파일 생성 -> 계정 생성 -> 계정 등록 순서대로 진행
* 기존 조교, 수강생 계정 모두 삭제 (* 재사용)
    - secret key 때문에 조교도 삭제해야 함.
* TA는 위와 같이 진행하면 됨
* 학생 계정
    - 통상 Lab 0가 개강 직후에 배부되고 과제 기한이 길지 않음.
    - 개강 직후 학생들이 수강 정정 등을 통해 계속 들어오므로 24/25년도에는 다음과 같은 방식으로 과제 마감 기한을 설정하였음.
        + 예: 3/5 배부, 3/11 마감
        + 3/5 신청자까지는 3/11 마감
        + 3/6 신청자까지는 3/12 마감
        + ... 3/12 신청자까지는 3/18 마감
    - 이를 위해 학기 초 수강 신청 기간에는 매일 18시에 추가 신청자를 확인하고, 해당 인원에 대한 계정을 생성하였음.
        + 이로 인해 Lab 0의 마감일은 가장 늦게 신청한 경우를 기준으로 설정하여야 함.
    - 신규 인원을 매번 찾아내기 귀찮으므로 다음과 같이 진행함.
        1. (3/5) eTL의 강의/출결에서 학생 목록을 excel로 다운로드 하고, 학번 부분만 복사하여 tmp/0305.raw.txt를 생성 -> 계정 생성
        2. (3/6) 1과 같이 tmp/0306.raw.txt 생성
        3. `./accounts/new-stdnts tmp/0305.raw.txt tmp/0306.raw.txt > tmp/0306.txt` 실행하여 3/6에 신규 유입된 수강생들 목록 생성
        4. tmp/0306.txt를 이용하여 계정 생성 절차를 진행하여 계정 생성.
        5. (3/7) 2-4 반복.
    - Lab 0 채점 시 0306.txt 등을 확인하여 각 수강생에 대한 마감 기한을 확인할 수 있음.
* 조교들이 테스트 해볼 수 있도록 테스트용 학생 계정을 생성.
    - 아이디: 클라이언트 설정 부분 확인
    - 비밀번호: 계정 생성 후 들어가서 비밀번호를 바꿔주어야 함. (일반 학생들 이용 방지)
* 수강 취소한 수강생 계정은 삭제. 자주 해줄 필요는 없으나 중간고사 즈음하여 있는 최종 수강 취소 기간 이후에는 해주는 것이 권장됨.

## 이외 서버 관리자가 학기 초에 해야할 것

### Lab 0
* Lab 0 문서를 수정하여 배포

### 서버 사용 문서 수정하여 배포
* 학생용: IP, 년도 등을 변경하여 배포
* 조교용: 테스트 계정 등을 변경하여 배포
* 추가/변경/삭제된 기능이 있다면 반영하여 배포
