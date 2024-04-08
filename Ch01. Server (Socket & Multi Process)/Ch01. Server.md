# Ch01. Server (Socket & Multi-Process)
1주차 워크북 - 서버란 무엇인가(소켓&멀티 프로세스)

<br>

# TCP
[TCP 공식문서](https://www.rfc-editor.org/rfc/rfc793)

TCP란 서버와 클라이언트 간 데이터를 송/수신을 위한 프로토콜이다. 
    
TCP의 특징
1. reliable data
2. connection-oriented(3-way handshake)
3. in-order data
4. congestion control
5. flow control
6. 양방향 전송
7. 1:1통신 (unicast) 등

TCP 소켓은 source의 IP, port와 destination의 IP, port로 구분한다.
<br>
<br>    

# UDP

[UDP 공식문서](https://www.rfc-editor.org/rfc/rfc768.txt)
     
속성
1. unreliable data
2. out-of-order → 순서가 뒤바뀐채 전달될 가능성O
3. 비연결성
4. checksum
5. 단방향 통신
6. 속도 빠름 
    
UDP 소켓은 source의 IP, destination의 IP로 구분한다.
<br>
<br>

# 시스템 콜?
응용프로그램이 요청할 때 커널에 접근할 수 있는 인터페이스이다. 파일을 읽어오거나 파일을 쓰거나 화면에 메시지를 출력하는 등의 서비스를 수행하기 위해 OS는 하드웨어를 직접 관리한다. 반면 응용프로그램은 OS가 제공하는 인터페이스를 통해서만 자원을 사용할 수 있는데, OS가 제공하는 인터페이스를 시스템콜 이라고 한다.

운영체제가 하드웨어, CPU를 사용하는 모드를 커널 모드라고 한다. 애플리케이션 코드를 실행하는 모드를 사용자 모드라고 한다. 사용자 모드에서 하드웨어 자원에 접근하기 위해서는(=커널 영역의 기능을 사용하기 위해서는) 시스템 콜을 사용해야 한다. 

보통 API를 통해 사용한다.

[Ref](https://didu-story.tistory.com/311)
<br>
<br>
    
# 하드웨어 인터럽트
CPU 외의 장치에서 예외상황이 발생하여 처리가 필요한 경우에 컨트롤러가 인터럽트 신호를 발생시켜 CPU에게 알리는 것을 말한다.

하드웨어 장치가 CPU에게 어떤 사실을 알려주거나 CPU의 서비스를 요청해야 하는 경우, "인터럽트 발생~" 하면서 인터럽트 라인을 세팅한다. 다른 명령을 수행하고 있던 CPU는 인터럽트 라인을 확인하면 하던 일을 멈추고 인터럽트를 처리하러 간다.

- **인터럽트 벡터**

인터럽트가 발생하면 CPU는 운영체제에게 제어권을 주고 인터럽트 처리를 위해 정의된 코드(=루트)를 찾는다. 운영체제는 인터럽트가 발생하면 정의된 코드를 찾을 수 있도록 처리해야 할 루틴의 주소를 보관하고 있는 테이블을 갖고 있다. 이 테이블을 인터럽트 벡터라고 한다.

- **인터럽트 핸들러**

운영체제 커널은 인터럽트가 들어왔을 때 처리해야 할 코드를 포함하고 있다.(이미 프로그래밍 되어 있다!~) 이 코드를 인터럽트 처리루틴(ISR), 인터럽트 핸들러라고 한다.

[Ref](https://baebalja.tistory.com/354)

<br>
<br>

# 리눅스의 파일과 파일 디스크립터
- 리눅스의 파일

리눅스는 모든 것을 파일로 취급한다. 디렉토리, 소켓 등등 모든 것을 파일로 관리한다. 

- 파일 디스크립터

리눅스에서 파일을 다룰 때 사용하는 개념이다. 프로세스에서 특정 파일에 접근할 때 사용하는 추상적인 값이다. stdin(표준입력, 0), stdout(표준출력, 1), stderr(표준에러, 2)는 이미 파일 디스크립터 값이 할당되어 있기 때문에 3 이상부터 사용할 수 있다.

리눅스에서 프로세스가 소켓에 접근한다면, 소켓은 파일이므로 파일 디스크립터를 이용할 것이다. 
<br>
<br>

# socket() 시스템 콜
새로운 소켓을 생성하고 파일 디스크립터를 반환한다.

    socket(domain, type, protocol)
- domain: IPv4는 AF_INET, IPv6는 AF_INET6
- type: TCP 소켓인 경우 SOCK_STREAM, UDP 소켓인 경우 SOCK_DGRAM
- protocol: 0은 시스템이 선택, 6은 TCP, 17은 UDP
<br>
<br>

# bind() 시스템 콜
생성한 소켓에 특정 IP주소와 포트 번호를 할당한다.

    bind(sockfd, sockaddr, socklen_t)
- sockfd: 바인딩 할 소켓의 파일 디스크립터
- sockaddr: 소켓에 바인딩 할 주소
- socklen_t: sockaddr의 크기
<br>
<br>

# listen() 시스템 콜
클라이언트의 connect 요청을 대기하는 상태가 된다. connection-oriented인 TCP에서만 사용한다.

    listen(sockfd, backlog)
- sockfd: listen을 시작할 소켓의 파일 디스크립터
- backlog: connection 요청을 대기하는 큐의 크기, 만약 큐가 꽉 차면 운영체제는 큐가 빌 때까지 새로운 connection 요청을 받지 않는다.
<br>
<br>

# accept() 시스템 콜
listening을 하다가 새로운 connection이 열리면 accept()는 새로운 소켓을 만들어서 리턴한다.

    accept(sockfd, sockaddr, socklen_t)
- sockfd: connection을 받을, listening 하고 있던 소켓의 파일 디스크립터
- sockaddr: 클라이언트의 주소 정보
- socklen_t: 위 주소 정보의 크기
<br>
<br>

# 멀티 프로세스
여러 개의 CPU가 하나 이상의 작업을 동시에 처리(병렬처리)한다.
(*프로세서=CPU코어)

자식프로세스 생성 - fork()

    pid_t fork()
리턴값이 0이면 자식 프로세스, 0이 아니면 부모 프로세스를 의미한다.

멀티 프로세스는 각 프로세스가 독립적이기 때문에 한 프로세스가 죽어도 다른 프로세스에는 영향을 주지 않아 안정성이 높다. 하지만 그만큼 많은 메모리 공간과 CPU 시간을 차지한다. 
<br>
<br>

# 병렬 처리
여러 CPU에서 각 프로세스를 병렬적으로 실행한다. 
