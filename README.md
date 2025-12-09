# Artnet Smart Artist Scraper

Scraper **Python** conçu pour enrichir une liste d’artistes Artnet à partir d’un fichier CSV d’URLs.  
Le script extrait automatiquement :

- **Auction Results** : nombre de résultats d’enchères par artiste
- **Objects / Mediums** : types d’objets listés dans les facets
- **Biography Timeline** : timeline d’expositions (page `/biography`)
- **Gestion des blocages** : détection 403 / challenge / structure cassée
- **Reprise automatique** : sauvegarde de progression dans un fichier d’état
- **Retry CSV** : export des artistes bloqués pour relance ultérieure

Le tout avec un **rythme de navigation “humain”** (jitter, micro-pauses, sessions par thread).

---

## ✨ Fonctionnalités

- **Multi-threading léger** (par défaut 2 workers) pour rester discret
- **Session cloudscraper par thread**  
  → cookies isolés, rotations de plateformes/navigateurs
- **Anti-pattern bot** :
  - délais aléatoires entre requêtes
  - micro-pauses “café” rares mais longues
  - pauses de groupe entre batches
- **Sauvegarde progressive** :
  - `output_artnet_smart.csv` alimenté au fil de l’eau
  - `last_position.txt` pour reprise
- **Fail-safe blocage** :
  - export dans `artists_to_retry.csv`
  - arrêt automatique après 20 blocages consécutifs

---

## ✅ Pré-requis

Python 3.9+ recommandé.

Dépendances :

```bash
pip install pandas cloudscraper beautifulsoup4 urllib3
📂 Fichiers attendus / générés
Entrée
input.csv

contient au minimum une colonne avec des URLs Artnet

le script détecte automatiquement la colonne URL

Exemple minimal :

csv
Copier le code
artist_name,url
Artist A,https://www.artnet.com/artists/artist-a
Artist B,https://www.artnet.com/artists/artist-b
Sorties
output_artnet_smart.csv
→ fichier enrichi avec 3 colonnes :

Scrap_Auctions

Scrap_Objects

Scrap_Timeline

artists_to_retry.csv
→ lignes bloquées à relancer plus tard

last_position.txt
→ index de la dernière position sauvegardée

🚀 Utilisation
Mets ton fichier input.csv à la racine du repo

Lance :

bash
Copier le code
python artnet_scraper.py
Le script :

reprend automatiquement si un last_position.txt existe

saute les artistes déjà présents dans output_artnet_smart.csv

⚙️ Configuration
Dans le script :

python
Copier le code
INPUT_FILE = 'input.csv'
OUTPUT_FILE = 'output_artnet_smart.csv'
RETRY_FILE = 'artists_to_retry.csv'
FICHIER_ETAT = 'last_position.txt'
SEP = ','
Mode intelligent :

python
Copier le code
MAX_WORKERS = 2   # nombre de threads
BATCH_SIZE = 20   # taille d’un batch avant sauvegarde CSV
Reprise manuelle (si tu veux forcer un départ) :

python
Copier le code
FORCE_START_INDEX = 0
Note : si last_position.txt existe, il écrase ce paramètre.

🧠 Stratégie anti-blocage (détails)
Chaque artiste suit ce pattern :

Pause aléatoire 0.5–3.5s avant requête

Micro-pause rare (5%) 8–15s

Pause courte entre page principale et bio

Pause de groupe en fin de batch (30% de chance)

Détection “block” si :

status code 403

présence de "security"/"challenge"

structure HTML inattendue (pas de <h1>)

🧯 Comportement en cas de blocage massif
Si le script détecte 20 blocages d’affilée :

export immédiat du dernier index dans last_position.txt

arrêt brutal (os._exit(1)) pour éviter le ban IP

tu peux relancer plus tard, il reprendra automatiquement

📌 Limitations connues
Le scraping dépend de la structure actuelle d’Artnet, donc peut casser si le site change.

Les pages bio peuvent être plus souvent protégées : dans ce cas on garde le reste des données.

Le script privilégie la discrétion à la vitesse.

⚠️ Notes légales / éthiques
Ce script est fourni à des fins d’analyse de données et de recherche.
Assure-toi de respecter :

les CGU du site

les contraintes légales locales

un usage raisonnable (ne pas surcharger les serveurs)

🛠️ Idées d’amélioration
Ajout d’une rotation de proxies

Backoff exponentiel au lieu d’un arrêt brutal

Export JSON optionnel

Logging structuré (niveau DEBUG/INFO)

