elasticsearch :


  elasticsearch.yml :
```bash
  cluster.name: my-application
    node.name: node-1
    network.host: 0.0.0.0
    http.port: 9200
    discovery.seed_hosts: []
    cluster.initial_master_nodes: ["node-1"]

  
  
  jvm.options:
  
    ## -Xms4g                ==> 512m    
    ## -Xmx4g
```  
  

