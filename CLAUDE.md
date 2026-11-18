# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Projet : Légibase, bibliothèque du droit français en Markdown.

## Objectif

Construire `legibase-fr/`, une skill Claude simple qui contient les codes et textes juridiques
français en Markdown. Chaque code a son propre dossier, et les liens entre les articles
(renvois, citations, jurisprudence) sont identifiés et navigables.

**Usage visé** : une personne télécharge le dossier depuis GitHub, l'installe comme skill,
pose ses questions, et Claude répond avec le texte de loi et sa source officielle.

## Principe de simplicité

- **Pour l'utilisateur** : aucune installation technique. Les fichiers .md sont déjà générés
  et publiés sur GitHub. Il télécharge, installe la skill, pose ses questions.
- **Pour le mainteneur** : Python sert uniquement à générer et actualiser les fichiers.
- **Technologies** : Python 3 avec la **bibliothèque standard uniquement**
  (`urllib`, `tarfile`, `xml.etree.ElementTree`, `json`, `re`, `pathlib`).
  Pas de paquets à installer, pas de base de données, pas de serveur.
- Le code doit rester court, lisible et commenté en français.
- Si un choix technique complique les choses, choisir la solution la plus simple et me le signaler.

## Règles absolues

- **Sources officielles uniquement.** Ne jamais écrire un texte de loi ou une décision de mémoire.
  Tout texte doit venir des sources listées ci-dessous.
- **Un code à la fois.** Ne jamais commencer le code suivant sans mon accord explicite.
- **Seulement le droit en vigueur.** Ne garder que les articles dont l'état est « VIGUEUR ».
- **Traçabilité.** Chaque article indique son identifiant officiel, sa date de version et son lien Légifrance.
- **Honnêteté pendant la construction.** Me dire toujours la vérité sur l'avancement : ne jamais
  annoncer qu'une étape est terminée si elle ne l'est pas, signaler chaque erreur, chaque article
  non traité et chaque doute, et ne jamais remplacer une donnée manquante par une donnée inventée.

## Sources

- **Textes** : base LEGI de la DILA (open data, Licence Ouverte 2.0),
  sur `echanges.dila.gouv.fr/OPENDATA/LEGI/`.
  Avant d'écrire un parseur, regarder comment des projets open source lisent ces XML
  (par exemple les projets Legilibre sur GitHub) pour comprendre le format,
  mais écrire notre propre script simple, sans dépendance.
- **Jurisprudence** : base CASS de la DILA (open data) ou API Judilibre de la Cour de cassation
  (via la plateforme PISTE, compte gratuit). Ne récupérer que la jurisprudence des thèmes que je choisis.
- **Règlement Intérieur National (RIN) des avocats** : il n'est pas dans LEGI. Le récupérer depuis
  le site du Conseil National des Barreaux et indiquer sa date de version.

## Structure

```
legibase-fr/
├── SKILL.md                      # mode d'emploi pour Claude (voir plus bas)
├── README.md                     # installation et usage, pour les humains
├── VERSION.md                    # date de la dernière actualisation + date de la base LEGI utilisée
├── INDEX.md                      # liste des codes, date de version, nombre d'articles
├── codes/
│   ├── code-civil/
│   │   ├── README.md             # plan du code (livres > titres > chapitres) avec liens
│   │   ├── livre-iii/titre-iii/sous-titre-ier/chapitre-ier.md  # un fichier par chapitre (minuscules, sans accents)
│   │   ├── livre-iii/titre-iii/articles-directs.md             # articles posés directement sur un livre ou un titre
│   │   ├── meta.json             # nom officiel, identifiant LEGITEXT, nombre d'articles et de fichiers
│   │   └── renvois.md            # liens vers et depuis les autres codes
│   ├── code-de-procedure-civile/
│   ├── code-du-travail/
│   └── ...
├── textes-non-codifies/          # ex : loi n° 71-1130, décret n° 91-1197, RIN
├── jurisprudence/
│   └── <theme>/                  # une fiche .md par décision
├── liens/
│   ├── graphe/<code>.json        # renvois (article source -> article cible), un fichier par code
│   └── carte-des-liens.md        # synthèse lisible : quels codes renvoient à quels codes
└── scripts/
    └── update.py                 # régénère tout et met à jour VERSION.md
```

