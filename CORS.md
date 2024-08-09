# SCG CORS + https 이슈


## 요청 헤더의 content-type, Access-Control-Allow-Headers의 preflight 응답 오류

### content-type, Access-Control-Allow-Headers 
```
Access to XMLHttpRequest at 
'http:///api/email- join:1 verifications' 
from origin "<http://localhost:3000>' has been blocked by CORS policy: 
Request header field content-type is not allowed by Access-Control-Allow-Headers 
in preflight response.
```

Front, Back에 문제가 없을 경우 Nginx에 문제가 있을 수 있다.
 -> http 환경에서 CORS 문제가 없으나, https 환경에서 CORS 문제가 있다면 Nginx 설정을 확인해보자.

 ```
 location /api {
     proxy_pass http://backend/api;
   proxy_hide_header Access-Control-Allow-Origin;
     add_header 'Access-Control-Allow-Origin' '*';
     proxy_http_version 1.1;
     proxy_set_header Upgrade $http_upgrade;
     proxy_set_header Connection "upgrade";
     proxy_set_header Origin "";
     proxy_set_header X-Real-IP $remote_addr;
     proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
     proxy_set_header Host $http_host;
}
 ```

 ``add_header 'Access-Control-Allow-Origin' '*';``의 경우 token, cookie를 제거하고 요청하도록 정책이 구성되어 있다.
 따라서, 토큰 및 쿠키를 이용한 인증 API를 구성하고 있다면, ``Access-Control-Allow-Origin``에서 와일드 카드를 제거해야 하고, ``withCredentials`` 옵션을 허용해야 한다.

 쿠키 및 토큰 사용 시 체크리스트
  - ``Access-Control-Allow-Origin``에서 와일드 카드 제거
  - ``withCredentials`` 허용

쿠키 및 토큰 사용 시 Nginx 설정 예시

```
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server localhost:8080;
    }

    server {
        listen 80;

        location /api {
            proxy_pass http://backend/api;
            add_header 'Access-Control-Allow-Credentials' 'true';
            add_header 'Access-Control-Allow-Origin' 'http://localhost:3000';
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Origin "";
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
        }
    }
}
```

 
 ### Preflight 요청 시 발생하는 문제

Preflight 요청은 ``Method OPTIONS``으로 요청한다.
또, CORS 문제 해결을 위해선 Preflight 요청에 응답이 반드시 Success 응답이어야 한다.
```
if ($request_method = 'OPTIONS') {
     add_header 'Access-Control-Allow-Origin' $allowed_origin always;
     add_header 'Access-Control-Allow-Methods' 'GET, POST, DELETE, PATCH, OPTIONS';
     add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
     add_header 'Access-Control-Allow-Credentials' 'true';
     return 204;
}
```

위와 같은 옵션을 추가하여, Preflight 요청을 허용할 수 있다.



### 정리


```

events {
    worker_connections 1024;
}

http {
    upstream backend {
        server localhost:8080;
    }

    server {
        listen 80;

        location /api {
            if ($request_method = 'OPTIONS') {
                add_header 'Access-Control-Allow-Origin' 'http://localhost:3000';
                add_header 'Access-Control-Allow-Methods' 'GET, POST, DELETE, PATCH, OPTIONS';
                add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization';
                add_header 'Access-Control-Allow-Credentials' 'true';
                return 204;
            }

            proxy_pass http://backend/api;
            add_header 'Access-Control-Allow-Credentials' 'true';
            add_header 'Access-Control-Allow-Origin' 'http://localhost:3000' always;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Origin "";
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
        }
    }
}
```

Nginx는 Access-Control-Allow-Origin 옵션은 하나밖에 설정이 되지 않는다.
여러 도메인을 허용하기 위해선 동적으로 설정해야 한다.
```
map $http_origin $allowed_origin {
    default "";
    "http://localhost:3000" $http_origin;
    "https://devridge-client.vercel.app" $http_origin;
}

add_header 'Access-Control-Allow-Origin' $allowed_origin always;
```

Reference: https://medium.com/@hee98.09.14/nginx-cors-https-%EC%84%A4%EC%A0%95-%EB%AC%B8%EC%A0%9C-%ED%95%B4%EA%B2%B0%ED%95%98%EA%B8%B0-5af71812f4a1