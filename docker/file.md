#  前端打包

```dockerfile
FROM nginx:1.14-alpine

MAINTAINER feilong.li@wisdragon.com

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo -e "server {\n listen 80;\n server_name localhost;\n \
    access_log /app/log/host.access.log main;\n location / {\n root /app;\n \
    index index.html;\n }\n }" > /etc/nginx/conf.d/default.conf

COPY resources/public /app

VOLUME /app/log
```



# 后端打包

```dockerfile
FROM openjdk:8-jre-alpine

MAINTAINER feilong.li@wisdragon.com

RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

COPY target/personnel.jar /app/personnel.jar

WORKDIR /app

CMD java -jar personnel.jar

EXPOSE 3000

VOLUME /app/log
```

```yaml
version: '3'

volumes:
  mysql_data:

services:
  hrms-db:
    image: pfs.wiseloong.com/wise/mysql:5
    container_name: hrms-db
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql

  hrms-s:
    image: pfs.wiseloong.com/hrms-s:drpeng
    container_name: hrms-s
    depends_on:
      - hrms-db
    ports:
      - "3000:3000"
    environment:
      POOL_SPEC__JDBC_URL: jdbc:mysql://192.168.16.17:3306/hrms?useSSL=false&serverTimezone=PRC
    volumes:
      - /data/wise/hrms/logs/service:/app/log

  hrms-w:
    image: pfs.wiseloong.com/hrms-w:drpeng
    container_name: hrms-w
    ports:
      - "8888:80"
    volumes:
      - /data/wise/hrms/logs/web:/app/log

```

```yaml
version: '3'

networks:
  hrms:

volumes:
  mysql_data:

services:
  hrms-db:
    image: pfs.wiseloong.com/wise/mysql:5
    container_name: hrms-db
    ports:
      - "3306:3306"
    networks:
      - hrms
    volumes:
      - mysql_data:/var/lib/mysql

  hrms-s:
    image: pfs.wiseloong.com/hrms-s:drpeng
    container_name: hrms-s
    depends_on:
      - hrms-db
    ports:
      - "3000:3000"
    networks:
      - hrms
    volumes:
      - /data/wise/hrms/logs/service:/app/log

  hrms-w:
    image: pfs.wiseloong.com/hrms-w:drpeng
    container_name: hrms-w
    ports:
      - "8888:80"
    volumes:
      - /data/wise/hrms/logs/web:/app/log

```

