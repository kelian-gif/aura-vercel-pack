---
name: setup-vercel-env
description: Lit .env.local et pousse toutes les variables sur Vercel (Production, Preview, Development). Met automatiquement en mode --sensitive les vraies clés secrètes, en mode normal les NEXT_PUBLIC_*. Ne montre jamais les valeurs à l'écran. Use when initial setup of env vars on Vercel, or after adding new secrets.
argument-hint: "[optional: chemin vers .env.local — défaut: ./.env.local]"
allowed-tools: Bash(vercel :*), Bash(test :*), Bash(cat :*), Bash(grep :*), Bash(awk :*), Bash(wc :*), Read
---

# Setup Vercel Env

Pousse toutes les variables d'environnement de `.env.local` sur Vercel en mode propre :
- `NEXT_PUBLIC_*` → mode normal (visible dashboard, c'est OK car déjà public)
- Tout le reste → mode `--sensitive` (chiffré, jamais visible après ajout)

Pour Production, Preview ET Development en une seule commande (avec exceptions documentées ci-dessous).

## Context

- Projet courant: !`pwd`
- Fichier env présent: !`test -f "${1:-.env.local}" && echo "✅ ${1:-.env.local} trouvé" || echo "❌ ${1:-.env.local} introuvable"`
- Lié à Vercel: !`test -f .vercel/project.json && echo "✅ projet lié" || echo "❌ pas lié — lance d'abord 'vercel link --yes'"`
- Compte Vercel: !`vercel whoami 2>/dev/null || echo "❌ pas logué — lance 'vercel login'"`
- Repo Git lié au projet Vercel: !`vercel git ls 2>&1 | grep -E "github|gitlab|bitbucket" | head -1 || echo "⚠️ projet Vercel pas lié à un repo Git — Preview env vars seront skip"`

## Préchecks bloquants

**Avant de continuer, vérifier :**

1. Si `.env.local` (ou le chemin fourni en argument) n'existe pas → STOP, dire à l'user de créer le fichier d'abord
2. Si `.vercel/project.json` n'existe pas → STOP, dire à l'user de lancer `vercel link --yes` d'abord
3. Si `vercel whoami` échoue → STOP, dire à l'user de faire `vercel login` d'abord

## Phase 1: Parser le fichier env

Lire le fichier (par défaut `.env.local`, sinon `$ARGUMENTS`) et extraire toutes les variables :

- Ignorer les lignes vides
- Ignorer les lignes qui commencent par `#` (commentaires)
- Format attendu : `NOM=valeur` (avec ou sans guillemets autour de la valeur)
- Stripper les guillemets `"..."` ou `'...'` si présents
- Garder les valeurs en mémoire **sans jamais les afficher à l'écran**

**Pour chaque variable, classer :**

| Préfixe | Mode | Environnements ciblés |
|---|---|---|
| `NEXT_PUBLIC_*` | Normal (sans `--sensitive`) | production + preview + development |
| Tout le reste | `--sensitive` | production + preview (development interdit par Vercel pour sensitive) |

**Affichage AVANT exécution** (sans les valeurs) :

```
📋 Variables détectées dans .env.local :

🔓 Mode Normal (NEXT_PUBLIC_*) → Production + Preview + Development :
  - NEXT_PUBLIC_SUPABASE_URL
  - NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY
  - NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY

🔒 Mode Sensitive (chiffré) → Production + Preview seulement (Vercel interdit Sensitive sur Development) :
  - SUPABASE_SECRET_KEY
  - STRIPE_SECRET_KEY
  - DATABASE_URL

→ Vais pousser sur les environnements adaptés.
```

## Phase 2: Pousser sur Vercel

**Syntaxe Vercel CLI 50.37+ (utilise `--value` au lieu de stdin pour le mode non-interactif) :**

```bash
# Pour les NEXT_PUBLIC_* (mode normal) — push sur les 3 environnements
vercel env add NOM production --value "$VALEUR" --force --yes
vercel env add NOM preview --value "$VALEUR" --force --yes
vercel env add NOM development --value "$VALEUR" --force --yes

# Pour les autres (mode sensitive) — push sur production + preview UNIQUEMENT
vercel env add NOM production --value "$VALEUR" --sensitive --force --yes
vercel env add NOM preview --value "$VALEUR" --sensitive --force --yes
# PAS de development pour sensitive (interdit Vercel par design)
```

**Règles importantes :**
- Utiliser `--value "$VALEUR"` (et **pas** `printf | vercel`) car le CLI Vercel 50.37+ ne supporte plus le stdin en mode non-interactif
- Toujours `--force --yes` pour écraser une valeur existante et confirmer "all preview branches" sans prompt interactif
- **Ne JAMAIS afficher la valeur** dans tes messages, ni dans les commandes que tu fais voir à l'user
- **Si la commande Preview échoue** avec une erreur du type `"reason": "git_branch_required"` ou `"Project does not have a connected Git repository"` : **continuer sans bloquer** et noter dans le résumé final que Preview a été skip parce que le projet Vercel n'est pas lié à un repo Git. L'utilisateur devra connecter son repo GitHub à Vercel pour activer Preview.
- Si une commande échoue pour une autre raison, reporter le nom de la var (pas la valeur) et continuer avec les suivantes

## Phase 3: Vérification

À la fin, lancer :

```bash
vercel env ls
```

Et reporter à l'user :

```
✅ Push terminé.

Résumé :
- 6 variables poussées
- 3 en mode 🔒 Sensitive (Production + Preview)
- 3 en mode 🔓 Normal (Production + Preview + Development)

⚠️ Si Preview a été skip : ton projet Vercel n'est pas encore lié à un repo Git.
   Va sur vercel.com/{team}/{projet}/settings/git pour le lier au repo GitHub.
   Une fois lié, relance /setup-vercel-env pour compléter Preview.

⚠️ Si certaines vars sont marquées "Needs Attention" dans le dashboard,
   elles existaient déjà en mode normal avant ce skill. Pour les passer
   en Sensitive, supprime-les manuellement et relance ce skill.
```

## Notes

- Les variables sensibles ne seront **plus jamais lisibles** dans le dashboard Vercel après push (par design Vercel). Si tu as besoin de les revoir, tu dois aller à la source (dashboard Stripe, Supabase, etc.).
- Pour rotate une variable : update à la source → relancer ce skill avec le `.env.local` mis à jour.
- Ce skill ne touche pas au code, uniquement à la config Vercel.
- **Vercel CLI 50.37+ ne supporte plus le stdin** pour `vercel env add` en mode non-interactif. La syntaxe `--value "..."` est la seule fiable. La valeur passe par argument shell donc reste locale à la session shell (jamais transmise ailleurs).
- **Sensitive vars sur Development = interdit par Vercel.** C'est by design : les sensitive vars ne peuvent pas être téléchargées en local pour rester chiffrées côté serveur.
- **Preview vars exigent un repo Git lié au projet Vercel.** Sinon le CLI demande explicitement une branche Git et ça échoue. Solution : lier le projet à GitHub via le dashboard Vercel (Settings → Git → Connect Git Repository).
