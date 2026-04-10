# 🏛️ CivicAI — Assistant Administratif Thaïlande

Assistant conversationnel basé sur Claude (Anthropic) pour aider les citoyens et expatriés à naviguer les démarches administratives en Thaïlande.

## Demo

![CivicAI Interface](static/demo.png)

---

## Stack technique

| Couche | Technologie |
|---|---|
| LLM | Claude Sonnet (Anthropic) |
| Agent | LangGraph |
| RAG | ChromaDB + Sentence Transformers |
| Recherche web | Tavily API |
| Backend | FastAPI + Uvicorn |
| Frontend | HTML/CSS/JS vanilla |
| Déploiement | Docker + Docker Compose |

---

## Architecture

```
Navigateur (interface web)
        ↓ POST /chat
FastAPI (api.py)
        ↓
Agent LangGraph (agent.py)
        ├── search_docs  → ChromaDB (RAG sur docs administratifs)
        └── web_search   → Tavily (infos récentes)
```

### Logique de routing

1. L'agent cherche **toujours dans les docs locaux** (`search_docs`) en premier
2. Si le score de similarité moyen est < 0.5 → bascule sur `web_search`
3. L'historique conversationnel est maintenu côté client et envoyé à chaque requête

---

## Fonctionnalités

- **RAG hybride** — base de connaissance locale + recherche web en fallback
- **Score de confiance** — détection automatique des questions hors-domaine
- **Historique conversationnel** — Claude se souvient du contexte de la conversation
- **Interface web** — chat avec suggestions de questions
- **Déployable** — Dockerfile multi-stage optimisé

---

## Installation locale

### Prérequis

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) — gestionnaire de packages
- Clés API : [Anthropic](https://console.anthropic.com) + [Tavily](https://tavily.com)

### Setup

```bash
git clone https://github.com/AlexMo-1205/civicai.git
cd civicai

# Installe les dépendances
uv sync

# Configure les variables d'environnement
cp .env.example .env
# Édite .env avec tes clés API

# Génère la base vectorielle
uv run python ingest.py

# Lance le serveur
uv run uvicorn api:app --reload
```

Ouvre [http://localhost:8000](http://localhost:8000)

---

## Déploiement Docker

```bash
# Build et lance
docker compose up --build

# Arrête
docker compose down
```

L'app est accessible sur [http://localhost:8000](http://localhost:8000).

La base vectorielle ChromaDB est persistée via un volume Docker — elle n'est pas régénérée à chaque redémarrage.

---

## Structure du projet

```
civicai/
├── docs/                    # Documents administratifs
│   ├── visa_touriste.txt
│   ├── permis_travail.txt
│   └── carte_residence.txt
├── static/
│   └── index.html           # Interface web
├── agent.py                 # Agent LangGraph
├── api.py                   # Backend FastAPI
├── ingest.py                # Ingestion RAG → ChromaDB
├── Dockerfile               # Multi-stage build
├── docker-compose.yml
├── pyproject.toml
└── .env.example
```

---

## Ajouter des documents

1. Ajoute tes fichiers `.txt` dans `docs/`
2. Relance l'ingestion :

```bash
uv run python ingest.py
# ou dans Docker :
docker compose up --build
```

---

## Variables d'environnement

Crée un fichier `.env` à partir de `.env.example` :

```
ANTHROPIC_API_KEY=sk-ant-...
TAVILY_API_KEY=tvly-...
```

---

## Choix techniques

**Pourquoi LangGraph plutôt que LangChain ?**
LangGraph modélise l'agent comme un graphe d'états explicite — les transitions sont lisibles, debuggables et extensibles sans modifier la logique existante.

**Pourquoi ChromaDB ?**
Base vectorielle locale, zéro infrastructure. Parfait pour un MVP — migrable vers pgvector ou Pinecone en prod sans changer le code d'ingestion.

**Pourquoi un score de confiance ?**
Sans seuil de pertinence, l'agent peut synthétiser des chunks non pertinents et halluciner une réponse confiante. Le fallback automatique vers `web_search` quand le score < 0.5 garantit une réponse toujours sourcée.

**Pourquoi du JS vanilla pour le frontend ?**
Zéro dépendance, zéro build step. L'interface est simple et déployable statiquement.

---

## Roadmap

- [ ] Support PDF en plus des fichiers txt
- [ ] Streaming des réponses (WebSockets)
- [ ] Déploiement GCP Cloud Run
- [ ] Support multilingue (thaï, anglais, français)
- [ ] Authentification utilisateur

---

## Auteur

**Alex Mo** — [@AlexMo-1205](https://github.com/AlexMo-1205)

Data Scientist | Bangkok, Thaïlande
