## 실시간 채팅 구현해보기

### REST API 와 WebSocket의 차이
#### REST API
REST API는 Representational State Transfer의 약자로, 웹 표준을 기반으로 하는 통신 방법이며 HTTP 메소드 (GET, POST, PUT, DELETE 등)를 사용하여 클라이언트와 서버 사이의 상호작용을 정의한다 이는 주로 **단방향 통신**을 의미하며, 클라이언트가 서버에 요청을 보내면 서버는 해당 요청에 대한 응답을 보내는방식이고 각 요청은 독립적이며 이전 요청의 상태 정보를 저장하지 않는다(stateless)
#### Websocket
WebSocket은 웹에서 실시간 **양방향 통신**을 가능하게 하는 프로토콜이며 클라이언트와 서버 간에 초기 핸드셰이크를 통해 연결이 이루어진 후, 이 연결은 네트워크 지연 시간 없이 양방향 데이터 전송을 지원한다 이러한 특성은 실시간 채팅 애플리케이션, 온라인 게임, 실시간 뉴스 피드 및 기타 실시간 웹 애플리케이션에 적합하고 핸드셰이크가 완료된 후 웹소켓은 지속적으로 열려 있어, 서버 또는 클라이언트가 언제든지 데이터를 주고받을 수 있다

#### Polling( 폴링 )이란?
클라이언트가 일정한 간격으로 서버에 요청을 보내서 결과를 전달받는 방식
```javascript
const POLL_TIME= 1000;
setInterval(()=>{
  fetch('https://example.com/location');
}, POLL_TIME)
```
#### Polling의 단점
위의 예제처럼 1초마다 서버에 요청을 보내 데이터를 받아올수있지만 서버의 상태가 변하지 않았을때에도(서버에서 바뀐 데이터가 없어도) 계속 요청을 보내서 받아와야하기때문에 필요없는 요청이 많아져서 서버의 성능에 부담을 줄 수 있고 주기가 길어진다면 실시간성이 떨어지게 됨

#### Long Polling 의 등장
Polling의 단점으로 인해 새롭게 Long Polling이 나왔으며 long polling또한 서버에 계속 요청을 보내지만 Polling과는 다르게 요청을 보내면 서버가 대기하고 있다가 이벤트가 발생했거나 타임아웃이 발생할때까지 기다린후에 응답을 주게되며 클라이언트는 응답을 받자마자 다시 요청을 보내게 된다 (polling방식보다는 서버의 부담이 줄어들지만 아직까진 효율적이지 못함)

#### Streaming 이란?
클라이언트에서 서버에 요청을 보내고 끊기지 않는 연결상태에서 계속 데이터를 수신하는 방식이며 양방향 소통보다는 서버에서 계속 요청을 받는것에 유용

#### 실시간 양방향 통신에 HTTP 프로토콜이 가진 문제점과 WebSocket이 해결할 수 있는 이유
polling, long polling, HTTP streaming 이 세가지는 결국에는 HTTP 프로토콜을 사용하여 HTTP request와 response에 Header가 같이 전달되는데 Header에 많은 데이터가 들어가 있어서 너무많은 요청과 응답의 교환은 결국 서버든 클라이언트든 부담을 가지게된다 그에반해 WebSocket은 처음에 접속확립(handshake)을 위해서만 HTTP를 이용하고 이후부터는 Header가 훨씬 간소화된 프레임을 사용하는 독립적인 프로토콜 **ws**(WebSocket)를 이용하게되며, ws은 핸드쉐이크가 완료되고 **연결을 끊기 전까지는 계속 연결**되어있기때문에 실시간데이터 처리에 적합하다 (HTTP 요청은 응답이 온후 저절로 연결이 끊긴다)

### WebSocket 이용해보기
서버에서는 ws라는 모듈로 웹소켓을 구현해줄수 있지만 클라이언트측(브라우저)은 지원을 안하기때문에 `native WebSocket`(JS에서 제공하는 WebSocket 객체)를 이용해서 클라이언트측 로직을 구현해야한다
1. `npm init -y`
2. `npm install ws`
![image](https://github.com/foriinrangelen/Real-time-chat/assets/123726292/22db6a5b-9613-462b-9647-e0b567a66763)

#### Socket IO 모듈 사용해보기
기존 클라이언트에서 사용했던 native websocket object와 서버에서 사용했던 ws모듈은 서로다른 인터페이스이고 WebSocket객체 또한 모든 브라우저에서 사용할 수있는 객체가 아니기 때문에 NodeJS에서 Websocket을 사용할 때 훨씬 더 간편하게 사용할 수있는 모듈인 Socket IO를 사용해보기 (**socket io 모듈은 같은 socket io모듈로 클라이언트 서버 모두 컨트롤이 가능하고 모든 브라우저에서 사용가능하다**)

`npm install socket.io`

#### Socket IO 모듈의 특징
1. WebSocket 연결을 설정할 수 없는 경우 long polling으로 대체되서 사용해준다
2. 주기적으로 연결상태를 확인하는 하트비트 메커니즘이 포함되어있어 연결이 불특정한이유로 끊어졌을 경우 **자동으로 다시 연결**해준다
3. **클라이언트가 연결이 해제되면 패킷이 자동으로 버퍼링되고 다시 연결되면 전송**한다(기본적으로 소켓이 연결되지않은 동안 발생한 모든이벤트는 다시 연결될때까지 버퍼링된다)
4. 이벤트를 보내고 응답을 받는 편리한 방법을 제공한다
5. 서버측에서는 **연결된 모든 클라이언트** 또는 **클라이언트 하위집합**에 이벤트를 보낼 수있다
6. 네임스페이스(멀티 플렉싱)

### Express + Socket.io 로 채팅 구현해보기
1. `npm install express socket.io`
2. `npm install nodemon -D`

### Namespace vs Rooms
`https://socket.io/docs/v4/emit-cheatsheet/`
![image](https://github.com/foriinrangelen/WebSocket-App/assets/123726292/39be7f0b-6188-41ec-8cee-1e122b179dd9)

#### 소스코드로 보기
```javascript
  // "myNamespace" 네임스페이스의 room1에 있는 모든 클라이언트에게
  io.of("myNamespace").to("room1").emit(/* ... */);
```
![image](https://github.com/foriinrangelen/WebSocket-App/assets/123726292/71c2ef58-d0d7-4381-ae93-a028c685e70f)

### Private chat (DM) 구현해보기
1. `npm install express socket.io mongoose`
2. `npm install nodemon -D`

