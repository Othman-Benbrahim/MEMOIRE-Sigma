---
name: memoire-sigma
version: 1.0
description: >
  MEMOIRE-Σ — Compression et restauration automatique de session, tout domaine.
  Généralise le sous-système mémoriel du CRC-R (Étape 0A + Étape 6) hors du cycle
  résonant. Deux déclencheurs : (1) RESTAURATION — dès qu'un message contient un
  bloc "=== SESSION ===" ou qu'un fichier journal-sessions.md est fourni, reconstruire
  le contexte sans demander de récapitulation ; (2) COMPRESSION — automatique, dès
  que la session montre des signes de clôture ("ok c'est bon", "on s'arrête là",
  "on continue demain", "merci c'est parfait", remerciement final) OU sur commande
  explicite ("⊜", "archive la session", "compresse la session", "bloc de session"),
  produire le Bloc-Session. Fonctionne pour toute session : code, symbolique,
  rédaction, recherche, conception. Aucun outil externe requis : le bloc EST l'outil.
---

# MEMOIRE-Σ — Mémoire de Session Compressée

> *« Le CRC-R archive ses cycles. MEMOIRE-Σ archive tout. »*

Extraction généralisée du mécanisme mémoriel du CRC-R : un cycle fermé
lecture → travail → écriture, où le bloc compressé est à la fois l'archive
et le point de restauration. Pas de transpileur, pas d'éditeur, pas d'IA locale.
Le journal de l'utilisateur est l'unique infrastructure.

---

## Principe fondateur

La compression est **lossy par conception** — héritée du CRC-R :
le bloc préserve la **structure** de la session (acquis, décisions, ouvertures),
pas son détail. Il permet la **reprise du travail**, pas la relecture de la conversation.

Règle d'or : un bloc doit suffire à reprendre la session **sans poser une seule
question de récapitulation**. Si une information est nécessaire à la reprise,
elle est dans le bloc. Sinon, elle est consentie comme perdue.

---

## DÉCLENCHEUR 1 — RESTAURATION (début de session)

S'active si le message contient un bloc `=== SESSION ===` (collé directement)
ou si un fichier `journal-sessions.md` est fourni.

### Protocole

1. **Lire** le ou les blocs fournis. Si journal complet : lire les **5 dernières entrées**.
2. **Reconstruire silencieusement** : domaine, objet, acquis, points ouverts, décisions.
3. **Lire le champ Feedback** s'il est rempli — ajuster les priorités en conséquence :
   - `[+] élément` → cet acquis est validé, le considérer comme solide
   - `[-] élément` → cet acquis est remis en cause, le re-questionner avant de bâtir dessus
   - `[!] élément` → point critique, le traiter en priorité
   - `[?] élément` → incertitude signalée, demander clarification avant usage
4. **Détecter les motifs inter-sessions** (si ≥ 3 blocs lus) : un même point ouvert
   présent dans 3 blocs consécutifs = **blocage cristallisé** → le signaler en une ligne.
5. **Reprendre le travail** directement sur le champ `Prochain` du dernier bloc.

### Format de reprise

Une seule ligne d'accusé, puis le travail :

> *Session restaurée — [objet], [N] points ouverts. On reprend sur : [prochain].*

Jamais de récapitulation longue. Jamais de "pour résumer où nous en étions".
Le bloc a déjà fait ce travail.

---

## DÉCLENCHEUR 2 — COMPRESSION (fin de session)

### Activation automatique

Produire le Bloc-Session **sans qu'on le demande** quand l'un de ces signaux apparaît :

- Clôture explicite : "on s'arrête là", "on continue demain", "ce sera tout",
  "je reviendrai", "à plus tard"
- Remerciement final : "merci c'est parfait", "super merci", "nickel" en fin
  d'échange substantiel
- Commande directe : `⊜`, "archive la session", "compresse la session", "bloc de session"

**Garde-fou anti-bruit** : ne pas déclencher sur un simple "merci" de milieu de
session, ni sur une session de moins de 4 échanges substantiels. En cas de doute,
proposer en une ligne : *"Session à archiver ? (⊜)"* — et n'archiver que sur confirmation.

### Protocole de compression

1. **Balayer la session entière** — pas seulement les derniers échanges.
2. **Extraire** dans l'ordre :
   - Le domaine et l'objet (≤ 8 mots)
   - Les acquis : faits établis, livrables produits, problèmes résolus (3-6, télégraphique)
   - Les décisions : choix actés ET leur raison courte (≤ 2 lignes chacune, max 3)
   - Les ouvertures : tâches non finies, questions en suspens, pistes écartées-mais-notées
   - Le prochain pas convenu (1 ligne)
3. **Générer la signature STÈLE** (règles ci-dessous).
4. **Produire le bloc** et inviter à le coller dans `journal-sessions.md`.

---

## Le Bloc-Session

