# CONTEXT.md — Il Viaggio delle Lettere

> 🤖 Auto-généré | 📅 2026-10-23 00:05 UTC
>
> **Pour charger** : "Lis https://raw.githubusercontent.com/antoineflohubert/il-viaggio-context/main/CONTEXT.md"

---

# Instructions Projet

# Viaggio Lettere

Interactive educational game teaching Italian syllables and letters to young children. Built with React + Vite, backed by Firebase and OpenAI Whisper for speech processing.

## Tech Stack

- **Frontend**: React 18, Vite 5, Tailwind CSS 3
- **Backend**: Firebase (Firestore, Auth, Hosting, Cloud Functions)
- **Speech**: OpenAI Whisper API (transcription), Web Speech Synthesis (TTS)
- **Runtime**: Node.js 22 for Cloud Functions

## Project Structure

```
src/
  components/    # React UI components (GameScreen, CorsaGame, WorldMap, etc.)
  hooks/         # Custom hooks (useFamily, useProgress, useSpeech, useWhisperSpeech, etc.)
  config/        # Firebase initialization and service layer
  data/          # Static data (worlds.js - 6 educational worlds)
  utils/         # Utilities (weightedSelection.js - spaced repetition)
  App.jsx        # Main app component and routing
functions/       # Firebase Cloud Functions (Whisper transcription endpoint)
```

## Règles de gouvernance

- **Pas de décision d'architecture structurante sans validation utilisateur.** Toute décision qui introduit une nouvelle dépendance, modifie le modèle de données Firestore, change la stratégie d'authentification, ajoute un service externe, ou modifie les workflows CI/CD doit être soumise à l'utilisateur avant implémentation. En cas de doute sur le caractère structurant, demander.

### Règle 1 — Chargement systématique sur action Knowledge Module

Toute action sur un Knowledge Module (lecture d'un use case, modification de `functional.md`/`technical.md`, écriture de code dans la zone couverte par le module, ajout d'ADR module, tests couvrant le périmètre) déclenche le parcours suivant **avant** tout autre travail :

1. Lire `README.md` à la racine du repo
2. Lire `knowledge/ubiquitous-language.md` (glossaire unique du BC `syllabic-learning`)
3. Parcourir `knowledge/modules/{name}/adr/index.md` (si présent)
4. Parcourir `docs/adr/index.md` (ADRs repo)

Cette règle est **sémantique, pas procédurale** : c'est systématique à chaque fois qu'un module est touché, pas optionnel. Elle se combine avec le step Contexte du flux spec-driven (voir § 1. Contexte).

### Règle 2 — Ubiquitous Language

Voir [`knowledge/ubiquitous-language.md`](knowledge/ubiquitous-language.md). Aucun terme métier ne doit être introduit sans y être défini. La **redéfinition de termes au niveau Knowledge Module est interdite** — si un terme manque, l'ajouter à l'UL. Si le besoin de redéfinir apparaît, c'est le signal que le BC est mal découpé (hors-scope actuel).

### Règle 3 — Structure Knowledge Module

Chaque Knowledge Module sous `knowledge/modules/{nom}/` contient :
- `functional.md` (obligatoire)
- `technical.md` (obligatoire, en paire avec functional)
- `adr/` avec `index.md` (optionnel)

**Aucun dossier `specs/` au niveau Knowledge Module** — les specs vivent dans `specs/pending|implemented/`.

**Aucun dossier `docs/features/` ni `docs/epics/`** — il n'y a pas de documentation knowledge aux niveaux Feature et Epic du backlog ; la traçabilité passe par les specs (User Stories) qui référencent les modules impactés.

### Cohérence knowledge / code

La knowledge base est la référence documentée de l'état attendu de l'application. Le code est l'état réel. Des écarts peuvent exister. Deux régimes gèrent leur détection et résolution :

**Régime 1 — Écarts détectés par les tests E2E (comportemental).** Les E2E sont la source de vérité comportementale. Quand un E2E hors-scope casse pendant un flux spec-driven, la règle de non-régression s'applique (voir § Code). Quand la knowledge est ajustée pour refléter le comportement réel révélé par les E2E : préparer toutes les modifications knowledge mais **ne pas committer**. Présenter un récap groupé à l'utilisateur dans le chat :
- Chaque écart constaté (ce que dit la knowledge vs ce que fait le code)
- Les modifications knowledge préparées
- La perception : régression ou évolution légitime non documentée

L'utilisateur valide, puis seulement on commit. Si rework demandé, ajuster avant de committer.

**Régime 2 — Écarts non capturés par les E2E (non-fonctionnel).** Data contracts, performance, services back, rules Firestore, Cloud Functions, configuration... tout ce qui n'a pas de test E2E comme filet. Ce régime repose sur un **devoir de vigilance** : à chaque fois que Claude lit du code et de la knowledge dans un même contexte (step Contexte, debug, analyse, review...), il compare activement. On accepte que ce régime a des trous — c'est de la vigilance, pas de la garantie. Quand un écart est détecté :
1. **Remonter à l'utilisateur** — décrire l'écart, jamais corriger silencieusement
2. **Qualifier la perception** :
   - **Évolution non documentée** (le code fait plus/mieux/pareil) → proposer un update knowledge
   - **Régression perçue** (le code fait moins : moins performant, moins de cas supportés, moins de comportements) → proposer un fix code pour re-matcher la knowledge
3. **L'utilisateur arbitre** — Claude ne tranche jamais seul sur régression vs évolution

## Flux de développement (spec-driven)

Toute évolution fonctionnelle ou technique suit ce flux. L'ensemble est piloté depuis Claude Code.

### 1. Contexte

Avant de coder, charger le contexte pertinent :
- Appliquer systématiquement la **Règle 1** (chargement parental : README, ubiquitous-language, index ADR module + repo) pour chaque module touché
- Lire `standards/index.md` → charger les pages standards pertinentes
- Lire `knowledge/index.md` → identifier les Knowledge Modules pertinents et charger leurs `functional.md` + `technical.md`. Les pages cross-cutting (`knowledge/cross-cutting/*.md`) sont chargées si la spec touche l'infra ou le tooling
- **Pour une feature** : vérifier si une branche `design/*` correspondante existe (`git branch -r --list 'origin/design/*'`). Si oui, consulter le code de cette branche comme **référence visuelle** pour comprendre l'UX validée. Ne jamais merger ni cherry-pick le code design — il sert uniquement à voir ce qui a été validé visuellement, pas comme base de code
- **Vigilance knowledge/code** (régime 2) : ce step est le moment naturel pour comparer activement la knowledge chargée avec le code lu. Tout écart détecté doit être remonté à l'utilisateur (voir § Cohérence knowledge / code)
- **Validated date** : quand une page knowledge est confirmée à jour (contenu toujours fidèle au code malgré des modifications source), mettre à jour le commentaire `<!-- validated: YYYY-MM-DD -->` en fin de page. Ce marqueur se commit avec le reste du workflow — pas de commit séparé

### 2. Spec

Créer la spec dans `specs/pending/` depuis le template approprié :
- **Feature** : `specs/templates/feature.md` → `specs/pending/features/[nom].md`
- **Technical** : `specs/templates/technical.md` → `specs/pending/technical/[nom].md`

