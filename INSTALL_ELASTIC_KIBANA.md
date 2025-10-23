## II. Installation et Configuration d'Elasticsearch et Kibana

### Étape 1 : Importer la clé GPG d'Elastic et configurer le dépôt

1.  **Installer les prérequis et la clé GPG :**
    ```bash
    sudo apt update
    sudo apt install apt-transport-https wget -y
    wget -qO - [https://artifacts.elastic.co/GPG-KEY-elasticsearch](https://artifacts.elastic.co/GPG-KEY-elasticsearch) | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
    ```

2.  **Ajouter le dépôt Elastic (8.x) :**
    ```bash
    echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] [https://artifacts.elastic.co/packages/8.x/apt](https://artifacts.elastic.co/packages/8.x/apt) stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
    ```

3.  **Mettre à jour :**
    ```bash
    sudo apt update
    ```

---
<br/>

### Étape 2 : Installer et configurer Elasticsearch (Base de données)

1.  **Installer le paquet :**
    ```bash
    sudo apt install elasticsearch -y
    ```

2.  **Configurer la mémoire (JVM Heap Size) :** ⚠️
    * **Ajustez la RAM** allouée (ex: 4g pour 8Go de RAM totale de la VM) :
        ```bash
        sudo nano /etc/elasticsearch/jvm.options
        ```
    * Modifiez les lignes :
        ```
        -Xms4g
        -Xmx4g
        ```

3.  **Démarrer et activer le service :**
    ```bash
    sudo systemctl enable elasticsearch
    sudo systemctl start elasticsearch
    ```

4.  **Générer le mot de passe de l'utilisateur `elastic` (Sécurité) :**
    * **NOTEZ BIEN CE MOT DE PASSE.**
        ```bash
        sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
        ```

5. **Tester l'installation**

    ```bash
    curl -u elastic https://localhost:9200 --cacert /etc/elasticsearch/certs/http_ca.crt
    ```

---
<br/>

### Étape 3 : Installer et configurer Kibana (Visualisation)

1.  **Installer le paquet :**
    ```bash
    sudo apt install kibana -y
    ```

2.  **Configurer Kibana :**
    * Modifiez le fichier de configuration :
        ```bash
        sudo nano /etc/kibana/kibana.yml
        ```
    * Décommentez et ajustez les lignes suivantes :
        ```yaml
        server.port: 5601
        server.host: "0.0.0.0" # Accès depuis l'extérieur de la VM
        elasticsearch.hosts: ["http://localhost:9200"]
        
        # Authentification pour se connecter à Elasticsearch
        elasticsearch.username: "elastic"
        elasticsearch.password: "VOTRE_MOT_DE_PASSE_ELASTIC"
        ```

3. **Configuration de base :**
   Configurer le fichier elasticsearch.yml
    ```bash
    sudo nano /etc/elasticsearch/elasticsearch.yml
    ```

   Modifiez ou ajoutez :

    ```bash
    network.host: 127.0.0.1
    http.port: 9200
    cluster.name: mon_cluster
    node.name: node-1
	discovery.seed_hosts: ["127.0.0.1"]
	cluster.initial_master_nodes: ["node-1"]
    ```



5.  **Démarrer et activer le service :**
    ```bash
    sudo systemctl enable kibana
    sudo systemctl start kibana
    ```

---
<br/>

### Étape 4 : Vérification et Accès

1.  Ouvrez un navigateur sur votre machine hôte ou cliente.
2.  Accédez à l'URL : `http://[IP_de_votre_VM]:5601`
3.  Connectez-vous avec l'utilisateur **`elastic`** et le mot de passe généré précédemment.

La **Pile Elastic** est opérationnelle et prête à recevoir les données du Wazuh Manager (prochaine étape).

---
<br/><br/>

### 🛠️ Résolution du problème "master not discovered yet" dans Elasticsearch 8.19.4

#### 4.1. 🔍 Contexte**

Lors du démarrage d’Elasticsearch, le message suivant apparaît dans les logs :
*master not discovered yet, this node has not previously joined a bootstrapped cluster...*

Cela signifie que le nœud ne parvient pas à élire un master, car il n’a jamais été bootstrappé correctement.

---

#### ✅ Étapes pour résoudre le problème

##### 1. Modifier la configuration du cluster

Ouvrir le fichier de configuration :

    ```bash
    sudo nano /etc/elasticsearch/elasticsearch.yml
    ```

 Modifiez ou ajoutez :

    ```bash
    network.host: 127.0.0.1
    http.port: 9200
    cluster.name: mon_cluster
    node.name: node-1
	discovery.seed_hosts: ["127.0.0.1"]
	cluster.initial_master_nodes: ["node-1"]
    ```


 ### 🛠️ Kibana ne peut plus utiliser le compte elastic pour se connecter à Elasticsearch
 
 *FATAL  Error: [config validation of [elasticsearch].username]: value of "elastic" is forbidden. This is a superuser account that cannot write to system indices that Kibana needs to function. Use a service account token instead. Learn more: https://www.elastic.co/guide/en/elasticsearch/reference/8.0/service-accounts.html*

