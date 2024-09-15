
# 1、概述


## 1\.1 什么是Subject Alternative Name（证书主体别名）


　　SAN(Subject Alternative Name) 是 SSL 标准 x509 中定义的一个扩展。**它允许一个证书支持多个不同的域名。通过使用SAN字段，可以在一个证书中指定多个DNS名称（域名）、IP地址或其他类型的标识符，这样证书就可以同时用于多个不同的服务或主机上。**这种灵活性意味着企业不需要为每个域名单独购买和安装证书，从而降低了成本和复杂性。


　　先来看一看 Google 是怎样使用 SAN 证书的，下面是 Youtube 网站的证书信息：


![](https://img2024.cnblogs.com/blog/624219/202409/624219-20240913185103995-1294299893.png)


 　　这里可以看到这张证书的 Common Name 字段是 \*.google.com，那么为什么这张证书却能够被 www.youtube.com 这个域名所使用呢。原因就是这是一张带有 SAN 扩展的证书，下面是这张证书的 SAN 扩展信息：



![](https://img2024.cnblogs.com/blog/624219/202409/624219-20240913185151312-569821346.png)![](https://img2024.cnblogs.com/blog/624219/202409/624219-20240913185404748-1863016726.png)


　　这里可以看到，这张证书的 Subject Alternative Name 段中列了一大串的域名，因此这张证书能够被多个域名所使用。对于 Google 这种域名数量较多的公司来说，使用这种类型的证书能够极大的简化网站证书的管理。


## 1\.2 SAN的由来






　　在早期的互联网中，每个SSL/TLS证书通常只包含一个CN字段，用于标识单一的域名或IP地址。随着虚拟主机技术的发展和企业对于简化管理的需求增加，需要一种机制能够允许单个证书有效地代表多个域名或服务。例如，一个企业可能拥有多个子域名，希望用单一的证书来保护它们。


　　为了解决这个问题，SAN扩展被引入到X.509证书标准中。最初在1999年的RFC 2459中提出，SAN提供了一种方法来指定额外的主题名称，从而使得一个证书能有效地代表多个实体。


## 1\.3 SAN的作用和重要性


1. 多域名保护：SAN使得一个证书可以保护多个域名和子域名，减少了管理的复杂性和成本。
2. 灵活性增强：企业和组织可以更灵活地管理和部署证书，根据需要快速调整和扩展保护范围。
3. 兼容性：随着技术的发展，现代浏览器和客户端软件都已经支持SAN。它们会优先检查SAN字段，如果找到匹配项，通常不会再回退到检查CN。


## 1\.4 如何使用SAN


　　在申请SSL/TLS证书时，可以指定一个或多个SAN值。这些值通常是你希望证书保护的域名或IP地址。证书颁发机构（CA）在颁发证书时会验证这些信息的正确性，并将它们包含在证书的SAN字段中。


# 2、如何在OpenSSL证书中添加SAN


在OpenSSL中创建证书时添加SAN，需要在配置文件中添加一个subjectAltName扩展。这通常涉及到以下几个步骤：


1. 准备配置文件：在配置文件中指定subjectAltName扩展，并列出要包含的域名和IP地址。
2. 生成密钥：使用OpenSSL生成私钥。
3. 生成证书签发请求（CSR）：使用私钥和配置文件生成CSR，该CSR将包含SAN信息。
4. 签发证书：使用CA（证书颁发机构）或自签名方式签发证书，证书中将包含SAN信息。


## 2\.1 准备带有SAN扩展的证书请求配置文件


* 创建带有SAN扩展字段的证书签名请求（CSR）的配置文件， alt\_names 的配置段为SAN扩展字段配置。确保在将其保存至文件（如csr.conf）。



[?](https://github.com)

| 1234567891011121314151617181920212223242526272829 | `[ req ]` `default_bits = 2048` `prompt = no` `default_md = sha256` `req_extensions = v3_ext` `distinguished_name = dn`  `[ dn ]` `C = CN` `ST = ShangDong``L = SZ` `O = Wise2c` `OU = Wise2c` `CN = zmc`  `[ req_ext ]` `subjectAltName = @alt_names`  `[ alt_names ]` `DNS.1 = *.zmcheng.com` `DNS.2 = *.zmcheng.net` `DNS.3 = *.zmc.com` `DNS.4 = *.zmc.net`  `[ v3_ext ]` `basicConstraints=CA:FALSE` `keyUsage=keyEncipherment,dataEncipherment` `extendedKeyUsage=serverAuth,clientAuth` `subjectAltName=@alt_names` |
| --- | --- |




> 注意 1：SAN扩展字段不仅可以配置域名，还可以配置邮箱、IP地址、URI。
> 
> 
> 
> [?](https://github.com)
> 
> | 1234567 | `// Subject Alternate Name values. (Note that these values may not be valid``// if invalid values were contained within a parsed certificate. For``// example, an element of DNSNames may not be a valid DNS domain name.)``DNSNames       []string``EmailAddresses []string``IPAddresses    []net.IP // Go 1.1``URIs           []*url.URL // Go 1.10` |
> | --- | --- |


## 2\.2 生成密钥


* 生成密钥位数为 2048 的 ca.key



[?](https://github.com)

| 1 | `openssl genrsa -out ca.key 2048` |
| --- | --- |



* 依据 ca.key 生成 ca.crt （使用 \-days 参数来设置证书有效时间）：



[?](https://github.com)

| 1 | `openssl req -x509 -new -nodes -key ca.key -subj "/CN=zmc" -days 10000 -out ca.crt` |
| --- | --- |



* 生成密钥位数为 2048 的 server.key



[?](https://github.com)

| 1 | `openssl genrsa -out server.key 2048` |
| --- | --- |



## 2\.3 生成证书签发请求


* 基于配置文件生成证书签名请求



[?](https://github.com):[楚门加速器p](https://tianchuang88.com)

| 1 | `openssl req -new -key server.key -out server.csr -config csr.conf` |
| --- | --- |



![](https://img2024.cnblogs.com/blog/624219/202409/624219-20240914084729383-551680855.png) 


## 2\.4 签发证书


* 使用 ca.key、ca.crt 和 server.csr 生成服务器证书



[?](https://github.com)

| 1 | `openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \-CAcreateserial -out server.crt -days 10000 \-extensions v3_ext -extfile csr.conf` |
| --- | --- |



* 查看证书，可以看到创建出带有SAN扩展字段证书



[?](https://github.com)

| 1 | `openssl x509  -noout -text -in ./server.crt` |
| --- | --- |



![](https://img2024.cnblogs.com/blog/624219/202409/624219-20240914085027846-1205922426.png)


# 3、客户端验证服务端证书步骤


客户端（一般是浏览器）在请求HTTPS网站时，验证服务端证书的过程确实是一个复杂且关键的安全步骤。以下是以浏览器为例，说下浏览器验证服务端证书步骤：


1. 发起HTTPS请求：
	* 用户通过浏览器输入HTTPS网站的URL，并按下回车键或点击链接，浏览器开始发起HTTPS请求。
2. 服务端响应并发送证书：
	* 服务器在收到HTTPS请求后，会将其SSL/TLS证书发送给浏览器。这个证书包含了服务器的公钥、证书颁发机构（CA）的信息、证书的有效期、以及可能包括的SAN（主题备用名称）和CN（通用名称）等字段。
3. 浏览器验证证书颁发机构（CA）：
	* 浏览器会检查证书是否由受信任的CA签发。浏览器内置了一个受信任的CA列表（也称为根证书列表），它会使用这些CA的公钥来验证证书链中每一级证书签名的有效性，直到找到信任的根证书。
4. 检查证书有效期：
	* 浏览器会验证证书的有效期，确保当前时间在证书的有效期内。
5. **验证证书域名**：
	* **浏览器会检查证书中的SAN字段（如果存在）和CN字段，确保它们与用户正在访问的域名相匹配。如果域名不匹配，浏览器将认为证书无效，并显示警告信息。**
6. 检查证书吊销状态（可选）：
	* 浏览器可能会通过OCSP（在线证书状态协议）或CRL（证书吊销列表）来检查证书是否已被吊销。这一步是可选的，但有助于及时发现并阻止使用已泄露私钥的证书。
7. 生成会话密钥（TLS握手过程的一部分）：
	* 如果证书验证通过，浏览器和服务器将进行TLS握手，以协商一个安全的会话密钥。这个密钥将用于后续通信的加密和解密。
8. 加密通信：
	* 一旦会话密钥协商完成，浏览器和服务器就可以开始加密通信了。浏览器会使用会话密钥加密发送给服务器的数据，而服务器则使用相同的会话密钥解密这些数据。


### 注意事项


* 如果浏览器在验证证书的过程中发现任何问题（如证书不受信任、已过期、域名不匹配等），它将向用户显示警告信息，并可能阻止用户继续访问该网站。
* SAN字段的优先级通常高于CN字段。如果证书中同时包含了SAN和CN字段，并且它们之间的域名不一致，浏览器将优先使用SAN字段中的域名进行验证。
* 浏览器内置的受信任CA列表可能会随着时间的推移而更新，以反映新的CA或撤销旧的CA。因此，用户应该保持其浏览器和操作系统的更新，以确保其能够识别最新的受信任CA。


以上步骤详细描述了浏览器在请求HTTPS网站时验证服务端证书的过程。这个过程确保了用户与服务器之间的通信是安全、可信的。


# 4、示例——Kubectl客户端访问KubeApiserver


## 4\.1 环境


KubeApiserver地址：10\.20\.32\.205:6443


KubeApiserver证书信息：



[?](https://github.com)

| 12345678910111213141516171819202122 | `[root@member-cluster2-master1 pki]# pwd``/etc/kubernetes/pki``[root@member-cluster2-master1 pki]# ls``apiserver.crt  apiserver-kubelet-client.crt  ca.crt  front-proxy-ca.crt  front-proxy-client.crt  sa.key``apiserver.key  apiserver-kubelet-client.key  ca.key  front-proxy-ca.key  front-proxy-client.key  sa.pub``[root@member-cluster2-master1 pki]# openssl x509  -noout -text -in ./apiserver.crt` `Certificate:``Data:``Version: 3 (0x2)``Serial Number: 495742113187184862 (0x6e13ad34cb940de)``Signature Algorithm: sha256WithRSAEncryption``Issuer: CN = kubernetes``Validity``Not Before: Sep 11 01:13:49 2024 GMT``Not After : Aug 18 01:16:09 2124 GMT``Subject: CN = kube-apiserver``.......` `X509v3 Subject Alternative Name:` `DNS:kubernetes, DNS:kubernetes.default, DNS:kubernetes.default.svc, DNS:kubernetes.default.svc.cluster.local, DNS:lb.zmc.local, DNS:localhost, DNS:member-cluster2-master1, DNS:member-cluster2-master1.cluster.local, IP Address:10.234.0.1, IP Address:10.20.32.205, IP Address:127.0.0.1``Signature Algorithm: sha256WithRSAEncryption``.......` |
| --- | --- |



## 4\.2 Kubectl正常访问KubeApiserver（证书SAN扩展字段值包含访问地址）


* 通过KubeApiserver ip地址访问


![](https://img2024.cnblogs.com/blog/624219/202409/624219-20240914141303493-859066482.png)


* 通过域名主机映射访问，域名在SAN里面


![](https://img2024.cnblogs.com/blog/624219/202409/624219-20240914141504809-26717656.png)


## 4\.3 Kubectl异常访问KubeApiserver（证书SAN扩展字段值不包含访问地址）


* 通过域名主机映射访问，域名不在SAN里面，客户端验证证书失败。


![](https://img2024.cnblogs.com/blog/624219/202409/624219-20240914142738505-1514652920.png)


# 5、其他——KubeApiserver证书SAN扩展字段是否要把整个集群IP都列上


第一次接触Kubernetes并安装集群时，网上翻了很多博客，在生成KubeApiserver证书环节，都写着要把整个待安装集群的ip地址都写到KubeApiserver证书请求文件的SAN扩展字段里面，也没讲什么原因。


至于在生成KubeApiserver证书环节是否要把整个集群的ip地址都写到KubeApiserver证书请求文件的SAN扩展字段里面，还是要看集群里面的组件如何访问KubeApiserver，如果：


* 调度器（kube\-scheduler）、控制器（kube\-controller\-manage）都是直接和当前节点的kube\-apiserver组件直接交互；
* kubelet、kubectl、kube\-proxy通过VIP代理到kube\-apiserver组件。


对于以上这种情况，SAN扩展字段可以只写VIP（负责均衡器IP等）、MasterIPs即可，WorkerIP没必要写到SAN扩展字段里面。


# 6、结论


　　Subject Alternative Name（主体别名）是一个证书扩展字段，用于指定证书所适用的主机名列表。当客户端连接到服务器时，会检查服务器返回的证书中的主体别名是否包含与请求的主机名匹配的条目。


　　SAN的引入极大地增强了数字证书的功能和应用范围，使得管理和保护多域名环境变得更加高效和简便。作为现代网络安全的一个关键组成部分，理解并正确使用SAN对于任何需要部署SSL/TLS保护的个人或组织都至关重要。随着网络环境的不断演变和新需求的不断出现，SAN将继续发挥其在保护网络通信安全中的重要作用。无论是IT专业人员还是普通用户，都应该了解SAN的基本概念和实践，以确保在数字世界中安全地通信和交互。





参考：[OpenSSL SAN 证书](https://github.com)
参考：[【HTTPS】 没有匹配的证书主体别名 (Subject Alternative Name)](https://github.com)
