요청 보내는 Client가 동일한 Origin이 아닐 경우 CORS 500 에러 발생

# Origin
``Origin`` request 헤더는 fetch가 시작되는 위치.
경로 정보는 포함하지 않고 서버 이름(출처)만 포함한다.
CORS requests와 함께 전송되고, Referer 헤더와 비슷하지만 전체 경로를 공개하지는 않는다.


POST요청과 Get 요청의 Origin ?


Same-Origin-Policy(SOP)
브라우저의 기본 정책

CORS
CORS 정책의 주체는 브라우저(브라우저에서 의해 실행)
CORS는 서버 간의 통신에는 적용되지 않음.
CORS는 브라우저에서 브라우저 사용자를 보호하기 위함.

CORS를 올바르게 설정하기 위해서는 요청을 받는 주체에서 정책을 설정한다.

CORS, SOP 설정은 서버에서, credentials 설정은 클라이언트에서.

- 클라이언트는 credentials를 요청에 포함
- 서버는 CORS, SOP 정책을 설정

### Client 

credentials
read-only 속성으로 요청에 credentials를 주고 받아도 되는지에 대한 정보.
 - credentials: Cookie, Authorization Header, TLS client certificates를 의미한다.
 - 기본 값은 ``same-site`` 같은 사이트에서만 credentials를 포함.

```js
fetch('https://example.com/hello', {
  method: 'GET',
  credentials: 'include'
})

axios.get('https://example.com/hello', {
  withCredentials: true
})
```
credentials 설정 시 포함가능.


### Server
- Origin
- Method
- Header
- Credentials
허용 여부에 대한 명시가 필요하다.


# Preflight
브라우저에서 진짜 요청을 보내기 전 Preflight 요청을 Method.OPTIONS 로 보내서, CORS 관련하여 서버에 요청을 보내도 되는지 사전 확인을 한다.

Simple Request라고 정의되는 몇몇 경우에 대해서는 Preflight를 보내지 않는다.

#### Simple Request
- GET, HEAD, POST 중 하나의 메서드 요청
- 요청 헤더에 user agent에 의해 자동적으로 정해지는 헤더(Connection, User-Agent, Fetch 스펙에 정의된 "forbidden header name")를 제외, 그리고  Fetch 스펙에 정의된 "CORS-safelisted request-header" 포함하는 경우

​
[출처] [Web] CORS 해결하기 - 1. Simple requests|작성자 자바쟁이

웹 프레임워크 사용 시, 미들웨어에서 이를 처리 해줄 수 있다.


Access-Control-Allow-Origin: 특정 Origin (Https의 경우 와일드 카드 제거)

Access-Control-Allow-Methods: 메소드 특정

Access-Control-Allow-Headers: Preflight가 아닌 실제 요청에 포함될 헤더

Access-Control-Allow-Credentials: 클라이언트에서 설정한 Credentials가 요청에 포함되었을 때, 이에 대한 응답을 클라이언트 코드에서 접근 가능하게 할 지의 여부
 -> Credentials를 허용할 경우 위 세 가지 헤더에 와일드 카드가 있어선 안된다.


브라우저가 Origin 헤더를 설정하는 경우 

 - 다른 Origin에 대해 요청을 보낼 때
 - 같은 Origin이라도 GET, HEAD를 제외한 나머지 메소드를 사용한 요청을 보낼때
 - 브라우저가 Origin 헤더를 설정하지 않은 경우
   - 다른 Origin이지만, GET, HEAD 메소드를 통한 no-cors 모드의 요청일때
- 








# Proxy
- API로 네트워크 요청/응답을 주고 받는 Client와 Server 사이를 중개하는 중간자 역할
- Proxy 설정 시 Origin을 바꿔 Server에 보낼 수 있음.

vite Proxy 설정

```js
export default defineConfig({
  plugins: [react()],
  server: {
    proxy: {
      '/api': {
        target: 'https://apis.data.go.kr',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
        secure: false,
        ws: true
      }
    }
  }
})
```
 - target: 도메인, 포트, 프로토콜을 서버가 허용한 주소와 매핑
 - changeOrigin: 출처(Origin)를 target URI로 바꿈
 -> /api 앞 부분인 origin을 target에 설정되어 있는 부분으로 바꿔줌.


 ### Cookie 
 Cookie는 Domain 단위, Port는 공유

 