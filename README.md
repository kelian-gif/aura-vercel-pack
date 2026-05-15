# Pack Skills Claude Code · Aura Formation Vercel

2 skills custom qui complètent le plugin Vercel officiel de Claude Code.

## Setup en 2 étapes

### 1️⃣ Installer le plugin Vercel officiel (une fois)

Dans Claude Code, tape :
```
/plugin install vercel
/reload-plugins
```

Tu récupères ~30 skills `vercel:*` maintenus par Vercel : `vercel:deploy`, `vercel:env`, `vercel:status`, `vercel:nextjs`, `vercel:turbopack`, etc.

### 2️⃣ Ajouter les 2 skills custom Aura

Dans Claude Code, demande simplement :

> Installe les 2 skills de https://github.com/kelian-gif/aura-vercel-pack dans `~/.claude/skills/`.

## Pourquoi ces 2 skills custom ?

Le plugin Vercel officiel couvre le déploiement et la gestion des variables d'environnement. Mais il ne gère pas le **workflow quotidien commit + monitor + auto-fix**. C'est ce que ces 2 skills ajoutent :

| Skill | À quoi ça sert |
|---|---|
| `commit-and-monitor` | Tu modifies, tu lances → Claude sauvegarde, déploie sur Vercel, **surveille en direct**, et **corrige si ça casse** |
| `ci-fixer` | Répare en boucle un déploiement bloqué (Vercel, GitHub Actions, Netlify) |

## Workflow complet

Une fois les 3 installés (plugin + 2 customs), tu as :

```
/vercel:deploy              # premier déploiement (plugin officiel)
/vercel:env                 # configs/secrets (plugin officiel)
/commit-and-monitor "msg"   # workflow quotidien (custom Aura)
/ci-fixer                   # réparation déploiement (custom Aura)
```

Tu n'as plus jamais à toucher le terminal pour déployer.
