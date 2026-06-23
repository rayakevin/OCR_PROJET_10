# Guide de présentation — Soutenance P10 (BrainScanAI)

> **But de ce document** : tout le contenu de ta présentation, prêt à jouer.
> Pour chaque slide : le **visuel** (et où le récupérer dans les notebooks), un **script oral**,
> la **transition** vers la slide suivante, et un **conseil de design**.
> Format cible : 14 slides + annexes · ≤ 15 slides au compteur · **15 min** de présentation.

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
- **Chiffres clés en gros** (ARI clustering 0.535, ARI consensus 0.875, self-training 93,3 % du pool à seuil 0.95, recall semi-supervisé 1.00, 0.00125 €/image). Un chiffre par slide qui « claque ».
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
  `Images → Features (ResNet) → Benchmark pseudo-labels → Semi-supervisé → Reco passage à l'échelle`. (À dessiner toi-même — PowerPoint/Excalidraw.)
- **Script** : « J'extrais des caractéristiques visuelles avec un réseau pré-entraîné, je benchmarke plusieurs façons de pseudo-labéliser les images non annotées, puis je teste une approche semi-supervisée et j'en tire une recommandation de passage à l'échelle. C'est le fil rouge de ma présentation. »
- **Transition** : « Mais avant toute modélisation : connaître ses données. »
- **Design** : ce schéma devient ta **mini-frise** en pied de slides suivantes (surligne le bloc en cours).

### Slide 4 — EDA : qualité & biais
- **Visuel** : **la carte de différence** (image moyenne cancer / image moyenne normal / différence) — *de loin* le visuel le plus parlant.
  → **Notebook 1 › section EDA › cellule « image moyenne + carte de différence »** (3 sous-images côte à côte).
  En vignette secondaire : un chiffre « **225 doublons/quasi-doublons retirés, 0 fuite restante** ».
- **Script** : « J'ai d'abord audité mes données. Deux constats. Un : il y avait des doublons exacts et des quasi-doublons perceptuels entre le jeu annoté et le jeu non annoté. J'ai donc dédoublonné avec **MD5 + phash**, ce qui réduit le dataset de 1 506 à **1 281 images** et laisse **0 fuite détectée**. Deux, plus subtil : en moyennant les images par classe et en les soustrayant *(montre la carte)*, la différence **s'allume sur les bords**, pas dans le cerveau. Autrement dit, ce qui sépare le plus mes deux classes, c'est le **cadrage**, pas la tumeur. C'est un **risque de biais** que je garde en tête pour la suite. »
- **Transition** : « Ces données propres, je les transforme maintenant en features exploitables. »
- **Design** : entoure en rouge les bords lumineux de la carte de différence + annote « biais de cadrage ».

### Slide 5 — Extraction de features (transfer learning)
- **Visuel** : un mini-schéma « image 224×224 → ResNet50 gelé (tête retirée) → vecteur 2048 » ; chiffre « **1 281 × 2 048** ».
- **Script** : « Je n'ai ni le budget ni assez de données pour entraîner un réseau de zéro. J'utilise donc le **transfer learning** : un ResNet50 pré-entraîné sur ImageNet, dont je **gèle** les couches et **retire la tête** de classification. Chaque image devient un vecteur de 2 048 caractéristiques. C'est rapide, gratuit en annotation, et ça réutilise un savoir visuel généraliste. »
- **Transition** : « Avec ces vecteurs, je cherche des regroupements naturels. »
- **Design** : très visuel, peu de texte ; insiste oralement sur « gelé » et « gratuit en annotation » (ça prépare le scaling).

### Slide 6 — Pseudo-labels : ne pas forcer les cas ambigus
- **Visuel** : à gauche **les trois nuages t-SNE** (K-Means / meilleur clustering / vrais labels), à droite une mini-table : clustering ARI **0,535** → SVC OOF ARI **0,768** → consensus filtré ARI **0,875**, **720 pseudo-labels retenus / 463 à revoir**.
  → **Notebook 1 › section 3.4-3.5** : table `benchmark_pseudo`, table `benchmark_consensus`, puis répartition des pseudo-labels retenus.
