# 🛡️ Architecture SIEM Légère : Détection d'Anomalies avec IDS, Syslog-NG et Stack ELK

Ce document décrit l'architecture, les outils et la méthodologie pour mettre en place un système de gestion des événements et d'information de sécurité (SIEM) léger. L'objectif est de centraliser les logs de sécurité pour la détection en temps réel des menaces potentielles et l'analyse des anomalies.

## 🎯 Objectif du Projet

Créer un système de détection d'anomalies et de gestion de logs pour identifier les menaces potentielles et améliorer la sécurité des réseaux, en transformant les données brutes en renseignements exploitables.

## 🛠️ Outils et Composants

Le projet est basé sur l'intégration de quatre composants principaux fonctionnant sur une machine virtuelle Linux (Ubuntu/Debian est recommandée).

| Composant | Rôle Principal | Détails |
| :--- | :--- | :--- |
| **IDS/IPS (Snort)** | **Détection d'Anomalies** | Analyse le trafic réseau ou les logs système pour identifier les schémas d'attaque connus ou les comportements anormaux. |
| **syslog-ng** | **Collecteur de Logs (Ingestion)** | Collecte les logs de sécurité (y compris les alertes IDS) et les achemine vers la base de données. |
| **Elasticsearch** | **Base de Données et Moteur de Recherche** | Stocke les logs collectés de manière évolutive et fournit un moteur de recherche rapide pour l'analyse. |
| **Kibana** | **Visualisation et Interface Utilisateur** | Fournit une interface Web pour interroger les données stockées dans Elasticsearch, créer des tableaux de bord et visualiser les anomalies. |

---

## 🏗️ Architecture du Projet (Flux de Données)

Le flux de données est linéaire, allant de la source (équipements réseau/IDS) à la visualisation (Kibana).

1.  **Détection d'anomalies par Snort et collecte des Logs (IDS/Réseau) vers Syslog-NG**
    * L'IDS/IPS (Snort) analyse le trafic ou les logs et génère ses propres **alertes de sécurité**.
    * **syslog-ng** fonctionne comme le point de réception central, capturant tous ces logs et alertes.

2.  **Traitement et Acheminement (Syslog-NG)**
    * `syslog-ng` est configuré avec un **destination driver (extension)** spécifique pour Elasticsearch (généralement via HTTP).
    * Le collecteur peut effectuer un prétraitement léger (parsing) des logs avant l'envoi.

3.  **Stockage et Indexation (Elasticsearch)**
    * Elasticsearch reçoit les logs de `syslog-ng` et les indexe dans un format JSON, les rendant interrogeables en quasi temps réel.

4.  **Analyse et Visualisation (Kibana)**
    * Kibana se connecte à Elasticsearch.
    * L'utilisateur crée des **dashboards** et des **visualisations** pour surveiller les métriques de sécurité (ex. : nombre d'attaques bloquées, tentatives de connexion échouées, détection d'alertes IDS).

### Schéma Conceptuel

<img width="339" height="520" alt="SecInf-IPS" src="https://github.com/user-attachments/assets/ca95d96c-8785-4a0e-bad5-6bd2c14358a9" />

## 📋 Étapes de Mise en Œuvre (Synopsis)

Bien que l'installation détaillée dépasse le cadre de ce document, voici les étapes clés :

### Étape 1 : Préparation de la Machine Linux(INSTALL_CONFIG_ELASTIC_STACK.md)


1.  Installer **Java Runtime Environment (JRE)**, car Elasticsearch en dépend.
2.  Installer la suite complète **Elastic Stack (Elasticsearch, Kibana)**.

### Étape 2 : Installation et Configuration d'Elasticsearch et Kibana(INSTALL_CONFIG_ELASTIC_STACK.md)


1.  **Elasticsearch :** Configurer `elasticsearch.yml` pour la grappe et l'écoute réseau. Démarrer le service.
2.  **Kibana :** Configurer `kibana.yml` pour pointer vers Elasticsearch. Démarrer le service et vérifier l'accès via le navigateur.

### [Étape 3 : Installation et Configuration de syslog-ng](INSTALL_CONFIG_SYSLOG.md)

1.  Installer les modules `syslog-ng` nécessaires, notamment le module **HTTP** ou **Elasticsearch** (si disponible).
2.  Éditer le fichier `/etc/syslog-ng/syslog-ng.conf` pour définir :
    * **Source :** Pour écouter les ports Syslog (UDP/TCP 514) et lire les fichiers logs de l'IDS.
    * **Destination :** Un bloc `destination` pointant vers l'API HTTP d'Elasticsearch.
    * **Log :** Lier les sources aux destinations pour créer le pipeline de logs.
3.  Redémarrer `syslog-ng` : `sudo systemctl restart syslog-ng`.
 


### [Étape 4 : Intégration IDS/IPS (Ex. : Snort)](INSTALL_SNORT.md)

1.  Installer et configurer l'IDS/IPS pour analyser le trafic réseau pertinent.
2.  Configurer l'IDS pour qu'il écrive ses alertes de sécurité dans un fichier log que `syslog-ng` pourra lire, ou qu'il les envoie directement en Syslog.


### [Étape 5 : Création des Tableaux de Bord (Kibana)](KIBANA_DASHBOARD.md)

1.  Dans l'interface Kibana, créer un **Index Pattern** pour interroger les données récemment reçues dans Elasticsearch (ex. : `syslog-*`).
2.  Utiliser l'outil **Visualize** pour construire des graphiques et des métriques basées sur les champs d'alerte IDS ou les logs critiques.
3.  Assembler les visualisations dans un **Dashboard** dédié à la sécurité et aux anomalies.

---

## 📈 Avantages du Projet

| Catégorie | Description |
| :--- | :--- |
| **Sécurité Améliorée** | Détection en temps réel des anomalies, des signatures d'attaques et des schémas de menaces potentielles, permettant une réponse proactive. |
| **Visibilité Totale** | Centralisation des logs de tous les équipements et applications, offrant une vue unifiée de l'état du réseau. |
| **Analyse Efficace** | Utilisation du moteur de recherche puissant d'Elasticsearch pour analyser rapidement des volumes massifs de logs (plus rapide que la simple lecture de fichiers texte). |
| **Gestion Simplifiée** | Réduction des coûts et de la complexité grâce à une plateforme Open Source unique pour la collecte, le stockage et la visualisation des logs de sécurité. |

## Ressources supplémentaires
- Documentation officielle de [syslog-ng](https://www.syslog-ng.com/)
- Documentation officielle de [Snort](https://www.snort.org/)
- Documentation officielle de [Wazuh](https://documentation.wazuh.com/)
- Documentation officielle d'[Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- Documentation officielle de [Kibana](https://www.elastic.co/guide/en/kibana/current/index.html)

## References
- https://letsdefend.io/blog/how-to-install-and-configure-snort-on-ubuntu
- https://www.sumologic.com/help/docs/send-data/hosted-collectors/cloud-syslog-source/install-syslog-ng/
