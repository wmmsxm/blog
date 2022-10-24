---
title: "K8s离线部署-内网使用https"
date: 2022-10-24T17:01:14+08:00
draft: false
tags: ["k8s"]
categories: ["k8s"]
---
## <center>K8s离线部署-内网使用https</center>
#### 证书生成
##### 生成证书请求文件
单独在一个文件夹iphttps新建openssl.cnf，内容如下：
```
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req

[req_distinguished_name]
countryName = Country Name (2 letter code)
countryName_default = CH
stateOrProvinceName = State or Province Name (full name)
stateOrProvinceName_default = GD
localityName = Locality Name (eg, city)
localityName_default = ShenZhen
organizationalUnitName  = Organizational Unit Name (eg, section)
organizationalUnitName_default  = organizationalUnitName
commonName = Internet Widgits Ltd
commonName_max  = 64

[ v3_req ]
# Extensions to add to a certificate request
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
subjectAltName = @alt_names

[alt_names]

# 改成自己的域名
#DNS.1 = kb.example.com
#DNS.2 = helpdesk.example.org
#DNS.3 = systems.example.net

# 改成自己的ip
IP.1 = 172.16.1.71
IP.2 = 172.16.1.72
IP.3 = 172.16.1.73
```

##### 生成私钥
san_domain_com 为最终生成得文件名，一般以服务器命名，可自行更改。
```
openssl genrsa -out san_domain_com.key 2048
```

##### 创建CSR文件
创建csr文件命令：
```
openssl req -new -out san_domain_com.csr -key san_domain_com.key -config openssl.cnf
```

##### 自签名并创建证书、
```
openssl x509 -req -days 3650 -in san_domain_com.csr -signkey san_domain_com.key -out san_domain_com.crt -extensions v3_req -extfile openssl.cnf
```

至此，使用openssl生成得证书已完成，可看到目录下又三个文件
```
san_domain_com.crt

san_domain_com.csr

san_domain_com.key
```

#### 创建secret资源
直接使用kubectl命令创建
```
cd /root/iphttps
kubectl create secret tls iphttps-secret --key san_domain_com.key --cert san_domain_com.crt -n fhzh

```

#### 创建ingress资源
创建ing资源，增加一个tls字段：
```
kind: Ingress
apiVersion: extensions/v1beta1
metadata:
  name: iphttps
  namespace: fhzh
spec:
  tls:
    - hosts:
        # 域名
        - www.iphttpsfhzh.com
      secretName: iphttps-secret
  rules:
      # 域名
    - host: www.iphttpsfhzh.com
      http:
        paths:
          - backend:
              # service的名称
              serviceName: fhzh-web-svc
              # service的内部端口
              servicePort: 80
    # 默认访问，解决内网ip无法访问的问题(弊端：不支持多个服务，后面考虑使用路径解析)
    - http:
        paths:
          - backend:
              serviceName: fhzh-web-svc
              servicePort: 80
```

#### https访问
内网使用域名访问的话，需要在客户端修改hosts，这里并不推荐。  
先查询ingress的暴露的端口
```
# 查看
kubectl get svc -n ingress-nginx
```
![配置](/blog/images/k8s/ingress.png)   
可以看出 80映射为 30865；443映射 31287   
至此访问路径 https://172.16.1.71:31287即可看到系统