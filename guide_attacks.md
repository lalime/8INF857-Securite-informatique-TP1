# Guide pratique — Scénarios d’attaques et preuves (pour SIEM‑Lab)

> **Important — sécurité / éthique**
> Tous les tests ci-dessous doivent être exécutés **uniquement** dans la VM de laboratoire (ou sur un réseau privé contrôlé). **NE lanceZ RIEN** sur Internet public ni sur des machines que vous ne possédez pas.

---

## Sommaire

* [Scénario 1 — Injection SQL (démo)]
* [Scénario 2 — Scan de ports (Nmap)]
* [Scénario 3 — DoS léger (hping3)]
* [Scénario 4 — Brute force SSH (auth logs)]
* [Scénario 5 — Téléchargement suspect (HTTP)]

---


## 1) Scénario 1 — Injection SQL (démo)

### Objectif

Détecter une tentative d'injection SQL et collecter les preuves issues de Suricata + Kibana.

### Préconditions

* Serveur HTTP local (ex. `python -m http.server` ou DVWA/bWAPP).
* Suricata configuré en mode inspection HTTP.

### Payload (exemple)

```bash
curl -i 'http://127.0.0.1/?q=1%27%20OR%20%271%27=%271'
```

### Requête KQL (Kibana Discover)

```kql
event_type: "alert" and suricata.alert.signature: "SQL" or http.request.body: "OR 1=1"
```

*(Adaptez la KQL selon vos signatures Suricata.)*

### Extraits à inclure dans le guide

* Capture du `curl` dans le terminal (screencap).
* JSON extrait `/var/log/suricata/eve.json` (partie pertinente).
* Capture d’écran Kibana Discover montrant l’événement (`url`, `http.request.body`, `suricata.alert.signature`).

### Exemple d’extrait JSON (format simplifié)

```json
{
  "timestamp": "2025-10-01T12:34:56.000Z",
  "event_type": "http",
  "http": {
    "request": {
      "method": "GET",
      "url": "/?q=1'%20OR%20'1'='1",
      "body": "q=1'%20OR%20'1'='1"
    }
  },
  "suricata": {
    "alert": {
      "signature": "SQL Injection attempt",
      "severity": 2
    }
  }
}
```

---

## 2) Scénario 2 — Scan de ports (Nmap)

### Objectif

Générer un SYN scan et observer les `flow`/`alert` produits par Suricata.

### Commande (exemple)

```bash
nmap -sS -p 1-1000 192.168.56.101
```

### Ce que Suricata doit produire

* `event_type: "alert"` avec une signature contenant `scan` (si règles IDS présentes).
* Plusieurs événements `flow` avec `network.transport: "tcp"` et différents `dest.port`.

### KQL d’exemple

```kql
event_type: "alert" and suricata.alert.signature: *scan*

# ou pour flows
event_type: "flow" and network.transport: "tcp" and dest.port: *
```

### Visualisation

* Histogramme temporel dans Discover montrant un pic pendant le scan.

---

## 3) Scénario 3 — DoS léger (hping3) — **LIMITÉ**

> **ATTENTION :** DoS peut rendre la VM instable. Limitez fortement l’intensité (ex. 200 paquets). N’exécutez jamais `--flood` sur un réseau partagé.

### Commande (exemple très limitée)

```bash
hping3 -S -p 80 --count 200 192.168.56.101
```

### Ce que Suricata doit produire

* Alerts pour flood / grand nombre de `flow` events.
* Pic temporel visible dans Kibana (time series).

---

## 4) Scénario 4 — Brute force SSH (auth logs)

### Objectif

Corréler les tentatives d’authentification SSH (`/var/log/auth.log`) avec les flux réseau détectés par Suricata.

### Commande d’attaque (ex. hydra - TEST LOCAL uniquement)

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.56.101 -t 4
```

*(N’utilisez hydra que dans un environnement contrôlé)*

### Ce qu’il faut collecter

* Extraits de `/var/log/auth.log` montrant les échecs (`sshd`).
* Suricata : connexions répétées TCP vers port `22` (flows/alerts).
* Kibana Discover : `event.dataset: "system.auth"` filtrant les `failure`.

### KQL d’exemple

```kql
event.dataset: "system.auth" and message: "Failed password"

# corrélation réseau
network.transport: "tcp" and dest.port: 22
```

---

## 5) Scénario 5 — Téléchargement suspect (HTTP)

### Objectif

Simuler un téléchargement avec un `User-Agent` suspect pour générer un événement HTTP Suricata.

### Commande (exemple)

```bash
curl -A "BadBot/1.0" -O http://192.168.56.101/malware.exe

# ou wget
wget --user-agent="BadBot/1.0" http://192.168.56.101/malware.exe
```

### Attendus Suricata

* `event_type: "http"` avec `http.request.headers.user_agent`, `http.response.status_code`, `url` ou `http.response.body` si deep inspection.

### À collecter

* Commande `curl/wget` et sa sortie.
* Extrait JSON dans `/var/log/suricata/eve.json`.
* Capture Kibana Discover montrant `user_agent` et `url`.

---

## Collecte de preuves

* **Exports** : JSON/CSV depuis Discover (bouton Export).
* **Captures d’écran** : Discover, Visualize, Dashboard.
* **Logs bruts** : extraits pertinents de `/var/log/suricata/eve.json` et `/var/log/auth.log`.
* **Pcap** : `tcpdump -w capture.pcap` si besoin pour corrélation réseau.

Exemple :

```bash
sudo tcpdump -i any host 192.168.56.101 -w /tmp/siemlab_capture.pcap
```

