---
layout: post
title: 'Setup project với nestjs, redis, postgresql và docker.'
date: 2020-11-01
categories: Nodejs Docker Nestjs Redis Postgresql
---

Hề lô hề lô, lại là mình đây ^^!. Hôm thứ 6 lỡ shutdown máy ở trên công ty nên về nhà không [ssh] vào để code tiếp được :D. Tình hình là cái [one man army project] mình nhắc tới ở bài trước đã running được khoảng tầm hơn một tháng rồi. Mà do chỉ có một mình mình làm mà thời gian lại có hơi gấp nên những lúc sau giờ làm mình thường hay mở source code ra để gõ tiếp vài dòng :D. Thứ 7 chủ nhật này xui xui sao tắt máy ở công ty nên xem như nghỉ ngơi vậy :D Tiện thể viết luôn bài blog trong lúc rảnh.

À thường thì sau giờ làm mình sẽ đi đánh cầu lông với mấy anh chị trong công ty, chủ nhật thì hay đi đá bóng với tụi bạn hồi cấp ba chứ cuộc sống không chỉ xoay quanh việc gõ code suốt ngày đâu :D

Okay, quay lại chủ đề chính của bài blog này. Cái dự án hiện tại mình đang làm được áp dụng khá là nhiều công nghệ cũ-người-mới-ta đối với mình. Mình cũng tốn kha khá thời gian để initiate nó, trong bài này mình sẽ nói những cái ưu điểm của việc gom tất tần tật các services vào trong một container của docker và làm sao để làm được việc đó :D Còn nhược điểm là gì thì mình chưa gặp cũng như chưa tìm hiểu nên sẽ không nhắc tới trong này, hi vọng mọi người có thể comment cho mình biết nhược của nó với ^^!

À, dạo này lười quá nên mọi người chủ động tìm hiểu khái niệm docker, container các kiểu là gì nha :D Mình sẽ để link tại đây về [Docker] ^^!

### I. Why Docker?

Mình xin trích dẫn một đoạn trong documents của docker:

```
Developing apps today requires so much more than writing code. Multiple languages, frameworks, architectures, and discontinuous interfaces between tools for each lifecycle stage creates enormous complexity. Docker simplifies and accelerates your workflow, while giving developers the freedom to innovate with their choice of tools, application stacks, and deployment environments for each project.

```

Việc developing bây giờ khác ngày xưa nhiều lắm, từ việc website ban đầu chỉ là một cục, đem lên server chạy (người ta hay gọi là [Monolithic Architecture] ý), khi đấy khi giao tiếp với một trang web thì chúng ta sẽ gửi một request tới server, sau đó server trả về toàn bộ nội dung trang web cho mình. Việc này khiến cho thời gian chờ để load cả một trang web lên rất lâu, rồi server cũng phải đảm nhiệm cả việc render html lên nữa,... Sau này khi mà người ta thấy quá nhiều nhược điểm cũng như là khả năng scale up của monolithic cũng gây khó cho developers. Từ đó, [Microservice Architecture] được ra đời để giải quyết các vấn đề trên, mỗi service sẽ làm đúng và duy nhất nhiệm vụ của mình, đảm bảo khả năng scale up dễ dàng hơn.

Ơ lại lan man rồi =)) Quay lại vấn đề chính nào.

Trong quá trình developing, chúng ta thường sẽ hay gặp vấn đề "It works on my machine" đúng hơm :D Hai người cùng cài một service nhưng lỡ có khác phiên bản là app crash ngay. Docker & container sinh ra để khắc phục vấn đề này. Docker cung cấp các container giúp cho developers chỉ cần cài đặt một lần duy nhất và sẽ chạy được trên tất cả các môi trường khác nhau, miễn là có thể cài được Docker.

Nghe giống giống slogan của Java đúng không =))) "Write once run everywhere". Docker cũng vậy, điều tuyệt vời của Docker là giúp cho thành viên mới khi tham gia vào dự án đỡ tốn thời gian setup hơn, việc deploying cũng dễ dàng hơn nữa chú. Hồi xưa mình làm dự án về Ruby, mỗi lần mà bị cài lại máy là xem như mất luôn một ngày chỉ để ngồi cài đặt từng cái modules để có thể code được =))).

### II. Initiate a project with Docker built on Nestjs, Redis, Postgresql.

#### 1. Prepair a Dockerfile.

Okay, để implement Docker vào project, chúng ta cần khai báo một file tên là `Dockerfile`. Docker sẽ dựa vào các command trong này để build thành một image. Image này sẽ dùng để chạy bên trong contaier của chúng ta.

Nội dung của `Dockerfile` cũng không có gì đặc biệt, đây là file `Dockerfile` của mình:

```javascript
# Init database for the first time.
FROM postgres:latest
COPY init.sql /docker-entrypoint-initdb.d/

# Check out https://hub.docker.com/_/node to select a new base image
FROM node:14.11.0

# Set to a non-root built-in user `node`
USER node

# Create app directory (with user `node`)
RUN mkdir -p /home/node/app
WORKDIR /home/node/app

# Install app dependencies & setup.
COPY --chown=node package.json /home/node/app
RUN yarn install

# Bundle app source code
COPY --chown=node . .

```