- **Script** : « J'ai d'abord renforcé le benchmark de clustering : K-Means, DBSCAN, agglomératif, Gaussian Mixture, spectral, et une baseline RAW2048 sans PCA. Le meilleur clustering atteint **ARI ≈ 0,535** : c'est mieux que K-Means, mais trop bruité pour pseudo-labéliser aveuglément 1 183 images. J'ai donc testé des pseudo-labelers en **5-fold out-of-fold**. Le SVC RBF atteint **ARI ≈ 0,768**, et le consensus SVC + ExtraTrees + LabelSpreading monte à **ARI filtré ≈ 0,875**. Résultat : je garde **720 pseudo-labels** et je laisse **463 images** pour annotation ciblée. »
- **Transition** : « La question devient : ces pseudo-labels plus propres peuvent-ils aider un modèle à détecter plus de cancers ? C'est l'objet du semi-supervisé. »
- **Design** : montre l'ancien clustering comme baseline, puis la stratégie finale. Mets « 720 retenus / 463 à revoir » en gros : c'est le lien avec le budget annotation.

### Slide 7 — Approche semi-supervisée (méthode + résultats)
- **Visuel** : à gauche un schéma `Images pseudo-labélisées → pré-entraînement → Images labélisées → affinage → Test` ; à droite **la table comparative** des métriques.
  → **Notebook 2 › section 4.5 › cellule de la table `comparaison`** (accuracy / precision / recall / F1 / ROC-AUC).
- **Script** : « Je compare deux modèles entraînés sur exactement le **même jeu de test, jamais vu**. Le modèle **supervisé** n'apprend que sur mes 68 images labélisées. Le modèle **semi-supervisé** se pré-entraîne d'abord sur les **720 pseudo-labels filtrés**, puis s'affine sur les vrais labels. Sur le split principal, le semi-supervisé atteint **recall cancer = 1,00** : aucun cancer raté. En revanche, il perd en précision (**0,83** contre **0,93**) et en accuracy (**0,90** contre **0,93**). L'AUC est équivalente : **0,991** pour les deux. »
- **Transition** : « Donc le semi-supervisé ne gagne pas partout : il déplace le compromis vers la sensibilité. »
- **Design** : surligne la ligne **recall (cancer)** côté semi-supervisé et la ligne **precision** côté supervisé. Garde la table lisible (arrondis à 2 décimales).

### Slide 8 — Résultat clé & sa justification
- **Visuel** : **les deux matrices de confusion** (A vs B) côte à côte ; un encadré « Pourquoi ? ».
  → **Notebook 2 › section 4.5 › cellule des matrices de confusion**. (Optionnel : la courbe ROC en annexe.)
- **Script** : « Le semi-supervisé **n'améliore pas la performance globale**, mais il améliore le recall. Sur 3 splits répétés, le supervisé est à **accuracy 0,91 / AUC 0,97 / recall 0,91**, contre **accuracy 0,86 / AUC 0,96 / recall 0,98** pour le semi-supervisé. J'ai aussi testé un **self-training itératif leakage-safe** : à seuil strict **0,95**, il pseudo-labélise en moyenne **93,3 %** du pool sans améliorer le SVC supervisé ; en descendant les seuils (progressif), on couvre **97,2 %**, mais les métriques baissent. Donc la bonne recommandation n'est pas "tout pseudo-labéliser", mais "pseudo-labéliser hautement confiant, puis annoter les cas ambigus". »
- **Transition** : « Ce constat est précisément ce qui guide ma recommandation pour passer à l'échelle. »
- **Design** : mets la table du split principal + un petit encart « stabilité 3 splits ». Encadré « Pourquoi ? » avec les 2 puces.

### Slide 9 — Passage à l'échelle : le verdict économique
- **Visuel** : **le calcul en gros** : `5 000 € ÷ 4 000 000 images = 0,00125 €/image`. À côté : « annotation clinique ≈ **0,5-5 €/image** → annoter tout = **2 à 20 M€** ».
- **Script** : « Clara, ta question : 4 millions d'images, 5 000 € — faisable ? Posons le calcul. Le budget, c'est **0,00125 € par image**. Or une annotation par un radiologue coûte, selon l'expertise, **0,5 à 5 € la pièce** — soit **400 à 4 000 fois plus**. Annoter les 4 millions à la main, ce serait **2 à 20 millions d'euros**. Donc **annoter tout à la main est exclu** — des centaines à des milliers de fois le budget. »
- **Transition** : « Mais "impossible à la main" ne veut pas dire "impossible". Voici l'architecture que je recommande. »
- **Design** : le 0,00125 €/image en ÉNORME. Un pictogramme « budget » barré pour l'annotation manuelle.

