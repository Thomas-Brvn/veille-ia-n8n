# Veille IA - Agrégateur intelligent de flux RSS

Agrégateur de veille technologique qui résume automatiquement les articles de plusieurs sources RSS grâce à **Llama 3.2**, orchestré par **n8n**.

## Architecture

```
┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│  Hacker News   │     │  TechCrunch    │     │  Ars Technica  │
│     (RSS)      │     │     (RSS)      │     │     (RSS)      │
└───────┬────────┘     └───────┬────────┘     └───────┬────────┘
        │                      │                      │
        └──────────────────────┼──────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │       n8n           │
                    │  (toutes les heures)│
                    │  Déduplique + filtre│
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Ollama            │
                    │   Llama 3.2         │
                    │   Résume en français│
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   summaries.json    │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Frontend (nginx)  │
                    │   localhost:8080     │
                    └─────────────────────┘
```

## Stack technique

| Composant | Rôle |
|-----------|------|
| **n8n** | Orchestre la collecte RSS et le résumé toutes les heures |
| **Ollama + Llama 3.2** | Résume chaque article en 2-3 phrases en français |
| **Nginx** | Sert la page web du dashboard |

## Prérequis

- [Docker](https://docs.docker.com/get-docker/) et Docker Compose
- ~4 Go de RAM disponible (pour Llama 3.2)

## Installation

### 1. Cloner le repo

```bash
git clone https://github.com/<ton-username>/veille-ia-n8n.git
cd veille-ia-n8n
```

### 2. Configurer l'environnement

```bash
cp .env.example .env
```

### 3. Lancer les services

```bash
docker compose up -d
```

### 4. Télécharger Llama 3.2

```bash
docker exec ollama ollama pull llama3.2
```

### 5. Importer le workflow dans n8n

1. Ouvrir http://localhost:5678
2. Créer un compte
3. **Workflows > Import from File** > sélectionner `workflows/01_rss_summarizer.json`
4. Activer le workflow (toggle en haut à droite)

### 6. Voir les résumés

Ouvrir http://localhost:8080

Le dashboard se met à jour automatiquement toutes les minutes.

## Personnaliser les sources RSS

Pour ajouter ou modifier les flux RSS, édite le workflow dans n8n :
- Ajoute un node **RSS Feed Read** avec l'URL du flux
- Connecte-le au node **Dédupliquer et limiter**

Exemples de flux :
- `https://www.theverge.com/rss/index.xml`
- `https://www.wired.com/feed/rss`
- `https://blog.google/rss/`
- `https://openai.com/blog/rss.xml`

## Structure du projet

```
.
├── docker-compose.yml              # n8n + Ollama + Nginx
├── .env.example                    # Variables de configuration
├── workflows/
│   └── 01_rss_summarizer.json      # Workflow n8n principal
├── frontend/
│   └── index.html                  # Dashboard des résumés
├── data/                           # Résumés générés (non versionné)
└── README.md
```

## Licence

MIT
