## lostash/conf.d   possibilité de mettre sous un seul fichier 

02-beats-input.conf   :
```bash
input {
  beats {
    port => 5044
  }
}
```

30-elasticsearch-output.conf : 
```bash
output {
 elasticsearch {
   hosts => ["localhost:9200"]
   manage_template => false
   index => "%{[@metadata][beat]}-%{[@metadata][version]}-%{+$
 }
}
```

10-syslog-filter.conf  :
```bash
filter {
      if [type] == "syslog" {
        grok {
          match => { "message" => "%{SYSLOGTIMESTAMP:syslog_t$
          add_field => [ "received_at", "%{@timestamp}" ]
          add_field => [ "received_from", "%{host}" ]
        }
        syslog_pri { }
        date {
          match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "$
        }
      }
    }
```



