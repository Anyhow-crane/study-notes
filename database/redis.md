

## Docker安装

```sh
docker pull redis:5-alpine
docker run --restart=always -d -e TZ=Asia/Shanghai --name redis -p 6379:6379 redis:5-alpine
```

