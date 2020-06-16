### docker container run
도커 컨테이너 실행하기

#### --name 옵션
컨테이너에 원하는 이름 붙이기

### docker container ls
실행 중인 컨테이너 목록 보기

#### -a 옵션
정지되어 있는 컨테이너를 포함한 목록 보기

#### -q 옵션
컨테이너 ID만 목록 보기

### docker container stop
실행 중인 컨테이너 중지시키기

### docker container rm
컨테이너 파기하기. 중지되어 있는 컨테이너만 파기할 수 있음.
run 명령으로 컨테이너를 실행할 때 --rm 옵션을 주면 컨테이너를 정지함과 동시에 파기할 수 있음.

### docker container logs
해당 컨테이너의 stdout을 볼 수 있음. -f 옵션으로 실시간으로 출력 확인 가능.

### docker container exec
실행 중인 컨테이너에서 명령 실행하기. 컨테이너 내부의 상태를 확인하거나 디버깅할 때 많이 활용함.
컨테이너 안에 든 파일을 수정하는 등의 명령은 절대 금지!!

#### -it 옵션
표준 입력 연결을 유지하는 -i 옵션과 유사 터미널을 할당하는 -t 옵션을 함께 줘서 컨테이너를 셸을 통해 다루는 것처럼 할 수 있음.
```
$ docker container exec -it echo sh
pwd
/go
```
