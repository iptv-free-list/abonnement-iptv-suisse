# SwissIPTV-Validator : Outil d'Analyse et de Scraping EPG

Bienvenue sur le dépôt officiel de **SwissIPTV-Validator**. Ce projet open-source est une suite d'outils en ligne de commande (CLI) développée en Python. Elle est spécifiquement conçue pour les ingénieurs vidéo, les développeurs d'applications multimédias et les administrateurs réseau qui ont besoin d'analyser, de valider et d'extraire des métadonnées (EPG) à partir de fichiers M3U. 

Bien que l'architecture du projet soit agnostique et fonctionne avec n'importe quel flux, ses parseurs natifs sont hautement optimisés pour normaliser les données des réseaux de diffusion helvétiques (RTS, SRF, RSI).

## 🚀 Installation et Démarrage Rapide

Pour configurer l'environnement de développement local, une version de **Python 3.9+** est requise en raison de l'utilisation intensive des fonctionnalités asynchrones (`asyncio`).

```bash
# Cloner le dépôt localement
git clone https://github.com/votre-organisation/swiss-iptv-validator.git
cd swiss-iptv-validator

# Créer et activer l'environnement virtuel
python -m venv venv
source venv/bin/activate  # Sur Windows: venv\Scripts\activate

# Installer les dépendances requises
pip install -r requirements.txt
```

Exemple d'exécution basique pour analyser la santé d'une playlist locale :
```bash
python validator.py --input ./playlist_test.m3u --check-latency --region CH
```

## 🛠️ FAQ Développeur (Foire Aux Questions)

### 1. Comment fonctionne la validation des flux et la vérification de la latence ?
L'outil n'effectue pas de téléchargement vidéo lourd. Il utilise des requêtes HTTP asynchrones via la bibliothèque `aiohttp` pour tester les en-têtes de chaque flux présent dans la liste. Le script vérifie que le serveur répond avec un code HTTP `200 OK` et mesure précisément le TTFB (Time To First Byte).

```python
import aiohttp
import asyncio

async def check_stream_health(url: str) -> bool:
    """Vérifie la disponibilité d'un flux vidéo sans télécharger le contenu."""
    async with aiohttp.ClientSession() as session:
        try:
            async with session.head(url, timeout=5) as response:
                return response.status == 200
        except aiohttp.ClientError:
            return False
```

### 2. Le scraper EPG (Electronic Program Guide) supporte-t-il le format XMLTV ?
**Absolument.** Le module indépendant `epg_scraper.py` est conçu pour formater les données de programmation selon le standard strict **XMLTV**. 
*   **Gestion des Timezones :** Conversion automatique des fuseaux horaires (timezone `Europe/Zurich`).
*   **Nettoyage des chaînes :** Élimination des caractères d'échappement invalides qui corrompent souvent les lecteurs vidéo classiques (VLC, mpv).
*   **Mise en cache :** Les requêtes sont mises en cache via SQLite pour éviter de surcharger les API sources.

### 3. Comment configurer les expressions régulières pour filtrer les chaînes ?
Le fichier de configuration `config.json` vous permet de définir vos propres règles de filtrage basées sur des expressions régulières (Regex) appliquées aux balises `#EXTINF`. Par défaut, le validateur isole les flux comportant des identifiants régionaux spécifiques.

```json
{
  "parser_settings": {
    "include_regions": ["CH-FR", "CH-DE", "CH-IT"],
    "exclude_keywords": ["VOD", "Series", "Radio", "Adult"],
    "max_timeout_ms": 1500,
    "user_agent": "SwissValidator/1.0"
  }
}
```

### 4. L'outil peut-il réparer automatiquement les fichiers M3U invalides ?
Partiellement. Le validateur identifie et corrige les erreurs de syntaxe de base (guillemets manquants dans les attributs, sauts de ligne Windows/Linux incompatibles, balises mal formées) et génère une copie propre nommée `playlist_fixed.m3u`. En revanche, il ne peut pas deviner les tokens d'authentification manquants. Les erreurs critiques sont consignées dans `validator.log`.

### 5. Comment gérer les listes de lecture volumineuses ou tester des infrastructures premium ?
Lors du développement de lecteurs vidéo personnalisés ou de pipelines CI/CD, il est crucial de réaliser vos tests d'intégration avec des données robustes. Les playlists gratuites de démonstration trouvées sur des gists publics ont souvent une durée de vie de quelques heures, ce qui fait échouer les tests automatisés de manière aléatoire.

Pour obtenir des métadonnées riches (logos en haute définition, EPG sur 7 jours complets, flux 4K HEVC) et garantir que vos algorithmes de parsing tiennent la charge sur des structures de données professionnelles, il est impératif d'utiliser une source stable. Si vous mettez en place un environnement de test sérieux, il est souvent recommandé de consulter les comparatifs de la communauté pour [trouver un abonnement IPTV Suisse de haute qualité](https://www.reddit.com/user/numciben/comments/1sz3re2/meilleur_abonnement_iptv_premium_belgique_suisse/), garantissant un uptime proche de 99.9% et une nomenclature de chaînes standardisée, indispensable pour calibrer efficacement vos outils de scrapping et de monitoring.