Un fichier par chapitre, pas un fichier par article.
`textes-non-codifies/` et `jurisprudence/` n'existent pas encore (étape 3 et jurisprudence non commencées).
`INDEX.md`, `VERSION.md`, `codes/` et `liens/` sont écrits par le générateur : ne pas les corriger à la main,
modifier `scripts/generer.py` et régénérer. `SKILL.md` et `README.md` sont écrits à la main.
Le dossier `scripts/` sert au mainteneur ; l'utilisateur n'en a pas besoin.

## Format d'un article

```markdown
### Article 1103
- Identifiant : LEGIARTI...
- État : en vigueur depuis le JJ/MM/AAAA
- Source : <lien Légifrance>
- Renvoie à : [art. X du code Y](chemin/relatif.md#article-x)
- Cité par : [art. Z du code W](chemin/relatif.md#article-z)
- Jurisprudence : [Cass. 2e civ., JJ/MM/AAAA, n° XX-XX.XXX](../../jurisprudence/theme/fiche.md)

<texte intégral de l'article>
```

## Format d'une fiche de jurisprudence

```markdown
# Cass. 2e civ., JJ/MM/AAAA, n° XX-XX.XXX
- Juridiction et formation :
- Date :
- Numéro de pourvoi :
- Solution : cassation / rejet / ...
- Articles visés : [liens vers les articles de la bibliothèque]
- Source : <lien Judilibre ou Légifrance>

## Résumé
<résumé fidèle de la décision, en distinguant les faits, la question posée et la solution>
```

## Liens entre les textes

1. Utiliser d'abord les liens officiels présents dans les XML LEGI (balises `LIENS` :
   citations, modifications, codifications).
2. Compléter en détectant dans le texte les mentions du type
   « article L. 1234-5 du code du travail » ou « article 10 de la loi du 31 décembre 1971 ».
3. Écrire chaque lien dans les deux sens (« Renvoie à » et « Cité par »), dans les fichiers
   `renvois.md` de chaque code et dans `liens/graphe/<code>.json`.
4. Relier chaque décision de jurisprudence aux articles qu'elle vise.
5. Quand un nouveau code est ajouté, mettre à jour les liens « Cité par » des codes déjà traités.
6. Lister à part les renvois vers des codes pas encore traités, pour les compléter plus tard.

## État d'avancement

- **Codes : les 76 codes en vigueur de LEGI sont générés** (13/11/2026, base LEGI du 08/11/2026), avec l'accord de
  l'utilisateur pour les faire tous à la suite : 159 998 articles, 13 858 fichiers, 317 172 renvois ; `codes/` 303 Mo,
  `liens/` 62 Mo, zip 63 Mo. Vérifiés : texte identique aux XML, 0 lien cassé, 0 ancre absente (695 033 liens internes).
  Poussé sur GitHub le 13/11/2026 (commit `5b2b679`).
- **Articles « VIGUEUR » non écrits** (20) : rangés par LEGI dans une section abrogée ou modifiée (consommation R822-x,
  éducation R337-31/45, santé publique R1333-172 à 175, transports L3124-8/L4316-3, CGI 302 B/591, communes de
  Nouvelle-Calédonie R234-11). Le code du domaine public fluvial a 0 article « VIGUEUR » (1 seul « ABROGE_DIFF »).
- **Décisions à demander à l'utilisateur** :
  - **taille** : la bibliothèque est sans doute trop grande pour une seule skill ; proposer de la séparer par domaine ;
  - des articles en vigueur aujourd'hui mais marqués « ABROGE_DIFF » (abrogation programmée) sont exclus par la règle
    « état VIGUEUR uniquement » (393 rien que pour civil, procédure civile, travail, consommation, commerce ; non compté
    pour les 76 codes). Proposition : les inclure avec « jusqu'au JJ/MM/AAAA » ;
  - 9 articles R822-x du code de la consommation (Commission des clauses abusives) sont « VIGUEUR » mais dans une
    section abrogée le 31/03/2026 : incohérence de LEGI, laissés de côté ;
  - 30 numéros d'articles portés par deux articles différents (surtout des annexes du code de commerce, ex. « Annexe 1-3 ») :
    les deux sont écrits, mais un lien vers ce numéro mène à un seul des deux.
