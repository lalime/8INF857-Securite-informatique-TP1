# Installer syslog-ng

Syslog-ng est un démon open source de gestion des journaux, fournissant l'implémentation du protocole Syslog pour les systèmes Unix et similaires. Vous pouvez installer Syslog-ng pour la gestion des journaux sources. Consultez [la section Source Syslog Cloud](/help/docs/send-data/hosted-collectors/cloud-syslog-source/) pour plus d'informations sur la configuration d'une source Syslog Cloud pour Syslog-ng.

**Vérifiez la version du système d'exploitation sur Système :**

```
$ lsb_release -aNo 
LSB modules are available.
Distributor ID:    Ubuntu
Description:    Ubuntu 18.04.2 LTS
Release:    18.04
Codename:    bionic
```

**Installer syslog-ng sur Ubuntu :**

```
$ sudo apt-get install syslog-ng -y
```

ou 

```
$ apt-get install syslog-ng syslog-ng-core
```

**Installer en utilisant yum :**

```
$ yum install syslog-ng
```

**Installer à l'aide d'Amazon EC2 Linux :**

Supprimez le rsyslog fourni avec EC2, puis installez syslog-ng.

```
$ sudo rpm -e --nodeps rsyslog$ sudo yum install --enablerepo=epel syslog-ng$ sudo yum install --enablerepo=epel syslog-ng-libdbi$ sudo /etc/init.d/syslog-ng start
```

**Vérifiez la version installée de syslog-ng :**

```
$ syslog-ng --versionsyslog-ng 3 (3.13.2)Config version: 3.13
```

**Assurez-vous que votre serveur syslog-ng fonctionne correctement :**

Ces commandes devraient renvoyer des messages de réussite. 

```
$ service syslog-ng status$ sudo systemctl status syslog-ng.service$ journalctl -xe
```

## Dépannage[​](#troubleshooting "Direct link to Troubleshooting")

Si vous recevez le message d'erreur **Impossible de localiser le paquet syslog-ng**  lors de l'installation d'un serveur syslog-ng sur Ubuntu, exécutez les commandes suivantes, puis essayez à nouveau d'installer syslog-ng.

```
$ sudo apt update$ sudo apt upgrade
```