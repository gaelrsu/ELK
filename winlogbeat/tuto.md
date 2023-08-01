# SOUS WINDOWS :
____________________________________________________________________________________

## Winlogbeat
```

  download => extraire => coller dans 'program' 
  Editer .yml => idem que sur debian     output ELK IP / ip Kibana
Voir le fichier de config proposé par elk : http://192.168.1.1:5601/app/home#/tutorial/windowsEventLogs
  
  ```
  
POWERSHELL en admin;
```
  cd '..\..\Program Files'
  cd .\winlogbeat
  .\install-service-winlogbeat.ps1
  ```
  
  si exécution powershell bloqué :  Set-ExecutionPolicy Unrestricted
  ```
  Start-Service winlogbeat
  ```

______________________________________________________________________________________
 ### SYSMON
  télécharger sysmon et coller dans 'program'
  Télécharger sysmon config file (github) et coller dans program/sysmon
  
POWERSHELL
```
  cd ..\Sysmon
  .\Sysmon.exe -accepteula -i .\sysmonconfig-export.yml   
 ``` 
  !!!!!!!!!!!!!fichier sysmon config!!!!!!!!!!
  
 Installation dashboard :
 ```
  .\winlogbeat.exe setup --dashboard
```  
  !!!!!!!!!!!il faut que elastic et kibana soit joignable!!!!!!!!!!!!
  