- **Dépôt GitHub** : `arkad-consulting/legibase-fr` (public, branche `main`). Transféré le 18/11/2026 du compte personnel
  `otonibar123` vers l'organisation `arkad-consulting` ; l'ancienne adresse redirige vers la nouvelle.
- **iCloud (réglé le 18/11/2026)** : le projet a été déplacé de `~/Documents` vers `~/Projects/legibase-fr`, hors d'iCloud.
  Historique : quand il était dans `~/Documents`, iCloud créait des doublons dans `codes/` après une
  régénération (noms terminés par « 2 » ou « 3 » : `livre-ier 2`, `meta 2.json`). 207 doublons supprimés le 18/11/2026,
  après vérification : aucun n'était suivi par git, aucun ne contenait de fichier absent de l'original, et les originaux
  étaient identiques au dernier commit. Après chaque génération, chercher les noms contenant une espace
  (`find codes liens -name '* *'`) et ne jamais faire `git add -A` avant de les avoir retirés.
  Par prudence, refaire la recherche des noms contenant une espace après la prochaine génération.
- **Prochaine étape proposée : textes de la profession d'avocat (étape 3)**, loi n° 71-1130, décret n° 91-1197, RIN.
  Ne pas commencer sans l'accord de l'utilisateur. Ce sont des textes non codifiés (`TNC_en_vigueur/JORF/...`) : il faut
  adapter l'extraction et le générateur (dossier `textes-non-codifies/`), et le RIN vient du site du CNB.
- **Points ouverts** :
  - vérifier à la main qu'un lien Légifrance « Cité par » vers un texte non codifié
    (`https://www.legifrance.gouv.fr/loda/article_lc/<id>`) s'ouvre bien ; sinon changer le format dans `lien_legifrance()` ;
  - détecter les mentions de lois citées par date (« article 10 de la loi du 31 décembre 1971 ») : à faire avec l'étape 3 ;
  - jurisprudence : pas commencée (attendre le choix des thèmes).

## Commandes

