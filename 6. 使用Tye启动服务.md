[Tye](https://github.com/dotnet/tye)是微软开源的一款开发人员工具， 能够用于简化微服务以及分布式应用程序的开发、测试以及部署过程。<br />Tye 的首要目标是简化微服务的开发，具体方式包括仅用一行命令执行多项服务、在容器中使用依赖项目，以及使用简单的方法探索其他服务的地址。
<a name="ClNMo"></a>
## 安装tye
首先我们安装tye，使用dotnet cli命令。
```powershell
dotnet tool install -g Microsoft.Tye --version "0.11.0-alpha.22111.1"
```
安装完后即可使用tye命令<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/730088/1673340106868-86c2d37d-d360-4bfb-94c6-84d61366072c.png#averageHue=%230b2d5d&clientId=u01896d63-b316-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=326&id=u47552aff&margin=%5Bobject%20Object%5D&name=image.png&originHeight=326&originWidth=587&originalType=binary&ratio=1&rotation=0&showTitle=false&size=8875&status=done&style=none&taskId=u5ee20240-0366-45fe-9977-3d498a781cf&title=&width=587)
<a name="RNLQ2"></a>
## 配置tye
首先我们使用tye init命令初始化tye.yaml配置文件<br />结构大致入下：
```yaml
name: funshow
services:
- name: 
  project: 
- name: 
  project:
```
我们需要在配置文件中添加我们的服务，包括绑定端口，环境变量等。<br />这里完整的配置文件如下：
```yaml
name: FunShow
services:
- name: auth-server
  project: apps/auth-server/src/FunShow.AuthServer/FunShow.AuthServer.csproj
  bindings:
    - protocol: https
      port: 44322
  env:
    - Kestrel__Certificates__Default__Path=../../../../etc/dev-cert/localhost.pfx
    - Kestrel__Certificates__Default__Password=e8202f07-66e5-4619-be07-72ba76fde97f
- name: administration-service
  project: services/administration/src/FunShow.AdministrationService.HttpApi.Host/FunShow.AdministrationService.HttpApi.Host.csproj
  bindings:
    - protocol: https
      port: 44367
  env:
    - Kestrel__Certificates__Default__Path=../../../../etc/dev-cert/localhost.pfx
    - Kestrel__Certificates__Default__Password=e8202f07-66e5-4619-be07-72ba76fde97f
- name: identity-service
  project: services/identity/src/FunShow.IdentityService.HttpApi.Host/FunShow.IdentityService.HttpApi.Host.csproj
  bindings:
    - protocol: https
      port: 44388
  env:
    - Kestrel__Certificates__Default__Path=../../../../etc/dev-cert/localhost.pfx
    - Kestrel__Certificates__Default__Password=e8202f07-66e5-4619-be07-72ba76fde97f
- name: logging-service
  project: services/logging/src/FunShow.LoggingService.HttpApi.Host/FunShow.LoggingService.HttpApi.Host.csproj
  bindings:
    - protocol: https
      port: 45124
  env:
    - Kestrel__Certificates__Default__Path=../../../../etc/dev-cert/localhost.pfx
    - Kestrel__Certificates__Default__Password=e8202f07-66e5-4619-be07-72ba76fde97f
- name: web-gateway
  project: gateways/web/src/FunShow.WebGateway/FunShow.WebGateway.csproj
  bindings:
    - protocol: https
      port: 44325
  env:
    - Kestrel__Certificates__Default__Path=../../../../etc/dev-cert/localhost.pfx
    - Kestrel__Certificates__Default__Password=e8202f07-66e5-4619-be07-72ba76fde97f  
```
bindings表示我们绑定https以及端口号。<br />env里面配置了我们的本地开发HTTPS证书。
<a name="X0dRd"></a>
## 创建本地证书
上面配置里面我们有加载本地证书，那么怎么创建证书呢，在tye仓库中也有说明<br />[https://github.com/dotnet/tye/blob/main/docs/tutorials/hello-tye/00_run_locally.md#generate-the-certificate](https://github.com/dotnet/tye/blob/main/docs/tutorials/hello-tye/00_run_locally.md#generate-the-certificate)<br />仓库中是在linux环境，但是在windows环境中localhost.conf是一样的
```powershell
[req]
default_bits       = 2048
default_keyfile    = localhost.key
distinguished_name = req_distinguished_name
req_extensions     = req_ext
x509_extensions    = v3_ca

[req_distinguished_name]
commonName                  = Common Name (e.g. server FQDN or YOUR name)
commonName_default          = localhost
commonName_max              = 64

[req_ext]
subjectAltName = @alt_names

[v3_ca]
subjectAltName = @alt_names
basicConstraints = critical, CA:false
keyUsage = keyCertSign, cRLSign, digitalSignature,keyEncipherment

[alt_names]
DNS.1   = localhost
DNS.2   = 127.0.0.1
```
创建etc/dev-cert目录，在目录下添加localhost.conf文件，内容如上。<br />然后执行dotnet dev-certs命令
```powershell
dotnet dev-certs https -v -ep localhost.pfx -p e8202f07-66e5-4619-be07-72ba76fde97f -t
```
就会在目录下面生成localhost.pfx证书文件。
<a name="TbpEK"></a>
## 使用tye运行微服务
在目录下面执行tye运行命令
```powershell
tye run --watch
```
效果如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/730088/1673341127964-e62d512c-58c4-48ff-86c4-019c0c9e3091.png#averageHue=%23222120&clientId=u01896d63-b316-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=668&id=u1281e2b5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=668&originWidth=1348&originalType=binary&ratio=1&rotation=0&showTitle=false&size=138330&status=done&style=none&taskId=ufce39b5a-2928-4190-8b9a-120ec0bba7d&title=&width=1348)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/730088/1673341159458-ab737a74-3ee4-47b5-beeb-53a97a8811a6.png#averageHue=%23e0ba87&clientId=u01896d63-b316-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=370&id=uea57cd50&margin=%5Bobject%20Object%5D&name=image.png&originHeight=370&originWidth=1428&originalType=binary&ratio=1&rotation=0&showTitle=false&size=50733&status=done&style=none&taskId=u25505139-a4f3-4199-babb-ba7c96227b0&title=&width=1428)<br />当然运行服务前我们需要把我们的基础服务都启动，如数据库，消息队列，redis等。<br />在tye dashboard可以查看服务的日志以及Metrics信息<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/730088/1673341321622-2154303e-2d1d-420b-aa72-95517770cb92.png#averageHue=%2345403b&clientId=u01896d63-b316-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=614&id=u4f595e0c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=614&originWidth=1092&originalType=binary&ratio=1&rotation=0&showTitle=false&size=132644&status=done&style=none&taskId=ud1455a18-65bb-4c08-96f2-5cfab1eb615&title=&width=1092)<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/730088/1673341355189-d3de796e-e08b-43ee-a573-1c5766cd9cfc.png#averageHue=%23f8f8f8&clientId=u01896d63-b316-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=634&id=ud6ad1ea9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=634&originWidth=1171&originalType=binary&ratio=1&rotation=0&showTitle=false&size=39025&status=done&style=none&taskId=u3368e3a9-8cfe-4f8c-89d2-f7a68504b6e&title=&width=1171)<br />下面是服务启动页面。<br />网关服务<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/730088/1673341249278-8ebf6220-87b1-4cd1-a95d-d2228017dc80.png#averageHue=%23e1e0e0&clientId=u01896d63-b316-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=535&id=ucb13e0ac&margin=%5Bobject%20Object%5D&name=image.png&originHeight=535&originWidth=1399&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52629&status=done&style=none&taskId=u4ca882cb-ebc8-441f-a81c-12669c0537f&title=&width=1399)<br />认证服务<br />![image.png](https://cdn.nlark.com/yuque/0/2023/png/730088/1673341267725-dd5961a0-7f24-4ac9-aabe-84d8e5051416.png#averageHue=%23f4fbfc&clientId=u01896d63-b316-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=756&id=ue0ca63c9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=756&originWidth=1142&originalType=binary&ratio=1&rotation=0&showTitle=false&size=25707&status=done&style=none&taskId=u64dab875-9022-4def-be3b-8e6105a801f&title=&width=1142)<br />到这我们后端功能就基本完成啦
