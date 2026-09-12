# 🗺️ Dashboard NC – Communes · Parcelles · Foncier

![HTML](https://img.shields.io/badge/HTML5-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Leaflet](https://img.shields.io/badge/Leaflet-199900?style=for-the-badge&logo=leaflet&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-yellow?style=for-the-badge)

> **Carte interactive du foncier et du cadastre de Nouvelle-Calédonie**

Application web monopage (SPA) permettant de visualiser la **répartition foncière par commune** (choroplèthe) et les **parcelles cadastrales** de Nouvelle-Calédonie, avec filtres par typologie de propriété et multiples sources de données (locale, API data.gouv.nc, release GitHub).

---

## 📋 Table des matières

- [Aperçu](#-aperçu)
- [Fonctionnalités](#-fonctionnalités)
- [Structure des données](#-structure-des-données)
- [Installation](#-installation)
- [Utilisation](#-utilisation)
- [Technologies](#-technologies)
- [Captures d'écran](#-captures-décran)
- [Contribuer](#-contribuer)
- [Licence](#-licence)

---

## 🎯 Aperçu

Ce dashboard cartographique offre une vision complète du foncier calédonien. Il combine :

- 🗺️ **Deux fonds de carte** : OpenStreetMap + Plan Cadastral WMS (Gouvernement NC, superposable)
- 📊 **Carte choroplèthe des communes** selon 4 indicateurs fonciers :
  - 🏠 Propriété privée (ha)
  - 🏛️ Propriété communale (ha)
  - 🏢 Domaine de l'État (ha)
  - 🌿 Terre coutumière (ha)
- 📍 **Parcelles cadastrales** avec clustering et coloration par typologie
- 🎛️ **Filtre par typologie** : Privé, Collectivité, Terre coutumière, Mixte, Non renseigné
- 🔄 **Chargement multi-source** avec fallback automatique :
  1. Fichier local (`foncier.json`, `communes-nc.geojson`, `parcelles-cadastrales-nc.geojson`)
  2. API `data.gouv.nc`
  3. Release GitHub via proxy CORS
  4. Branche `main` via jsDelivr

---

## ✨ Fonctionnalités

### 📊 Carte choroplèthe des communes
- **Coloration dégradée** selon la valeur de l'indicateur sélectionné
- **Tooltip** au survol : nom de la commune + valeur en hectares
- **Popup** détaillé : les 4 indicateurs fonciers (Privé, Communal, État, Terre coutumière)
- **Légende dynamique** avec 5 paliers de couleur

### 📍 Parcelles cadastrales
- Clustering (Leaflet.markercluster) pour de gros volumes
- **Coloration par typologie** :

| Typologie | Couleur | Libellé |
|-----------|---------|---------|
| 🏠 PRIVE | 🟢 `#2e7d32` | Privé |
| 🏛️ COLLECTIVITE | 🔵 `#1565c0` | Collectivité |
| 🌿 TERRE COUTUMIERE | 🟣 `#6a1b9a` | Terre coutumière |
| 🔀 MIXTE | 🟠 `#f57f17` | Mixte |
| ❓ NON RENSEIGNE | ⚪ `#9e9e9e` | Non renseigné |

### 🔍 Popups détaillés (parcelles)
| Champ | Description |
|-------|-------------|
| 🏛️ Commune | Nom de la commune |
| 📍 NIC | Identifiant NIC |
| 📋 Lot | Numéro de lot |
| 📐 Section | Section cadastrale |
| 📏 Surface | Surface (convertie en hectares) |
| 🏷️ Typologie | Badge coloré selon la nature |

### 🔄 Chargement robuste multi-source

Le script tente **dans l'ordre** :

1. **Fichiers locaux** (pour le développement) :
   - `foncier.json`
   - `communes-nc.geojson`
   - `parcelles-cadastrales-nc.geojson`

2. **API `data.gouv.nc`** via proxy CORS (`allorigins`, `corsproxy.io`, `thingproxy`) :
   - Dataset `repartition-fonciere-par-nature-de-proprietaire`
   - Dataset `communes-nc` (export GeoJSON)

3. **Release GitHub** (`nc-cadastre`) via proxy `allorigins`

4. **Branche `main`** via **jsDelivr CDN**

> 💡 Si toutes les sources échouent, un GeoJSON vide est retourné pour éviter de bloquer l'application.

---

## 📂 Structure des données

```
📁 repository/
├── index.html
├── README.md
├── LICENSE
├── serve.sh
├── foncier.json                        # (optionnel) Données foncières locales
├── communes-nc.geojson                 # (optionnel) Contours communaux locaux
├── parcelles-cadastrales-nc.geojson    # (optionnel) Parcelles locales
└── 📁 data/                            # (optionnel) Données additionnelles
```

### Format attendu

**`foncier.json`** (répartition foncière par nature de propriétaire) :
```json
{
  "results": [
    {
      "type_zone": "Commune",
      "annee": "2025",
      "zone": "Nouméa",
      "proprietaire_nature": "PRIVE",
      "superficie_ha": "1234.5"
    }
  ]
}
```

**`communes-nc.geojson`** :
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": { "type": "Polygon", "coordinates": [[[165.5, -21.5], "..."]] },
      "properties": {
        "nom_commune": "Nouméa"
      }
    }
  ]
}
```

**`parcelles-cadastrales-nc.geojson`** :
```json
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": { "type": "Point", "coordinates": [165.5, -21.5] },
      "properties": {
        "commune": "Nouméa",
        "typologie": "PRIVE",
        "nic": "NC000000A0123",
        "num_lot": "123",
        "section_cadastrale": "AB",
        "surface_cadastrale": "45ha 20a 00ca"
      }
    }
  ]
}
```

### Propriétés reconnues (variantes acceptées)

| Champ affiché | Propriétés reconnues |
|---------------|----------------------|
| Nom commune | `nom_commune`, `nom`, `NAME`, `commune` |
| Typologie | `typologie`, `nature`, `proprietaire_nature`, `type` |
| NIC | `nic`, `id` |
| Lot | `num_lot`, `numero`, `lot` |
| Section | `section_cadastrale`, `section` |
| Surface | `surface_cadastrale`, `contenance` |

---

## 🚀 Installation

### Prérequis
- Un navigateur moderne (Chrome, Firefox, Edge, Safari)
- Un serveur HTTP local pour éviter les restrictions CORS sur `fetch()`

### Installation rapide (script Bash)

```bash
# 1. Cloner le dépôt
git clone https://github.com/gunout/dashboard-nc-cpf.git
cd dashboard-nc-cpf

# 2. Lancer le serveur local
./serve.sh
```

### Installation manuelle

```bash
# Python 3
python3 -m http.server 8000

# Node.js (avec npx)
npx serve . -l 8000

# PHP
php -S localhost:8000
```

Puis ouvrir : **http://localhost:8000**

---

## 🛠️ Script Bash : `serve.sh`

Script utilitaire pour lancer automatiquement le dashboard avec détection du serveur HTTP disponible.

```bash
#!/usr/bin/env bash
# ================================================================
# 🗺️  Dashboard NC - Communes · Parcelles · Foncier
# Script de lancement rapide du serveur local
# ================================================================

set -euo pipefail

# ---------- Configuration ----------
PORT="${PORT:-8000}"
HOST="${HOST:-localhost}"

# ---------- Couleurs ----------
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m'

log_info()  { echo -e "${BLUE}ℹ️  $*${NC}"; }
log_ok()    { echo -e "${GREEN}✅ $*${NC}"; }
log_warn()  { echo -e "${YELLOW}⚠️  $*${NC}"; }
log_error() { echo -e "${RED}❌ $*${NC}" >&2; }

# ---------- Vérification des dépendances ----------
check_command() {
  command -v "$1" &>/dev/null
}

# ---------- Détection du serveur HTTP ----------
detect_server() {
  if check_command python3; then
    echo "python3"
  elif check_command python; then
    echo "python"
  elif check_command npx; then
    echo "npx"
  elif check_command php; then
    echo "php"
  else
    echo ""
  fi
}

# ---------- Vérification des données ----------
check_data() {
  local missing=0
  for f in "foncier.json" "communes-nc.geojson" "parcelles-cadastrales-nc.geojson"; do
    if [[ -f "$f" ]]; then
      log_ok "Fichier local détecté : $f"
    else
      log_warn "Fichier local absent : $f (fallback API/release/jsDelivr)"
      missing=1
    fi
  done
  if [[ "$missing" -eq 0 ]]; then
    log_ok "Tous les fichiers locaux sont présents."
  fi
}

# ---------- Lancement du serveur ----------
start_server() {
  local server="$1"
  log_info "Démarrage du serveur sur http://${HOST}:${PORT}"
  log_info "Appuyez sur Ctrl+C pour arrêter."
  echo

  case "$server" in
    python3|python)
      "$server" -m http.server "$PORT" --bind "$HOST"
      ;;
    npx)
      npx --yes serve . -l "$PORT"
      ;;
    php)
      php -S "${HOST}:${PORT}"
      ;;
    *)
      log_error "Aucun serveur HTTP disponible."
      log_info "Ouvrez 'index.html' directement dans votre navigateur."
      exit 1
      ;;
  esac
}

