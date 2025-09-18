# Agent IA de Réponse Automatique pour Redmine
**Proof of Concept (PoC) – Automatisation des réponses aux tickets sans réponse**

---

## 🎯 Objectif du Projet
Développer un **agent IA léger** qui :
- **Détecte** les tickets Redmine sans réponse depuis **24h**.
- **Génère une réponse automatique** en utilisant :
  - La **base de connaissances** (wiki Redmine).
  - Les **réponses existantes** dans des tickets similaires.
- **Publie** la réponse en commentaire sur le ticket concerné.

**Exemple de réponse générée** :
> *"Bonjour, ce ticket n’a pas reçu de réponse depuis 24h. Voici une suggestion basée sur notre documentation et des cas similaires : [réponse IA]. ⚠️ Ce message est automatique. Un humain validera sous peu."*

---

## 🛠 Technologies à Utiliser
| Technologie | Rôle                                                                 |
|-------------|----------------------------------------------------------------------|
| **Python**  | Langage principal pour le script et la logique métier.              |
| **Ollama**  | Génération de réponses via un modèle local (ex: `mistral` ou `llama3`). |
| **Redmine** | Source des données (tickets, wiki) via [l’API REST](https://www.redmine.org/projects/redmine/wiki/Rest_api). |

**Librairies recommandées** :
- `python-redmine` ou `requests` pour interagir avec l’API Redmine.
- `ollama` pour communiquer avec le modèle IA local.
- `python-dotenv` pour gérer les variables d’environnement.

---

## 📋 Fonctionnalités Minimales

### 1. Détection des Tickets
- **Critère** : Tickets ouverts depuis **>24h** sans commentaire ni mise à jour.
- **Optionnel** : Filtrer par priorité ou projet spécifique.

### 2. Récupération des Données
- **Données du ticket** : Titre, description, historique.
- **Sources pour l’IA** :
  - Pages wiki Redmine (`/projects/{id}/wiki/index.json`).
  - 5 derniers tickets **résolus** avec des mots-clés similaires.

### 3. Génération de Réponse
- **Prompt Ollama** :
  ```text
  Tu es un assistant technique. Voici un ticket sans réponse depuis 24h :
  [TITRE] : {title}
  [DESCRIPTION] : {description}

  Utilise ces ressources pour répondre :
  - Wiki : {wiki_extraits}
  - Tickets similaires : {réponses_passées}

  Réponds en **3 phrases max**. Si tu n’as pas assez d’informations, dis :
  *"Je n’ai pas trouvé de réponse adaptée. Un expert humain interviendra bientôt."*
  ```

### 4. Publication
- Ajouter un **commentaire** sur le ticket via l’API Redmine (`POST /issues/{id}.json`).
- **Format** :
  ```markdown
  > 🤖 [Réponse Automatique]
  > {réponse_générée}
  > *Source : [liens vers les tickets/wiki utilisés]*
  ```

---

## 📂 Exemple de structure du Projet
```bash
redmine-ai-agent/
├── .env                # Variables d’environnement (API Redmine, Ollama)
├── .gitignore          # Fichiers à ignorer
├── README.md           # Documentation du projet
├── pyproject.toml      # Configuration du projet (dépendances, build)
├── src/
│   ├── __init__.py
│   ├── main.py         # Point d’entrée du projet
│   ├── redmine/
│   │   ├── __init__.py
│   │   ├── client.py   # Client pour l’API Redmine
│   │   └── models.py   # Modèles de données (Ticket, WikiPage)
│   ├── ia/
│   │   ├── __init__.py
│   │   ├── prompt.py   # Construction du prompt pour Ollama
│   │   └── generator.py # Génération de la réponse via Ollama
│   └── utils/
│       ├── __init__.py
│       ├── config.py   # Gestion de la configuration
│       └── logger.py   # Gestion des logs
└── tests/
    ├── __init__.py
    ├── test_redmine/
    │   └── test_client.py
    ├── test_ia/
    │   └── test_generator.py
    └── test_utils/
        └── test_config.py
```
```

---

## 🧪 Tests
Le projet devra inclure des **tests unitaires** pour valider le bon fonctionnement des composants clés :

**Exemple de structure des tests** :
```bash
tests/
├── __init__.py
├── test_redmine_client.py
├── test_response_generator.py
└── test_config.py
```

**Exécution des tests** :
```bash
pytest tests/
```
---

## 🚀 Livrables
1. **Code fonctionnel** :
   - Script Python exécutable (`python main.py`).
   - Mode "dry-run" pour tester sans publier.
   - les tests unitaires
2. **Documentation** :
   - `README.md` avec instructions d’installation et d’exécution.
   - Exemple de logs et captures d’écran.
3. **Améliorations possibles** :
   - Système de feedback ("✅ Utile" / "❌ Non pertinent") qui demandera la création d'une API avec FastAPI.
   - Utilisation d’**embeddings** pour améliorer la recherche de tickets similaires.

---

## ⚠️ Contraintes
- **Sécurité** :
  - Ne pas commiter `.env`.
  - Limiter les permissions de l’API Redmine.
- **Performance** :
  - Traiter les tickets par lots (ex: 10 à la fois).
  - Utiliser un cache local pour éviter de re-télécharger le wiki.

---

## 🔧 Configuration : exemples
1. **Fichier `.env`** :
   ```ini
   REDMINE_URL=https://votre-redmine.fr
   REDMINE_API_KEY=votre_clé_api
   OLAMA_MODEL=mistral
   DRY_RUN=True  # Désactive la publication pour les tests
   ```

2. **Installation** :
   ```bash
   pip install -r requirements.txt
   ```

3. **Exécution** :
   ```bash
   python main.py
   ```

---

## 🎯 Critères d’Évaluation
| Critère               | Poids |
|-----------------------|-------|
| Fonctionnalité de base | 40%   |
| Qualité du code       | 30%   |
| Originalité           | 20%   |
| Documentation         | 10%   |

---

## 🔍 Ressources
- [Documentation API Redmine](https://www.redmine.org/projects/redmine/wiki/Rest_api)
- [Ollama Python Client](https://github.com/jmorganca/ollama-python)
- [Guide de Prompt Engineering](https://www.promptingguide.ai/)

---

## 💬 Questions Fréquentes
**Q : Comment éviter les réponses en boucle ?**
→ Ajouter un tag ou un statut "Répondu par IA" après publication.

**Q : Quel modèle Ollama utiliser ?**
→ `mistral` ou `llama3` pour un bon compromis rapidité/qualité.

**Q : Comment tester sans risquer de publier ?**
→ Utiliser le mode `DRY_RUN=True` dans `.env`.
