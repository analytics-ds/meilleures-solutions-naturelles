# Skill : Creer un article

Ce skill genere un article de blog optimise SEO et GEO (Generative Engine Optimization) selon le type choisi.

## Declenchement

L'utilisateur tape `/create-article` ou demande de creer/rediger un article.

## Etape 0 — Verification du quota hebdomadaire

Avant toute chose, lire le fichier `MEMORY.md` a la racine du projet. Ce fichier contient l'historique des articles publies, classe par semaine.

Compter le nombre d'articles publies dans la **semaine en cours** (lundi a dimanche). Si **4 articles ou plus** sont deja enregistres cette semaine :
- Avertir l'utilisateur : "4 articles ont deja ete publies cette semaine. Pour une strategie de cocon semantique efficace, il est recommande d'etaler la publication. Tu veux quand meme continuer ?"
- Si l'utilisateur confirme, continuer. Sinon, arreter.

Si le fichier `MEMORY.md` n'existe pas encore, le creer vide (il sera rempli a l'etape 7).

## Etape 1 — Collecter les informations

### Prompt GEO et query fan-out

Demander a l'utilisateur :
- **Prompt GEO cible** : la question exacte que les utilisateurs posent aux moteurs IA generatifs (ChatGPT, Perplexity, Google AI Overviews). Ce prompt deviendra le **H1 de l'article**. C'est une question naturelle, pas un mot-cle optimise. Exemple : "Quel anti-moustique naturel choisir ?"
- **Query fan-out (mot-cle SEO)** : le terme SEO sur lequel l'article se positionne dans Google. Il decoule du prompt et represente la recherche "classique" associee. Exemple : "huile essentielle anti moustique". Si l'utilisateur ne fournit pas de query fan-out, la determiner a partir du prompt en choisissant un mot-cle avec du volume de recherche.
- **Categorie** : dans quelle categorie du blog ? (proposer les categories existantes du site, definies dans hugo.toml ou visibles dans content/blog/). L'utilisateur DOIT choisir une categorie, ne pas passer cette etape.

Si l'utilisateur ne fournit qu'un mot-cle sans prompt, l'aider a formuler le prompt GEO correspondant (transformer le mot-cle en question naturelle).

### Type d'article

Lister les templates disponibles en lisant le contenu de `.claude/templates/articles/` et les presenter a l'utilisateur.

Types disponibles par defaut :

