# Lab3 - voting app

### 1. `docker-compose.yml` 수정

> swarm/swarm-app-1

- manager 컨테이너
  - 포트포워딩 설정 추가
    - `5001:81`



### 2. `voting_app.yml` 추가

> swarm/swarm-app-1/stack

- volume x 1 (`db-data:/var/lib/postgresql/data`)
- networks x 2 (frontend, backend - overlay)
- services x 5 (voting-app, redis, db, worker, result-app)
- stack x 1 (my-vote-app)



#### 2.1 포트포워딩

> 컨테이너의 포트는 이미지에 의해 이미 결정되어있었음 `80`

- voting-app
  - `80:80`
- result-app
  - `81:80`



#### 2.2 네트워크 설정

``` yaml
networks:
  frontend:
    external: true

  backend:
    external: true
```

##### _** 중요! **_

![network_1](https://user-images.githubusercontent.com/20036670/72504392-160b5700-3881-11ea-9f5c-aa4482514da5.PNG)



![network_2](https://user-images.githubusercontent.com/20036670/72504439-26233680-3881-11ea-86c6-8b59796d232a.PNG)

- worker 서비스는 frontend 네트워크를 사용하지만 backend 네트워크에 있는 db 서비스에 데이터를 전달해야 하기 때문에 worker 서비스는 frontend, backend 네트워크를 둘 다 사용해야 한다.

  ```yaml
  networks:
     - frontend
     - backend
  ```

  

#### 2.3 볼륨 설정

- `db` 서비스에 볼륨 설정 추가

``` yaml
volumes: 
    - db-data:/var/lib/postgresql/data
```

- 서비스 바깥 쪽에 볼륨 설정 추가

``` yaml
volumes:
  	db-data:
```





### 3. `docker-compose.yml` 실행

``` shell
$ docker-compose up
```





### 4. 스웜 설정

- 매니저 접속

``` shell
$ docker exec -it manager sh
```

``` shell
매니저
$ docker swarm init
## join 토큰으로 work01~03 join 해주기
```





### 5. 네트워크 생성

``` shell
매니저
$ docker network create --driver=overlay --attachable frontend
$ docker network create --driver=overlay --attachable backend
```



### 6. `voting_app.yml` 배포하기

``` shell
매니저
$ docker stack deploy -c /stack/voting_app.yml my_vote_app
```



### 7. 서비스 확인

``` shell
$ docker service ls
```



### 8. 웹브라우저 확인

- `localhost:8000` 

  - voting-app

  ![vote-app](https://user-images.githubusercontent.com/20036670/72504936-34258700-3882-11ea-852f-36b0740fb8a5.png)

- `localhost:5001`

  - result-app

  ![vote-result](https://user-images.githubusercontent.com/20036670/72504988-4bfd0b00-3882-11ea-8ef1-56bead3dbc60.png)