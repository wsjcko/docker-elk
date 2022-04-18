
##### docker-compose 命令使用
docker-compose -f ./docker-stack.yml  up -d 
docker-compose -f ./docker-stack.yml  down
docker-compose -f ./docker-stack.yml   ps
docker-compose -f ./docker-stack.yml  restart kibana
##### 查看日志
docker-compose -f ./docker-stack.yml logs -f kibana |grep ERROR

##### 重置密码 elastic, kibana
 [官方文档](https://www.elastic.co/guide/cn/kibana/current/index.html)
docker exec -it elk_elasticsearch /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

配置 elk_elasticsearch 的yml   xpack.security.enrollment.enabled: true
docker exec -it elk_elasticsearch /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
docker exec -it elk_elasticsearch /usr/share/elasticsearch/bin/elasticsearch-plugin install

curl -H "Content-Type:application/json" -XPOST -u elastic 'http://192.168.140:9200/_xpack/security/user/elastic/_password' -d '{ "password" : "123456" }'


##### 
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,kibana_system,logstash_system,beats_system,remote_monitoring_user.
[官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/8.1/built-in-users.html)
需要root用户权限  -u root
docker exec -it  elk_elasticsearch /usr/share/elasticsearch/bin/elasticsearch-reset-password  -u kibana_system
docker exec -it  elk_elasticsearch /usr/share/elasticsearch/bin/elasticsearch-reset-password  -u logstash_system


##### 即将弃用 重置密码方式
 docker exec -u root -i -t elk_elasticsearch /bin/bash
 /usr/share/elasticsearch/bin#  ./elasticsearch-setup-passwords interactive



#### kibana报错, kibana_system连接elasticsearch 通信，密码重新获取
elk_kibana  | [2022-04-16T16:36:00.147+00:00][ERROR][elasticsearch-service] Unable to retrieve version information from Elasticsearch nodes. connect ECONNREFUSED 172.30.0.2:9200
elk_kibana  | [2022-04-16T16:36:15.522+00:00][ERROR][elasticsearch-service] Unable to retrieve version information from Elasticsearch nodes. security_exception: [security_exception] Reason: unable to authenticate user [kibana_system] for REST request [/_nodes?filter_path=nodes.*.version%2Cnodes.*.http.publish_address%2Cnodes.*.ip]


#### logstash 报错
##### xpack监控elasticsearch和 连接elasticsearch通信，都是 logstash_system，密码重新获取，
elk_logstash  | [2022-04-16T17:20:06,251][ERROR][logstash.monitoring.internalpipelinesource] Failed to fetch X-Pack information from Elasticsearch. This is likely due to failure to reach a live Elasticsearch cluster.
elk_logstash  | [2022-04-16T17:20:36,245][ERROR][logstash.licensechecker.licensereader] Unable to retrieve license information from license server {:message=>"Got response code '405' contacting Elasticsearch at URL 'http://elasticsearch:9200/9200/_xpack'"}

### logstash template报错
 logstash.outputs.elasticsearch][main] Failed to install template {:message=>"Got response code '403' contacting Elasticsearch at URL 'http://elasticsearch:9200/_index_template/ecs-logstash'

<!-- curl -u logstash_system 'http://localhost:9200/_index_template/ecs-logstash' 

curl -H "Content-Type:application/json" -XPOST -u logstash_system 'http://localhost:9200/_shield/authenticate' -d '{ "password" : "T*UebpqQGYxVlmqQApT1" }' -->


[官方文档](https://www.elastic.co/guide/en/logstash/8.1/ls-security.html)
role:  logstash_writer                            manage_ilm
{
  "cluster": ["manage_index_templates", "monitor"], 
  "indices": [
    {
      "names": [ "logstash-*" ], 
      "privileges": ["write","create","create_index"] 
    }
  ]
}

user: logstash_internal
{
  "password" : "x-pack-test-password",
  "roles" : [ "logstash_writer"],
  "full_name" : "Internal Logstash User"
}