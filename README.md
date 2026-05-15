# Pack Skills Claude Code · Vercel

4 skills pour déployer ton site sur Vercel via Claude Code, utilisés dans la formation **Aura Académie**.

## Installation rapide

Dans Claude Code, demande simplement :

> Installe les 4 skills de https://github.com/kelian-gif/aura-vercel-pack dans `~/.claude/skills/`, et les outils Vercel CLI + GitHub CLI dont j'ai besoin.

Claude clone le repo, copie les skills, installe les outils, et te fait un résumé. C'est tout.

## Skills inclus

| Skill | Type | À quoi ça sert |
|---|---|---|
| `deploy-to-vercel` | 🟢 Officiel Vercel | Met ton site en ligne sur Vercel pour la première fois |
| `setup-vercel-env` | 🟡 Custom Aura | Pousse tes configs (.env.local) sur Vercel en chiffré |
| `commit-and-monitor` | 🟡 Custom Aura | Sauvegarde + déploie + auto-fix si ça casse |
| `ci-fixer` | 🟡 Custom Aura | Répare en boucle un déploiement bloqué |

## Mode d'emploi par skill

Une fois installés, dans Claude Code :

```
/deploy-to-vercel
/setup-vercel-env
/commit-and-monitor "message du commit"
/ci-fixer
```

Tu n'as plus jamais à toucher au terminal pour déployer.