```
=== SESSION ===
Date       : [JJ/MM/AAAA]
Domaine    : [code / symbolique / rédaction / recherche / conception / mixte]
Objet      : [3-8 mots]
Acquis     :
  - [fait ou livrable, télégraphique]
  - [...]
Décisions  :
  - [choix] — [raison courte]
Ouvert     :
  - [tâche ou question non résolue]
Prochain   : [l'étape suivante convenue, 1 ligne]
Signature  : ⟦[chaîne STÈLE]⟧ [statut]
Mots-codes : [transcription verbale de la chaîne]
Feedback   : [vide — rempli par l'utilisateur après session]
===============
```

**Budget : ≤ 150 tokens.** Si le bloc dépasse, comprimer les acquis avant
les ouvertures — ce qui est acquis se redécouvre, ce qui est ouvert se perd.

---

## Signature STÈLE de session

Adaptation de la table CRC-R aux sessions de travail générales.
Structure : `⟦ [1-2 substances] [1-2 opératives] [0-2 modales] ⟧` — max 12 signes.

### Substances (ce qu'était la session)

| Condition de session | Glyphe | Mot-code |
|---|---|---|
| Objectif clair dès le départ | `⊙` | SOURCE |
| Livrable stable produit | `◯` | FORME |
| Session exploratoire, sans objet fixé | `Ø` | VIDE |
| Travail en cours, transformation active | `∿` | MOUVEMENT |
| Limite atteinte, piste close ou abandonnée | `⊥` | LIMITE |
| Travail sur une structure portante du projet | `✦` | AXE |
| Session de reprise sur travail antérieur | `◊` | TRACE |
| Plusieurs problèmes entremêlés | `⊛` | NOEUD |

### Opératives (ce qui s'est accompli)

| Condition de session | Glyphe | Mot-code |
|---|---|---|
| Blocage cassé, problème résolu par rupture d'approche | `⟁` | FRACTURER |
| Connexion établie (entre outils, idées, sessions) | `⊗` | LIER |
| Découverte — quelque chose d'inattendu est apparu | `✶` | RÉVÉLER |
| Session pleinement aboutie | `⊜` | SCELLER |
| Piste identifiée mais mise en réserve | `⌒` | PLIER |
| Renversement — la direction du projet a changé | `⥀` | INVERSER |
| Passage de relais (handoff vers session suivante) | `⟶` | TRANSMETTRE |

### Modales (comment ça s'est configuré)

| Condition | Glyphe | Mot-code |
|---|---|---|
| Problème récurrent (déjà vu en sessions passées) | `↻` | CYCLE |
| Très forte intensité de travail ou enjeu | `▲` | INTENSITÉ |
| Échec, refus, impasse sur l'élément précédent | `⊘` | NÉGATION |
| Session inachevée, suspendue en cours | `◐` | CONDITIONNEL |
| Un sujet majeur volontairement non traité | `◌` | SILENCE |

### Statuts

- **Clos** — objectif atteint, rien d'urgent en suspens → opérative `⊜` probable
- **Ouvert** — travail interrompu, reprise prévue → modale `◐` en fin de chaîne
- **Bifurqué** — la session a changé la direction du projet → opérative `⥀` probable

### Exemples

Session de dev aboutie sur une fonctionnalité prévue :
```
⟦⊙∿◯⊜⟧ Clos
SOURCE.MOUVEMENT.FORME.SCELLER
```

Session exploratoire qui a révélé une nouvelle direction puis suspendue :
```
⟦Ø✶⥀◐⟧ Bifurqué
VIDE.RÉVÉLER.INVERSER.CONDITIONNEL
```

Reprise sur blocage récurrent, cassé pendant la session :
```
⟦◊⊛⟁↻◯⟧ Clos
TRACE.NOEUD.FRACTURER[CYCLE].FORME
```

---

## Résonance inter-sessions

*Applicable en RESTAURATION si ≥ 3 blocs lus.*

Comparer les signatures STÈLE des blocs :
- **Glyphe récurrent ≥ 3 fois** → motif cristallisé du projet. Le nommer en une ligne.
  Ex : *"⊛ revient dans 4 des 5 dernières sessions — la complexité s'accumule sans se résoudre."*
- **Mutation** → un glyphe change de polarité (ex : `⊛` disparaît au profit de `◯`)
  → signaler la progression.
- **`◐` persistant** → sessions systématiquement inachevées → suggérer de réduire
  le périmètre des objectifs de session.

Trois lignes maximum. Ce n'est pas une analyse, c'est un relevé.

---

## Principes opératoires

**Compression fidèle** — Le bloc préserve la structure, pas le détail.
Ne jamais prétendre qu'il restitue la session complète.

**Automatisme discipliné** — Le déclenchement automatique sert l'utilisateur,
il ne le surprend pas. En cas de doute sur la clôture : proposer, ne pas imposer.

**Le journal appartient à l'utilisateur** — Le skill produit le bloc ; coller,
organiser et annoter le journal reste un geste de l'utilisateur. C'est ce geste
qui fait la mémoire, pas le mécanisme.

**Un bloc = une session** — Jamais de bloc cumulatif multi-sessions.
La granularité est la session ; l'agrégation se fait par la lecture du journal.

**Honnêteté épistémique** — héritée du CRC-R : ce skill modélise une mémoire
de travail externalisée. Les signatures STÈLE sont des coordonnées symboliques,
pas des preuves ni des prédictions.
