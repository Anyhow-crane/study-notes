# Harbor-私有仓库

## 安装

## 更新配置

### centos系统添加crt/cer证书到信任列表，[参考](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html)

```sh
# 1: 复制根证书到相应的目录
sudo cp ~/root.crt /etc/pki/ca-trust/source/anchors/
# 2: 更新安装证书
sudo update-ca-trust
# 3: 重启docker
sudo systemctl restart docker
# 4: 登陆docker仓库（不行可以尝试重新登陆ssh，再不行重启系统）
docker login docker.wisdragon.com
```