### Slide 10 — Passage à l'échelle : l'architecture recommandée
- **Visuel** : un schéma en 3 briques **chiffrées** :
  **1. Features + inférence à l'échelle** — `4 M ÷ ~1 000 img/s ≈ 1 h GPU ≈ 1-2 €` · `~33 Go de features` · **< 0,1 % du budget** ·
  **2. Active learning** — `5 000 € ÷ 0,5-5 €/label = 1 000 à 10 000 annotations ciblées` (≈ ×2-3 vs aléatoire) ·
  **3. Vérification humaine des positifs** — coût **opérationnel** ∝ taux de positifs (~40 000 revues à 1 %).
  + un encart « Conditions ».
- **Script** : « Trois briques, et je les chiffre.
  **Un — le compute n'est pas le problème.** Extraire les features ResNet50 et inférer sur 4 millions d'images, c'est de l'ordre de **1 000 images/seconde** sur un GPU standard, soit **~1 heure de calcul, autour de 1 à 2 €** ; les vecteurs pèsent **~33 Go**, stockage négligeable. Le compute, c'est donc **moins de 0,1 % des 5 000 €**. Ancrage concret : j'ai déjà extrait 1 281 vecteurs ici, ça tient dans **10 Mo** — l'extrapolation est linéaire.
  **Deux — tout le budget va à l'annotation, mais ciblée.** Je n'utilise pas le clustering seul comme vérité. Une annotation médicale coûte, selon l'expertise, **0,5 à 5 € la pièce** : les 5 000 € achètent donc **~1 000 à 10 000 labels** — et choisis par **active learning**, ils valent **2 à 3 fois** des labels tirés au hasard. Pourquoi c'est indispensable : mon self-training laisse **~3 % du pool en zone ambiguë** (97,2 % couverts, mesuré NB2) ; à 4 millions d'images, ça fait **~120 000 cas incertains**, dont le budget n'annote au mieux que **quelques milliers**. On ne peut donc pas annoter tout l'ambigu : il faut **prioriser les cas les plus informatifs**.
  **Trois — vérification humaine systématique des positifs**, sécurité patient oblige. C'est un coût **opérationnel récurrent**, distinct des 5 000 € de construction : à ~1 % de positifs sur 4 M, c'est **~40 000 cas à revoir**. Le seuil est calibré haut sur le **recall** : on accepte des fausses alertes que l'humain filtre, **jamais un cancer raté non vu**.
  Conditions : outil de triage R&D, pas diagnostic ; seuil calibré recall ; validation clinique ; monitoring de dérive ; ré-entraînement. »
- **Transition** : « En résumé, chacun de mes choix techniques répond à une contrainte métier — chiffres à l'appui. »
- **Design** : 3 briques numérotées avec icônes, **un chiffre-choc par brique** (`< 0,1 %` · `1 000-10 000 labels` · `~40 000 revues`) ; encadré « Conditions » en bas. C'est **LA** slide à soigner. ⚠️ Présente ces nombres comme **ordres de grandeur avec hypothèses** (débit GPU, €/label, taux de positifs) — un ordre de grandeur défendable bat un chiffre faussement précis.

