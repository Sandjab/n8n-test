# Morning Briefing API - Tutoriel n8n

Tutoriel pour decouvrir **n8n** depuis zero sur Windows 11.
On construit une API personnelle qui retourne un "briefing du matin" avec meteo, citation inspirante, et actualites resumees par IA.

## Progression du tutoriel

| Etape | Workflow | Ce qu'on apprend |
|-------|----------|------------------|
| 1. Hello World | `01-hello-world.json` | Bases : trigger, node, execution |
| 2. Briefing API | `02-briefing-api.json` | Webhook, HTTP Request, Merge, Code |
| 3. Briefing IA | `03-briefing-claude.json` | RSS, HTTP POST, IA/LLM, `$('NodeName')` |

## Prerequis

- Node.js installe (v18+)
- Un terminal (Git Bash, PowerShell, ou CMD)
- Une cle API Anthropic pour l'etape 3 ([console.anthropic.com](https://console.anthropic.com/))

## Demarrage rapide

```bash
# 1. Lancer n8n (premiere fois : telecharge automatiquement)
npx n8n

# 2. Ouvrir dans le navigateur
#    http://localhost:5678

# 3. Importer un workflow (exemple : etape 2)
#    Dans n8n : Workflows > Add workflow > Import from file
#    Ou en CLI :
npx n8n import:workflow --input=workflows/02-briefing-api.json

# 4. Activer le workflow (toggle en haut a droite) puis tester :
curl -s http://localhost:5678/webhook/briefing
curl -s http://localhost:5678/webhook/briefing-ai   # etape 3
```

Au premier lancement, n8n demande de creer un compte local (owner account).
Les workflows importes sont persistes localement (`~/.n8n/`) : au prochain `npx n8n`, tout est deja en place.

---

## Etape 1 : Hello World

**Fichier** : `workflows/01-hello-world.json`

Ce workflow minimal illustre les bases :
- **Manual Trigger** : declenche le workflow a la main
- **Edit Fields** : definit un message `{ "message": "Hello n8n!" }`

### Importer le workflow

1. Dans n8n, aller dans **Workflows** > **Add workflow** > **Import from file**
2. Selectionner `workflows/01-hello-world.json`
3. Cliquer sur **Execute Workflow** (bouton play)
4. Observer le resultat dans le panneau de sortie de chaque node

### Concepts appris

- Workflow, node, connexion
- Execution manuelle
- Inspection des donnees de sortie

---

## Etape 2 : Morning Briefing API

**Fichier** : `workflows/02-briefing-api.json`

Ce workflow construit une API de briefing matinal avec deux appels en parallele :

```
Webhook GET /briefing
    |
    +---> Get Quote (zenquotes.io)
    |
    +---> Get Weather (open-meteo.com - Paris)
    |
    v
  Merge (combine les deux)
    |
    v
  Format Briefing (Code JavaScript)
    |
    v
  Respond to Webhook (retourne le JSON)
```

### Importer et activer

1. Importer `workflows/02-briefing-api.json` dans n8n
2. **Activer le workflow** (toggle en haut a droite)
3. Tester dans le navigateur : `http://localhost:5678/webhook/briefing`

### Reponse attendue

```json
{
  "briefing": {
    "date": "vendredi 21 fevrier 2026",
    "meteo": {
      "ville": "Paris",
      "temperature": "8°C",
      "condition": "couvert"
    },
    "citation": {
      "texte": "The only way to do great work is to love what you do.",
      "auteur": "Steve Jobs"
    },
    "message": "Bonjour ! Il fait 8°C a Paris (couvert). Citation du jour : \"...\" - Auteur"
  }
}
```

### Detail des nodes

| Node | Type | Role |
|------|------|------|
| Webhook | Trigger | Ecoute les requetes GET sur `/briefing` |
| Get Quote | HTTP Request | Appelle `zenquotes.io/api/random` |
| Get Weather | HTTP Request | Appelle `api.open-meteo.com` (meteo Paris) |
| Merge | Merge | Combine citation + meteo en un seul item |
| Format Briefing | Code (JS) | Formate le JSON de reponse |
| Respond to Webhook | Response | Retourne le JSON au client |