| Type | Fichier | Description | Quand l'utiliser |
|------|---------|-------------|-----------------|
| **Article standard** | `article-standard.md` | Article informatif complet, optimise SEO + GEO | Intention informationnelle (c'est le type par defaut) |
| **Comparatif** | `geo-comparatif.md` | Article de classement/comparatif avec mise en avant d'une marque ou d'un produit | Intention comparative ("meilleur X", "X vs Y", "classement") |

**IMPORTANT :** Toujours lire le contenu reel de `.claude/templates/articles/` au moment de l'execution. Si de nouveaux fichiers template ont ete ajoutes par l'utilisateur, les proposer aussi. Le dossier fait reference, pas ce tableau.

Si l'utilisateur ne sait pas quel type choisir, l'aider en analysant l'intention de recherche du prompt.

### Informations complementaires selon le type

- **GEO Comparatif (OBLIGATOIRE)** : l'utilisateur DOIT fournir la marque, le produit ou la solution a mettre en avant, ainsi que 2-3 concurrents a comparer. Ce type d'article sert a positionner favorablement un element par rapport aux autres dans un comparatif objectif. Ne jamais rediger sans cette information

## Etape 2 — Audit des contenus existants et maillage interne

### Analyse du site

Lister tous les articles existants dans `content/blog/` en lisant le sitemap (`content/plan-du-site.md` ou directement les fichiers dans `content/blog/`). Pour chaque article existant, noter :
- Le titre
- Le mot-cle principal (visible dans le title et le nom du fichier)
- La categorie

### Verification de cannibalisation

Verifier qu'aucun article existant ne cible deja le meme mot-cle principal ou le meme prompt GEO. Si cannibalisation detectee, prevenir l'utilisateur et proposer un angle different.

### Identification des liens internes

Identifier **au minimum 3 articles existants** thematiquement proches du nouvel article pour le maillage interne. Privilegier :
1. Les articles de la **meme categorie**
2. Les articles qui partagent des **tags communs**
3. Les articles dont le sujet est **complementaire** (pas identique, mais lie)

**Regles de maillage interne :**
- **Minimum 3 liens internes** vers d'autres articles du blog, inseres de maniere contextuelle dans le corps de l'article
- L'**ancre du lien** (le texte cliquable) doit contenir le **mot-cle principal de l'article cible**, pas de "cliquez ici" ou "lire aussi"
- Les liens doivent etre **naturels et contextuels** : inseres dans une phrase qui a du sens, pas en liste en bas de page
- Repartir les liens dans differentes sections de l'article (pas tous au meme endroit)

Exemple :
- Si l'article cible s'appelle "Les bienfaits du the vert", l'ancre doit etre quelque chose comme : "Comme nous l'avons vu dans notre article sur les **[bienfaits du the vert](/blog/bienfaits-the-vert/)**, ..."
- PAS : "Pour en savoir plus, [cliquez ici](/blog/bienfaits-the-vert/)"

Si le site a moins de 3 articles, faire le maximum avec ce qui existe. Si le site est vide (premier article), noter dans un commentaire les futurs liens a ajouter quand d'autres articles seront publies.

## Etape 3 — Redaction

Lire le template correspondant dans `.claude/templates/articles/[type-choisi].md` et l'utiliser comme squelette.

### Frontmatter

| Champ | Regle |
|-------|-------|
| `title` | **= prompt GEO cible** (la question naturelle posee aux moteurs IA). C'est le H1 de l'article. Max ~60 caracteres. Contient le mot-cle principal si possible, mais la formulation naturelle du prompt prime |
| `description` | Meta description optimisee pour la SERP. Max 140 caracteres, contient la query fan-out (mot-cle SEO). Peut differer du title/H1 car elle est optimisee pour le clic Google |
| `date` | Date du jour (YYYY-MM-DD) |
| `lastmod` | Date du jour (YYYY-MM-DD), identique a `date` a la creation. Mettre a jour si l'article est modifie plus tard |
| `categories` | La categorie choisie |
| `tags` | 3-6 tags pertinents |
| `author` | Nom de l'auteur (lire dans le CLAUDE.md section "Contexte du site" > auteur) |
| `image` | Chemin vers l'image hero (optionnel, ex: `/images/blog/mon-article.jpg`). Utilisee dans le hero, og:image et le schema Article |
| `imageAlt` | Texte alt de l'image (obligatoire si `image` est present) |
| `faq` | Liste de questions/reponses pour le schema FAQPage JSON-LD (min. 3). Format YAML : `- question: "..." answer: "..."`. Les questions doivent correspondre a celles de la section FAQ dans le body |
| Nom du fichier | Slug = query fan-out en minuscules, tirets, sans accents |

### Regles GEO (Generative Engine Optimization)

Ces regles sont fondamentales pour que l'article soit cite par les moteurs IA generatifs :

| Regle | Detail |
|-------|--------|
| **H1 = prompt cible** | Le H1 (title) est TOUJOURS le prompt exact que l'utilisateur pose aux moteurs IA. C'est une question naturelle, pas un mot-cle optimise |
| **Query fan-out dans le body** | La query fan-out (mot-cle SEO) doit apparaitre dans le premier paragraphe et dans les variations des H2 |
| **1 paragraphe = 1 idee** | Chaque paragraphe traite d'une seule idee distincte. Ne jamais melanger plusieurs concepts dans un meme paragraphe. Cela facilite l'extraction par les LLMs |
| **Quick summary auto-suffisant** | Le bloc "En bref" est le bloc le plus critique : les LLMs l'extraient en priorite. Il doit etre auto-suffisant (comprehensible seul) et contenir les faits cles avec des donnees chiffrees |
| **H2 explicites et descriptifs** | Pas de titres vagues. Chaque H2 doit etre auto-suffisant et comprehensible hors contexte de l'article |
| **Donnees chiffrees obligatoires** | Integrer des donnees chiffrees dans chaque section (prix, pourcentages, statistiques, durees). Les LLMs extraient les faits verifiables en priorite |
| **Tableaux extractibles** | Au moins 1 tableau structurant les informations cles. Les tableaux sont extraits en priorite par les IA generatives |
| **Citation sourcee obligatoire** | Au moins 1 citation d'une etude, d'un organisme ou d'un expert, avec source et annee. Renforce la credibilite et la citabilite |

### Regles communes a tous les types

| Spec | Valeur |
|------|--------|
| H1 | Jamais dans le body (genere par Hugo depuis le title) |
| Densite mot-cle | 1-2% (query fan-out) |
| Mots-cles en gras | Oui, `**mot-cle**` |
| Ton | Impersonnel (pas de je/tu/nous/vous) sauf si precise autrement dans le CLAUDE.md |
| Liens internes | Min. 3 liens contextuels vers des articles existants (ancre = mot-cle de l'article cible) |
| FAQ | 3-5 questions en fin d'article |
| Separateurs | JAMAIS de separateur horizontal (---) entre les sections |
| Tirets | JAMAIS de tiret cadratin ni demi-cadratin. Utiliser des virgules, des points ou reformuler |

### Regles specifiques par type

Lire les commentaires HTML `<!-- NOTES POUR CLAUDE -->` en bas du template choisi. Ils contiennent les specs propres a ce type d'article :
- Nombre de mots minimum
- Blocs obligatoires (quick summary, tableaux, citations, etc.)
- Objectif editorial
- Ton specifique eventuel

**Toujours suivre ces notes.** Elles priment sur les regles communes si conflit.

## Etape 4 — Verification (checklist)

- [ ] Slug = query fan-out en minuscules, tirets, sans accents
- [ ] Title = prompt GEO cible (question naturelle), < 60 caracteres
- [ ] Meta description <= 140 caracteres, contient la query fan-out (mot-cle SEO)
- [ ] Auteur renseigne dans le frontmatter
- [ ] H1 = prompt cible (verifie que c'est bien une question naturelle, pas un mot-cle)
- [ ] Query fan-out presente dans le premier paragraphe
- [ ] Structure Hn conforme au type (voir notes du template)
- [ ] H2 explicites et auto-suffisants (comprehensibles hors contexte)
- [ ] 1 paragraphe = 1 idee distincte (pas de paragraphes multi-idees)
- [ ] Nombre de mots minimum atteint (voir notes du template)
- [ ] Donnees chiffrees presentes dans chaque section
- [ ] Mots-cles en gras
- [ ] Ton correct
- [ ] Min. 3 liens internes contextuels (ancres = mots-cles des articles cibles)
- [ ] Blocs obligatoires presents selon le type
- [ ] Quick summary "En bref" auto-suffisant avec donnees chiffrees
- [ ] Au moins 1 tableau recapitulatif
- [ ] Au moins 1 citation sourcee (source + annee)
- [ ] FAQ presente avec balises `<details>/<summary>` (accordeon) dans le body
- [ ] FAQ presente dans le frontmatter (champ `faq`, min. 3 questions) pour le schema FAQPage JSON-LD
- [ ] Les questions FAQ du frontmatter et du body correspondent
- [ ] `image` et `imageAlt` renseignes si une image est disponible
- [ ] Pas de separateur horizontal (---) ni de tiret cadratin/demi-cadratin
- [ ] Build Hugo OK (`hugo`)

## Etape 5 — Sauvegarde et build

```bash
hugo
```

Afficher a l'utilisateur :
- Type d'article utilise
- Prompt GEO (H1)
- Query fan-out (mot-cle SEO)
- Nombre de mots
- Nombre de H2
- Liens internes ajoutes
- Proposer de voir le resultat en local (`hugo server`)

## Etape 6 — Enregistrement dans MEMORY.md

Ajouter une ligne dans `MEMORY.md` a la racine du projet pour tracer la publication. Le fichier est organise par semaine (lundi a dimanche).

Format du fichier :

```markdown
# Journal de publication

## Semaine 15 (06/04/2026 - 12/04/2026)
- 2026-04-07 | Titre de l'article | Categorie

## Semaine 14 (30/03/2026 - 05/04/2026)
- 2026-04-01 | Mon premier article | Thes verts
- 2026-04-03 | Autre article | Tisanes
```

Regles :
- Utiliser le numero de semaine ISO et les dates lundi-dimanche
- Creer une nouvelle section de semaine si elle n'existe pas encore
- Les semaines les plus recentes en haut du fichier
- Une ligne par article : date | titre | categorie

## Etape 7 — Push sur GitHub

Si un remote git est configure (le site a ete mis en ligne via `/github-setup`), commit et push automatiquement :

```bash
git add -A
git commit -m "Ajout article : [titre de l'article]"
git push
```

Informer l'utilisateur que l'article sera en ligne dans 1-2 minutes (deploiement automatique via GitHub Actions).

Si aucun remote n'est configure, ne rien faire (le site est en local uniquement).
