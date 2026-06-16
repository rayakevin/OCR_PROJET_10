# Guide de présentation — Soutenance P10 (BrainScanAI)

> **But de ce document** : tout le contenu de ta présentation, prêt à jouer.
> Pour chaque slide : le **visuel** (et où le récupérer dans les notebooks), un **script oral**,
> la **transition** vers la slide suivante, et un **conseil de design**.
> Format cible : 13 slides + annexes · ≤ 15 slides au compteur · **15 min** de présentation.

---

## 0. Ton général & posture

- **Tu es Data Scientist junior** qui présente à **Clara** (ta responsable). Parle comme à une collègue technique mais pressée : claire, structurée, orientée décision.
- **Assume l'honnêteté scientifique.** Ton résultat phare est *négatif* (le semi-supervisé n'améliore pas) — c'est une **force** : un junior qui explique *pourquoi* un truc ne marche pas vaut bien mieux qu'un qui gonfle ses chiffres. Répète-toi : « la donnée pilote mes choix ».
- **Relie en permanence technique ↔ métier** : budget, volume, temps, et surtout **sécurité patient** (rater un cancer = l'erreur grave).
- **Ne lis pas tes slides.** Les slides portent les visuels et 3-4 mots-clés ; toi, tu **racontes l'histoire**.
- **Rythme** : ~1 à 1,5 min par slide, plus de temps sur clustering + scaling (le cœur).

### Budget temps (15 min)
| Bloc | Slides | Durée |
|---|---|---|
| Intro & contexte | 1-2 | 1,5 min |
| Démarche | 3 | 1 min |
| EDA | 4 | 1,5 min |
| Features | 5 | 1 min |
| Clustering | 6 | 2 min |
| Semi-supervisé + résultat | 7-8 | 2,5 min |
| **Passage à l'échelle** | 9-10 | **3 min** |
| Tech ↔ métier | 11 | 1 min |
| Limites | 12 | 1 min |
| Conclusion | 13 | 1 min |

---

## 1. Conseils visuels généraux

- **Une idée par slide.** Titre = **un message**, pas un thème (« Le clustering ne sépare pas la pathologie », pas « Clustering »).
- **Peu de texte**, gros visuels lisibles. Exporte tes figures en **PNG haute résolution** depuis Jupyter : clic droit sur la figure → *Enregistrer l'image*, ou ajoute en fin de cellule `fig.savefig("reports/figures/nom.png", dpi=150, bbox_inches="tight")`. Range-les dans `reports/figures/`.
- **Annoter les figures clés** : une flèche / un cercle rouge sur la zone qui compte (ex. les bords sur la carte de différence, le cluster « mélangé » sur le t-SNE). L'œil de Clara doit aller direct au bon endroit.
- **Chiffres clés en gros** (ARI 0.12, recall 0.93, 0.00125 €/image). Un chiffre par slide qui « claque ».
- **Palette sobre et cohérente** ; numérote les slides ; mets une mini-frise du pipeline en bas de chaque slide technique pour situer où on en est.

---

## 2. Déroulé slide par slide

### Slide 1 — Titre
- **Visuel** : titre du projet, sous-titre, ton nom, date. Optionnel : une IRM en filigrane (prends-en une depuis `Notebook 1 › section EDA › grille d'exemples`).
- **Script** : « Bonjour Clara. Voici la première phase du projet BrainScanAI : une exploration analytique pour évaluer si on peut **automatiser la détection de tumeurs cérébrales** sur IRM, avec très peu d'images annotées. »
- **Transition** : « Commençons par le cadre et ce que je considère comme un objectif atteint. »
- **Design** : épuré, une seule image de fond, titre lisible.

### Slide 2 — Contexte & mission (+ definition of done)
- **Visuel** : 3 chiffres — **1 500 IRM**, **100 annotées** (50/50), **~1 400 non annotées** ; et un encart « Definition of done ».
- **Script** : « On a un sous-ensemble minuscule annoté par des radiologues, et une grande majorité d'images sans label — annoter coûte cher. Ma question directrice : peut-on exploiter ces images non annotées ? Et surtout, **qu'est-ce qu'un bon modèle ici ?** En imagerie médicale, **rater une tumeur — un faux négatif — est bien plus grave qu'une fausse alerte**. Donc ma métrique de réussite, ce n'est pas l'accuracy, c'est le **recall sur le cancer**. »
- **Transition** : « Pour y répondre, j'ai déroulé un pipeline en trois temps. »
- **Design** : les 3 chiffres en gros ; la *definition of done* dans un encadré coloré (on y reviendra).

### Slide 3 — Démarche d'ensemble
- **Visuel** : **un schéma du pipeline** en 4 blocs avec flèches :
  `Images → Features (ResNet) → Clustering (labels faibles) → Semi-supervisé → Reco passage à l'échelle`. (À dessiner toi-même — PowerPoint/Excalidraw.)
- **Script** : « J'extrais des caractéristiques visuelles avec un réseau pré-entraîné, je regroupe les images non annotées par clustering pour produire des labels "faibles", puis je teste une approche semi-supervisée, et j'en tire une recommandation de passage à l'échelle. C'est le fil rouge de ma présentation. »
- **Transition** : « Mais avant toute modélisation : connaître ses données. »
- **Design** : ce schéma devient ta **mini-frise** en pied de slides suivantes (surligne le bloc en cours).

### Slide 4 — EDA : qualité & biais
- **Visuel** : **la carte de différence** (image moyenne cancer / image moyenne normal / différence) — *de loin* le visuel le plus parlant.
  → **Notebook 1 › section EDA › cellule « image moyenne + carte de différence »** (3 sous-images côte à côte).
  En vignette secondaire : un chiffre « **186 doublons, dont 32 fuites corrigées** ».
- **Script** : « J'ai d'abord audité mes données. Deux constats. Un : **186 images en doublon exact**, dont 32 images annotées clonées dans le jeu non annoté — une **fuite** que j'ai corrigée par dédoublonnage, sinon mon évaluation aurait été biaisée. Deux, plus subtil : en moyennant les images par classe et en les soustrayant *(montre la carte)*, la différence **s'allume sur les bords**, pas dans le cerveau. Autrement dit, ce qui sépare le plus mes deux classes, c'est le **cadrage**, pas la tumeur. C'est un **risque de biais** que je garde en tête pour la suite. »
- **Transition** : « Ces données propres, je les transforme maintenant en features exploitables. »
- **Design** : entoure en rouge les bords lumineux de la carte de différence + annote « biais de cadrage ».

### Slide 5 — Extraction de features (transfer learning)
- **Visuel** : un mini-schéma « image 224×224 → ResNet50 gelé (tête retirée) → vecteur 2048 » ; chiffre « **1 410 × 2 048** ».
- **Script** : « Je n'ai ni le budget ni assez de données pour entraîner un réseau de zéro. J'utilise donc le **transfer learning** : un ResNet50 pré-entraîné sur ImageNet, dont je **gèle** les couches et **retire la tête** de classification. Chaque image devient un vecteur de 2 048 caractéristiques. C'est rapide, gratuit en annotation, et ça réutilise un savoir visuel généraliste. »
- **Transition** : « Avec ces vecteurs, je cherche des regroupements naturels. »
- **Design** : très visuel, peu de texte ; insiste oralement sur « gelé » et « gratuit en annotation » (ça prépare le scaling).

### Slide 6 — Clustering : le constat clé
- **Visuel** : **les deux nuages t-SNE côte à côte** (clusters K-Means vs vrais labels) + **ARI ≈ 0.12** en gros.
  → **Notebook 1 › section 3.5 › cellule de visualisation (2 scatterplots)** et le print **ARI** de la cellule juste avant.
- **Script** : « Je réduis la dimension par PCA, puis je teste **deux** algorithmes de clustering, K-Means et DBSCAN — réglé proprement via la courbe des k-distances. Verdict commun : **les clusters ne se superposent pas à la pathologie**. L'ARI, qui mesure l'accord avec les vrais labels, est de **0,12** — faible. *(montre les nuages)* À gauche mes clusters, à droite la vraie étiquette : ça ne coïncide pas. Concrètement, mon étiquetage faible **ne récupère que ~40 % des cancers**. Ce n'est pas un échec : c'est cohérent avec le biais de cadrage vu en EDA — le clustering capte surtout le type d'acquisition, pas la tumeur. »
- **Transition** : « La question devient : ces labels faibles, même imparfaits, peuvent-ils quand même aider un modèle ? C'est l'objet du semi-supervisé. »
- **Design** : place une flèche sur le cluster « mélangé » du t-SNE ; ARI 0.12 en énorme à côté.

### Slide 7 — Approche semi-supervisée (méthode + résultats)
- **Visuel** : à gauche un schéma `Jeu faible (pseudo-labels) → pré-entraînement → Jeu fort → affinage → Test` ; à droite **la table comparative** des métriques.
  → **Notebook 2 › section 4.5 › cellule de la table `comparaison`** (accuracy / precision / recall / F1 / ROC-AUC).
- **Script** : « Je compare deux modèles entraînés sur exactement le **même jeu de test, jamais vu**. Le modèle **supervisé** n'apprend que sur mes 69 images fortement annotées. Le modèle **semi-supervisé** se pré-entraîne d'abord sur les images pseudo-labellisées par clustering, puis s'affine sur les vrais labels. *(montre la table)* Les deux sont **bons** — recall cancer 0,93, AUC 0,99 — grâce au transfer learning. »
- **Transition** : « Mais regardez la comparaison de près : le semi-supervisé n'apporte rien. Pourquoi ? »
- **Design** : surligne la ligne **recall (cancer)** et la colonne du supervisé. Garde la table lisible (arrondis à 2 décimales).

### Slide 8 — Résultat clé & sa justification
- **Visuel** : **les deux matrices de confusion** (A vs B) côte à côte ; un encadré « Pourquoi ? ».
  → **Notebook 2 › section 4.5 › cellule des matrices de confusion**. (Optionnel : la courbe ROC en annexe.)
- **Script** : « Le semi-supervisé **n'améliore pas** le supervisé — il fait même une fausse alerte de plus. Et l'écart se résume à **une seule image sur 30**, donc dans le bruit. Pourquoi pas de gain ? **Deux raisons.** Un : mon baseline supervisé **sature déjà** la tâche — le transfer learning suffit avec 69 images. Deux : mes pseudo-labels sont **trop bruités** — on l'a vu, le clustering ne récupère que 40 % des cancers — donc le pré-entraînement n'apporte pas de signal fiable, et l'affinage sur les vrais labels l'écrase. **Le facteur limitant, ce n'est pas le volume de données, c'est la qualité du clustering.** »
- **Transition** : « Ce constat est précisément ce qui guide ma recommandation pour passer à l'échelle. »
- **Design** : sous chaque matrice, écris en clair « 2 erreurs / 30 » et « 3 erreurs / 30 ». Encadré « Pourquoi ? » avec les 2 puces.

### Slide 9 — Passage à l'échelle : le verdict économique
- **Visuel** : **le calcul en gros** : `5 000 € ÷ 4 000 000 images = 0,00125 €/image`. À côté : « annotation experte ≈ plusieurs dizaines de centimes → des **centaines de milliers d'€** ».
- **Script** : « Clara, ta question : 4 millions d'images, 5 000 € — faisable ? Posons le calcul. Ça fait **0,00125 € par image**. Or une annotation par un radiologue coûte, même au plus optimiste, quelques dizaines de centimes : annoter les 4 millions, c'est **des centaines de milliers d'euros**. Donc **annoter tout à la main est exclu** — on est à ~1 000 fois le budget. »
- **Transition** : « Mais "impossible à la main" ne veut pas dire "impossible". Voici l'architecture que je recommande. »
- **Design** : le 0,00125 €/image en ÉNORME. Un pictogramme « budget » barré pour l'annotation manuelle.

### Slide 10 — Passage à l'échelle : l'architecture recommandée
- **Visuel** : un schéma en 3 briques : **1. Features + inférence à l'échelle** (compute ≈ qq €) · **2. Active learning** (annoter seulement les cas incertains) · **3. Vérification humaine ciblée**. + un encart « Conditions ».
- **Script** : « Trois briques. Un : l'**extraction de features et l'inférence** d'un modèle sur 4 millions d'images coûtent une poignée d'euros de GPU — négligeable. Deux : plutôt que le clustering, qu'on a vu peu fiable, j'investis le budget dans une **annotation experte ciblée par active learning** — on ne fait annoter que les cas où le modèle hésite. À quelques dizaines de centimes l'annotation, **5 000 € achètent plusieurs milliers d'annotations bien choisies** ; rappel : avec seulement 100 labels, on atteignait déjà recall 0,93 ! Trois : **vérification humaine systématique des cas positifs**, sécurité patient oblige. Conditions : c'est un **outil de triage R&D, pas un diagnostic** ; seuil calibré sur le recall ; validation clinique ; monitoring de dérive ; ré-entraînement. »
- **Transition** : « En résumé, chacun de mes choix techniques répond à une contrainte métier. »
- **Design** : 3 briques numérotées avec icônes ; encadré « Conditions » en bas. C'est **LA** slide à soigner.

### Slide 11 — Choix techniques ↔ contraintes métier
- **Visuel** : un **tableau** à 3 colonnes : *Choix technique* | *Contrainte métier adressée* | *Bénéfice*.
  Lignes : transfer learning → budget/temps → pas d'entraînement coûteux · seed expert + active learning → budget annotation → modèle fiable à coût minimal · métrique recall cancer → sécurité patient → on ne rate pas de tumeur · dédoublonnage / test étanche → fiabilité → résultats crédibles.
- **Script** : « Pour résumer la cohérence de mes décisions : *(parcours le tableau)* le transfer learning répond au budget et au temps ; le couple seed expert + active learning répond au coût d'annotation ; le choix du recall répond à la sécurité patient ; et la discipline anti-fuite garantit que mes chiffres sont crédibles. »
- **Transition** : « Bien sûr, cette PoC a ses limites, que je veux poser clairement. »
- **Design** : tableau propre, une ligne = une décision. Aère.

### Slide 12 — Limites & conditions
- **Visuel** : liste courte : test petit (30 img) → validation croisée ; performance plafonnée par la qualité du clustering ; **triage R&D ≠ diagnostic** ; validation clinique + monitoring requis.
- **Script** : « Je reste lucide. Mon jeu de test est petit — 30 images — donc je conditionne mes chiffres et je recommanderais une **validation croisée** pour les stabiliser. La performance est plafonnée par la qualité du clustering, pas par le volume. Et surtout : ceci est un **outil de triage de recherche, jamais un dispositif de diagnostic** — toute mise en usage exige une validation clinique et une surveillance. »
- **Transition** : « Je conclus. »
- **Design** : icône « ⚠️ » discrète ; ne surcharge pas — 4 puces max.

### Slide 13 — Conclusion & recommandation
- **Visuel** : 3 messages : ✔ pipeline complet livré · ✔ métrique et résultats justifiés · ➡️ **scaling faisable sous conditions** (transfer learning + active learning + vérification).
- **Script** : « En une phrase : oui, le passage à l'échelle est **faisable**, mais à condition de **ne pas chercher à tout labelliser**. La valeur n'est pas dans le clustering non supervisé — qu'on a démontré insuffisant — mais dans un **petit jeu d'annotations expertes bien choisi**, exploité par transfer learning et active learning. Mon livrable : un pipeline reproductible, une comparaison honnête, et une recommandation chiffrée. Je suis prêt pour vos questions. »
- **Design** : 3 lignes, beaucoup de blanc. Termine sur la recommandation, pas sur les limites.

---

## 3. Slides d'annexe (backup pour la discussion)

À garder après la slide 13, pour répondre à chaud sans chercher dans tes notebooks :
- **A1** — Courbes ROC des deux modèles → *Notebook 2 › section 4.5 › cellule ROC*.
- **A2** — Courbe des k-distances (justifie le réglage de DBSCAN) → *Notebook 1 › section clustering › cellule k-distances*.
- **A3** — Histogrammes d'intensité par classe + scatter luminosité×contraste → *Notebook 1 › EDA*.
- **A4** — Visualisation de paires de doublons / fuites → *Notebook 1 › EDA › cellule doublons*.
- **A5** — Détail du dédoublonnage (1506 → 1410, étanchéité = 0).

---

## 4. Préparer la discussion (10 min) — questions probables de Clara

> Réponds court, assume, et **ramène toujours à une mesure**.

- **« Pourquoi ResNet et pas un autre modèle / from scratch ? »**
  → Trop peu de données pour entraîner de zéro (sur-apprentissage) ; le transfer learning réutilise un savoir visuel et coûte zéro annotation. ResNet est un standard robuste et léger à servir.
- **« Un ARI de 0,12, c'est un échec ? »**
  → Non, c'est un **résultat informatif** : il prouve que la séparation visuelle dominante n'est pas la pathologie mais le cadrage/l'acquisition (cf. carte de différence). C'est ce qui motive la suite.
- **« Pourquoi le semi-supervisé n'aide pas ? »**
  → Baseline déjà saturé (transfer learning) **+** pseudo-labels qui ratent 60 % des cancers. Le facteur limitant est la qualité du clustering, pas le volume.
- **« Ton test fait 30 images, c'est fiable ? »**
  → Non, c'est petit, je l'assume — d'où la validation croisée en perspective. L'écart entre mes deux modèles (1 image) est dans le bruit ; je ne sur-interprète pas.
- **« Pourquoi le recall et pas l'accuracy ? »**
  → Parce que rater un cancer (faux négatif) est l'erreur la plus grave. L'accuracy masquerait des faux négatifs sur un jeu déséquilibré.
- **« Comment garantis-tu l'absence de fuite ? »**
  → Dédoublonnage exact + quasi (MD5 + phash), vérification « 0 empreinte commune » entre annoté et non annoté, et test issu uniquement du jeu fort, jamais vu à l'entraînement.
- **« Le scaling est-il vraiment faisable ? »**
  → Oui **sous conditions** : compute négligeable + budget concentré sur l'annotation ciblée (active learning) + vérification humaine + validation clinique. Pas l'annotation exhaustive.
- **« Que ferais-tu avec plus de temps/budget ? »**
  → Validation croisée ; tester un fine-tuning partiel du backbone ; corriger le biais de cadrage (recadrage/normalisation) ; boucle d'active learning réelle ; calibration du seuil sur le recall.

---

## 5. Checklist finale (couverture de la grille d'auto-éval)

- [ ] Recommandations techniques pour le déploiement à grande échelle (4 M images, 5 000 €) → slides 9-10
- [ ] Lien choix techniques ↔ contraintes métier (volume, budget, temps) → slide 11
- [ ] Arguments préparés pour justifier choix techniques **et** métier → section 4
- [ ] Cohérence résultats notebooks ↔ recommandations → mêmes chiffres partout (ARI 0.12, recall 0.93, 0,00125 €/image)
- [ ] ≤ 15 slides (13 + annexes) · durée 15 min (±5)
