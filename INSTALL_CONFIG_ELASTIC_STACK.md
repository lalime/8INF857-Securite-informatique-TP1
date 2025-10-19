🛡️ Étapes 1 & 2 : Installation et configuration de l’Elastic Stack (Elasticsearch + Kibana)
Cette documentation détaille la préparation de la machine Linux, l’installation et la configuration d’**Elasticsearch** et **Kibana**, éléments centraux de la Stack ELK utilisés pour le stockage, la recherche et la visualisation des logs collectés.


📋 **Prérequis**
- Machine virtuelle Linux (Ubuntu 22.04 LTS)
- Accès root ou sudo  
- Ports ouverts : 9200 (Elasticsearch) et 5601 (Kibana)  
- Connexion réseau stable  



 +Étape 1 : Préparer la VM et les dépendances

 **Mettre à jour le système et installer Java (OpenJDK 17)**  


sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-17-jre-headless apt-transport-https curl wget gnupg jq ntp

timedatectl status
java -version
✔️ System clock synchronized: yes
✔️ openjdk version "17.0.xx"

+Étape 2 : Installer Elasticsearch et Kibana

-Transférer les paquets depuis Windows

scp .\elasticsearch-8.15.3-amd64.deb amal@192.168.2.37:~
scp .\kibana-8.15.3-amd64.deb amal@192.168.2.37:~
-Installer les paquets sur Ubuntu
sudo dpkg -i elasticsearch-8.15.3-amd64.deb
sudo dpkg -i kibana-8.15.3-amd64.deb
sudo apt -f install -y

-Configurer Elasticsearch

Fichier : /etc/elasticsearch/elasticsearch.yml
yaml
cluster.name: siem-cluster
node.name: siem-node-1
network.host: 0.0.0.0
http.port: 9200
discovery.type: single-node
-Configurer Kibana
Fichier : /etc/kibana/kibana.yml
server.port: 5601
server.host: "0.0.0.0"
elasticsearch.hosts: ["http://localhost:9200"]

-Activer et démarrer les services

sudo systemctl daemon-reload
sudo systemctl enable --now elasticsearch kibana
sudo systemctl status elasticsearch --no-pager -l
sudo systemctl status kibana --no-pager -l
Les deux services doivent afficher : Active (running)

-Tester Elasticsearch
curl -s http://localhost:9200 | jq .


-Créer un tunnel SSH depuis la machine locale :

ssh -L 5601:localhost:5601 amal@192.168.2.37
Puis ouvrir : http://localhost:5601
✔️ L’interface Kibana doit s’afficher.

📚 Ressources utiles

Documentation officielle Elasticsearch

Documentation officielle Kibana

Guide Elastic Stack 8.x – Installation
