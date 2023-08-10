# install 
```bash
curl -L -O https://www.elastic.co/fr/downloads/beats/auditbeat
unzip
cd /auditbeat
nano auditbeat.yml
```
```
output.elasticsearch
  hosts: ["<ip:port>"]
setup.kibana:
  host: "<kibanaip:port>"
```
launch / upload dashboard
```bash
./auditbeat setup
```


