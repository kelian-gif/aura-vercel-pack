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

Pour Production, Preview ET Development en une seule commande.

## Context

- Projet courant: !`pwd`
- Fichier env présent: !`test -f "${1:-.env.local}" && echo "✅ ${1:-.env.local} trouvé" || echo "❌ ${1:-.env.local} introuvable"`
- Lié à Vercel: !`test -f .vercel/project.json && echo "✅ projet lié" || echo "❌ pas lié — lance d'abord 'vercel link --yes'"`
- Compte Vercel: !`vercel whoami 2>/dev/null || echo "❌ pas logué — lance 'vercel login'"`

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

| Préfixe | Mode |
|---|---|
| `NEXT_PUBLIC_*` | Normal (sans `--sensitive`) |
| Tout le reste | `--sensitive` |

**Affichage AVANT exécution** (sans les valeurs) :

```
📋 Variables détectées dans .env.local :

🔓 Mode Normal (NEXT_PUBLIC_*) :
  - NEXT_PUBLIC_SUPABASE_URL
  - NEXT_PUBLIC_SUPABASE_PUBLISHABLE_KEY
  - NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY

🔒 Mode Sensitive (chiffré) :
  - SUPABASE_SECRET_KEY
  - STRIPE_SECRET_KEY
  - DATABASE_URL

→ Vais pousser sur Production, Preview, Development.
```

## Phase 2: Pousser sur Vercel

**Pour chaque variable, pour chaque environnement (production, preview, development) :**

Utiliser stdin pour passer la valeur sans l'afficher :

```bash
# Pour les NEXT_PUBLIC_* (mode normal)
printf '%s' "$VALEUR" | vercel env add NOM production --force
printf '%s' "$VALEUR" | vercel env add NOM preview --force
printf '%s' "$VALEUR" | vercel env add NOM development --force

# Pour les autres (mode sensitive)
printf '%s' "$VALEUR" | vercel env add NOM production --sensitive --force
printf '%s' "$VALEUR" | vercel env add NOM preview --sensitive --force
printf '%s' "$VALEUR" | vercel env add NOM development --sensitive --force
```

**Règles importantes :**
- Utiliser `printf '%s'` (pas `echo`) pour éviter d'ajouter un newline parasite
- Toujours `--force` pour écraser une valeur existante sans prompt interactif
- **Ne JAMAIS afficher la valeur** dans tes messages, ni dans les commandes que tu fais voir à l'user
- Si une commande échoue, reporter le nom de la var (pas la valeur) et continuer avec les suivantes

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
- 3 en mode 🔒 Sensitive
- 3 en mode 🔓 Normal
- Toutes présentes en Production, Preview, Development

⚠️ Si certaines vars sont marquées "Needs Attention" dans le dashboard,
   elles existaient déjà en mode normal avant ce skill. Pour les passer
   en Sensitive, supprime-les manuellement et relance ce skill.
```

## Notes

- Les variables sensibles ne seront **plus jamais lisibles** dans le dashboard Vercel après push (par design Vercel). Si tu as besoin de les revoir, tu dois aller à la source (dashboard Stripe, Supabase, etc.).
- Pour rotate une variable : update à la source → relancer ce skill avec le `.env.local` mis à jour.
- Ce skill ne touche pas au code, uniquement à la config Vercel.