# ---------- Main ----------
main() {
  echo -e "${BLUE}"
  echo "╔══════════════════════════════════════════════════════════╗"
  echo "║  🗺️  Dashboard NC - Communes · Parcelles · Foncier      ║"
  echo "╚══════════════════════════════════════════════════════════╝"
  echo -e "${NC}"

  if [[ ! -f "index.html" ]]; then
    log_error "Fichier 'index.html' introuvable. Exécutez ce script depuis la racine du projet."
    exit 1
  fi

  check_data

  local server
  server="$(detect_server)"
  if [[ -z "$server" ]]; then
    log_error "Aucun serveur HTTP trouvé (Python, Node, PHP)."
    exit 1
  fi
  log_ok "Serveur détecté : $server"

  start_server "$server"
}

main "$@"
```

### Utilisation

```bash
# Rendre exécutable
chmod +x serve.sh

# Lancer sur le port par défaut (8000)
./serve.sh

# Lancer sur un autre port
PORT=3000 ./serve.sh
```

---

## 🎮 Utilisation

1. **Sélectionner un indicateur** dans le premier menu (Privé, Communal, État, Terre coutumière)
2. La **carte choroplèthe** des communes se met à jour avec un dégradé de couleurs
3. **Survoler une commune** pour voir la valeur de l'indicateur
4. **Cliquer sur une commune** pour voir le détail des 4 indicateurs fonciers
5. **Filtrer les parcelles** par typologie via le second menu
6. **Cliquer sur un marqueur** pour voir les détails de la parcelle (NIC, lot, section, surface, typologie)
7. **Basculer entre les fonds de carte** (OSM / Plan cadastral WMS) via le sélecteur en haut à gauche

---

## 🛠️ Technologies

| Technologie | Usage |
|-------------|-------|
| [Leaflet 1.9.4](https://leafletjs.com/) | Carte interactive |
| [Leaflet.markercluster 1.5.3](https://github.com/Leaflet/Leaflet.markercluster) | Clustering des marqueurs |
| [WMS Cadastre NC](https://carto.gouv.nc/) | Plan cadastral superposable |
| [data.gouv.nc API](https://data.gouv.nc/) | Données foncières et communales |
| [jsDelivr](https://www.jsdelivr.com/) | CDN fallback pour la release |
| [OpenStreetMap](https://www.openstreetmap.org/) | Fond de carte principal |

---

## 📸 Captures d'écran

> *Ajoutez ici vos captures d'écran du dashboard*

```
📷 [Capture 1 : Carte choroplèthe des communes de NC]
📷 [Capture 2 : Zoom sur parcelles colorées par typologie]
📷 [Capture 3 : Popup détaillé d'une commune avec 4 indicateurs fonciers]
```

---

## 🤝 Contribuer

Les contributions sont les bienvenues !

1. Fork le projet
2. Créer une branche (`git checkout -b feature/amelioration`)
3. Commit les changements (`git commit -m 'Ajout fonctionnalité X'`)
4. Push (`git push origin feature/amelioration`)
5. Ouvrir une Pull Request

### Idées d'amélioration
- [ ] Ajout d'un graphique de répartition foncière par commune
- [ ] Comparaison multi-communes
- [ ] Export CSV / GeoJSON des parcelles filtrées
- [ ] Recherche par NIC ou numéro de lot
- [ ] Mode sombre
- [ ] Affichage des polygones de parcelles (et non plus seulement les centroïdes)
- [ ] Ajout d'indicateurs supplémentaires (Superficie totale, Densité, etc.)
- [ ] Cache local (IndexedDB) pour éviter les rechargements

---

## 📄 Licence

Ce projet est distribué sous licence **MIT**. Voir le fichier [LICENSE](LICENSE) pour plus d'informations.

### Fichier `LICENSE` (MIT)

```
MIT License

Copyright (c) 2025 Dashboard NC – Communes · Parcelles · Foncier

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

---

## 🙏 Remerciements

- [Gouvernement de Nouvelle-Calédonie](https://cadastre.gouv.nc/) – Plan cadastral WMS
- [data.gouv.nc](https://data.gouv.nc/) – Données ouvertes foncières et communales
- [IGN](https://www.ign.fr/) – Référentiels géographiques
- [OpenStreetMap](https://www.openstreetmap.org/) – Fond de carte
- Communauté Leaflet

---

<p align="center">
  <strong>🗺️ Fait avec ❤️ pour la Nouvelle-Calédonie</strong><br>
  <em>Dashboard NC – 2025</em>
</p>


# Liens Website :

    https://gunout.github.io/dashboard-nc-cpf/


# EXAMPLE

<img width="1807" height="761" alt="Screenshot 2026-09-03 at 08-00-35 🗺️ Dashboard NC – Communes · Parcelles · Foncier" src="https://github.com/user-attachments/assets/9da593b4-b605-4e30-a939-5220610b2b9c" />

---

<div align="center">

### 🇫🇷 Gunout · 2026

![Made in France](https://img.shields.io/badge/Made_in-France-002395?style=flat-square&labelColor=FFFFFF&logo=data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHZpZXdCb3g9IjAgMCA5MDAgNjAwIj48cmVjdCB3aWR0aD0iOTAwIiBoZWlnaHQ9IjYwMCIgZmlsbD0iIzAwMjM5NSIvPjxyZWN0IHdpZHRoPSI5MDAiIGhlaWdodD0iNDAwIiB5PSIxMDAiIGZpbGw9IiNmZmYiLz48cmVjdCB3aWR0aD0iOTAwIiBoZWlnaHQ9IjIwMCIgeT0iNDAwIiBmaWxsPSIjZWQyOTM5Ii8+PC9zdmc+)
![GitHub](https://img.shields.io/badge/GitHub-gunout-181717?style=flat-square&logo=github&logoColor=white)
![Year](https://img.shields.io/badge/2026-ED2939?style=flat-square&labelColor=FFFFFF)

<sub>© 2026 <strong>gunout</strong> — Tous droits réservés.</sub>

</div>
