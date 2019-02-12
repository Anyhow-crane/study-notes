# 开启ssl

## 证书

1. 阿里云有[免费版](https://common-buy.aliyun.com/?spm=5176.7968328.1266638..feee1232kFUd5K&commodityCode=cas#/buy)，选择Symantec，再点增强型OV，就会出来免费型DV选项了，一年期，单域名，因该不能适应通配符域名。
2. [Certbot](https://certbot.eff.org)，使用[Let's Encrypt](https://letsencrypt.org/)，没试成功，尴尬。
3. [FreeSSL.org](https://freessl.cn)，挺好用的证书申请网站。
4. [acme.sh](https://github.com/Neilpang/acme.sh)，certbot太复杂，可以尝试这个，貌似还可以使用docker运行。

## 自签证书

使用[GitHub](https://github.com/Fishdrowned/ssl)别人写好的脚本，下载后根据说明自定义你的根证书名称和组织。

> C=CN 国家，ST=Anhui 省，L=Hefei 所在地，O=Wisdragon 组织，OU=www.wisdragon.com 组织单位/部门， CN=*.wisdragon.com 证书名称，emailAddress=feilong.li@wisdragon.com 邮箱。

- gen.cert.sh 可以设置为`/C=CN/ST=Anhui/L=Hefei/O=Wisdragon/OU=$1/CN=*.$1`，$1为执行命令后的*.wisdragon.com泛域名。
- gen.root.sh 可以设置为`/C=CN/ST=Anhui/L=Hefei/O=Wisdragon/OU=www.wisdragon.com/CN=Wisdragon ROOT CA`。

``` sh
./gen.cert.sh *.wisdragon.com
```

[想自己创建的可以参考]: https://wangye.org/blog/archives/732/

## 证书使用

推荐使用nginx反向代理，