**Rédaction basée sur la knowledge** : les comportements attendus (Given/When/Then) doivent s'appuyer sur les pages knowledge chargées à l'étape 1. Vérifier les règles métier existantes, les edge cases documentés, les dépendances fonctionnelles et les états/transitions déjà en place. Les critères doivent être cohérents avec l'état actuel de la knowledge — ne jamais contredire ou ignorer une règle documentée sans le signaler explicitement.

**Complétude BDD** : les critères doivent couvrir la totalité des comportements de la feature — happy path, edge cases, états d'erreur. Ce sont les specs d'implémentation, pas un résumé.

**Data contract** : si la feature touche Firestore (nouveau champ, nouvelle collection, modification de rules) ou les Cloud Functions, remplir les sections Data contract et Répartition front/back du template. Chaque élément du data contract doit pointer vers la page knowledge technique qu'il impacte (`→ knowledge/modules/{module}/technical.md § [section]` ou `→ knowledge/cross-cutting/{page}.md § [section]`).

**Pointage knowledge exhaustif** : le pointage `→ knowledge/modules/{module}/{functional|technical}.md § [section]` ne se limite pas aux AC. Le data contract, la répartition front/back, et les notes techniques doivent aussi pointer vers leurs pages knowledge cibles. Toute information de la spec destinée à enrichir la knowledge doit avoir un pointage explicite.

**Front-matter Knowledge Modules impactés** : chaque spec déclare dans son front-matter YAML les Knowledge Modules qu'elle touche, avec `impact_type: primary | secondary`. Au moins un `primary` requis (sauf Epic architectural — hors-scope actuel).

**Règle de découpage** : une spec = un scope atomique. Si une spec impacte plus de 3 pages knowledge (au total, modules + cross-cutting), la découper en sous-specs indépendantes.

**Check ADR** : avant de soumettre la spec, évaluer si la solution proposée justifie un nouvel ADR ou l'amendement d'un ADR existant.

Processus :
1. Scanner la spec à travers la grille de signaux ci-dessous
2. Comparer avec les ADRs existants : index repo (`docs/adr/index.md`) **et** les indexes des modules touchés (`knowledge/modules/{M}/adr/index.md`) — est-ce un sujet nouveau, ou une évolution d'un ADR en place, au repo ou au module ?
3. Agir selon le résultat :
   - **Signal fort** → rédiger l'ADR (nouveau ou amendement) depuis `docs/adr/000-template.md` et l'afficher dans le chat après la spec, pour validation conjointe
   - **Signal faible** → signaler en une ligne sous la spec ("Pas d'ADR proposé, mais à noter que [observation]") — l'utilisateur décide
   - **Aucun signal** → ne rien mentionner

Grille de signaux :

| Signal | ADR | Exemples projet |
|--------|-----|-----------------|
| Nouvelle dépendance externe (lib, service, API) | Nouveau | Ajout Stripe, migration Whisper → Deepgram |
| Nouveau pattern architectural non couvert par un ADR | Nouveau | Event sourcing (cf. ADR-015 module `progression-mastery`), WebSocket, SSR |
| Changement de stratégie data (nouveau modèle Firestore, migration schema) | Nouveau ou amend ADR-007 | Nouvelle collection avec pattern différent de l'existant |
| Changement de stratégie CI/CD ou testing | Nouveau ou amend ADR-005/013 | Nouveau type de test, changement de pipeline |
| Changement d'approche auth/sécurité | Amend ADR-010 | Passage auth non-anonyme, ajout rate limiting |
| Abandon ou remplacement d'un choix documenté dans un ADR | Amend → deprecated + nouveau | Remplacement de Tailwind, abandon Web Audio synthesis |
| Feature utilisant exclusivement des patterns existants | Non | ambient-noise (Web Audio + localStorage existants) |
| Refactoring sans changement de pattern | Non | Extraction hook, renommage, cleanup |
| Optimisation perf sans nouvelle stratégie | Non | Mémoisation, lazy loading d'un composant |
| Ajout d'une constante, config, ou paramètre de tuning | Non | Nouveau seuil, nouveau flag |

**Test décisif pour les specs techniques** : un ADR est proposé seulement si la solution introduit un pattern ou une contrainte qui affectera les futures décisions. Question : *"Est-ce qu'un développeur qui arrive dans 6 mois a besoin de comprendre POURQUOI on a fait ce choix plutôt qu'un autre ?"* Si oui → ADR. Si la réponse est évidente depuis le code → non.

**Format dans le chat** : quand un ADR est proposé, l'afficher après la spec, clairement séparé. L'utilisateur valide spec et ADR indépendamment (spec ✓ + ADR ✓, ou spec ✓ + ADR ✗).

Afficher le contenu complet de la spec dans le chat et attendre la validation de l'utilisateur.

### 3. Code

- Créer une branche `feat/[nom]` ou `refactor/[nom]`
- Implémenter selon la spec, les standards et la knowledge
- Feature : implémenter dans `src/`, créer tests unitaires, ajouter `data-testid`
- Technical : modifier `src/`, assurer la non-régression

**Règle de non-régression E2E hors-scope** : si l'implémentation d'une spec casse des tests E2E en dehors du scope de la spec, il est **interdit** de corriger les E2E pour les faire passer. C'est le signal d'une incohérence entre la knowledge et l'état réel de l'application. Flux en cas de cassure hors-scope :
1. **Stop** — ne pas continuer l'implémentation
2. **Root cause analysis** — identifier pourquoi la knowledge a mené à une spec qui entre en conflit avec le comportement existant
3. **Fix knowledge** — mettre à jour les pages knowledge pour refléter l'état réel de l'application
4. **Fix spec** — mettre à jour la spec pour être cohérente avec la knowledge corrigée
5. **Re-implémenter** — reprendre l'implémentation sur la base corrigée
6. **Récap avant commit** — présenter un récap groupé des ajustements knowledge à l'utilisateur pour validation avant de committer (voir § Cohérence knowledge / code — Régime 1)

### 4. Knowledge

Mettre à jour la knowledge base en intégrant **toute la connaissance portée par la spec** — pas seulement les AC :
- **AC / Comportement attendu** → Règles métier, États & transitions, Cas limites, Flux UX
- **Motivation / Problème** → Section Objectif des pages knowledge (le "pourquoi" donne du sens aux règles)
- **Data contract** → `knowledge/modules/{module}/technical.md` ou `knowledge/cross-cutting/firebase-infrastructure.md`
- **Répartition front/back** → Section Fichiers source des pages fonctionnelles + pages techniques des modules
- **Hors scope** → Section Cas limites ou Dépendances fonctionnelles (les décisions de ne PAS faire sont de la connaissance)
- **Notes techniques / Risques** → `knowledge/modules/{module}/technical.md` concernés

