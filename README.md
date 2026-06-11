# MEMOIRE-Σ — Mémoire de Session Compressée

> Compression et restauration automatique de sessions de travail humain-IA.
> Un seul fichier. Zéro infrastructure. Le bloc EST l'outil.

---

## Concept

Chaque session de travail avec un LLM se termine par une perte : le contexte
disparaît, et la session suivante recommence par un long récapitulatif — coûteux
en tokens, imprécis, incomplet.

**MEMOIRE-Σ** ferme ce cycle. À la clôture d'une session, le skill produit
automatiquement un **Bloc-Session** compressé (≤ 200 tokens) que l'utilisateur
colle dans son journal. À la session suivante, ce bloc suffit à reprendre le
travail — sans récapitulation, sans question, sans perte des points critiques.

```
Session N
   ↓  clôture détectée → Bloc-Session généré automatiquement
journal-sessions.md  ← l'utilisateur colle le bloc
   ↓  session N+1 : bloc fourni → contexte restauré + synthèse validée
Session N+1 reprend exactement où N s'est arrêtée
```

---

## Origine

MEMOIRE-Σ généralise le sous-système mémoriel du skill
[CRC-R (Cycle de Résonance Cognitive Récursif)](https://github.com/Othman-Benbrahim/CRC-R-Cycle-de-Resonance-Cognitive-Recursif) —
qui archivait ses cycles résonants en signatures STÈLE — hors de son domaine
d'origine, vers **toute session de travail** : code, rédaction, recherche,
conception, symbolique.

La v1.1 intègre les résultats d'une analyse multi-corpus (93 papiers sur les
protocoles de transmission médicale I-PASS/SBAR, 82 sur la mémoire externe LLM,
76 sur les boundary objects) — voir [Fondements scientifiques](#fondements-scientifiques).

---

## Installation

Un seul fichier, aucune dépendance.

```
skills/
└── memoire-sigma/
    └── SKILL.md
```

Placer `SKILL.md` dans le dossier skills de Claude Code, ou charger son contenu
en instructions de projet dans Claude.ai.

L'utilisateur crée lui-même son journal : un simple fichier texte
`journal-sessions.md` où il colle les blocs produits. C'est tout.

---

## Utilisation

### Fin de session — compression automatique

Le skill détecte les signaux de clôture ("on s'arrête là", "on continue demain",
remerciement final) et produit le bloc **sans qu'on le demande**. Commande
explicite également disponible : `⊜` ou "archive la session".

```
=== SESSION ===
Date       : 11/06/2026
Domaine    : code
Objet      : refonte du plugin RSS
Acquis     :
  - signal_engine.py opérationnel, stdlib uniquement
  - bug de corruption fichier résolu
Décisions  :
  - parsing maison — feedparser trop lourd (écarté : lxml)
Ouvert     :
  - [!] détection de doublons défaillante sur flux > 500 items
  - [ ] icônes des catégories à harmoniser
Contingence: si le flux X repasse en 403, basculer sur le miroir — éviter blocage
Prochain   : corriger la détection de doublons
Signature  : ⟦⊙∿⊛⟁◐⟧ Ouvert
Mots-codes : SOURCE.MOUVEMENT.NOEUD.FRACTURER.CONDITIONNEL
Feedback   : 
===============
```

### Début de session — restauration validée

Coller le bloc (ou fournir le journal). Le skill reconstruit le contexte et
**synthétise sa compréhension en une ligne** — l'utilisateur valide ou corrige
avant la reprise (mécanisme *receiver synthesis*, hérité d'I-PASS) :

```
Session restaurée — refonte du plugin RSS. Compris : moteur opérationnel,
un blocage critique sur les doublons, on attaque ça en premier.
Correct ? (ou corrige-moi avant qu'on reprenne)
```

### Récupération guidée

Si le journal complet est fourni avec un objectif ("on reprend le travail sur X"),
le skill balaie **tout le journal** et sélectionne les blocs pertinents — même
anciens, même non consécutifs. Aucun index, aucun embedding : le modèle est
lui-même le moteur de récupération.

### Feedback inter-sessions

Après la session, l'utilisateur peut annoter le champ `Feedback` :

| Signal | Sens | Effet à la session suivante |
|---|---|---|
| `[+] élément` | acquis validé | considéré comme solide |
| `[-] élément` | remis en cause | re-questionné avant usage |
| `[!] élément` | critique | traité en priorité |
| `[?] élément` | incertain | clarification demandée |

---

## Le Bloc-Session

### Champs

| Champ | Rôle | Obligatoire |
|---|---|---|
| Date, Domaine, Objet | Identification | Oui |
| Acquis | Faits établis, livrables (3-6, télégraphique) | Oui |
| Décisions | Choix + raison + alternatives écartées | Oui |
| Ouvert | Tâches en suspens, **avec priorité** `[!]` `[*]` `[ ]` | Oui |
| Contingence | "Si condition, alors action" | Si risque identifié |
| Sources | Fichiers, URLs mobilisés | Si pertinent |
| Prochain | L'étape suivante convenue | Oui |
| Signature | Chaîne STÈLE + statut (Clos / Ouvert / Bifurqué) | Oui |
| Feedback | Annotations utilisateur post-session | Optionnel |

### Budget et règle de sacrifice

**≤ 200 tokens.** En cas de dépassement, ordre de sacrifice :
Sources → alternatives écartées → acquis anciens.
**Jamais les `[!]` ni la contingence** — ce qui est critique ne se sacrifie pas.

### Format canonique strict

Clés fixes, ordre fixe, préfixes littéraux : le bloc texte est directement
parsable par machine, sans représentation JSON séparée (principe du
*boundary object* : un seul artefact, deux lectures). Export JSON sur demande.

### Signature STÈLE

Chaque bloc se clôt par une signature glyphique de 3 à 12 signes — coordonnée
symbolique de la session, pas résumé. Elle encode la nature de la session
(substances), ce qui s'est accompli (opératives), et sa configuration (modales).

```
⟦⊙∿◯⊜⟧ Clos       → objectif clair, travail, livrable, session aboutie
⟦Ø✶⥀◐⟧ Bifurqué   → exploration, découverte, changement de cap, suspendue
⟦◊⊛⟁↻◯⟧ Clos      → reprise, nœud récurrent, blocage cassé, forme produite
```

La table de mapping complète (25 primitives, 4 strates) est embarquée dans le
skill. Le skill [STÈLE](https://github.com/Othman-Benbrahim) chargé à côté
enrichit la lecture des signatures, mais n'est **pas requis**.

### Détection de motifs inter-sessions

À la restauration (≥ 3 blocs lus), le skill relève — sans analyser :
- un glyphe récurrent ≥ 3 fois → **motif cristallisé** du projet
- un `[!]` identique sur 3 sessions → **blocage critique persistant**, remonté en tête
- un `◐` persistant → sessions systématiquement inachevées → périmètre à réduire

---

## Fondements scientifiques

La v1.1 résulte d'une analyse croisée de trois corpus (251 papiers, via SciSpace) :

**I-PASS / SBAR (transmission médicale)** — La structure mnémonique fixe, les
priorités explicites, les plans de contingence et la *synthèse par le receveur*
sont les mécanismes validés en essais multicentriques pour réduire les omissions
de transmission (Starmer et al., *NEJM* 2014). MEMOIRE-Σ les transpose au
handoff humain-IA — avec le receveur inversé : c'est le modèle qui synthétise,
l'humain qui valide.

**Mémoire externe et compression contextuelle LLM** — Le Bloc-Session est un
*resume block* au sens de la littérature (état de session sérialisé en bloc
compact restituant décisions et rationales). La compression lossy y est
documentée comme inévitable ; MEMOIRE-Σ l'assume par conception et la discipline
par l'ordre de sacrifice.

**Boundary objects (Star & Griesemer, 1989)** — Le bloc est conçu comme un
artefact-frontière : structure stable et reconnaissable, interprétable à la fois
par l'humain (lecture), la machine (parsing du format canonique) et l'écosystème
symbolique (signature STÈLE).

Le rapport d'analyse complet est disponible dans ce dépôt :
[`rapport_amelioration_memoire_sigma.md`](./rapport_amelioration_memoire_sigma.md)

---

## Principes

**Compression fidèle** — Le bloc préserve la structure, pas le détail.
Il permet la reprise du travail, pas la relecture de la conversation.

**Automatisme discipliné** — Le déclenchement automatique sert l'utilisateur,
il ne le surprend pas. En cas de doute sur la clôture : proposer, ne pas imposer.

**Le journal appartient à l'utilisateur** — Le skill produit le bloc ; coller,
organiser et annoter le journal reste un geste humain. C'est ce geste qui fait
la mémoire, pas le mécanisme.

**Vérifier avant de reprendre** — Une ligne de synthèse validée coûte 10 tokens ;
une restauration faussée coûte la session.

**Honnêteté épistémique** — Ce skill modélise une mémoire de travail
externalisée. Les signatures STÈLE sont des coordonnées symboliques, pas des
preuves ni des prédictions.

---

## Écosystème

MEMOIRE-Σ s'inscrit dans l'écosystème IRIS∞ :

- **[CRC-R](https://github.com/Othman-Benbrahim/CRC-R-Cycle-de-Resonance-Cognitive-Recursif)** — origine du mécanisme mémoriel
- **STÈLE** — alphabet des 25 primitives, source des signatures
- Compatible avec tout autre skill : MEMOIRE-Σ archive la session quel que soit
  le travail effectué dedans

---

## Roadmap

```
v1.0 ✓  Extraction du mécanisme CRC-R, généralisation tout domaine
v1.1 ✓  Intégration corpus scientifiques (I-PASS, LLM memory, boundary objects)
v1.2    Validation en conditions réelles — mesure des taux d'omission
        à la restauration, ajustement du budget
v2.0    (Exploratoire) Agrégation de journal : rapport de trajectoire
        de projet généré depuis N blocs
```

---

## Licence

GPL-3.0

## Auteur

**Othman BEN BRAHIM**
[github.com/Othman-Benbrahim](https://github.com/Othman-Benbrahim)

---

*MEMOIRE-Σ v1.1 — Juin 2026*