### Slide 10bis — Le funnel : 4 M non labellisées → 4 M labellisées pour 5 000 €
- **Visuel** : **un entonnoir / boucle active learning** en 2-3 passes — on dépense le budget sur les **cas ambigus** (uncertainty sampling), le self-training propage, et on livre 4 M pseudo-labels **scorés en fiabilité**. Blueprint à redessiner en PowerPoint/Excalidraw :
```
                4 000 000 images NON labellisées
                          │  Features ResNet50 + inférence  (compute ≈ 2 €)
                          ▼
 ┌── PASSE 1 — SEED CLINIQUE CIBLÉ ───────────────────────────────┐
 │  quelques centaines de labels CLINIQUES, priorité aux CAS       │
 │  AMBIGUS de notre PoC (les 463 « à revoir » du NB1 / zone       │
 │  incertaine) = uncertainty sampling      ── part du budget      │
 └────────────────────────────────────────────────────────────────┘
                          │  ré-entraînement sur le seed
                          ▼
   SELF-TRAINING PROGRESSIF (seuils de confiance décroissants)
     couvre ~97 % du volume   →  ~3 880 000 img auto-labellisées
     (mesuré NB2 : 93 % strict / 97,2 % progressif ; ≥ 97 % avec le seed)
                          │  reste ambigu ~3 %
                          ▼
                ~120 000 cas encore incertains
                          │
 ┌── PASSE 2 — 2e LABELLISATION CIBLÉE ───────────────────────────┐
 │  quelques centaines de labels sur le résidu le plus informatif  │
 │                                          ── reste du budget     │
 └────────────────────────────────────────────────────────────────┘
                          │  ré-entraînement final
                          ▼
   LIVRAISON : 4 000 000 pseudo-labels + SCORE DE FIABILITÉ
     • grande majorité HAUTE fiabilité (auto-confiant + propagé)
     • ~1 000 graines EXPERTES cliniques (cas durs, quelques centaines/passe)
     • résidu FAIBLE fiabilité : marqué, NON forcé
   Budget : ~5 000 € (≈ 1 000 labels cliniques à ~5 €, en 2-3 passes) + 2 € compute
   ↻ 1 à 3 passes — même principe (boucle active learning)
```
- **Script** : « La question de Clara, frontalement : passer de 4 millions d'images **non** labellisées à 4 millions **labellisées** pour 5 000 € — possible ? Oui, avec une **boucle d'active learning** en deux ou trois passes. Features plus inférence — **~2 € de compute** — donnent à chaque image un score. **Passe 1 : je dépense une partie du budget sur un seed clinique CIBLÉ** — quelques centaines de labels de radiologue, mais pas n'importe lesquels : **les cas ambigus de mon travail**, ceux que le modèle ne tranche pas. C'est de l'**uncertainty sampling** : on annote là où ça fait le plus progresser le modèle. Je ré-entraîne, puis **self-training progressif** : il couvre **~97 % du volume** — et ça, c'est **mesuré** dans mon NB2, 97,2 %. Reste **~3 %, environ 120 000 cas** incertains. **Passe 2 : deuxième labellisation ciblée** sur ce résidu, avec le reste du budget, puis ré-entraînement final. À l'arrivée : **4 millions de pseudo-labels, chacun avec un score de fiabilité**, la grande majorité en **haute fiabilité** — et ce taux est **optimisé** justement parce que j'ai mis l'argent sur les cas durs. L'humain n'aura touché que **~0,02 % des images**. Une passe ou trois, le principe est le même : **on ne paie que les cas qui font progresser le modèle, le reste se propage**. »
- **Transition** : « Donc faisable — à condition d'assumer une livraison de pseudo-labels **tiérés par fiabilité**, pas 4 millions de labels cliniques. »
- **Design** : sablier + **flèche de boucle** (retour active learning). Code couleur : seed clinique **rouge** (cas ambigus), self-training **bleu**, livraison **verte tiérée par fiabilité**. Deux chiffres-chocs : **« ~0,02 % annotés à la main »** et **« 97 % couverts par self-training (mesuré) »**. Marque bien le **résidu faible fiabilité non forcé** — c'est ce qui rend la promesse honnête. ⚠️ Présenter les couvertures comme **mesurées** : **97,2 %** (progressif) / **93,3 %** (strict 0,95).

### Slide 11 — Choix techniques ↔ contraintes métier
- **Visuel** : un **tableau** à 3 colonnes : *Choix technique* | *Contrainte métier adressée* | *Bénéfice*.
  Lignes : transfer learning → budget/temps → pas d'entraînement coûteux · seed expert + active learning → budget annotation → modèle fiable à coût minimal · métrique recall cancer → sécurité patient → on ne rate pas de tumeur · dédoublonnage / test étanche → fiabilité → résultats crédibles.
- **Script** : « Pour résumer la cohérence de mes décisions : *(parcours le tableau)* le transfer learning répond au budget et au temps ; le couple seed expert + active learning répond au coût d'annotation ; le choix du recall répond à la sécurité patient ; et la discipline anti-fuite garantit que mes chiffres sont crédibles. »
- **Transition** : « Bien sûr, cette PoC a ses limites, que je veux poser clairement. »
- **Design** : tableau propre, une ligne = une décision. Aère.