Processus :
- Modifier les pages `knowledge/modules/{M}/{functional|technical}.md` et/ou `knowledge/cross-cutting/{nom}.md` impactées (listées dans la spec via les pointages `→`)
- Créer les nouvelles pages si nécessaire
- **Relire chaque page modifiée dans son intégralité** et la réécrire si nécessaire pour maintenir la cohérence narrative. L'objectif n'est pas d'ajouter des lignes à une page existante mais de produire un document qui se suffit à lui-même comme référence 360°.
- **Mettre à jour `knowledge/index.md`** à chaque création, modification ou suppression de page : ajouter/supprimer la ligne du tableau, ajuster la description si les concepts-clés changent. L'index est un aiguillage (mots-clés de routage), pas un résumé — le détail est dans les pages elles-mêmes
- Si les modifications touchent plus de 2 sections d'une même page, afficher la **page complète** dans le chat (pas seulement le diff) pour validation. Sinon, afficher le diff.

**Intégrité des pointages** : le linter `scripts/lint-knowledge-pointers.mjs` valide tous les pointages `→` entre knowledge pages (cross-références). Il s'exécute automatiquement au pre-commit quand des fichiers `knowledge/` ou `specs/` sont staged. Si un renommage de section casse des pointages cross-pages, propager le renommage dans tous les fichiers référençant l'ancienne section avant de re-committer. Commandes : `npm run lint:pointers` (validation standard) ou `npm run lint:pointers:all` (inclut les specs archivées).

**Santé de la knowledge base** : le linter `scripts/lint-knowledge-health.mjs` audite trois dimensions — pages stale (fichiers source modifiés après la page), redondance (H3 partagés ou fichiers source chevauchants entre pages), et complexité (seuils dérivés des contraintes LLM). Non-bloquant — outil d'audit manuel. Voir ADR-021. Commandes : `npm run lint:knowledge` (tout) ou `--stale`, `--redundancy`, `--complexity`.

### 5. Finalize

