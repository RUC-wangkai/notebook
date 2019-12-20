# ELK

ELK可以都部署在同一台服务器，docker-compose.yml文件内容：

```yaml
version: "3"
services:
  es-service:
    image: "elasticsearch:latest"
    ports:
      - "9200:9200"

  logstash-service:
    image: "logstash:latest"
    ports:
      - "9600:9600"
      - "5959:5959"
      - "5960:5960"
    volumes:
      - ./logstash-conf:/etc/logstash/conf.d
    command: logstash -f /etc/logstash/conf.d/logstash.conf


  kibana-service:
    image: "kibana:latest"
    ports:
      - "5601:5601"

    environment:
      - ELASTICSEARCH_URL=http://es-service:9200

```

其中，logstash-conf文件夹中新建一个logstash.conf，内容为：

```ruby
input {
    beats {
        port => 5959
    }

    tcp {
        port => 5960
    }
}

filter {
  mutate {
    rename => { "[host][name]" => "host" }
  }
}
output {
    stdout {

    }
    elasticsearch {
        hosts => ["es-service:9200"]
        index => "fuck"
    }
}

```


注意事项：

1. service之间的通信可以通过service name来作为host alias

2. docker-compose up后，测试elk是否正常的方法非常简单，运行命令"echo hello world | nc localhost 5960"

3. logstash的配置中，input有两个模块，一个是tcp，监听5960端口的输入；一个是beats的输入，从5959端穿数据过来，所以docker-compose.yml中需要开启5959端口和5960端口（可自定义）。

4. logstash的配置中，fileter中的mutate的作用是做一个转换，把filebeats的输入json字符串中的key转换一下。如果不转换，那么filebeats的输入传给logstash，然后logstash再传给es会插入数据失败。

5. logstash的配置中，beats的端口不可以和tcp端口一样。filebeats的配置中，也不能够传到logstash监听的tcp端口，否则会有乱码问题。

6. logstash的配置中，output有两个模块，一个是输出的标准输出，一个是输出到es的指定index中。

7. 打开Kibana Web UI，在首页查看需要检索的索引时，输入在logstash.conf的索引名称，比如上例子中的"fuck"。

8. Filebeat（6.5.x版本）需要分别部署在各个服务器上，监听服务器上的log。我用的是官网的压缩包而不是docker。修改filebeat.yml的配置：


(1) 开启log监听：

```
- type: log

  # Change to true to enable this input configuration.
  enabled: false
```

改为：

```
- type: log

  # Change to true to enable this input configuration.
  enabled: false
```

(2) 把你自己想要监听的log文件夹写在paths下面：

```
  paths:
    - /home/monitor1379/Workspaces/gopath/src/shannonpdf/logs/*.log
    # - /var/log/*.log
    #- c:\programdata\elasticsearch\logs\*

```

(3) 关闭filebeat直接输出到es的功能，因为耗资源，插入的这一步交给logstash来做：

```
# output.elasticsearch:
  # Array of hosts to connect to.
  # hosts: ["localhost:9200"]

```


(4) 开启filebeat输出到logstash的功能：

```
output.logstash:
  # The Logstash hosts
  hosts: ["localhost:5959"]

```

这里的5959端口就是logstash.conf配置里设置的：

```
input {
    beats {
        port => 5959
    }
}
```



start filebeat:
         sudo chown root filebeat.yml 
         sudo ./filebeat -e