Mọi người có thế dựa vào [Nodejs Docker App] hoặc [Dockerfile reference] để tự xây dựng `Dockerfile` của mình, à ngoài ra cũng nên follow theo [Nodejs Docker best pratice] nữa nhé ;)

OKay, chúng ta đã có một container gồm Nodejs và Postgreql để dùng rồi, việc tiếp theo là define `docker-compose` để có thể chạy multi-container.

#### 2. Writing a docker-compose.yml.

Với compose, chúng ta có thể define và chạy nhiều container và kết nối chúng lại với nhau. Vì mỗi container sẽ sử dụng một image riêng biệt, và vấn đề mà ta cần giải quyết là làm sao cho các container đó có thể giao tiếp được với nhau. Lúc này thì docker-compose sẽ giúp mình làm việc đó, nó sẽ tự động tạo ra một network, nơi mà tất cả các container được define trong này có thể nói chuyện với nhau y như thể tụi nó được cài đặt trên cùng một cái container vậy á :)).

Khó hiểu đúng không? Chúng ta có thể tưởng tượng cái máy tính của mình là một container đi he. Công ty cấp cho mình một cái PC có cài đặt Nodejs, thế nhưng vì ổ cứng hết dung lượng nên không thể cài đặt thêm Postgresql và Redis được nên phải đi mượn ông A cái laptop đề cài lên. Vấn đề xảy ra lúc này là app chạy trên platform Nodejs ở máy công ty không thể nào kết nối tới Postgresql trên laptop của ông A để thao tác xoá sửa dữ liệu được, thì docker-compose sẽ là cái thằng được configure sẵn cho phép cái App giao tiếp được với database. Việc mình cần làm là define nhưng service nào được phép nói chuyện với nhau cho thằng docker-compose biết. Ok hỉu hơm hỉu hơm ^^

Được rồi he, đây là file `docker-compose.yml` mà mình đã dùng để define những cái service trên.

```javascript
version: '3'

volumes:
  postgres_data_local: {}
  postgres_backup_local: {}

services:
  db:
    image: postgres:latest
    container_name: db_local
    restart: always
    ports:
      - '5432:5432'
    env_file:
      - ./.env
    volumes:
      - postgres_data_local:/var/lib/postgresql/data
      - postgres_backup_local:/backups

  api:
    build: .
    image: api_local
    container_name: api_local
    restart: always
    depends_on:
      - db
      - redis
    links:
      - db
      - redis
    volumes:
      - ./:/home/node/app
    env_file:
      - ./.env
    ports:
      - '3000:3000'
    command: './start.sh'

  redis:
    restart: always
    image: redis
    container_name: redis_local
    ports:
      - '6379:6379'
    env_file:
      - ./.env
    command: 'redis-server --requirepass ${REDIS_PASSWORD}'

```

Giải thích một chút, `Compose` file này sẽ chứa ba service: Api, Db, và redis.
Ngoài ra mình có define một cái lable là `volumes`, lable này dùng trong việc backup lại dữ liệu, để mỗi lần mình stop container sau đó start lại thì data của mình vẫn còn nguyên ở đấy :D
Ngoài ra, api của mình còn phải _depends_on_ tụi `db` và `redis` nữa. Mục đích của việc này là force thằng `api` phải phụ thuộc vào `redis` và `db`, tức là khi `docker-compose up` thì thằng `db` và `redis` sẽ phải khởi động trước thằng `api`. Các bạn có thể đọc thêm về [depends_on].

Các command để dùng trong `docker-compose` các bạn có thể tìm đọc thêm ở phần [docker-compose docs] này.

### IV. Kết thúc.

Phần cài đặt Docker & sử dụng Docker-compose mình xin kết thúc tại đây, trên đây chỉ là những cái khái niệm cơ bản khi sử dụng docker mà mình tìm hiểu được, kiến thức còn hạn chế, mong mọi người có thể góp ý để mình bổ sung giúp cho bài viết tốt hơn nhé ^^.

Anw, nếu có thắc mắc gì thì cứ inbox [Facebook] mình hoặc comment ở dưới nha ^^ Mình sẽ giải đáp cho các bạn :D

Love you all :\*

[ssh]: https://en.wikipedia.org/wiki/Ssh_(Secure_Shell)
[one man army project]: https://nguyenvu.dev/nodejs/firebase/typescript/2020/09/26/firebase-implementation.html
[docker]: https://docs.docker.com/get-started/overview/
[microservice architecture]: https://microservices.io/patterns/microservices.html
[monolithic architecture]: https://microservices.io/patterns/monolithic.html
[nodejs docker app]: https://nodejs.org/en/docs/guides/nodejs-docker-webapp/
[nodejs docker best pratice]: https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md
[dockerfile reference]: https://docs.docker.com/engine/reference/builder/
[depends_on]: https://docs.docker.com/compose/compose-file/#depends_on
[docker-compose docs]: https://docs.docker.com/compose/compose-file/
