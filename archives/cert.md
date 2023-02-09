---
title: 生成CA自签名根证书和颁发证书和证书提取
date: 2023-02-09T14:11:41+08:00
Description:
Tags: 
Categories:
DisableComments: false
---

### 生成CA证书私钥

```bash
# 生成aes128位编码的 密码为Test@2022 2048位的 key 文件  (带密码 、加密格式 aes、des 3des等)
openssl genrsa -aes128 -passout pass:Test@2022 -out ca_private.key 2048
# 生成 2048位的 key 文件  (不带密码，加密格式 等)
openssl genrsa -out ca_private.key 2048 
# 也可生成  ca_private.pem 文件，将后缀.key 变更为.pem
openssl genrsa -aes128 -passout pass:Test@2022 -out ca_private.pem 2048
openssl genrsa -out ca_private.pem 2048
```

### 生成CA证书请求文件
```bash
# 有效期20年
# 也可以 将后缀.key 变更为.pem 带密码（注意目录，文件放在哪个目录下，一会用的到，别找不到路径）
openssl req -new -key ca_private.key -passin pass:Test@2022 -out ca_req.csr -days 7300
# 不带密码
openssl req -new -key ca_private.key  -out ca_req.csr -days 7300 
# 填写 国家、机构、密码等，按实际情况填写即可
```

### 生成CA根证书
```bash
# 注意目录，文件放在哪个目录下，一会用的到，别找不到路径
openssl x509 -req -in ca_req.csr -signkey ca_private.key -out ca_root.crt -days 7300 -passin pass:Test@2022
# 不带密码
openssl x509 -req -in ca_req.csr -signkey ca_private.key -out ca_root.crt -days 7300
```
> 注：接下来服务器证书要根据 以上证书 来生成

### 服务器证书生成
与根节点服务器证书类似，只是生成 服务器证书的第三部要依赖 生成的 ca 根证书

```bash
# 1. 生成服务器私钥 
openssl genrsa -aes128 -passout pass:Test@2022 -out server_private.key 2048
## 带秘钥 带加密方式 等同 ca 第一步
## 可以 去掉密码 去掉加密 方式
openssl genrsa -out server_private.key 2048

# 2. 生成服务端的待签名证书
## 有效期10年
openssl req -new -key server_private.key -passin pass:Test@2022 -out server_req.csr -days 3650
## 无密码可以去掉密码部分
openssl req -new -key server_private.key -out server_req.csr -days 3650

# 3. 使用CA根证书对服务端证书签名
## key版
openssl x509 -req -in server_req.csr -days 3650  -CAkey ca_private.key -CA ca_root.crt -CAcreateserial  -out server.crt
## pem 版本
openssl x509 -req -in server_req.csr -days 3650  -CAkey ca_private.pem -CA ca_root.pem -CAcreateserial  -out server.crt
```


### chrome错误
错误码：`NET::ERR_CERT_COMMON_NAME_INVALID`

错误信息：此服务器无法证实它就是 mousemin.com - 它的安全证书没有指定主题备用名称。这可能是因为某项配置有误或某个攻击者拦截了您的连接。


#### 解决方案
新建一个文件 `ext.ini`，写入以下内容：

```ini
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names
 
[alt_names]
DNS.1 = *.mousemin.com
DNS.2 = mousemin.com
```

`alt_names` 下面填写上所有的域名即可，然后签发证书的时候带上参数：

```bash
openssl x509 ... -extfile ext.ini
```

### 服务器证书生成脚本
```bash
#!/usr/bin/env bash
set -e

File=$1
if [[ "${File}" == "" ]]
then
  echo "gen.sh filename [domain...]"
  exit
fi

if [[ "$#" < 1 ]]
then
  echo "gen.sh filename [domain...]"
  exit
fi

domains=($*)

# 生成私钥
openssl genrsa -out ${File}.key 2048

# 生成服务端的待签名证书
openssl req -new -key ${File}.key -out ${File}.csr -days 3650

# 生成ext.ini
# 生成ext.ini
echo "basicConstraints = CA:FALSE" > ${File}_ext.ini
echo "keyUsage = nonRepudiation, digitalSignature, keyEncipherment" >> ${File}_ext.ini
echo "subjectAltName = @alt_names" >> ${File}_ext.ini
echo "[alt_names]" >> ${File}_ext.ini

for(( i=1; i<${#domains[@]}; i++)) do
  echo "DNS.${i} = ${domains[i]}" >> ${File}_ext.ini
done

for(( i=0;i<${#domains[@]};i++)) do
  echo "DNS.${i} = ${domains[i]}" >> ${File}_ext.ini
done

# 使用CA根证书对服务端证书签名
openssl x509 -req -in ${File}.csr -days 3650 -CAkey ca_private.key -CA ca_root.crt -CAcreateserial -out ${File}.crt -extfile ${File}_ext.ini

rm -f ${File}.csr ${File}_ext.ini

```

## 一键生成

```bash
openssl req -newkey rsa:4096 -x509 -sha256 -days 3650 -nodes -out mousemin.com.crt -keyout mousemin.com.key
```