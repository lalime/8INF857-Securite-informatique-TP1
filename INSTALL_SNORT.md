[⬅ Retour à l'accueil](README.md)
<br/>

## 🛡️ Étape 5 : Intégration IDS/IPS (Snort)

Cette documentation détaille l'installation, la configuration de **Snort (IDS/IPS)**, et son intégration avec **syslog-ng** pour analyser le trafic réseau et transmettre les alertes dans un fichier ou via Syslog.

---

### 📋 Prérequis

- Une machine virtuelle Linux (**Ubuntu 20.04 LTS** recommandé)
- Accès `root` ou via `sudo`
- Interface réseau active (ex. : `eth0`)
- `syslog-ng` installé et fonctionnel
- Connexion réseau stable

---
<br/><br/>
## ✅ Étape 3.1 : Installer et configurer Snort

### 🔧 Installation de Snort

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install snort -y
````

✅ Vérifier l'installation :

```bash
snort -V
```

---

### ⚙️ Configuration de Snort

Modifier le fichier `/etc/snort/snort.conf` :

```bash
sudo nano /etc/snort/snort.conf
```

#### ➤ Définir les variables réseau :

```conf
ipvar HOME_NET 192.168.1.0/24  # Remplacez par votre réseau
ipvar EXTERNAL_NET any
```

#### ➤ Activer l’interface réseau à surveiller :

Assurez-vous que Snort est lancé avec la bonne interface (ex. : `eth0`).

---

### 📥 Télécharger les règles communautaires Snort

```bash
sudo wget https://www.snort.org/downloads/community/community-rules.tar.gz -O /etc/snort/rules/community-rules.tar.gz
sudo tar -xvzf /etc/snort/rules/community-rules.tar.gz -C /etc/snort/rules
```

Puis ajouter dans `snort.conf` :

```conf
include $RULE_PATH/community-rules/community.rules
```

---

### 🧪 Tester la configuration

```bash
sudo snort -T -c /etc/snort/snort.conf -i eth0
```

✅ Corriger toute erreur avant de continuer.

---

### 🚀 Lancer Snort en mode IDS (console)

```bash
sudo snort -c /etc/snort/snort.conf -i eth0 -A console
```

Options :

* `-i eth0` : interface réseau à surveiller
* `-A console` : affiche les alertes en temps réel

---
<br/><br/>

## ✅ Étape 3.2 : Envoyer les alertes Snort à syslog-ng

### 🔸 Option 1 : Écriture dans un fichier lisible par syslog-ng

#### 📁 Modifier `snort.conf` pour ajouter une sortie `alert_fast`

```conf
output alert_fast: /var/log/snort/alerts.log
```

Puis relancer Snort :

```bash
sudo snort -c /etc/snort/snort.conf -i eth0 -l /var/log/snort
```

#### 🛠️ Configurer syslog-ng pour lire ce fichier

Fichier : `/etc/syslog-ng/syslog-ng.conf`

```conf
source s_snort {
    file("/var/log/snort/alerts.log" follow_freq(1) flags(no-parse));
};

destination d_elasticsearch {
    elasticsearch2(
        index("snort_logs")
        type("alerts")
        servers("localhost:9200")
    );
};

log {
    source(s_snort);
    destination(d_elasticsearch);
};
```

Redémarrer syslog-ng :

```bash
sudo systemctl restart syslog-ng
```

---

### 🔸 Option 2 : Envoi direct en Syslog

#### 🛠️ Modifier `snort.conf` :

```conf
output alert_syslog: LOG_AUTH LOG_ALERT
```

Relancer Snort :

```bash
sudo snort -c /etc/snort/snort.conf -i eth0
```

#### 🔧 Vérifier syslog-ng (fichier `/etc/syslog-ng/syslog-ng.conf`) :

```conf
source s_syslog {
    syslog();
};

destination d_elasticsearch {
    elasticsearch2(
        index("snort_logs")
        type("alerts")
        servers("localhost:9200")
    );
};

log {
    source(s_syslog);
    destination(d_elasticsearch);
};
```

Redémarrer syslog-ng :

```bash
sudo systemctl restart syslog-ng
```

---

## 🧪 Validation

### 🎯 Générer du trafic (exemple : nmap)

```bash
sudo nmap -sS -Pn -T5 -p- 172.16.245.128  # Remplacer par l’IP de votre machine cible
```

### 📂 Vérifier les alertes :

* **Option 1 :** `/var/log/snort/alerts.log`
* **Option 2 :** `/var/log/syslog` ou `/var/log/messages`

---

### 🔍 Vérification dans Kibana

1. Accédez à : `http://<IP_de_la_machine>:5601`
2. Recherchez l’index `snort_logs`
3. Visualisez les alertes injectées dans Elasticsearch

---

## ⚠️ Remarques importantes

* Assurez-vous que `syslog-ng` a les droits de lecture :

```bash
sudo chmod 644 /var/log/snort/alerts.log
```

* Pour lancer Snort comme service :

```bash
sudo systemctl enable snort
sudo systemctl start snort
```

---

## 📚 Ressources utiles

* [Documentation officielle de Snort](https://www.snort.org/documents)
* [Documentation officielle de syslog-ng](https://www.syslog-ng.com/technical-documents/list/syslog-ng-open-source-edition/)
* [Sumologic](https://www.sumologic.com/help/docs/send-data/hosted-collectors/cloud-syslog-source/install-syslog-ng/)


<br/><br/>

## 🔐 Création d'un rôle Elasticsearch et d'un utilisateur dédié à syslog-ng

### Objectif

Créer un utilisateur `syslog-ng` dans Elasticsearch 8.x avec uniquement les **droits d'écriture** dans les index `syslog-ng-*`.

---

### ✅ Étape 1 : Créer un rôle `syslog-ng-writer`

Ce rôle autorise l'écriture dans tous les index commençant par `syslog-ng-`.

```bash
curl -u elastic:VOTRE_MOT_DE_PASSE \
  -X POST "https://localhost:9200/_security/role/syslog-ng-writer" \
  -H "Content-Type: application/json" \
  --cacert /etc/elasticsearch/certs/http_ca.crt \
  -d '{
    "indices": [
      {
        "names": [ "syslog-ng-*" ],
        "privileges": ["create_index", "write", "create", "index"]
      }
    ]
  }'
````

> 🔐 Remplacez `VOTRE_MOT_DE_PASSE` par le mot de passe de l'utilisateur `elastic`.

---

### ✅ Étape 2 : Créer l'utilisateur `syslog-ng` avec mot de passe

```bash
curl -u elastic:VOTRE_MOT_DE_PASSE \
  -X POST "https://localhost:9200/_security/user/syslog-ng" \
  -H "Content-Type: application/json" \
  --cacert /etc/elasticsearch/certs/http_ca.crt \
  -d '{
    "password": "UN_MOT_DE_PASSE_SECURISE",
    "roles": [ "syslog-ng-writer" ],
    "full_name": "Syslog-NG Writer",
    "email": "syslog@example.com"
  }'
```

> 📌 Remplacez `"UN_MOT_DE_PASSE_SECURISE"` par un mot de passe fort, que vous utiliserez ensuite dans la configuration de `syslog-ng`.

---

### ✅ Étape 3 : Tester avec `curl` (optionnel)

Vérifiez que l'utilisateur peut bien écrire :

```bash
curl -u syslog-ng:UN_MOT_DE_PASSE_SECURISE \
  --cacert /etc/elasticsearch/certs/http_ca.crt \
  -X POST "https://localhost:9200/syslog-ng-test/_doc" \
  -H 'Content-Type: application/json' \
  -d '{"test": "message depuis syslog-ng"}'
```

Vous devez recevoir une réponse avec `"result": "created"`.

---

### ✅ Bonnes pratiques

* Créez des mots de passe forts (`openssl rand -base64 20`)
* Ne donnez pas plus de privilèges que nécessaire
* Isolez les index par application (ex. : `syslog-ng-*`, `audit-*`, etc.)
* Utilisez un pipeline d'ingestion pour parser les messages côté Elasticsearch si besoin

<br/><br/>
[⬅ Retour à l'accueil](README.md)