### Slide 12 — Limites & conditions
- **Visuel** : liste courte : test petit (30 img) → splits répétés ; raccourcis intensité/cadrage ; pseudo-labels imparfaits ; **triage R&D ≠ diagnostic**.
- **Script** : « Je reste lucide. Mon jeu de test est petit — 30 images — donc j'ai ajouté des splits répétés et des baselines. Un point important : des features simples d'image atteignent déjà **AUC ≈ 0,92**, donc une partie du signal peut venir du cadrage ou de l'intensité, pas uniquement de la tumeur. Et surtout : ceci est un **outil de triage de recherche, jamais un dispositif de diagnostic** — toute mise en usage exige une validation clinique et une surveillance. »
- **Transition** : « Je conclus. »
- **Design** : icône « ⚠️ » discrète ; ne surcharge pas — 4 puces max.

### Slide 13 — Ouverture R&D : SSL avancé à tester
- **Visuel** : une roadmap courte en 3 niveaux : **Aujourd'hui validé** → consensus/self-training strict · **Prochain test** → FixMatch / Mean Teacher · **Plus long terme** → FlexMatch / MixMatch / SGAN. À droite, un encart « À valider par test sanctuarisé ».
- **Script** : « En ouverture, les ressources de cours donnent des pistes SSL plus avancées. **FixMatch** pourrait améliorer l'exploitation des images non labélisées via augmentations faibles/fortes et seuil de confiance. **FlexMatch** pourrait adapter les seuils par classe, utile si le pool est déséquilibré. **Mean Teacher** pourrait stabiliser l'apprentissage avec un modèle professeur moyenné. **MixMatch** et les **SGAN** sont plus ambitieux, mais aussi plus coûteux et plus instables. Le gain espéré serait d'augmenter la couverture pseudo-labélisée sans perdre en précision, voire d'améliorer le recall cancer. Mais je ne les vends pas comme acquis : il faudra les valider avec le même protocole que le self-training, c'est-à-dire folds sanctuarisés, métriques recall/F1/AUC, et contrôle des faux positifs/faux négatifs. »
- **Transition** : « Donc ma recommandation reste pragmatique : on industrialise ce qui est validé, et on garde ces méthodes comme backlog R&D mesurable. »
- **Design** : sépare bien *validé maintenant* et *à tester ensuite*. Mets en gros : « Piste ≠ recommandation tant que non validée ».

### Slide 14 — Conclusion & recommandation
- **Visuel** : 3 messages : ✔ pipeline complet livré · ✔ métrique et résultats justifiés · ➡️ **scaling faisable sous conditions** (transfer learning + active learning + vérification).
- **Script** : « En une phrase : oui, le passage à l'échelle est **faisable**, mais à condition de **ne pas chercher à tout labelliser** et de ne pas confondre pseudo-labels et labels médicaux. La valeur est dans le couple **transfer learning + active learning + validation experte ciblée**. Mon livrable : un pipeline reproductible, un benchmark plus robuste, une comparaison honnête, et une recommandation chiffrée. Je suis prêt pour vos questions. »
- **Design** : 3 lignes, beaucoup de blanc. Termine sur la recommandation, pas sur les limites.

---

## 3. Slides d'annexe (backup pour la discussion)

À garder après la slide 14, pour répondre à chaud sans chercher dans tes notebooks :
- **A1** — Courbes ROC des deux modèles → *Notebook 2 › section 4.5 › cellule ROC*.
- **A2** — Courbe des k-distances (justifie le réglage de DBSCAN) → *Notebook 1 › section clustering › cellule k-distances*.
- **A3** — Histogrammes d'intensité par classe + scatter luminosité×contraste → *Notebook 1 › EDA*.
- **A4** — Visualisation de paires de doublons / fuites → *Notebook 1 › EDA › cellule doublons*.
- **A5** — Détail du dédoublonnage (1506 → 1281, étanchéité MD5 + phash = 0).
- **A6** — Table benchmark clustering + baselines supervisées → *Notebook 1 › benchmark clustering* et *Notebook 2 › benchmarks additionnels*.

---

## 4. Préparer la discussion (10 min) — questions probables de Clara

> Réponds court, assume, et **ramène toujours à une mesure**.

