构建ELK监控环境

incps@incpshome:~$ docker pull elasticsearch:5.5

incps@incpshome:~$ docker pull kibana:5.5

incps@incpshome:~$ docker pull logstash:5.5





docker run -p 9200:9200 -e "http.host=0.0.0.0" -e "transport.host=127.0.0.1" --name my-elastic -d elasticsearch:5.5

docker run -p 5601:5601 -e "ELASTICSEARCH\_URL=http://127.0.0.1:9200" --name my-kibana --network host -d kibana:5.5



logstash.yml

```
http.host: "0.0.0.0"
path.config: /usr/share/logstash/pipeline
xpack.monitoring.elasticsearch.url: http://localhost:9200
xpack.monitoring.elasticsearch.username: elastic
xpack.monitoring.elasticsearch.password: changeme
```

logstash.conf

```
input {
  file {
    path => "/tmp/access_log"
    start_position => "beginning"
  }
}
output {
  elasticsearch {
    hosts => ["localhost:9200"]
    user => "elastic"
    password => "changeme"
  }
}
```



docker run -v /home/incps/devsoft/elkspack/logstash/conf.d:/usr/share/logstash/pipeline/:ro -v /tmp:/tmp:ro -v /home/incps/devsoft/elkspack/logstash/logstash.yml:/usr/share/logstash/config/logstash.yml:ro --name my-logstash --network host -d logstash:5.5

