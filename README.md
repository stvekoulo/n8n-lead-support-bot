# 🤖 Bot de Qualification de Leads et Support Client Intelligent

> **Mon premier workflow n8n** - Un système automatisé qui qualifie les messages entrants, gère les prospects et crée des tickets de support avec l'intelligence artificielle.

[![n8n](https://img.shields.io/badge/n8n-Workflow-FF6D5A?logo=n8n)](https://n8n.io/)
[![OpenRouter](https://img.shields.io/badge/OpenRouter-AI-blue)](https://openrouter.ai/)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

---

## 📋 Table des Matières

- [À Propos](#-à-propos)
- [Fonctionnalités](#-fonctionnalités)
- [Architecture](#-architecture)
- [Prérequis](#-prérequis)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Utilisation](#-utilisation)
- [Structure des Données](#-structure-des-données)
- [Contribuer](#-contribuer)
- [Roadmap](#-roadmap)
- [Licence](#-licence)

---

## À Propos

Ce workflow automatise complètement la gestion des messages entrants (formulaires de contact, emails) en utilisant l'IA pour :
- **Qualifier** automatiquement les messages (Prospect, Support, Spam)
- **Router** vers les bonnes actions selon le type
- **Notifier** l'équipe en temps réel via WhatsApp
- **Créer** des tickets de support avec ID unique
- **Enregistrer** tout dans Google Sheets

### Pourquoi ce projet ?

Ceci est **ma toute première automatisation n8n** ! J'ai voulu créer quelque chose d'utile et complet pour apprendre. Le code n'est peut-être pas parfait, mais il fonctionne et je suis ouvert à toutes vos suggestions d'amélioration !

---

## Fonctionnalités

### Classification Intelligente par IA
- Analyse automatique du contenu avec OpenRouter (openai/gpt-oss-20b:free)
- Détection du type : **Commercial**, **Support** ou **Spam**
- Extraction des informations : nom, email, résumé, niveau d'urgence
- Score de confiance pour chaque classification

### Gestion des Prospects (Commercial)
1. Enregistrement dans Google Sheets "Prospects"
2. Email de confirmation personnalisé au prospect
3. Notification WhatsApp instantanée à l'équipe commerciale

### Gestion du Support
1. Génération d'un ticket unique (ex: `1730419200000-A3F9`)
2. Enregistrement dans Google Sheets "Tickets Support"
3. Email de confirmation avec numéro de ticket au client
4. Notification WhatsApp si urgence **HAUTE**

### Filtrage du Spam
1. Log dans Google Sheets "Spam"
2. Aucune action supplémentaire (pas d'email, pas de notification)

---

## Architecture

```
┌─────────────────┐
│  Webhook/IMAP   │  Réception du message
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Nettoyage Data  │  Formatage et validation
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│   OpenRouter    │  Analyse IA
│   (openai/gpt) │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  Parse JSON     │  Extraction résultats
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│     Router      │  Aiguillage par type
└───┬─────┬───┬───┘
    │     │   │
    │     │   └──────────────────┐
    │     │                      │
    ▼     ▼                      ▼
┌────────┐ ┌─────────┐    ┌──────────┐
│Prospect│ │ Support │    │   Spam   │
└───┬────┘ └────┬────┘    └─────┬────┘
    │           │               │
    ├→ Sheet    ├→ Gen Ticket   └→ Log Sheet
    ├→ Email    ├→ Sheet
    └→ WhatsApp ├→ Email
                └→ WhatsApp (si urgent)
```

---

## Prérequis

### Services Nécessaires

| Service | Utilité | Coût |
|---------|---------|------|
| **n8n** | Plateforme d'automatisation | Gratuit (self-hosted) ou ~20€/mois (cloud) |
| **OpenRouter** | API d'IA pour l'analyse | Gratuit |
| **Google Sheets** | Base de données | Gratuit |
| **Gmail** | Envoi d'emails | Gratuit (500 emails/jour) |
| **WasenderAPI** | Notifications WhatsApp | Voir leur tarification | 3 Jours d'éssaie gratuit |

### Comptes à Créer

1. [n8n Cloud](https://n8n.io/) ou self-hosted
2. [OpenRouter](https://openrouter.ai/) - Récupère ta clé API
3. [WasenderAPI](https://wasenderapi.com/) - Récupère ta clé API
4. Compte Google (Gmail + Sheets)

---

## Installation

### 1. Télécharger le Workflow

```bash
git clone https://github.com/stvekoulo/n8n-lead-support-bot.git
cd n8n-lead-support-bot
```

### 2. Importer dans n8n

1. Ouvre n8n
2. Menu (☰) → **Import from File**
3. Sélectionne `Bot Qualification Leads & Support Client.json`
4. Clique sur **Import**

### 3. Créer les Google Sheets

#### **Sheet 1 : Prospects**
Crée un Google Sheet nommé "Prospects" avec ces colonnes :

| Date | Nom | Email | Résumé Demande | Urgence | Confiance IA | Traité | Message Complet | Notes |
|------|-----|-------|----------------|---------|--------------|--------|-----------------|-------|

#### **Sheet 2 : Tickets Support**
Crée un Google Sheet nommé "Tickets Support" avec ces colonnes :

| Ticket ID | Date | Nom | Email | Résumé | Urgence | Statut | Message Complet | Notes |
|-----------|------|-----|-------|--------|---------|--------|-----------------|-------|

#### **Sheet 3 : Logger Span**
Crée un Google Sheet nommé "Logger Span" :

| Date | Email | Message | Confiance |
|------|-------|---------|-----------|

---

## Configuration

### 1. Créer les Credentials

#### **A) OpenRouter API**
1. n8n : **Settings** → **Credentials** → **Add Credential**
2. Type : **HTTP Header Auth**
3. Configuration :
   - **Name** : `OpenRouter API`
   - **Header Name** : `Authorization`
   - **Header Value** : `Bearer sk-or-v1-VOTRE_CLE_API`
4. **Save**

#### **B) WasenderAPI**
1. **Add Credential** → **HTTP Header Auth**
2. Configuration :
   - **Name** : `WasenderAPI`
   - **Header Name** : `Authorization`
   - **Header Value** : `Bearer VOTRE_CLE_WASENDER`
3. **Save**

#### **C) Google Sheets**
1. **Add Credential** → **Google Sheets OAuth2 API**
2. Clique sur **Connect my account**
3. Autorise l'accès à ton compte Google
4. **Save**

#### **D) Gmail**
1. **Add Credential** → **Gmail OAuth2 API**
2. Clique sur **Connect my account**
3. Autorise l'accès à Gmail
4. **Save**

### 2. Configurer les Nœuds

#### Dans chaque nœud Google Sheets :
- Sélectionne ton **credential Google Sheets**
- Choisis le bon **Document** (Prospects / Tickets Support / Logs)
- Sélectionne le bon **Sheet Name**

#### Dans les nœuds WhatsApp :
Remplace `VOTRE_NUMERO_WHATSAPP` par ton numéro (format international sans +)
```json
"to": "22997123456"  // Exemple pour le Bénin
```

#### Dans les nœuds Email :
- **Email au Prospect** : Remplace `votre-email@gmail.com`
- **Email Confirmation Support** : Remplace `support@votre-entreprise.com`

### 3. Activer le Workflow

1. Clique sur le toggle **OFF → ON** en haut à droite
2. Copie l'**URL du Webhook** générée
3. Utilise cette URL dans ton formulaire de contact

---

## Utilisation

### Option 1 : Via Webhook (Formulaire Web)

Envoie une requête POST à l'URL du webhook :

```bash
POST https://votre-n8n.com/webhook/VOTRE-ID
Content-Type: application/json

{
  "name": "Jean Dupont",
  "email": "jean@example.com",
  "message": "Bonjour, je souhaite obtenir un devis pour votre service premium"
}
```

**Exemple avec JavaScript (formulaire web) :**

```javascript
const formData = {
  name: document.getElementById('name').value,
  email: document.getElementById('email').value,
  message: document.getElementById('message').value
};

fetch('https://votre-n8n.com/webhook/VOTRE-ID', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(formData)
})
.then(response => response.json())
.then(data => {
  console.log('Message envoyé avec succès !', data);
})
.catch(error => console.error('Erreur:', error));
```

### Option 2 : Via IMAP (Email)

Si tu veux que le bot analyse les emails reçus :
1. Remplace le nœud **Webhook** par un nœud **IMAP**
2. Configure ton serveur email (Gmail, Outlook, etc.)
3. Le reste du workflow reste identique

---

## Structure des Données

### Format d'Entrée (Webhook)

```json
{
  "name": "string (optionnel)",
  "email": "string (optionnel mais recommandé)",
  "message": "string (requis)"
}
```

### Réponse de l'IA (OpenRouter)

```json
{
  "type": "commercial | support | spam",
  "nom": "string | null",
  "email": "string | null",
  "resume": "string",
  "urgence": "haute | moyenne | basse",
  "confiance": 0.95
}
```

### ID de Ticket Généré

Format : `TIMESTAMP-RANDOM`
Exemple : `1730419200000-A3F9`

---

## Contribuer

Les contributions sont **très bienvenues** ! C'est mon premier workflow n8n, donc il y a sûrement des améliorations possibles.

### Comment Contribuer

1. **Fork** ce dépôt
2. Crée une **branche** pour ta modification :
   ```bash
   git checkout -b amelioration/ma-super-feature
   ```
3. **Modifie** le workflow dans n8n
4. **Exporte** le workflow : Menu → Export → Download
5. **Renomme** le fichier exporté en incrémentant la version :
   ```
   Bot Qualification Leads & Support Client.json_v2.json
   Bot Qualification Leads & Support Client.json_v3.json
   etc.
   ```
6. **Commit** tes changements :
   ```bash
   git add Bot Qualification Leads & Support Client.json_v2.json
   git commit -m "Ajout: Intégration Slack pour les notifications"
   ```
7. **Push** vers ta branche :
   ```bash
   git push origin amelioration/ma-super-feature
   ```
8. Ouvre une **Pull Request** en décrivant tes modifications

### Important

- **NE MODIFIE PAS** `Bot Qualification Leads & Support Client_v2.json` (fichier original)
- **AJOUTE** toujours un nouveau fichier avec la version incrémentée
- Documente tes changements dans la Pull Request
- Teste ton workflow avant de soumettre

### Idées d'Améliorations

- [ ] Intégration Slack/Discord
- [ ] Support multi-langues
- [ ] Dashboard de statistiques
- [ ] Auto-réponse IA pour les questions simples
- [ ] Détection de sentiment
- [ ] Intégration CRM (HubSpot, Salesforce)
- [ ] Système de priorité automatique
- [ ] Réponses IA personnalisées par type

---

## Roadmap

### Version 1.0 (Actuelle)
- Classification IA (Commercial/Support/Spam)
- Notifications WhatsApp
- Gestion tickets avec ID unique
- Emails automatiques
- Enregistrement Google Sheets

### Version 2.0 (À venir)
- [ ] Interface de gestion des tickets
- [ ] Statistiques et analytics
- [ ] Réponses IA automatiques pour support niveau 1
- [ ] Multi-canaux (Slack, Discord, Telegram)
- [ ] Base de connaissances IA
- [ ] Auto-apprentissage sur les réponses
- [ ] Intégration CRM complète
- [ ] API REST pour interroger les données

---

## Changelog

### v1.0.0 - 31/10/2024
- Version initiale
- Classification IA avec OpenRouter
- Emails automatiques (prospects + support)
- Notifications WhatsApp
- Enregistrement Google Sheets
- Système de tickets unique

---

## Problèmes Connus

- Les émojis dans les messages peuvent parfois causer des erreurs d'encodage
- Le temps de réponse peut être long si OpenRouter est surchargé
- Limite Gmail : 500 emails/jour

---

## Licence

MIT License - Tu es libre d'utiliser, modifier et distribuer ce workflow.

---

## Remerciements

- **n8n** pour cette plateforme incroyable
- **OpenRouter** pour l'accès aux modèles IA
- **La communauté n8n** pour l'inspiration
- **Toi** pour utiliser ou améliorer ce workflow ! 

---

## Contact

- **GitHub Issues** : Pour les bugs et suggestions
- **Discussions** : Pour les questions générales
- **Email** : stevencrdkoulo@gmail.com

---

## ⭐ Star ce Projet

Si ce workflow t'a aidé, n'hésite pas à mettre une ⭐ sur GitHub ! Ça m'encourage à continuer. 🚀

---

**Made with by Steven - Mon premier workflow n8n !**
