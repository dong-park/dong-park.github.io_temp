---
layout: post
title: "소켓통신 야매 클래스"
author: "dongpark"
category: post
---

## 소켓이란?

- TCP/IP 프로토콜 기반 통신을 위해 양 프로그램 끝단에 생성되는 링크의 단자를 지칭
- *그럼 TCP/IP는 무엇 일까?*
    - 프로토콜 : 복수의 컴퓨터 사이나 중앙 컴퓨터와 단말기 사이에서 데이터 통신을 원활하게 하기 위해 필요한 **통신 규약**
    - 이 TCP/IP는 인터넷 통신에서 사용하는 핵심적인 프로토콜
    - 인터넷을 사용하고 조금이라도 커스텀한 연결을 해본 유저라면 익숙한 TCP,UDP,IP,PORT 모두 이 프로토콜에서 사용되는 언어
- *소켓과 무슨상관 일까요?*
    - 네트워크를 연결하기 위해 만들어진 연결부, 110v 220v 전기코드에도 규약이 필요하듯 소켓은 TCP/IP 프로토콜을 따릅니다.
    - 소켓은 당연히 TCP 와 연관되어 4계층 전송계층, 5계층 단위로 활용됩니다.
    - 당연히 인터넷 통신에서 사용되는 IP와 PORT를 이용해 통신을 하겠죠?
    - 보통 TCP/IP 프로토콜을 언급하면서 나오는게 TCP방식과 UDP 방식입니다.둘 예기가 나오면 보통 언급되는게 패킷에 대한 응답으로 하는 연결성 서비스인 TCP 방식 비연결성 서비스인 UDP방식 풀어 말하자면 신뢰성이 체크를 매 패킷마다 수행하는것을 TCP 방식이라 생각하시면 편하고 UDP 방식은 CheckSum을 통해 오류 체크를 진행합니다, 이외 [특이점](https://mangkyu.tistory.com/15)
    - 전세계 천재님들이 언어별로 API 형태로 (자바의 경우엔 소켓 클래스가 존재합니다) 소켓 인터페이스를 기반해 다.. 만들어져 있습니다. 덕분에 응용 개발자들은 통신 구조 설계없이 비즈니스 로직만 작성하면 됩니다.
    
    ## 소켓의 작동원리
    
    이론은 이론일뿐.. 그럼 비즈니스 로직을 어떻게 적용 할까요?

    - 게임을 생각하시면 편합니다. 서버와 클라이언트 둘로 쪼개지는 경우가 거진입니다.
    - 뭔가 복잡해보이지만, 게임이랑 다를게 없습니다
    - ***서버소켓(방장) 입장*** : bind() [방을 만든다 아이피 / 포트를 이용해] - > listen() [참가자 올때까지 기다린다..] -> accept() [참가자가 들어왔다] -> send(), recv() [게임,채팅 데이터 전송이 왔다갔다..] -> close() [방장이 나갔다]
    - ***클라이언트소켓(참가자) 입장*** : connect() [방장이 바인딩한 방에 입장] -> send(), recv() [데이터 왔다리 갔다리..] -> close() [유저가 나갔다]
    - 풀어서 볼까요?
        - socket()
            - 소켓 객체를 생성하거나 모듈을 받아오는등 선언하는 경우가 대부분입니다.
        - bind()
            - 아이피와 포트를 이용해 소켓 접속정보를 갱신합니다. 보통은 생성자나 초기화(보통 initializing, 초기값을 넣어주는) 메소드를 이용합니다. **당연히!** 기존에 사용하는 아이피 포트는 이용 해선 안됩니다. 시작이 되지도 않겠지만.. 시작되면 더문제 입니다. 소켓이 갱신되서 해당 포트를 사용하던 어플리케이션이 뻑이 나는 불상사가 일어납니다. 1~1000포트는 **[잘 알려진 포트](https://ko.wikipedia.org/wiki/TCP/UDP%EC%9D%98_%ED%8F%AC%ED%8A%B8_%EB%AA%A9%EB%A1%9D)**(well-known port) 로서 되도록이면 사용을 지양하도록 합시다.
        - listen()
            - 클라이언트의 요청을 수신하는 단계입니다. 다른 클라이언트 소켓의 응답을 기다립니다.
        - connect()
            - 클라이언트 소켓에서 아이피와 포트로 바인딩된 서버 소켓으로 연결을 시도합니다.
        - accept()
            - listen상태에서 클라이언트 소켓의 연결요청이 들어오면 연결처리를 합니다. 보통 이 과정 이후에 연결과정을 수립하는 과정을 handshaking 이라고 부릅니다.
        - send()
            - 바인딩된 서버 혹은 클라이언트 에게로 메시지를 보냅니다. 각각의 소켓기반 API 마다 정해진 양식이 있는편이지만 보통은 메시지의 종류를 나뉘는 헤더와 실제 내용이 담긴 바디문으로 전송되는 편입니다(사실 Http Request 자체가 다 이런식으로 이루어지고 있습니다). API들마다 특이점이 나뉠수도 있는데 node 진영의 socket.io 같은 경우에는 구독 개념으로 접근해 전송된 메시지를 다른 클라이언트들에게 전송하는등 컨셉에 따라 나뉘는것 같습니다.
        - recv()
            - 위에서 말한 메시지를 받습니다. 받은 메시지들은 개발자가 짜놓은 로직에 따라 핸들링되어 처리되는 경우가 대부분 입니다.
        - close()
            - 클라이언트라면 서버와의 연결된 세션을 종료하고 서버라면 다른 클라이언트들과 연결을 종료합니다.
    - 정말 작동원리만 알면 될까요?
        - 제 생각에는 위의 과정을 이해하면 보통 동일한 이름으로 API메소드들을 지원하는 경우가 대부분이기 때문에 이후 프로토콜쪽이나 다른 커스터마이징이 필요하기 전까진 기본적인 프로토콜의 대한 이해와 작동원리만 숙지하고 계셔도 충분히 어플리케이션 제작을 시작하셔도 된다고 생각합니다.
    - 예를들어서 어떤 소켓 API가 있을까요?
    
    제가 경험한 부분은 node.js 의
    
    [socket.io](https://socket.io/docs/)
    
    와 spring framework에서는
    
    [Websocket,stomp](https://www.egovframe.go.kr/wiki/doku.php?id=egovframework:rte3.5:ptl:stomp)
    
    가 있었습니다. 참조하시는데 도움이 됐으면 좋겠습니다.

