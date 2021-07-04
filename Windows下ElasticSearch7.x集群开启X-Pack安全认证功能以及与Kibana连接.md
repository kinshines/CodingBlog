# Windows下ElasticSearch7.x集群开启X-Pack安全认证功能以及与Kibana连接

### 1.生成CA证书和私钥

cmd窗口执行命令

> bin/elasticsearch-certutil.bat ca

将产生新文件 elastic-stack-ca.p12文件，该文件位于根目录下，该 elasticsearch-certutil 命令还会提示你输入密码以保护文件和密钥，一般无需设置该密码。

> bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12

将产生新文件 elastic-certificates.p12文件。系统同样会提示你输入密码，可以输入证书和密钥的密码，一般无需设置该密码，按Enter键将密码留空。

为便于管理，可将以上两个文件拷贝至config目录下。

### 2.集群节点配置

将config目录中的 *elastic-stack-ca.p12*、 *elastic-certificates.p12*、*elasticsearch.keystore* 这三个文件copy到其它的集群节点对应的config目录中。

### 3.配置集群中的每个节点以使用其签名证书标识自身并在传输层上启用TLS

在每个ES集群的名节点的elasticsearch.yml配置文件中添加如下配置：

```yaml
#启用X-Pack(TLS)并指定访问节点证书所需的信息
xpack.security.enabled: true
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: config/elastic-certificates.p12
xpack.security.transport.ssl.truststore.path: config/elastic-certificates.p12
```

### 4.重启各节点的ES服务

重启ES集群服务后在浏览器中请求任何一个节点都会需要进行登录操作，***这时需要进行密码设置***。

### 5.为系统默认用户创建密码

在 ***ES集群服务启动的情况下*** 为五个默认用户（elastic,apm_system,kibana,logstash_system,beats_system）设置密码。

> bin/elasticsearch-setup-passwords.bat interactive

***设置完成密码后，ES集群的各个节点会进行同步，不需要在单独进行密码设置。***
之后，再访问各个节点输入用户名密码后即可查看节点信息。

### 6.Kibana连接ES集群

***在 Kibana的配置文件 kibana.yml 添加 kibana连接 ES集群需要的用户名和密码以及节点地址。***

```yaml
elasticsearch.username: "kibana_system"
elasticsearch.password: "your_password"
```

### 7.java api带密码访问

```java
//初始化ES操作客户端
        final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
        credentialsProvider.setCredentials(AuthScope.ANY,
                new UsernamePasswordCredentials("elastic", "123456"));  //es账号密码（默认用户名为elastic）
        RestHighLevelClient esClient =new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("127.0.0.1",9200)
                ).setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
                    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                        httpClientBuilder.disableAuthCaching();
                        return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
                    }
                })/.setMaxRetryTimeoutMillis(2000)/
        );
```



