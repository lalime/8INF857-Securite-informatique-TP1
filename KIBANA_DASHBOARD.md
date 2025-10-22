[⬅ Retour à l'accueil](README.md)
<br/>

## 📊 Étape 4 : Création d’un Tableau de Bord dans Kibana


### ✅ Prérequis

* Avoir accès à Kibana via un navigateur (ex. : [http://localhost:5601](http://localhost:5601))
* Avoir des données indexées dans Elasticsearch
* Avoir créé un index pattern dans **Stack Management > Index Patterns**

---

### 1. 🔍 Créer une visualisation


1. Accédez à **Kibana > Visualize Library**

![Kibana Menu gauche](./docs/step-4/ev-0.png "Kibana Menu gauche")

2. Cliquez sur **"Create visualization"**

![Visualize Library](./docs/step-4/ev-4.png "Visualize Library")

3. Choisissez un type de visualisation :
   - 📈 Line, Bar, Pie, Heat map, etc.
   - 📋 **Data Table** (pour affichage en colonnes)

4. Sélectionnez l’**Index pattern** (ex. : `syslog-ng`)

![Choisir visualisation](./docs/step-4/ev-11.png "Choisir visualisation")

5. Configurez la visualisation :
   - Ajoutez une métrique (ex. : Count)
   - Ajoutez une **Split rows** ou **Split columns** (par ex. `host`, `program`, etc.)

6. Cliquez sur **"Save and return"**

![Enregistrer la visualisation](./docs/step-4/ev-5.png "Enregistrer la visualisation")

<br/>

---

### 2. 🧩 Créer un nouveau tableau de bord

1. Allez dans **Kibana > Dashboard**

![Visualize Library](./docs/step-4/ev-12.png "Visualize Library")

2. Cliquez sur **"Create new dashboard"**

3. Cliquez sur **"Add from library"**

![Ajouter de la bibliotheque](./docs/step-4/ev-13.png "Ajouter de la bibliotheque")

4. Sélectionnez les visualisations que vous avez créées

![Sélectionner visualisation](./docs/step-4/ev-14.png "Sélectionner visualisation")

5. Réorganisez les éléments par glisser-déposer

6. Cliquez sur **"Save"**
   - Donnez un nom au tableau de bord (ex. : `Tableau de bord Syslog`)
   - Cochez **"Store time with dashboard"** si vous voulez sauvegarder l'intervalle de temps

![Enregistrer le tableau de bord](./docs/step-4/ev-9.png "Enregistrer le tableau de bord")


---

### 3. ⏱️ Ajuster la période de temps


1. En haut à droite, cliquez sur le sélecteur de dates
2. Choisissez une période (ex. : "Last 15 minutes", "Today", "This month", etc.)
3. Rafraîchissez automatiquement les données si besoin (Auto-refresh)

![Ajuster la periode](./docs/step-4/ev-10.png "Ajuster la periode")



---

### 4. 🧪 Tester et partager

1. Vérifiez que les visualisations affichent bien les données attendues
2. Cliquez sur **"Share"** pour copier un lien public ou intégrer dans une iframe
3. Vous pouvez exporter le dashboard au format JSON (via API ou Dev Tools)

---

## 💡 Astuces


- Utilisez des filtres globaux pour affiner les résultats (champ `program`, `host`, `severity`, etc.)
- Ajoutez un **Search bar** pour filtrer dynamiquement par mots-clés
- Combinez plusieurs types de graphiques (camembert, histogramme, data table)


---

<br/><br/>
[⬅ Retour à l'accueil](README.md)
