 # DECOMMENTER L'OUTPUT VALIDE / METTRE LOCAL HOST OU L'IP DU SERVEUR ELK
filebeat.yml :  
```bash
 output.elasticsearch
      ...
      #output.elasticsearch:
        # Array of hosts to connect to.
        #hosts: ["localhost:9200"]
      ...
   outputlogstash
      output.logstash:
        # The Logstash hosts
        hosts: ["localhost:5044"]
```    
       
 ============filebeat unputs =============
 filebeat.inputs
 ```bash
 - type: log
    - /var/log/nginx/*.log
    
 exclude_lines: ["^127.0.0.1]             //retirer 
 exclude_files: [".gz$"]
 ignore_older: 24h
 tags: ["back", "system"]
 fields:
     env: prod
     part: back
```

 