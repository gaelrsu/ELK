# Metricbeat 

## Download package 
```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
```

## Install metrcibeat
```bash
sudo apt-get update && sudo apt-get install metricbeat
```

## launch
```bash
sudo systemctl enable metricbeat
```
