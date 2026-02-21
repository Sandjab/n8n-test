# Morning Briefing API - Tutoriel n8n

Tutoriel pour decouvrir **n8n** depuis zero sur Windows 11.
On construit une API personnelle qui retourne un "briefing du matin" avec meteo + citation inspirante.

## Prerequis

- Node.js installe (v18+)
- Un terminal (Git Bash, PowerShell, ou CMD)

## Demarrage rapide

```bash
# 1. Lancer n8n (premiere fois : telecharge automatiquement)
npx n8n

# 2. Ouvrir dans le navigateur
#    http://localhost:5678

# 3. Tester l'API briefing (une fois le workflow importe et publie)
curl -s http://localhost:5678/webhook/briefing
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

Ce workflow complet construit une API de briefing matinal :

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
3. Tester dans le navigateur : **http://localhost:5678/webhook/briefing**

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

## APIs utilisees

| API | URL | Auth | Description |
|-----|-----|------|-------------|
| ZenQuotes | `https://zenquotes.io/api/random` | Aucune | Citation aleatoire |
| Open-Meteo | `https://api.open-meteo.com/v1/forecast` | Aucune | Meteo mondiale (codes WMO) |

### Changer la ville pour la meteo

Modifier les parametres `latitude` et `longitude` dans l'URL du node "Get Weather" :
- Paris : `latitude=48.8566&longitude=2.3522`
- Berlin : `latitude=52.52&longitude=13.405`
- New York : `latitude=40.7128&longitude=-74.006`
- Tokyo : `latitude=35.6762&longitude=139.6503`

> Note : Open-Meteo est une API gratuite et open-source couvrant le monde entier. Pas besoin de cle API.

---

## Structure du projet

```
n8n-test/
  README.md                    # Ce fichier
  workflows/
    01-hello-world.json        # Workflow Hello World (etape 1)
    02-briefing-api.json       # Workflow Briefing API (etape 2)
```

---

## Extensions possibles

- **Schedule Trigger** : executer automatiquement chaque matin a 7h
- **Email** : envoyer le briefing par email (node Gmail ou SMTP)
- **Autres sources** : actualites (RSS), cours crypto (CoinGecko API)
- **Claude AI** : generer un resume personnalise avec l'API Claude
- **Telegram Bot** : envoyer le briefing sur Telegram

---

## Depannage

| Probleme | Solution |
|----------|----------|
| `npx n8n` ne demarre pas | Verifier Node.js : `node --version` (v18+ requis) |
| Port 5678 occupe | `npx n8n --port 5679` ou arreter le processus existant |
| Webhook ne repond pas | Verifier que le workflow est **active** (toggle ON) |
| API meteo en erreur | Open-Meteo peut etre temporairement indisponible, reessayer |
| Citation vide | ZenQuotes a un rate limit (5 req/30s), patienter |