- **« Pourquoi ResNet et pas un autre modèle / from scratch ? »**
  → Trop peu de données pour entraîner de zéro (sur-apprentissage) ; le transfer learning réutilise un savoir visuel et coûte zéro annotation. ResNet est un standard robuste et léger à servir.
- **« Pourquoi ne pas garder K-Means comme pseudo-labels ? »**
  → Parce que le benchmark montre qu'il est moins bon : K-Means est à **ARI ≈ 0,13**, l'agglomératif PCA100 atteint **ARI ≈ 0,535**, et le consensus de pseudo-labelers atteint **ARI filtré ≈ 0,875** en out-of-fold. Je garde K-Means comme baseline, pas comme meilleur choix.
- **« Pourquoi réduire la dimension si ResNet sort 2048 features ? »**
  → Je ne le présume pas : je l'ai testé. Agglomerative RAW2048 et Agglomerative PCA100 donnent le même ARI ≈ **0,535** et la même balanced accuracy ≈ **0,869**. Je garde PCA100 parce qu'elle est plus compacte sans perte mesurée sur ce critère.
- **« Pourquoi le semi-supervisé ne domine pas ? »**
  → Baseline déjà forte grâce au transfer learning **+** pseudo-labels encore imparfaits. Sur 3 splits, le semi-supervisé augmente le recall moyen (**0,978 vs 0,911**) mais baisse en precision/accuracy/F1 ; c'est une piste pour réduire les faux négatifs, pas encore un gain global établi.
- **« Pourquoi ne pas itérer jusqu'à pseudo-labéliser presque tout ? »**
  → Je l'ai testé en outer CV 5 folds sanctuarisée. À seuil strict **0,95**, le self-training couvre **93,3 %** du pool sans gain net ; avec seuils progressifs, il couvre **97,2 %**, mais l'accuracy/F1/recall baissent. Les derniers cas sont les plus ambigus : ils doivent aller vers l'annotation experte.
- **« Pourquoi pas FixMatch / MixMatch / Mean Teacher / GAN semi-supervisé ? »**
  → Ce sont de bonnes pistes R&D, vues dans les ressources de cours, mais elles demandent des augmentations fortes calibrées pour l'IRM et une validation plus coûteuse. Pour ce livrable, le cahier des charges demande surtout features pré-entraînées, clustering, ARI et comparaison semi-supervisée. J'ai donc poussé les méthodes les plus alignées avec le problème métier : consensus, LabelSpreading et self-training leakage-safe.
- **« Ton test fait 30 images, c'est fiable ? »**
  → C'est petit, donc je ne sur-interprète pas un split. J'ai ajouté des baselines en validation croisée répétée et une comparaison CNN sur 3 splits pour mesurer la stabilité.
- **« Pourquoi le recall et pas l'accuracy ? »**
  → Parce que rater un cancer (faux négatif) est l'erreur la plus grave. L'accuracy masquerait des faux négatifs sur un jeu déséquilibré.
- **« Comment garantis-tu l'absence de fuite ? »**
  → Dédoublonnage exact + quasi (MD5 + phash), vérification « 0 hash commun » et « 0 paire phash ≤ 5 » entre annoté et non annoté, et test issu uniquement du jeu fort, jamais vu à l'entraînement.
- **« Le scaling est-il vraiment faisable ? »**
  → Oui **sous conditions** : compute négligeable + budget concentré sur l'annotation ciblée (active learning) + vérification humaine + validation clinique. Pas l'annotation exhaustive.
- **« Que ferais-tu avec plus de temps/budget ? »**
  → Validation croisée ; tester un fine-tuning partiel du backbone ; corriger le biais de cadrage (recadrage/normalisation) ; boucle d'active learning réelle ; calibration du seuil sur le recall.

---

## 5. Checklist finale (couverture de la grille d'auto-éval)

- [ ] Recommandations techniques pour le déploiement à grande échelle (4 M images, 5 000 €) → slides 9-10
- [ ] Lien choix techniques ↔ contraintes métier (volume, budget, temps) → slide 11
- [ ] Arguments préparés pour justifier choix techniques **et** métier → section 4
- [ ] Cohérence résultats notebooks ↔ recommandations → mêmes chiffres partout (ARI clustering 0.535, ARI consensus 0.875, self-training 93,3 %/97,2 %, recall semi 1.00, 0,00125 €/image)
- [ ] ≤ 15 slides (14 + annexes) · durée 15 min (±5)
