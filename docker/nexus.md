# Nexus 学习

## docker部署

docker仓库地址：<https://hub.docker.com/r/sonatype/nexus3/>

``` sh
# 1: 下载镜像
docker pull sonatype/nexus3
# 2: 创建工作文件夹
mkdir /data/nexus-data && chown -R 200 /data/nexus-data
# 3: 运行个容器
docker run -d -p 8081:8081 --name nexus -v /data/nexus-data:/nexus-data sonatype/nexus3
```

> 初始账号密码：admin:admin123

## Nginx反向代理开启https

```
server {
    listen 80;
    server_name dc.wisdragon.com;
    rewrite ^(.*)$ https://${server_name}$1 permanent; 
}
server {
    listen 443 ssl;
    server_name dc.wisdragon.com;
    ssl_certificate /nginx/cert/wisdragon.com.bundle.crt;
    ssl_certificate_key /nginx/cert/cert.key.pem;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_pass http://192.168.19.25:8081;
    }
}
```

> 如果不添加这个`proxy_set_header X-Forwarded-Proto "https";`会一直卡在 Initialize...

## 开启ssl后idea信任证书

开启ssl后，idea无法使用仓库，下载jar包是报错，需要信任证书（不知是不是自签证书造成的，有机会验证下），不是在系统安装的jdk里信任证书，而是在idea自带的jre里信任证书，（如果使用自己安装的maven，则在系统jdk里信任），只需要信任nginx的证书就可以了，也可以直接信任根证书。

``` sh
/Applications/IntelliJ\ IDEA.app/Contents/jdk/Contents/Home/jre/bin/keytool -import -alias wisdragon -keystore /Applications/IntelliJ\ IDEA.app/Contents/jdk/Contents/Home/jre/lib/security/cacerts -file /Users/loong/app/cert/wisdragon.com.bundle.crt -storepass changeit
```

> 上面为mac命令，使用idea.app里jre的keytool命令添加证书，不是直接keytool命令添加，windows同理使用应用里的jre。
>
> -alias 为给个别名，-keystore 密钥库路径，-file 为证书位置，-storepass 密钥库密码，changeit为默认密码。

## 开启ssl后Java信任证书

``` sh
keytool -import -alias wisdragon -keystore /Library/Java/JavaVirtualMachines/jdk1.8.0_181.jdk/Contents/Home/jre/lib/security/cacerts -file /Users/loong/app/cert/root.crt -storepass changeit
```

## 使用中遇到的问题

### idea 仓库更新index出错

![事实上](../images/nexus-001.png)

解决方法：需要创建定时更新index文件任务（admin登陆nexus）

![Pasted Graphic 1.tiff](../images/nexus-002.png)

![Pasted Graphic 2.tiff](../images/nexus-003.png)

![Pasted Graphic 3.tiff](../images/nexus-004.png)