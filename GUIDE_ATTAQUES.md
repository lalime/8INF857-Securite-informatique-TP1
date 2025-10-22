# 🔒 Guide pratique — Scénarios d’attaques et preuves (pour SIEM-Lab)

## 🚨 Important : Sécurité et éthique
**Tous les tests ci-dessous doivent être exécutés uniquement dans la VM de laboratoire (ou sur un réseau privé contrôlé). NE JAMAIS lancer ces tests sur Internet public ou sur des machines que vous ne possédez pas.**

## 📋 Sommaire
- [🔒 Guide pratique — Scénarios d’attaques et preuves (pour SIEM-Lab)](#-guide-pratique--scénarios-dattaques-et-preuves-pour-siem-lab)
  - [🚨 Important : Sécurité et éthique](#-important--sécurité-et-éthique)
  - [📋 Sommaire](#-sommaire)
  - [🛠 Préambule — Architecture des logs](#-préambule--architecture-des-logs)
  - [🔍 Commandes de vérification initiales](#-commandes-de-vérification-initiales)
  - [💉 Scénario 1 — Injection SQL (démo)](#-scénario-1--injection-sql-démo)
    - [🎯 Objectif](#-objectif)
    - [📋 Préconditions](#-préconditions)
    - [Étapes](#étapes)
    - [📊 Ce que ~~Suricata~~ doit produire](#-ce-que-suricata-doit-produire)
    - [🔎 Requête KQL (Kibana)](#-requête-kql-kibana)
    - [📸 Collecte de preuves](#-collecte-de-preuves)
  - [🔍 Scénario 2 — Scan de ports (Nmap)](#-scénario-2--scan-de-ports-nmap)
    - [🎯 Objectif](#-objectif-1)
    - [📊 Ce que ~~Suricata~~ doit produire](#-ce-que-suricata-doit-produire-1)
  - [⚠️ Scénario 3 — DoS léger (hping3) — très limité](#️-scénario-3--dos-léger-hping3--très-limité)
    - [🚨 ATTENTION](#-attention)
    - [📊 Ce que ~~Suricata~~ doit produire](#-ce-que-suricata-doit-produire-2)
    - [📈 Visualisation](#-visualisation)
  - [🔐 Scénario 4 — Brute force SSH (auth logs)](#-scénario-4--brute-force-ssh-auth-logs)
    - [🎯 Objectif](#-objectif-2)
    - [📊 Ce que ~~Suricata~~ doit produire](#-ce-que-suricata-doit-produire-3)
    - [📸  Collecte de preuves](#--collecte-de-preuves)
  - [📥 Scénario 5 — Téléchargement suspect (HTTP)](#-scénario-5--téléchargement-suspect-http)
    - [🎯 Objectif](#-objectif-3)
    - [📊 Ce que ~~Suricata~~ doit produire](#-ce-que-suricata-doit-produire-4)
    - [Étapes](#étapes-1)
    - [📸 Collecte de preuves](#-collecte-de-preuves-1)
  - [Collecte de preuves](#collecte-de-preuves)
  - [📊 Requêtes KQL utiles et création de dashboard Kibana](#-requêtes-kql-utiles-et-création-de-dashboard-kibana)


## 🛠 Préambule — Architecture des logs

- **Pipeline** : ~~Suricata~~ → Filebeat → Elasticsearch → Kibana
- **Description** : ~~Suricata~~ capture le trafic réseau et génère des logs (eve.json). Filebeat collecte les logs système (ex. : auth.log). Elasticsearch indexe les données, et Kibana permet leur visualisation et analyse.

## 🔍 Commandes de vérification initiales

- Vérifier que ~~Suricata~~ est actif : `sudo systemctl status ~~Suricata~~`
- Vérifier Filebeat : `sudo systemctl status filebeat`
- Vérifier Elasticsearch : `curl -X GET "localhost:9200/_cluster/health?pretty"`
- Vérifier Kibana : Accéder à `http://localhost:5601`
- Vérifier les logs ~~Suricata~~ : `tail -f /var/log/~~Suricata~~/eve.json`

<br/>

## 💉 Scénario 1 — Injection SQL (démo)

### 🎯 Objectif

Simuler une tentative d’injection SQL et détecter l’attaque via ~~Suricata~~ et Kibana.

### 📋 Préconditions

- Serveur web local actif (ex. : `/var/www/html` ou `python -m http.server`).
- Idéalement, utiliser DVWA ou bWAPP pour simuler une application vulnérable.
- ~~Suricata~~ configuré pour inspecter le trafic web (mode IDS/IPS avec règles appropriées).

### Étapes

1. Exécuter une requête avec payload d’injection SQL :
   ```bash
   curl -i 'http://127.0.0.1/?q=1%27%20OR%20%271%27=%271'
   ```

### 📊 Ce que ~~Suricata~~ doit produire

- **event_type** : "alert" avec signature liée à SQL injection (si règles IDS activées).
- Champs : `http.request.body`, `url`.

### 🔎 Requête KQL (Kibana)

```kql
event_type:alert AND http.url:*OR* AND http.request.body:*%27*
```

### 📸 Collecte de preuves

- **Capture terminal** : Commande `curl` exécutée.
- **Extrait JSON** : Contenu pertinent dans `/var/log/~~Suricata~~/eve.json`.
- **Capture Kibana** : Écran Discover montrant l’événement et les champs `url` / `http.request.body`.

<br/>

## 🔍 Scénario 2 — Scan de ports (Nmap)

### 🎯 Objectif
Détecter un scan de ports (SYN scan) par ~~Suricata~~ et le visualiser dans Kibana.

### 📊 Ce que ~~Suricata~~ doit produire

- **event_type** : "alert" avec signature contenant `scan` (si règles IDS incluses).
- Nombreux **event_type** : "flow" avec `network.transport: "tcp"` et différents `dest.port`.
- **Kibana Discover** : Histogramme temporel montrant le pic pendant le scan.

<br/>

## ⚠️ Scénario 3 — DoS léger (hping3) — très limité

### 🚨 ATTENTION

**Un DoS peut rendre la VM instable. Limiter fortement l’intensité (ex. : 200 paquets). Ne jamais utiliser `--flood` sur un réseau partagé.**

### 📊 Ce que ~~Suricata~~ doit produire

- Alerts pour flood / nombreux événements `flow`.
- Pic temporel visible dans Kibana.

### 📈 Visualisation

- Time series Kibana montrant le pic d’activité.

<br/>

## 🔐 Scénario 4 — Brute force SSH (auth logs)

### 🎯 Objectif

Montrer la corrélation entre tentatives SSH dans `/var/log/auth.log` (Filebeat) et événements réseau détectés par ~~Suricata~~.

### 📊 Ce que ~~Suricata~~ doit produire

- Connexions répétées TCP sur le port 22 (`flows`/`alerts`).

### 📸  Collecte de preuves

- Extraits de `/var/log/auth.log`.
- **Kibana Discover** : `event.dataset: "system.auth"` affichant les échecs.
- **Corrélation** : Montrer les timestamps correspondants entre `flow` ~~Suricata~~ et log auth.

<br/>

## 📥 Scénario 5 — Téléchargement suspect (HTTP)

### 🎯 Objectif

Simuler un téléchargement de fichier ou un User-Agent suspect pour que ~~Suricata~~ logge un événement HTTP.

### 📊 Ce que ~~Suricata~~ doit produire

- **event_type** : "http" avec `http.request.headers.user_agent`, `http.response.status_code`, `url` ou `http.response.body` (si inspection profonde activée).

### Étapes

- Commande `wget` ou `curl` pour simuler le téléchargement.

### 📸 Collecte de preuves

- Extrait JSON de `/var/log/~~Suricata~~/eve.json`.
- **Kibana Discover** : Affichage des champs `user_agent` et `url`.

<br/>
<br/>

## Collecte de preuves

- **Logs** : Extraire les événements pertinents de `/var/log/~~Suricata~~/eve.json` et `/var/log/auth.log`.
- **Exports** : Exporter les résultats Kibana en CSV/JSON.
- **Captures d’écran** : Inclure les vues Discover et dashboards Kibana pour chaque scénario.

<br/>
<br/>

## 📊 Requêtes KQL utiles et création de dashboard Kibana
- **Exemple KQL** :
  - Injection SQL : `event_type:alert AND http.url:*OR*`
  - Scan de ports : `event_type:flow AND network.transport:tcp AND dest.port:*`
  - Brute force SSH : `event.dataset:system.auth AND message:*fail*`

<br/>

- **Dashboard Kibana** :

  - Créer un dashboard avec :
    - Histogramme temporel pour pics d’activité (scénarios 2, 3).
    - Tableaux pour événements HTTP (scénario 5).
    - Graphique de corrélation pour SSH (scénario 4).
  - Sauvegarder sous : "SIEM-Lab-Demo-Dashboard".