Se placer dans `legibase-fr/` (Python 3, bibliothèque standard uniquement ; `-I` isole l'interpréteur des fichiers téléchargés).

```
python3 -I scripts/update.py                       # télécharge la dernière base LEGI, extrait, régénère tout
python3 -I scripts/update.py --sans-telechargement # réutilise l'archive déjà dans .cache/
python3 -I scripts/update.py --sans-telechargement --sans-extraction  # réutilise aussi les XML déjà extraits
```

- `CODES` de `scripts/update.py` contient les 76 codes en vigueur de LEGI (`dossier: identifiant LEGITEXT`, dossier = titre
  officiel TITREFULL en minuscules sans accents). Génération complète : ~11 minutes, ~1,3 Go de mémoire. Tout est régénéré à chaque lancement, donc les liens
  « Cité par » des codes déjà traités se mettent à jour d'eux-mêmes.
- **`.cache/`** (2 Go, exclu de GitHub par `.gitignore`) contient l'archive `Freemium_legi_global_*.tar.gz` (1,26 Go)
  et un dossier `extrait-<code>/` par code. L'extraction relit toute l'archive : compter plus de 10 minutes
  (lancer en arrière-plan). Les scripts sont dans `scripts/generer.py` (lecture XML, plan des fichiers, liens,
  écriture) et `scripts/update.py` (téléchargement, extraction, appel).
- Après chaque génération, vérifier par script (sans afficher les fichiers) : nombre d'articles = nombre d'articles
  « VIGUEUR » des XML, texte identique à la source, liens relatifs et ancres existants.
  Il n'y a ni tests ni script de vérification dans le dépôt : ces contrôles ont été faits avec des scripts
  temporaires, à réécrire à chaque fois (ou à ajouter dans `scripts/` avec l'accord de l'utilisateur).
- `--sans-telechargement` prend l'archive la plus récente de `.cache/`. Piège : `--sans-extraction` seul interroge
  quand même la DILA, télécharge une archive plus récente s'il y en a une, et écrit sa date dans `VERSION.md` alors
  que les XML relus sont les anciens ; toujours le combiner avec `--sans-telechargement`. Sans `--sans-extraction`, tous les dossiers
  `extrait-*` sont effacés puis recréés. `update.py` affiche à la fin un résumé par code et les compteurs `stats`
  (articles écartés et pourquoi, liens écartés, numéros en double) : les lire après chaque génération.

## Architecture des scripts

`update.py` ne fait que télécharger, extraire et appeler `generer.ecrire_tout()`. Tout le reste est dans
`generer.py`, en trois temps, pour tous les codes à la fois (les liens entre codes l'exigent) :

1. **Lecture** (`lire_tout`) : pour chaque code, `lire_structure` suit `texte/struct` puis les `section_ta` et construit
   un arbre de nœuds `{titre, id, enfants: [("art", id) | ("sec", nœud)]}` ; `planifier` transforme l'arbre en liste de
   fichiers (un nœud dont le titre commence par « Chapitre », ou sans sous-section, = un fichier ; les niveaux au-dessus
   = des dossiers) ; `lire_article` lit chaque article et ses liens `CITATION`.
2. **Liens** (`construire_liens`) : liens officiels LEGI, puis mentions repérées dans le texte (`re_mention_code`).
   Produit `aretes` (liste `{de, vers, origine: "LEGI" | "texte"}`) et les tables `renvoie` / `cite_par`.
3. **Écriture** (`ecrire_tout`) : fichiers de chapitre, puis par code `README.md`, `renvois.md`, `meta.json`,
   puis `liens/graphe/<code>.json`, `liens/carte-des-liens.md`, `INDEX.md` et `VERSION.md`.

La table `emplacement` relie les trois temps : `(nom officiel du texte, numéro d'article)` → `(chemin du fichier,
ancre, identifiant LEGIARTI, dossier du code)`. Un article est « dans la bibliothèque » s'il est dans cette table ;
sinon `lien_md` écrit un lien Légifrance. La clé est le nom officiel (TITREFULL), pas le dossier : tout nom lu dans un
libellé ou dans le texte doit d'abord être ramené à ce nom (`cidtexte`, sinon `complet()`).

Un code sans aucun renvoi n'a pas de fichier dans `liens/graphe/` (73 fichiers pour 76 codes).

## Particularités de la base LEGI (à connaître avant de modifier le parseur)

- **Emplacement** : `legi/global/code_et_TNC_en_vigueur/code_en_vigueur/LEGI/TEXT/.../<LEGITEXT>/`, avec
  `texte/struct`, `texte/version`, `section_ta/` et `article/`. Une section peut contenir à la fois des
  sous-sections et des articles ; les anciennes versions sont mélangées aux versions en vigueur.
- **État et dates** : l'état « VIGUEUR » ne suffit pas. Il faut aussi que la période de validité (début, fin) couvre
  la date de la base (cas de l'article 214 du code civil : 2e version « VIGUEUR » avec début = fin = 1976).
  Les articles « VIGUEUR_DIFF » (vigueur future) sont exclus.
- **Sections** : une section « ABROGE_DIFF » ou « VIGUEUR_DIFF » peut contenir des articles en vigueur aujourd'hui ;
  le générateur les parcourt (le tri se fait article par article). Un même article peut être rattaché à deux versions
  d'une section : il n'est écrit qu'une fois, sous la section « VIGUEUR » de préférence. Les fichiers de chapitre
  sans article en vigueur ne sont pas écrits, et le dossier d'un code est vidé avant d'être régénéré.
- **Noms des codes** : un même code est nommé de façons diverses dans les libellés de liens (« Code général des impôts, CGI. »,
  majuscules, coupé à 50 caractères avec ou sans « ... », ancien nom : « Code rural (nouveau) » = « Code rural et de la
  pêche maritime »). Les liens sont rattachés par l'identifiant `cidtexte` ; les autres noms relevés sont listés dans
  `INDEX.md` et servent à repérer les mentions dans le texte.
- **Articles sans numéro** (6) : titrés « Article (sans numéro dans LEGI) » et pas inscrits dans la table des liens
  (sinon tout lien vers un texte entier pointerait vers eux). Noms de fichiers coupés à 80 caractères (limite macOS).
- **Sens des liens** (`typelien="CITATION"`) : `sens="source"` = l'article cite la cible (« Renvoie à ») ;
  `sens="cible"` = l'article est cité par l'autre texte (« Cité par »).
- **Libellés de lien** : « Code civil - art. 1366 (V) ». L'état entre parenthèses (V, VD = gardés ; M, Ab... = anciennes
  versions, écartés). Sans état : gardé seulement si l'article cité est dans la bibliothèque. Noms de textes irréguliers
  (« Code général des impôts, CGI. », liens vers une section) normalisés par `lire_libelle()` ; les noms longs sont
  coupés par LEGI à 50 caractères avec « ... » (conservés tels quels).
- **Titres** : LEGI écrit parfois « Titre : XIV : Des ... » (`prefixe()` gère ce cas pour les noms de fichiers).
- **Plafond d'affichage** : au plus 10 renvois vers des textes hors bibliothèque par article (`MAX_EXTERNES`) ;
  la liste complète est dans `renvois.md`.

## Actualisation

- `scripts/update.py` télécharge la dernière base LEGI, régénère les fichiers et écrit dans
  `VERSION.md` la date de l'actualisation et la date de la base LEGI utilisée.
- Pour l'utilisateur, actualiser = télécharger la dernière version du dossier sur GitHub.
- Option à proposer à la fin du projet (seulement si c'est simple) : une GitHub Action qui lance
  `update.py` une fois par mois et publie la nouvelle version automatiquement.

## Méthode de travail : un code à la fois

- **Étape 1** : préparer les outils (téléchargement de LEGI, parseur, `scripts/update.py`),
  puis générer uniquement le code civil.
- **À la fin de chaque code**, s'arrêter et me donner :
  - le nombre d'articles et de fichiers créés ;
  - la taille du dossier de ce code et la taille totale de `legibase-fr/` ;
  - 2 ou 3 exemples d'articles avec leurs renvois, pour que je vérifie le format ;
  - les renvois vers des codes pas encore traités ;
  - le code proposé ensuite.

  Puis **attendre ma réponse** avant de continuer.
- **Ordre proposé** (je peux le changer) :
  1. code civil
  2. code de procédure civile
  3. textes sur la profession d'avocat (loi n° 71-1130, décret n° 91-1197, RIN)
  4. code du travail
  5. code de la consommation
  6. code de commerce
- **Taille** : si la bibliothèque devient trop grande pour une skill, me prévenir avant de continuer
  et proposer de la séparer en plusieurs skills (une par domaine).

## Économie de tokens

- Ne jamais afficher ni lire en entier les fichiers XML de LEGI ou les fichiers .md générés.
- Pour vérifier un résultat, utiliser des scripts qui affichent seulement des comptes,
  des tailles et quelques lignes d'exemple.
- Tester le parseur sur un petit échantillon (quelques articles) avant de le lancer sur tout un code.

## SKILL.md et README.md

Les deux fichiers existent et sont écrits à la main. Ce qu'ils doivent toujours contenir :

- **`SKILL.md`** : une description qui dit quand utiliser la skill (toute question sur le droit français, un article de
  code, une loi ou une décision de justice française) ; la section « Honnêteté », qui passe avant toutes les autres
  (ne jamais inventer, ne jamais répondre de mémoire, dire « Ce code n'est pas encore dans Légibase » ou « Je n'ai pas
  trouvé cette information dans Légibase » au lieu de combler le vide, séparer le texte de l'interprétation) ; la façon
  de chercher (`INDEX.md`, puis le dossier du code, puis les renvois et la jurisprudence liée) ; et les quatre règles
  de chaque réponse : texte avec sa source et sa date de version, date de la bibliothèque lue dans `VERSION.md`,
  mise en garde si cette date a plus d'un mois, avertissement final (« pas un avocat, pas un conseil juridique »).
- **`README.md`** (pour les humains, court, en français) : ce que fait et ne fait pas Légibase, l'installation comme
  skill pas à pas, 2 ou 3 exemples de questions, comment actualiser, les sources et leur licence (Licence Ouverte 2.0).

Ne pas affaiblir ces règles en modifiant l'un des deux fichiers ; si le contenu de la bibliothèque change
(textes non codifiés, jurisprudence), mettre à jour « Comment chercher » et « Limites à connaître » de `SKILL.md`.