- Déplacer la spec : `specs/pending/` → `specs/implemented/`
- Ajouter la référence PR dans la spec
- Si un ADR a été validé à l'étape 2 : le committer au niveau approprié (repo dans `docs/adr/` OU module dans `knowledge/modules/{M}/adr/`), mettre à jour l'index ADR correspondant (`docs/adr/index.md` ou celui du module)
- **Validation knowledge** : avant le commit final, lancer un agent indépendant (sans contexte de l'implémentation) qui reçoit la spec complète et les pages knowledge mises à jour, et vérifie : (a) tous les éléments de la spec sont capturés dans les pages knowledge, (b) aucune contradiction interne entre les pages modifiées, (c) les pointages cross-pages sont cohérents. Si des manques sont identifiés, corriger avant de finaliser.
- Commit final, créer la PR, et **merger la PR** dans `main` sans demander confirmation (la validation utilisateur a lieu en amont, lors de l'approbation de la spec et des commits)

### Chemin CI alternatif

Les workflows `implement-spec.yml` et `update-knowledge.yml` permettent le même flux sans CLI : une spec pushée sur `main` dans `specs/pending/` déclenche l'implémentation automatique par Claude Code Action, et le merge de la PR archive les specs et met à jour la knowledge.

## Knowledge Base

Documentation fonctionnelle et technique de l'application dans `knowledge/`, structurée en **Knowledge Modules** sous le BC `syllabic-learning`.

- **`knowledge/ubiquitous-language.md`** — Glossaire unique du BC (entités, commands, events, états, faux amis). Référence sémantique obligatoire.
- **`knowledge/index.md`** — Index plat routage (modules + cross-cutting + artefacts BC)
- **`knowledge/modules/{name}/functional.md` + `technical.md`** — Paire obligatoire par module. 6 modules : `family-identity`, `world-navigation`, `gameplay`, `progression-mastery`, `speech`, `companion-breaks`.
- **`knowledge/modules/{name}/adr/`** — ADRs spécifiques au module (optionnel, avec `index.md` si présent)
- **`knowledge/cross-cutting/{nom}.md`** — Pages techniques transverses à plusieurs modules (firebase-infrastructure, ci-cd, dev-tooling, design-sessions, spec-system)

## Standards

Principes de design et UX pour enfants 4-5 ans dans `standards/`.

- **`standards/pages/UI-CHILDREN-4-5.md`** — Touch targets, typography, couleurs, animations, audio, layout
- **`standards/pages/UX-CHILDREN-4-5.md`** — Navigation, feedback, progression, onboarding, scaffolding, sécurité contenu
- **`standards/index.md`** — Index avec résumés

## Development

```bash
npm run dev          # Start dev server (port 5173)
npm run build        # Run tests then production build
npm run preview      # Preview build (port 4173)
npm run test         # Run unit tests in watch mode
npm run test:run     # Run unit tests once
npm run test:e2e     # Run E2E tests — Chromium only
npm run test:e2e:smoke  # Smoke E2E — Chromium only — runs on pre-commit
npm run test:e2e:full   # Full E2E — 3 browsers (Chromium + WebKit + Android) — runs in CI
npm run test:e2e:ui  # Run E2E tests with interactive UI
npm run lint:knowledge      # Audit knowledge health (stale, redundancy, complexity)
npm run knowledge:validate  # Mark knowledge page(s) as validated
```

### Fins de ligne (LF)

`.gitattributes` force LF depuis la PR #27, mais un dossier de travail extrait avant garde ses fichiers en CRLF (Git for Windows, `core.autocrlf=true`) et des tests de scripts échouent alors au pre-commit. **En arrivant dans un dossier de travail**, vérifier :

```bash
git ls-files --eol | grep 'w/crlf' | grep -v 'eol=crlf'   # doit être vide
```

S'il liste des fichiers, les convertir sur place (garde les modifications non commitées) :

```bash
git ls-files --eol | grep 'w/crlf' | grep -v 'eol=crlf' | cut -f2 | xargs -d '\n' sed -i 's/\r$//'
```

Ne jamais convertir un fichier à la main ni committer de CRLF : `git add --renormalize .` ne doit rien changer.

## Testing

- **Always** create a `.test.jsx` file for every new component, hook, or utility
- Use **vitest** + **React Testing Library** (`@testing-library/react`, `@testing-library/user-event`)
- Test descriptions (`describe`, `it`) in **English**
- Tests are **co-located** next to the source file (e.g., `components/Gufo.test.jsx`)
- **Render-without-crash + key interactions** — no exhaustive branch coverage
- **Never assert on exact Italian text** — verify structure, roles, element presence instead
- **No snapshot tests**
- Shared mock helpers live in `src/test/` (Firebase, Web Audio API, SpeechSynthesis, etc.)
- Web Audio / SpeechSynthesis APIs must be **stubbed** in jsdom (not skipped)
- Naming convention: `ComponentName.test.jsx` / `hookName.test.jsx` / `utilName.test.jsx`
- Setup file: `src/test/setup.js` (imports `@testing-library/jest-dom`)

### E2E Tests (Playwright)

- E2E tests live in `e2e/` (separate from unit tests in `src/`)
- Use **Playwright** with custom fixtures defined in `e2e/fixtures/test-utils.js`
- Four fixtures: `appPage` (fresh app), `withFamily` (family created), `withFamilyNoTutorial` (tutorial not done), `withProfile` (family + profile → WorldMap)
- Mocks in `e2e/fixtures/mocks.js`: getUserMedia, AudioContext, Whisper route interception
- **No Firebase mock needed** — Playwright config forces local mode via env var overrides
- **No SpeechSynthesis mock needed** — works silently in headless Chromium
- Use `data-testid` attributes for reliable selectors (not Italian text)
- Test descriptions in **English**
- When adding a new screen or major UI flow, add corresponding E2E smoke tests
- **Pre-commit hook** runs unit tests + E2E smoke (Chromium) — blocks commit on failure. **Exception branches `design/*`** : skip unit tests, ne lance que les E2E staged (ajoutés/modifiés dans le commit)
- **CI pipeline** (`.github/workflows/deploy.yml`) runs full E2E on 3 browsers — blocks Firebase deploy
- **Do NOT run tests manually** — the pre-commit hook handles it. Never run `vitest`, `playwright test`, or any test command after finishing work. Just commit and let the hook validate.
- See ADR-013 for architecture decisions

## Accessibility (WCAG 2.1 AA)

This project targets **WCAG 2.1 Level AA** compliance. Follow these rules for every UI change:

- **Semantic HTML**: use `<button>`, `<h1>`–`<h6>`, `<label>`, `<nav>`, `<dialog>` — not `<div onClick>`
- **ARIA attributes**: `aria-label` on icon-only buttons, `aria-modal="true"` + `role="dialog"` on overlays, `role="progressbar"` with `aria-valuenow/min/max`, `aria-hidden="true"` on decorative elements
- **Live regions**: use `AriaLive` provider + `useAnnounce()` hook to announce dynamic changes (state transitions, success/failure) to screen readers
- **Focus management**: use `useFocusTrap()` on modals and dialogs; ensure Escape closes overlays; never trap focus without an escape route
- **Keyboard support**: all interactive elements must be reachable and operable via keyboard (Enter/Space to activate, Escape to dismiss)
- **Color + shape**: never convey meaning by color alone — always pair with an icon, text, or shape indicator (e.g., SyllableGrid uses color AND `✓`/`●` icons)
- **Touch targets**: use `ChildTapButton` (or equivalent min 44×44px) for all tap targets — children need generous hit areas
- **Timed content**: provide pause/stop controls for auto-advancing content (CorsaGame pause, `corsaTimingPercent` setting); satisfies WCAG 2.2.1
- **Text alternatives**: `role="img"` + `aria-label` on SVG illustrations (Gufo, decorative backgrounds)

## Key Patterns

- Functional React components with custom hooks for state/logic
- All UI text is in Italian
- Props-based data flow, no external state management library
- Tailwind utility classes + custom CSS animations in index.css
- Environment variables prefixed with `VITE_` (see .env.example)
- Firebase Cloud Functions deployed to europe-west1

## Design Sessions (UI Refinement)

Voir [ADR-020](docs/adr/020-design-sessions.md) pour le flux complet (start → iterate → finalize), les conventions de branches `design/*`, les tests allégés, et le cleanup automatique.

## Architecture Decision Records (ADRs)

Les ADRs existent à **deux niveaux** (DELTA-015) :

- **Repo** (`docs/adr/`) — décisions structurantes du BC entier. Index : [docs/adr/index.md](docs/adr/index.md) (16 ADRs).
- **Module** (`knowledge/modules/{M}/adr/`) — décisions spécifiques à un module. Indexes :
  - `speech` : [knowledge/modules/speech/adr/index.md](knowledge/modules/speech/adr/index.md) (008, 012)
  - `progression-mastery` : [knowledge/modules/progression-mastery/adr/index.md](knowledge/modules/progression-mastery/adr/index.md) (009, 011, 015)
  - `world-navigation` : [knowledge/modules/world-navigation/adr/index.md](knowledge/modules/world-navigation/adr/index.md) (014)

**Règle de placement** : un ADR est placé au niveau le plus bas où sa portée s'applique. Un ADR module ne peut pas contredire un ADR repo ; en cas de conflit, le repo prévaut.

**Règle d'index** (DELTA-016) : le dossier `adr/` est optionnel à tout niveau. **Mais si un dossier `adr/` existe, un fichier `index.md` y est obligatoire** et doit lister tous les ADR du dossier (ID, titre, statut, résumé). Cette règle vaut pour `docs/adr/` comme pour `knowledge/modules/{M}/adr/`.

**Consulter les ADRs existants avant toute décision architecturale** (index repo + index du module touché — voir § Règle 1).

Pour ajouter un ADR : copier `docs/adr/000-template.md`, numéroter à la suite (prochain libre : 023, numérotation séquentielle à travers tout le repo), placer au niveau approprié, et mettre à jour l'index correspondant.

---

# Knowledge Base Index

# Knowledge Base — Il Viaggio delle Lettere

BC : `syllabic-learning`. Routage **simple** (index plat) — le nombre de pages est bien sous le seuil d'escalade 100-150 (DELTA-012).

Ordre de chargement systématique pour tout agent qui agit sur un Knowledge Module (règle DELTA-017 / CLAUDE.md § Règle 1) :

1. [README.md](../README.md) à la racine du repo
2. [ubiquitous-language.md](ubiquitous-language.md) — glossaire unique du BC
3. `adr/index.md` du module touché (si présent)
4. [docs/adr/index.md](../docs/adr/index.md) — index ADR repo

## Knowledge Modules

| Module | Functional | Technical | Périmètre (mots-clés de routage) |
|---|---|---|---|
| family-identity | [→](modules/family-identity/functional.md) | [→](modules/family-identity/technical.md) | Rule `hasOnly` étendue à `introducedStagesByLanguage`, `seenWordsByLanguage` et `wordGameAnnouncedByLanguage` (reset les vide, déploiement manuel), Famiglia code 6 chiffres, Profili avatars + Lingua + genre grammatical facultatif (accord des messages enfant), saisie du genre à la création (`GenderPicker`), réglages genre / Lingua dans Settings (`updateProfile` optimiste + rollback, sélecteur de Lingua si plusieurs Pacchetti), reset complet toutes Lingue (`ResetProfilo`), consentement vocal RGPD, espace parent (Settings, ParentTutorial avec barre d'action fixe et reprise à l'étape mémorisée `parentTutorialStep`, FAQ propres à chaque Lingua, liens légaux par Lingua, section Crediti : attribution CC BY 4.0 de ZIPA `ZIPA_CREDIT`), long-press 3s, multi-device via `arrayUnion(deviceId)`, contraste AA des écrans parent (fond `from-purple-700` `PARENT_BACKGROUND_CLASS`, textes secondaires ≥ `text-white/90`, boutons de confirmation `bg-orange-700` / `bg-red-600`, réponses du tutoriel `green-700` / `orange-700`, `parentScreenColors.js`, garde-fou `contrast.test.js` qui lit le code) |
| world-navigation | [→](modules/world-navigation/functional.md) | [→](modules/world-navigation/technical.md) | Routing par état React, guide de page de la WorldMap (premier Mondo jouable, Tesori ; après la notification « Nouveau jeu ») et de ModeSelect (Gioca, Corsa, retour ; aide Silero 20 s repoussée), ModeSelect en grandes icônes sans texte, sans emoji du Mondo ni nombre de Syllabe, sous-titre du Mondo bonus `sr-only` (tient sans défilement à 360 × 640), bouton retour standard `BackButton`, WorldMap sans défilement (360 × 740 / 390 × 844 / 360 × 640 `short:`, paysage au mieux `landscape:`, Profilo haut gauche, Gufo haut droite, boutons de Mondo rectangulaires 2 colonnes avec ratio dans le coin haut droit, Tesori et « Mots » en icônes sans libellé, titre `sr-only`, pas de lien de reset), Mondo verrouillé tappable (`aria-disabled`, cadenas qui se balance, phrase « plus tard » `world_locked`), contraste AA des boutons de Mondo et Tesori (`contrast.js`, dégradés assombris), du prénom du Profilo et du numéro de version (voile `bg-black/20` sur le fond de l'app), exceptions assumées aux standards (pulsations Tesori et badge « +N », boucles de la notification « Nouveau jeu », textes 12 / 16 px, numéro de version 12 px, marge basse ; animation de Gufo : companion-breaks), `worstTextContrast` (texte semi-transparent, voiles empilés), pas de message de Gufo sur la WorldMap, écran de chargement avec le nom de l'app, WorldMap hub (Mondi du Pacchetto Lingua actif, 6 en `it`, 9 en `fr` draft, progression, compteur Tesori ; bouton « Mots » caché / verrouillé / badge « +N », notification plein écran « Nouveau jeu » sans texte écrit, bouton ✓ avec indice « tape ici »), écran `wordGame` sans Mondo, ModeSelect (attente de Silero au premier lancement : sablier, icône atténuée, `aria-disabled`, tap sans lancement + Gufo THINKING, annonce « Giochi pronti! », aide Gufo à 20 s, pas de sablier sous 300 ms / non supporté / échec, retour à ModeSelect après consentement), thèmes Corsa, déverrouillage séquentiel, overlays modaux, remontage `key={pack.id}` et retour WorldMap au changement de Lingua, orientation (APK Android verrouillé en portrait par le manifeste, web non verrouillé, exception WCAG 1.3.4) |
| gameplay | [→](modules/gameplay/functional.md) | [→](modules/gameplay/technical.md) | Écran toujours allumé pendant les jeux (`useScreenWakeLock`, API Screen Wake Lock : Gioca, Corsa pause comprise, jeu de mots, résultats ; rendu en quittant, redemandé au retour au premier plan, refus et navigateurs sans l'API ignorés, APK non vérifiée, sans plugin natif), Sans réseau (spec X 6 : écran « Pas de réseau » `OfflineScreen` au démarrage hors ligne ou sur raison `offline`, Sessione en pause, essai non compté ; « Joue avec un adulte » en Gioca et Mots : ✓ / ✗ de l'adulte en appui long 1 s `HoldButton`, bandeau, Attempts `source: 'adult'`, introduction de Tappa sans Attempt, voix à la carte suivante au retour du réseau `useOfflinePlay` ; Corsa en pause sans option adulte ; moteur sur l'appareil prioritaire), Jeu de mots (`WordGameScreen`, `WordCard` : Sessione de 6 mots pondérés, imprimerie minuscule sans emoji, aides visuelles qui s'estompent (couleurs de syllabes, lettres muettes grises et fines, pointillés), accepted → mot dit + emoji + Attempt `{lang}:word:{wordId}`, 3 échecs ou 3 incertains → lecture du compagnon syllabe par syllabe sans Attempt, présentation non comptée d'un palier ≥ 2, cible `word.oral` / G2P, résumé « + N nouveaux mots », écran sans texte pour l'enfant et emplacement d'état fixe `word-state-slot` comme Gioca, règle du palier dite sans être écrite et présentation à la mise en page de la première carte, `100dvh` + compactage sous 720 px), Introduzione di Tappa avant la 1re carte (`StageIntroduction` : Sillaba-modello, animation de fusion, graphèmes allumés `GraphemeLetters` / `fusion.js`, phrase de Tappa, `extraModels`, « Tocca a te! » avec le micro montré par l'indice « tape ici » `TapHint` jusqu'au premier tap ou à sa fermeture, phrases dites sans sous-titre, emplacement d'état fixe `stage-intro-state-slot` pour toutes les phases et le mode adulte, répétition non comptée, ✕ / Échap vers la WorldMap), échos jamais comptés (items présentés hors des 2 premières cartes, tap hors READY sans audio, grille qui retire la carte, TEACHING par la fusion), mot de référence à la réussite Gioca (« la… comme lapin » : `playSequence` cible + liaison `connectorLike` / `connectorIn` calculée à l'exécution + mot, emoji du mot révélé au mot, rotation après Maestria, `referenceWord.js`, SFX seul au-delà de 3 réussites, avance à la fin de la séquence, repli « Bravo » sans `referenceWord`), emoji masqué pendant la lecture (emplacement neutre, révélé au succès / à l'enseignement ; Corsa sans emoji), Sessione Gioca (« Parla! » puis micro ouvert, « Parla! » limité aux 10 premières fois `parlaCount`, consigne d'arrivée `gameready` aux 3 premières Sessioni `gameReadyCount`, tap Gufo en READY, indice « retape » après 4 s : aide `gameready` + micro qui rebondit + main 👆 (`TapHint`), fondu de la voix au tap, écran tenu en `100dvh` et compacté sous 720 px, machine à états READY→SPEAKING→LISTENING→PROCESSING→SUCCESS\|PROMPTING\|TEACHING, attente de transcription : micro barré + Gufo THINKING + bulle de réflexion, annonce unique, 8 Syllabe, 3 tentatives, Attempt seulement au succès / TEACHING / tap carte, essai non entendu ou incertain jamais compté, message `uncertain`), carte Gioca dans le style affiché (cursive en police du Pacchetto, Attempt sous ce style, nom accessible identique), Corsa (sans écran de départ ni bouton « Inizia! » : Presentazione del gioco aux 3 premiers lancements `gameIntroCount` dans `CorsaGame`, « Ancora! » compris, puis départ direct, 60 s, au plus 6 cartes de 96 px tirées par sélection pondérée parmi les Tappe ouvertes et introduites ou la première Tappa, carte cible de Gufo pondérée, Gufo vole toujours : jamais l'emplacement quitté ni une carte sans verdict ou aux étoiles en cours, carte jouée gardée jusqu'à son verdict puis remplacée par un item non affiché (succès : à la fin des étoiles ; sortie 250 ms, sans animation en mouvement réduit), échéance au départ de la carte suivante (remplacée sans son, verdict tardif compté en silence ; après la fin de la course, course précédente ou écran quitté : succès compté pour la progression seule sous la Sessione de sa course, résultats figés), pause : jeu figé (ni vol ni micro, verdicts appliqués à la reprise, Gufo repart), grille 3 × 2 paysage / 2 × 3 portrait dans les safe zones, `corsaCards.js`, `CORSA_MAX_CARDS`, lab `corsa.maxCards`, MAJ seule sans cursive, timer par Syllaba compté depuis l'ouverture du micro (5s, plancher 3s, parole en cours jamais coupée, verdict attendu), succès seuls enregistrés, pas de déverrouillage), Attempts par mode, SessionSummary (guide de page après les phrases enchaînées, variante « Mots » `word_summary`, ✓ en rond, retour standard, bouton 🔄 seul ; phrases de Gufo enchaînées sans superposition, phrase « progression » tant que le Profilo n'a aucun Tesoro dans la Lingua : icône 🏆, dite et annoncée, jamais écrite ; fin de Sessione annoncée, pastille 💬, titre `sr-only`), CorsaResults (phrase de fin annoncée `corsaEndPhraseKey`, pastille 💬, titre `sr-only` ; guide de page : signe vert, 🔄, retour ; ✓ en rond vert, retour standard, bouton 🔄 seul sans texte ; trophée MAJ ≥ 4), Gufo volant inline, étoiles du succès sur la carte Corsa (SFX 3), pastille d'attente Corsa (micro barré, sans rotation, écoute prioritaire, masquée pendant la pause), confettis ≥ 60%, écran Gioca sans score visible (grille des Syllabe à la demande exceptée), écran Gioca sans texte pour l'enfant (barre du Mondo sans compteur, bouton micro sans libellé, message `uncertain` dit et annoncé sans être écrit) et emplacement d'état de hauteur fixe `state-slot` (voix / adulte : Gufo et carte immobiles), micro barré « \ » (`rotate-45`), transcription en dev seulement (journal lab en build lab), version affichée (12 px, exception assumée), contraste de la grille des Syllabe (consigne et légende `text-white/90` sur `bg-black/60`, tuiles colorées sous le seuil non traitées) |
| progression-mastery | [→](modules/progression-mastery/functional.md) | [→](modules/progression-mastery/technical.md) | Mondi par Lingua (6 `it`, 9 `fr` en 41 Tappe), scoring ±1 par Stile d'écriture [0-5] (`maj` / `min` / `cursive_min`), clé `{lang}:{worldId}:{letter}:{style}` (`progressKey.js`), progression par Lingua (filtre Lingua attiva), `successes` / `successSessionIds` / `lastSource` (`adult` : réussites validées hors réseau, sans indicateur), `sessionId`, migration locale v2→v3, Maestria paramétrable par Pacchetto (`mastery.js`, règle `sessions` : 3 réussites sur 2 Sessioni distinctes et score courant ≥ 2 (`minScore`) par style d'imprimerie, `it` compris ; règle `score` ; Tesori préservés par la migration), Tappe (`stages.js`, Tappa implicite, Sillaba-modello, `stageUnlock`, `unlockedStagesByLanguage`, `unlockStage`, Tappe introduites `introducedStagesByLanguage` / `markStageIntroduced` / `viaggio-introduced`, tirage limité aux Tappe introduites), déverrouillage des Mondi `worldUnlock` (`percent` 60 / `lastStage` de `fr`, Gioca seul) par Lingua (`unlockedWorldsByLanguage`), Parole (`words.js`, mots lisibles / seulement entendus, prérequis, paliers de lettres muettes P0-P5, clé `{lang}:word:{wordId}`), déblocage des mots (`wordGame.js` : compteur d'ouverture à 15 mots P0-P1, paliers ouverts à la fin d'un Mondo, jamais de reverrouillage, `seenWordsByLanguage` / `wordGameAnnouncedByLanguage` + marquage local de secours, niveaux d'aide dérivés des réussites, poids d'un mot + facteur de rappel), Risposta non contata, styles d'écriture (`writingStyles.js` : registre, `cursive_min` débloquée par MIN acquise, `cursive_maj` hors v1 règle 2/3 par Mondo, choix du style affiché, polices cursives Playwrite IT Trad / FR Trad auto-hébergées, cursive sans effet sur Tesori), sélection pondérée (Gioca et Corsa) sur le plus faible score des styles disponibles parmi les Tappe ouvertes, facteur de rappel temporel (`RECALL_STEPS`, jours calendaires, max < 5, lab `progression.recall.*`), Carta di ripasso inter-Mondi (`selectionPool.js`, 1 sur 8, scorée sous son Mondo), pool < 4 complété par le Mondo précédent, reset toutes Lingue, rules permissives (backlog prod #4), Tesori, Album (vide : Gufo dit comment gagner des Tesori, `treasures_empty`, à chaque ouverture, rien d'écrit (🔮, titre `sr-only`) ; guide de page : premier Tesoro puis retour standard) + mode spectacle « Mostra a tutti! » (boutons en grandes icônes 🎬 / ✕ / 🔄 / ✓, fin du défilement redessinée, dialogue modal `TreasureShow` fermé au tap n'importe où ou par Échap, focus rendu à 🎬, « Ancora! » sans flash, fond assombri au contraste AA `treasureShowColors.js` / `worstTextContrastOnGradient`, « Ha collezionato » 24 px, frise qui ne rebondit plus après la fin), compteurs de Tesori = progression et non score, CF processAttempt en transaction serveur + shim clés historiques |
| speech | [→](modules/speech/functional.md) | [→](modules/speech/technical.md) | Whisper Cloud Function (whisper-1, warmup 4min, silence 600ms, max 5s), détection de parole Silero VAD par défaut en prod (ADR-031 amendé : `initSilero` au lancement, états `idle`/`loading`/`ready`/`unsupported`/`failed` + `useSileroReadiness`, test de capacités WASM SIMD + AudioWorklet, nouvelle tentative 30 s, contexte audio au tap `prepareSileroAudio`, chien de garde 300 ms, décision sur probabilités par trame `createSileroDetector` 0.5 / 0.35 / 600 ms, micro ouvert après « Parla! » en Gioca (armement retiré), inférence à vide, assets versionnés `vad/<vad-web>-<ort>/`, voix de fond = limite acceptée, pas de Web Speech en mode Silero), repli détection historique spectre + Web Speech, essai non entendu jamais compté (offline/error/no-speech/hallucination/empty ; `offline` → écran « Pas de réseau », moteur sur l'appareil prioritaire `canRecognizeWithoutNetwork`), signaux de capture (`vadFallbackReason`, `vadFrames`, `vadReadyMs`, `vadInferMsP50/P95`, `shadowDetection`), réglages lab `speech.*` (`speech.detection` défaut `silero`), journal `speech.attempt` / `speech.engine`, vérification sur appareil (critère de merge), moteurs de transcription (ADR-030 : `whisper-1` / `gpt-transcribe`, shadow mode, repli, budget, `promptMode`, `noSpeechProb` / `avgLogprob`, échéance whisper-1 2027-02-26), moteur phonème sur l'appareil `phoneme-local` lab (ADR-032 : wav2vec2 XLSR-53 q4, Cache Storage, worker ORT, CTC masqué + `engineScore`), second moteur `phoneme-zipa` lab et prod (ADR-034, ADR-036 : ZIPA-CR-small int8 71 Mo CC BY 4.0 avec attribution, registre `PHONEME_MODELS`, fbank Kaldi 80 bandes `fbank.js` validé contre lhotse, `tokens.txt`, blank `▁` / `ː`, tilde de nasalité recollé, alias ɾ ɕ ʑ, `speech.zipaEngine` coché par défaut, votant en verdict combiné (en prod aussi, modèle auto-hébergé `/models/zipa-<rev8>/`, téléchargé après le consentement vocal, ADR-036), observateur avec un moteur seul, ≈ 57 ms contre ≈ 1,6 s), verdict combiné par défaut (ADR-035 : OU whisper-1 + gpt-transcribe + ZIPA prêt, `transcribeCombined`, `matcherForEngine`, répétitions, liste blanche par son `foreignReadings`), matcher à trois issues `accepted`/`rejected`/`uncertain` (ADR-025 : `legacy` exact > sous-chaîne > variantes, défaut `it` ; `phonetic` forme phonétique + classes + oppositions strictes + noms de lettres, module `phonetics` par Pacchetto ; `low_confidence` via seuils `speech.uncertain.*` ; `speech.matcher` + shadow `match.shadow` ; message `uncertain` ; crochets `preprocess` / `optionalFinalOmission` / `hesitations` / `letterNameScope` token / `elisions` / `repetitionWithinWord`, `parseIpa`), cible explicite d'un mot (`item.phonemes` : `word.oral` ou G2P du texte, jeu de mots), phonétique française prête (ADR-026 : G2P maison par règles + lexique d'exceptions, cibles API de l'annexe, classes T1-T11, noms de lettres par token, « euh » incertain, `src/content/fr/`), hallucinations d'un seul mot par égalité, `fr` dans la liste blanche CF (redéploiement manuel), paramètres vocaux par Lingua (`speech` / `voice` du Pacchetto, voix TTS paresseuses par locale, hallucinations par Lingua, casse locale), audio `public/audio/{lang}/` (targets / companion / words / fusion, slug ASCII `audioSlug`, manifeste + `generate_voices.py`), audio `fr` pré-généré (spec FR 4 : Google Cloud TTS `fr-FR-Neural2-F` / Boubou `fr-FR-Neural2-G`, SSML `<phoneme>` des cibles API, `oral` des mots, fusion au débit 25 % + repli, /ɲ/ → [nj], « ː » refusé, idempotent + `--force`, caractères facturés, journal `scripts/audio-generation/`, contrôle de durée 150 ms à 6 s `mp3Duration.js`, `REVIEW.md` à écouter avant activation), cibles MP3 sans repli TTS (`audio.missing`, test de présence `draft` compris sauf `DRAFT_PACKS_WITHOUT_AUDIO`), compagnon MP3 + TTS secours, `playSequence` sans trou (`onSegmentStart`, coupé par `stop`, segment manquant sauté sans TTS, `audio.sequence.long` > 2 s, liaisons `connectorLike` / `connectorIn` sans TTS), effets sonores CC0 (`sfx/`, Freesound / Kenney, montage ffmpeg, succès / succès court / échec « Aww » / « Go! » / tapes micro, Tesoro joué une fois par la notification, URL versionnées `?v=SFX_VERSION` contre le cache), détection bruit RMS 0.06, Presentazione del gioco (écran de Gufo sans bouton ni texte visible, phrase annoncée au lecteur d'écran) aux 3 premiers lancements de chaque jeu (`gameIntroCount_{jeu}_{profileId}`, amorçage depuis `gameIntroSeen` / `corsaIntroCount` / `gameReadyCount`, jeu lancé 1 s après la voix, 3 à 15 s, voix Gioca = consigne de lecture « Leggi la carta e dilla forte! » + endroit calme, voix Corsa = consigne explicite + endroit calme, MP3 `it` de `game_intro_*` et `noise_warning` + exception de durée 6,95 s, contexte `corsa_intro` retiré, journal `it.json`), escalade 3/12, phrase « progression » du résumé Gioca (`progress_hint`), `useSound.stop` coupe la voix du compagnon (MP3 + TTS) et les lectures interruptibles, `onStart(durée)` pour l'allumage des graphèmes, `playFusion` / `speakSyllable` résolvent `false` si le MP3 manque, `speakGufoSpecial`, `gufo_your_turn.mp3`, phrases du guide de page `voice.help.guide_*` (MP3 `it` / `fr`) |
| companion-breaks | [→](modules/companion-breaks/functional.md) | [→](modules/companion-breaks/technical.md) | Guide de page (`usePageGuide` : phrases courtes qui montrent chacune un élément par l'indice « tape ici » `TapHint`, auto aux 10 premières visites ou 1×/jour les 3 premiers jours de jeu, `pageGuide_{profileId}` localStorage, avis « toccami », tap Gufo = guide (nom accessible `companion.guideAriaLabel` ; pendant le guide `aria-disabled` + description `companion.speaking`), phrases du guide et avis annoncés au lecteur d'écran phrase par phrase (`useAnnounce`), élément montré décrit par sa phrase (`highlightDescription`, `TapHint` `description` → `aria-describedby` vers un texte `hidden`, focus inchangé), règle « phrases du compagnon dites, jamais écrites » (règle générale : pas de sous-titres ; « Pas de réseau » exception documentée, exceptions restantes listées dans les standards UX § 9), annonce des phrases hors guide `useCompanionAnnounce` (fin de Sessione, fin de course, Tesoro, pause, aides d'une phrase), pastille 💬 sans indisponibilité `voicePlaying`, écran de pause sans texte écrit, coupure au tap avec fondu, voix automatiques jamais superposées, écrans guidés WorldMap / ModeSelect / CorsaResults / résumés / Album, option `always` de l'Album vide), Gufo 6 humeurs (NEUTRAL, HAPPY, EXCITED, ENCOURAGING, LISTENING, THINKING = attente de transcription Gioca), animation permanente de Gufo selon l'humeur = exception assumée au standard d'animation (tous les écrans, figée en mouvement réduit), compagnon de l'introduction de Tappa (« Tocca a te! »), nom par Lingua (Gufo, Boubou), phrases parlées (neutres, 8 mots max), SVG inline, GufoClickable, GufoFlying non branché (Gufo Corsa inline), rappel pause 20 min à la place de la WorldMap, jamais pendant un écran de jeu (audio `gufo_help_pause`), `corsaTimingPercent` = durées Corsa seulement |

## Artefacts de niveau BC

| Artefact | Localisation | Rôle |
|---|---|---|
| Ubiquitous Language | [ubiquitous-language.md](ubiquitous-language.md) | Glossaire unique (entités dont « écran de jeu », commands, events, états, faux amis) |
| ADR index repo | [../docs/adr/index.md](../docs/adr/index.md) | Décisions structurantes du BC (19 ADRs) |
| Standards transverses | [../standards/index.md](../standards/index.md) | UI-CHILDREN-4-5, UX-CHILDREN-4-5 |

## Cross-cutting pages techniques

Pages techniques transverses à plusieurs modules (infrastructure, tooling, méta-process) — hors structure des Knowledge Modules.

| Page | Description |
|---|---|
| [firebase-infrastructure](cross-cutting/firebase-infrastructure.md) | Hosting (cache immutable `/vad/**` et `/models/**` (modèle ZIPA, `.onnx` en `application/octet-stream`), vérification des types MIME après déploiement, quota de transfert), Firestore (4 sous-collections families→users→progress+attempts, `labTrials` du lab avec TTL `expireAt` 90 jours, champs Profilo `language` / `unlockedWorldsByLanguage` / `introducedStagesByLanguage` / `seenWordsByLanguage` / `wordGameAnnouncedByLanguage`), attempts `source` `asr` / `adult` (mode « Joue avec un adulte », rule déjà déployée), CF transcribeAudio (`language` / `prompt` / `engine` / `keywords`, listes blanches, 400, `verbose_json`) + processAttempt (shim i18n), migration i18n (script dry-run/copy/cleanup, ordre de bascule), 8 env vars Vite, canaux live/design/lab, rules (blocage par défaut, `language` / `grammaticalGender` typés à la création et à la modification, `introducedStagesByLanguage`, `seenWordsByLanguage` et `wordGameAnnouncedByLanguage` dans `hasOnly` à déployer à la main avant le Hosting, `progress/` ouvert au client, backlog prod #4) |
| [ci-cd](cross-cutting/ci-cd.md) | 8 workflows GitHub Actions (deploy + canal lab, contrôle des marqueurs lab dans `dist/`, modèle ZIPA récupéré au build avec cache CI `zipa-model-<révision>`, preview, specs, knowledge, design, cleanup, notifications ntfy.sh) |
| [language-packs](cross-cutting/language-packs.md) | Pacchetti Lingua (ADR-023) : clés retirées `modeSelect.syllableCount` / `newTreasure.badge`, `src/content/`, registre + import dynamique, statut `draft` / `available` (draft = lab seulement, jamais choisi automatiquement ; audio `fr` généré, activation par l'utilisateur après `REVIEW.md`), Pacchetti disponibles (`it` ; `fr` draft : 9 Mondi, 41 Tappe, 174 items, 299 Parole, `silentTiers`, `wordGame` (validés : `validateWordGame`, règles des paliers `wordGame.tier.{n}`), messages complets au neutre, typographie française, FAQ et tutoriel FR, voix de Boubou), Lingua attiva (Profilo > device > repli fr/it), `LanguagePackProvider` / `useLanguagePack`, changement à chaud, `<html lang>`, Mondo bonus générique, `getNextWorldId`, dictionnaire de messages (`useT` / `t`, clé `gameIntro.ready` retirée, genre `{ f, m, neutral }`, `useProfileGender`, repli `it`), noms de Mondo et de thème en messages, compagnon par Lingua, pages légales par Lingua (politique de confidentialité § 3.3 : reconnaissance ZIPA sur l'appareil), sections `speech` / `voice` / `audioLanguage` / `phonetics` (+ `speech.defaultMatcher`), police d'emojis embarquée (Noto Color Emoji réduite), Pacchetto `test` lab-only pseudo-localisé (`__VIAGGIO_TEST_PACK__`, deux Tappe dans le Mondo 2), couleurs de Mondo AA (texte blanc ≥ 4,5:1), schéma Tappe / `mastery` / `stageUnlock` / `worldUnlock` / `connectorCompare` (`ignoreAccents` `it`, `strict` `fr`) / Parole (`words`, `referenceWord`, `rotationWords`) / `styles` + `cursiveFont` (police cursive auto-hébergée), `introPhrase` / `extraModels` / `fusionTimings` / `graphemes`, 2 à 15 items par Tappa, invariants `validatePack` + `validateModel` + `validateStyles` |
| [lab](cross-cutting/lab.md) | Panneau lab hors prod (ADR-022) : mode testeur `lab.testerMode` (bonne réponse / je me trompe / partie normale non mesurée), faux positifs et solution « au moins un moteur » du tableau de bord, onglets Réglages / Tableau de bord, essais copiés dans Firestore `labTrials` 90 jours (ADR-033, `labTrialSync.js`, export JSON retiré), appareil de test `device` (`labDevice.js`), réglages `lab.testerMode` et `lab.unlockAll` (« Tout débloquer », coché par défaut en lab, sans effet en prod), tableau de bord de la reconnaissance vocale (`LabDashboard.jsx`, `speechStats.js` : solutions vs bonne réponse, par appareil, par graphème), `VITE_ENABLE_ADMIN`, exclusion du build prod (moteurs phonème `phoneme-local` lab-only / `phoneme-zipa` lab et prod (ADR-036), `speech.zipaEngine` déclaré en lab seulement ; Silero n'est plus lab-only), registre de réglages `defineLabSetting`/`useLabSetting`, statuts et actions `defineLabStatus`/`defineLabAction` (« Modèle ZIPA » + attribution CC BY 4.0), journal d'essais `logTrial` (`speech.attempt`, `speech.engine`, `audio.missing`, `audio.sequence.long`, `match.shadow`, `selection.recall`), réglages `progression.recall.*` et `corsa.maxCards`, appui long 2s, canal Hosting `lab` |
| [dev-tooling](cross-cutting/dev-tooling.md) | Vite 5 (plugin `vadAssets` : assets Silero versionnés dans tous les builds, `__VAD_ASSET_DIR__` ; plugin `zipaModel` : modèle ZIPA `models/zipa-<rev8>/`, `VITE_ZIPA_MODEL=off`), Vitest jsdom, Playwright 3 browsers, pre-commit hook, Tailwind CSS 3, garde-fous du dictionnaire (couverture, scan des clés, pseudo-localisation, police d'emojis) |
| [design-sessions](cross-cutting/design-sessions.md) | Branches design/* preview Firebase 7j, code = référence visuelle, cleanup auto |
| [spec-system](cross-cutting/spec-system.md) | Templates feature/technical, cycle pending→implemented, pointage knowledge, flux CLI vs CI |

---

# Standards Index

# Standards — Il Viaggio delle Lettere

## Structure

| Standard | Description |
|----------|-------------|
| [pages/ui-guidelines.md](pages/UI-CHILDREN-4-5.md) | UI design specs: touch targets, typography, color, animation, audio, layout |
| [pages/ux-guidelines.md](pages/UX-CHILDREN-4-5.md) | UX patterns: navigation, feedback, progression, onboarding, accessibility |

## Résumés

### UI-CHILDREN-4-5.md

Spécifications opérationnelles pour le design d'interfaces destinées aux enfants de 4-5 ans. Couvre les tailles de zones tactiles (min 64×64px), la gestion des événements touch adaptée aux jeunes enfants (tolérance 300ms, 30px de mouvement), le drag & drop avec zones de drop généreuses, la typographie (min 24px, polices arrondies), la palette de couleurs (vives, pas de rouge pour les erreurs), l'iconographie (remplie, simple, littérale), les animations (ease-out/bounce, jamais linéaire), le design audio (feedback multimodal), les états visuels des éléments interactifs, le layout avec zones sûres, les transitions d'écran, et les considérations d'accessibilité motrice et visuelle. Deux standards propres à Viaggio Lettere : l'**indice « tape ici »** (élément qui rebondit + main 👆, `TapHint`, seul moyen de montrer un élément à l'enfant) et le **bouton retour** (haut gauche, rond, blanc opaque, 64 px, sans texte visible, `BackButton`).

### UX-CHILDREN-4-5.md

Principes UX pour les interfaces enfants 4-5 ans (pré-lecteurs). Couvre l'architecture d'information (max 2 niveaux, max 4 choix par écran), le modèle d'interaction (tap prioritaire, pas de double-tap/pinch/rotate), les systèmes de feedback (multimodal, jamais punitif, encouragement constant), la progression et motivation (récompenses immédiates, jamais de perte), l'onboarding sans texte (démonstration animée par le personnage), l'interaction vocale (débit 0.85x, max 8 mots), le scaffolding adaptatif (indices progressifs après inaction), l'intégration parents (accès caché par triple-tap), la gestion d'état (sauvegarde automatique permanente), et la sécurité du contenu (pas de liens externes, pas de PII).