### Concepts appris

- **Webhook** : transformer un workflow en API
- **HTTP Request** : appeler des APIs externes
- **Merge** : combiner plusieurs flux de donnees
- **Code** : transformer les donnees en JavaScript
- **Respond to Webhook** : retourner une reponse HTTP

---

## Etape 3 : Briefing IA avec Claude + Actualites RSS

**Fichier** : `workflows/03-briefing-claude.json`

Ce workflow enrichit le briefing avec des actualites RSS et un resume genere par Claude (IA).
Contrairement a l'etape 2, le flux est **sequentiel** (pas de Merge) :

```
Webhook GET /briefing-ai
    |
    v
  Get Weather (Open-Meteo - Paris)
    |
    v
  Get Quote (zenquotes.io)
    |
    v
  Get News (RSS Le Monde - titres d'actualite)
    |
    v
  Build Prompt (Code JS - agrege les donnees, construit le prompt Claude)
    |
    v
  Call Claude (HTTP POST - api.anthropic.com)
    |
    v
  Format Response (Code JS - structure le JSON final)
    |
    v
  Respond to Webhook (retourne le JSON)
```

### Pourquoi un flux sequentiel ?

- Le node RSS retourne **N items** (un par article), ce qui complique le Merge avec 3+ sources
- Le pattern **`$('NodeName')`** permet de recuperer les donnees de n'importe quel node en amont
- Plus simple a suivre pour un tutoriel

### Importer et configurer

1. Importer `workflows/03-briefing-claude.json` dans n8n
2. **Configurer la cle API Anthropic** (voir section ci-dessous)
3. **Activer le workflow** (toggle en haut a droite)
4. Tester : `http://localhost:5678/webhook/briefing-ai`

### Configurer la cle API Anthropic

Le node "Call Claude" necessite une cle API Anthropic pour fonctionner :

