# ELK

I/ Installation
Elasticsearch fonctionne donc sous le principe d’une base nosql distribuée sur un ensemble de machine lui permettant de créer une résilience plus forte selon la taille du cluster.
Il indexe tous les documents en offrant une qualité de recherche en terms frequency (fréquence des mots) et en Inverse different frequency, c’est-à-dire que moins un mot est connu plus il aura de poids ou de valeur dans la recherche.
Commencez l’installation d’elasticsearch avec la commande présentée :
sudo apt install elasticsearch

II/ Configuration
Utilisez maintenant l’éditeur de texte afin de modifier le ficher de configuration d’elasticsearch :
(Il faudra porter une attention particulière sur le fichier, celui-ci étant en format YAML, il faudra bien conserver le format d’indentation).
sudo nano /etc/elasticsearch/elasticsearch.yml
Le document se présentera sous cette forme :
cluster.name : my-application ➔nom du cluster devant être le même pour toute les machines afin de communiquer
node.name : node-1 ➔nom du noeuds
network.host : localhost ➔ip d’écoute
http.port : 9200 ➔port d’écoute
discovery.seed_hosts : [] ➔permet de découvrir tous les hôtes
Elasticsearch écoute tout le trafic sur le port 9200, il faudra limiter l’accès pour empêcher les personnes extérieures de lire les données ou de modifier le cluster. Il sera donc important de décommenter la ligne network.host et remplacer sa valeur par localhost. localhost permettra d’écouter sur toutes les interfaces et les IP liés. (Possibilité de remplacer le localhost par 0.0.0.0 afin d’écouter toutes les interfaces).
Il faudra maintenant démarrer le service à l’aide de la commande :
sudo systemctl start elasticsearch
Utilisez la commande ici-bas afin qu’il s’active à chaque démarrage du serveur :
sudo systemctl enable elasticsearch
5
Testez maintenant le bon fonctionnement du service elasticsearch en envoyant une requête http :
curl -X GET "localhost:9200"
Le résultat de la requête devrait correspondre à celle-ci :
{
"name" : "ELKSERV",
"cluster_name" : "my-application",
"cluster_uuid" : "z3gXpvA8SyC50QOf4YBnNg",
"version" : {
"number" : "7.15.1",
"build_flavor" : "default",
"build_type" : "deb",
"build_hash" : "83c34f456ae29d60e94d886e455e6a3409bba9ed",
"build_date" : "2021-10-07T21:56:19.031608185Z",
"build_snapshot" : false,
"lucene_version" : "8.9.0",
"minimum_wire_compatibility_version" : "6.8.0",
"minimum_index_compatibility_version" : "6.0.0-beta1"
},
"tagline" : "You Know, for Search"
Il est possible de réduire la consommation d’elasticsearch en modifiant la configuration du fichier jvm.option se trouvant dans /etc/elasticsearch/ et en modifiant les lignes qui possèdent la valeur 1G pour 512m.
KIBANA
I/ Installation
Kibana sert d’interface et d’exploitation des données, il est aussi un assistant d’installation en ce qui concerne les systèmes métriques, logs et SIEM. On peut aussi lui implémenter un jeu de données afin de servir d’exemple.
Une fois les composants elasticsearch en place, procédez à l’installation de kibana :
sudo apt install kibana

II/ Configuration
Une fois installé, modifiez le fichier YAML en décommentant les lignes et en les modifiant comme ci-dessous :
server.port: 5601 ➔port d’écoute
server.host: 0.0.0.0 ➔ permet de passer en écoute
elasticsearch.hosts: ["http://127.0.0.1:9200"] ➔recherche d’hôtes
6
Activez ensuite le service et le démarrer :
sudo systemctl enable kibana
sudo systemctl start kibana
Attention son installation est longue et Kibana peut ne pas être opérationnel malgré un « status up » affiché par le résultat de la commande :
sudo systemctl status kibana
Utilisez la commande afin de vérifier sa bonne installation :
tail -f /var/log/syslog
192.168.1.168 :5601
(L’ip sera celle du serveur ELK)
Afin d’accéder à l’interface créez un utilisateur administratif, ainsi que son mot de passe qui sera stocké dans htpasswd.users :
echo "kibanaadmin:`openssl passwd -apr1`" | sudo tee -a /etc/nginx/htpasswd.users
Logstash
I/ Installation
Logstash est un outil open source très puissant pour le traitement de données par un pipe.
Il récupère les données simultanément de sources diverses (applications, serveurs etc.) pour les traiter en 3 étapes : Récupération des données et envoi vers le pipe input ➔ filtrage (les données sont triées, il restera uniquement ce qui nous intéresse, c’est-à-dire que cela nous permettra de faire ressortir du lot la ou les donnée(s) désirée(s) ) ➔ elles ressortent ensuite par le output en direction d’elasticsearch.
Lancez l’installation à l’aide de cette commande :
sudo apt install logstash
7

II/ Configuration
La configuration se fait et se fera au fur et à mesure du traitement des données dans le dossier /etc/logstash/con.d
Une fois installé, débutez la configuration de celui-ci.
Dans un premier temps, créez un fichier d’entrée Filebeat :
sudo nano /etc/logstash/conf.d/02-beats-input.conf
Avec à l’intérieur :
input {
beats {
port => 5044
}
}
Testez maintenant la configuration de Logstash à l’aide de la commande ci-dessous :
sudo -u logstash /usr/share/logstash/bin/logstash --path.settings /etc/logstash -t
Si le résultat suivant s’affiche on peut alors passer à la suite.
output
Config Validation Result: OK. Exiting Logstash
Une fois la vérification faite, activez et lancez le service :
sudo systemctl start logstash
sudo systemctl enable logstash
Nginx
I/ Installation
Nginx est un proxy inverse, c’est-à-dire qu’il va traiter et redistribuer les requêtes des clients à un serveur. En plus de travailler sur l’équilibrage entre les serveurs, le proxy inverse peut gérer les services qui ne font pas nécessairement parti des applications des serveurs (exemple : compression, SSL encryption)
Pour l’installer :
sudo apt install nginx
Attention vous pourrez avoir un conflit dans le cas où apache2 serait installé sur le serveur.
Une erreur du type apparaitra alors :
8
Failed to start A high performance web server and a reverse proxy server.
Désactivez alors le service apache2 à l’aide de la commande :
Systemctl stop apache2.service
Supprimez du serveur apache2 si le problème persiste avec la commande :
sudo apt-get remove --purge apache2
Avant de tester si Nginx fonctionne bien, ajoutez les règles de Nginx ou niveau du pare-feu :
sudo ufw allow 'Nginx HTTP'
Vérifiez le changement à l’aide de la commande :
sudo ufw status
Status: inactive
Dans le cas comme ici où le statut apparaît en inactif il vous faudra taper ces commandes afin de l’activer pour ensuite vérifier les règles :
sudo ufw enable
sudo ufw default deny
sudo iptables -L
sudo ufw status
output
Status: active
To Action From
-- ------ ----
Nginx HTTP ALLOW Anywhere
Nginx HTTP (v6) ALLOW Anywhere (v6)
II/ Configuration
A présent, vérifiez que le service est opérationnel :
systemctl status nginx
Le service peut tourner correctement mais le meilleur moyen de valider et d’en être sûr est d’envoyer une requête :
Curl ‘ipduserveur ‘
Afin de garantir une sécurité c’est l’utilisateur www-data du groupe adm qui est propriétaire du fichier et qui a les droits d’accès. Si ce n’est pas le cas rendez-lui ses droits à l’aide de la commande :
Chown -R www-data : adm /var/log/nginx
La création d’un pattern dans Logstash permet de capturer des éléments et de les charger dans une variable. Par exemple avec la récupération des logs Nginx :
9
Mkdir /etc/logstash/pattern ➔création du dossier
Chmod 755 -R /etc/logstash/pattern ➔droits d’accès et modification
Nano /etc/logstash/pattern/nginx ➔modification du fichier
Complétez avec ce qui suit :
NGUSERNAME [a-zA-A\ .\@\-\+_%]+
NGUSER %{NGUSERNAMES} ➔ Cela permettra de récupérer les
utilisateurs dans la ligne de log Nginx
Après la création de nouveau pattern il faudra les créer dans l’interface kibana afin d’en extraire les données.
La création de nouveaux patterns permettra de distinguer les différents types de log envoyés de logstash à notre elasticsearch.
Dans l’interface kibana :
Index pattern ➔ create index pattern ➔ cherchez selon le nom de l’index, dans ce cas entrez nginx-* (l’étoile lui permettant de prendre tout ce que commencera par nginx- , cela permettra en cas de liste par des dates de toutes les traiter. Exemple : nginx-10/01/22 nginx-11/01/22 etc...) ➔ choisissez ensuite @timestamp (qui est le format horaire).
Filebeat

I/ Installation
Elastic Stack utilise plusieurs expéditeurs de données légères appelés Beats pour collecter des données en provenance de diverses sources pour les envoyer vers Logstash ou Elasticsearch.
Filebeat : recueille et expédie les fichiers journaux.
Metricbeat : collecte les métriques de vos systèmes et services.
Packetbeat : recueille et analyse les données du réseau.
Winlogbeat : collecte les journaux des événements Windows.
Auditbeat : collecte les données du framework de vérification Linux et surveille l'intégrité des fichiers.
Heartbeat : surveille activement la disponibilité des services.
Dans ce cas-là nous utiliserons Filebeat, installons-le :
sudo apt install filebeat
10

II/ Configuration
Procédez à la configuration de son fichier YML :
sudo nano /etc/filebeat/filebeat.yml
Configurez comme montré ici-bas :
#output.elasticsearch:
# Array of hosts to connect to.
#hosts: ["localhost:9200"]
output.logstash:
# The Logstash hosts
hosts: ["192.168.1.168:5044"]
output.kibana :
hosts : ["192.168.1.168 :5601"]
Dans le cas où il s’agit d’un serveur ELK distant on n’utilisera pas l’ip ‘localhost’ mais celle de notre serveur. Si vous choisissez de passer par Elasticsearch il vous faudra commenter le output de Logstash et décommenter l’output.elasticsearch.
Ici nous utiliserons le module system, qui permettra de collecter et analyser les journaux créés par le service de journalisation du système des distributions Linux courantes.
sudo filebeat modules enable system
Consultez la liste des modules par le biais de cette commande :
sudo filebeat modules list
Modifiez le fichier de configuration de filebeat de notre serveur externe (qui ne possède pas ELK) dans /etc/filebeat/filebeat.yml
On remplacera le type dans :
filebeat.input :
-type : log
Enabled : true
Path :
-/var/log/nginx/access.log (ou par access* afin de prendre tout ce qui commencera par access)
Output.logstash :
Hosts : ["192.168.1.168:5044"]
On commentera les lignes de l’output.elasticsearch et hosts par un # cela nous permettra de passer non par elasticsearch directement mais par logstash.
11
Sur notre serveur ELK il suffira de modifier l’input afin de lui confirmer un port d’écoute comme présenté si dessous :
Input {
Beats {
Port => 5044
}
}
On créera ensuite notre pattern dans l’interface de Kibana comme démontré précédemment.
filebeat à plusieurs volets input de type log voici une présentation de ses différentes options :
-enabled : activation.
-paths : chemin d’accès (avec ou sans *).
-fields : champs additionnels.
-recursive_glob.enabled : récursif “massif ‘’.
-include/exclude lines/files : inclure ou exclure des lignes ou fichiers (exemple exclure le curl sur la loopback).
-json : format de données semblable à la syntaxe des objets JavaScript.
-ignore_older : ignore les vieux fichiers.
-symlink : suivre les liens symboliques.
-tags : équivalent à la commande find.
-index : si elasticsearch est saisi en output.
Tout cela se retrouve dans le fichier filebeat.yml.
Passez à l’étape de vérification en interrogeant l’index Filbeat POUR en exécutant la commande :
curl -XGET 'http://localhost:9200/filebeat-*/_search?pretty'
En réponse vous devriez avoir :
Output
...
{
{
"took" : 4,
"timed_out" : false,
"_shards" : {
"total" : 2,
"successful" : 2,
"skipped" : 0,
"failed" : 0
},
"hits" : {
"total" : {
"value" : 4040,
12
"relation" : "eq"
},
"max_score" : 1.0,
"hits" : [
{
"_index" : "filebeat-7.7.1-2020.06.04",
"_type" : "_doc",
"_id" : "FiZLgXIB75I8Lxc9ewIH",
"_score" : 1.0,
"_source" : {
"cloud" : {
"provider" : "Dexter",
"instance" : {
"id" : "194878454"
},
"region" : "nyc1"
},
"@timestamp" : "2020-06-04T21:45:03.995Z",
"agent" : {
"version" : "7.7.1",
"type" : "filebeat",
"ephemeral_id" : "cbcefb9a-8d15-4ce4-bad4-962a80371ec0",
"hostname" : "june-ubuntu-20-04-elasticstack",
"id" : "fbd5956f-12ab-4227-9782-f8f1a19b7f32"
},
En revanche si le résultat de la requête affiche 0, Elasticsearch ne charge aucun journal, il faudra donc revoir la configuration.