1. **Obtenir une cle** : aller sur [console.anthropic.com](https://console.anthropic.com/) > API Keys > Create Key
2. **Editer le node** : dans n8n, ouvrir le workflow, double-cliquer sur le node **"Call Claude"**
3. **Remplacer le placeholder** : dans les headers, remplacer `YOUR_ANTHROPIC_API_KEY` par votre vraie cle
4. **Sauvegarder** et activer le workflow

> **Note** : pour un usage en production, utilisez les **Credentials** n8n (Header Auth) plutot qu'une cle en dur dans le node. Voir la [doc n8n sur les credentials](https://docs.n8n.io/credentials/).

### Reponse attendue

```json
{
  "briefing": {
    "date": "samedi 22 fevrier 2026",
    "meteo": {
      "ville": "Paris",
      "temperature": "8°C",
      "condition": "couvert"
    },
    "citation": {
      "texte": "The only way to do great work is to love what you do.",
      "auteur": "Steve Jobs"
    },
    "actualites": [
      "Titre article 1",
      "Titre article 2",
      "Titre article 3",
      "Titre article 4",
      "Titre article 5"
    ],
    "resume_ia": "Bonjour ! En ce samedi matin sous un ciel couvert et 8°C a Paris...",
    "modele": "claude-sonnet-4-20250514"
  }
}
```

### Detail des nodes

| # | Node | Type | Role |
|---|------|------|------|
| 1 | Webhook | Trigger | Ecoute les requetes GET sur `/briefing-ai` |
| 2 | Get Weather | HTTP Request | Appelle Open-Meteo (meteo Paris) |
| 3 | Get Quote | HTTP Request | Appelle zenquotes.io/api/random |
| 4 | Get News | RSS Feed Read | Lit le flux RSS du Monde (actualites) |
| 5 | Build Prompt | Code (JS) | Agrege meteo + citation + titres, construit le prompt |
| 6 | Call Claude | HTTP Request | POST vers l'API Anthropic (Claude Sonnet) |
| 7 | Format Response | Code (JS) | Extrait la reponse Claude, structure le JSON final |
| 8 | Respond to Webhook | Response | Retourne le JSON au client |

### Concepts appris

- **RSS Feed Read** : lire un flux RSS (actualites, blogs, podcasts)
- **HTTP POST avec headers** : envoyer des requetes authentifiees (cle API)
- **Integration IA/LLM** : utiliser Claude pour generer du contenu
- **`$('NodeName')`** : referencer les donnees de n'importe quel node en amont (pas seulement le precedent)
- **Flux sequentiel** : alternative au Merge pour les cas avec des sorties de tailles differentes

---

## APIs utilisees

| API | URL | Auth | Description |
|-----|-----|------|-------------|
| ZenQuotes | `https://zenquotes.io/api/random` | Aucune | Citation aleatoire (rate limit : 5 req/30s) |
| Open-Meteo | `https://api.open-meteo.com/v1/forecast` | Aucune | Meteo mondiale gratuite (codes WMO) |
| Le Monde RSS | `https://www.lemonde.fr/rss/une.xml` | Aucune | Flux RSS actualites (etape 3) |
| Anthropic | `https://api.anthropic.com/v1/messages` | Cle API (`x-api-key`) | Generation de texte IA (etape 3) |

### Changer la ville pour la meteo

Modifier les parametres `latitude` et `longitude` dans l'URL du node "Get Weather" :

| Ville | Latitude | Longitude |
|-------|----------|-----------|
| Paris | 48.8566 | 2.3522 |
| Berlin | 52.52 | 13.405 |
| New York | 40.7128 | -74.006 |
| Tokyo | 35.6762 | 139.6503 |

> Open-Meteo est une API gratuite et open-source couvrant le monde entier. Pas besoin de cle API.

---

## Structure du projet

```
n8n-test/
  CLAUDE.md                    # Instructions pour Claude Code
  README.md                    # Ce fichier
  workflows/
    01-hello-world.json        # Workflow Hello World (etape 1)
    02-briefing-api.json       # Workflow Briefing API (etape 2)
    03-briefing-claude.json    # Workflow Briefing IA Claude (etape 3)
```

---

## Extensions possibles

- **Schedule Trigger** : executer automatiquement chaque matin a 7h
- **Email** : envoyer le briefing par email (node Gmail ou SMTP)
- **Telegram Bot** : envoyer le briefing sur Telegram
- **Autres sources** : cours crypto (CoinGecko API), meteo multi-villes
- **Base de donnees** : sauvegarder les briefings quotidiens

---

## Depannage

| Probleme | Solution |
|----------|----------|
| `npx n8n` ne demarre pas | Verifier Node.js : `node --version` (v18+ requis) |
| Port 5678 occupe | `npx n8n --port 5679` ou arreter le processus existant |
| Webhook ne repond pas | Verifier que le workflow est **active** (toggle ON) |
| API meteo en erreur | Open-Meteo peut etre temporairement indisponible, reessayer |
| Citation vide | ZenQuotes a un rate limit (5 req/30s), patienter |
| Erreur 401 sur Call Claude | Verifier la cle API dans le header `x-api-key` du node |
| Erreur 400 sur Call Claude | Verifier que le body JSON est bien forme (voir Build Prompt) |
| RSS Le Monde indisponible | Le flux RSS peut etre temporairement en maintenance |
| `resume_ia` = "Resume indisponible" | Verifier la cle API et les credits sur [console.anthropic.com](https://console.anthropic.com/) |

---

## Liens utiles

- [Documentation n8n](https://docs.n8n.io/)
- [n8n Node Reference](https://docs.n8n.io/integrations/builtin/core-nodes/)
- [API Anthropic (Claude)](https://docs.anthropic.com/en/api/messages)
- [Open-Meteo API](https://open-meteo.com/en/docs)
- [ZenQuotes API](https://zenquotes.io/)
