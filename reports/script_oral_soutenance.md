# Script oral — Soutenance BrainScanAI (deck final, 13 slides)

> **Durée cible : ~15 min.** Tu ne lis pas les slides : elles portent l'image et 3 mots-clés, **toi tu racontes**.
> **Fil rouge à répéter mentalement : « la donnée pilote chaque décision ».**
> **Arc narratif :** la promesse → le succès trop beau → le retournement (biais) → la reconstruction honnête → la vision (passage à l'échelle).
> Les **transitions en gras** sont à dire telles quelles : chacune ouvre une question que la slide suivante vient résoudre.

| Slide | Durée | Rôle narratif |
|---|---|---|
| 1 Titre | 0:20 | accroche |
| 2 Contexte/DoD | 1:00 | la promesse & la contrainte |
| 3 Démarche | 0:45 | la carte du voyage |
| 4 Doublons/fuite | 1:00 | la rigueur d'abord |
| 5 Biais d'acquisition | 1:30 | **le soupçon** |
| 6 Features | 0:45 | l'outil |
| 7 Clustering/consensus | 1:15 | la stratégie pseudo-labels |
| 8 Trop beau | 1:30 | **le retournement** |
| 9 Neutraliser le biais | 1:15 | la reconstruction |
| 10 Sup vs semi | 1:30 | le verdict |
| 11 Self-training | 1:00 | l'exploitation poussée |
| 12 Passage à l'échelle | 2:00 | **la vision chiffrée** |
| 13 Conclusion | 1:00 | la promesse tenue |

---

## Slide 1 — Titre
**[À l'écran]** *Détecter les tumeurs cérébrales sur IRM — exploration semi-supervisée.*

« Bonjour à toutes et à tous. Je suis Kévin Raya, Data Scientist junior. Je vais vous présenter une enquête en vision par ordinateur autour d'une question très concrète : **peut-on détecter des tumeurs cérébrales sur IRM quand presque aucune image n'est annotée ?** Vous le verrez, c'est une histoire avec un rebondissement — un résultat qui paraissait parfait, et qui ne l'était pas du tout. Et un fil rouge : à chaque étape, **c'est la donnée qui décide, pas mon envie que ça marche.** »

> **→ Transition :** « Et tout commence par un constat de terrain assez frustrant. »

---

## Slide 2 — Contexte, mission & Definition of Done
**[À l'écran]** *1506 IRM · répartition annotées / non labellisées · budget labellisation 300 € · DoD · métrique = Recall.*

« On nous confie **1 506 IRM** en 512×512. Mais regardez la répartition : à peine une **centaine** sont annotées par des radiologues — le reste, plus de mille, est **sans label**. Pourquoi ? Parce qu'**annoter coûte cher** : notre budget de labellisation est dérisoire, **300 €**. D'où ma mission : **peut-on exploiter ces images non annotées** pour aider un modèle ?

Et avant même de coder, une question que je trouve plus importante : **qu'est-ce qu'un bon modèle ici ?** En imagerie médicale, l'erreur grave, ce n'est pas une fausse alerte — c'est de **rater une tumeur**, un faux négatif. Donc ma métrique de réussite n'est pas l'accuracy, c'est le **recall sur le cancer**. Mon cadre de qualité, ma *Definition of Done*, tient en trois points : un **pipeline reproductible sans fuite**, un **clustering validé par l'ARI**, et une **comparaison rigoureuse supervisé / semi-supervisé**. »

> **→ Transition :** « Pour répondre à ça proprement, j'ai déroulé une démarche en cinq temps — la voici. »

---

## Slide 3 — Démarche d'ensemble
**[À l'écran]** *EDA → Prétraitement & Extraction → Apprentissage → Modélisation → Passage à l'échelle.*

« Cinq étapes. D'abord **l'EDA**, connaître mes données. Ensuite **le prétraitement et l'extraction de features** avec un réseau pré-entraîné. Puis **l'apprentissage** : du clustering pour générer des pseudo-labels. Ensuite **la modélisation** : comparer CNN supervisé et semi-supervisé. Et enfin **le passage à l'échelle**, la recommandation pour l'entreprise. Je vais suivre ce fil exactement. »

> **→ Transition :** « Première étape, et pour moi non négociable : **on regarde les données avant de modéliser.** »

---

## Slide 4 — EDA : Doublons & fuite
**[À l'écran]** *MD5 : 186 · quasi-doublons : 179 · 32 fuites annoté/non annoté · dataset final 1 281 (98 / 1 183).*

« Premier réflexe : auditer. Et je trouve deux problèmes. Des **doublons exacts** — détectés par empreinte MD5, 186 images. Des **quasi-doublons** perceptuels — détectés par hachage perceptuel, 179 images. Et surtout, le plus dangereux : **32 fuites**, des images qui se retrouvent à la fois dans l'annoté et le non-annoté. Si je n'y touche pas, le modèle "révise avec le corrigé" et mes scores sont **artificiellement gonflés**.

Je nettoie donc tout ça. On passe de 1 506 à **1 281 images** : **98 annotées, 1 183 non annotées**, et zéro fuite résiduelle. C'est moins de données, mais des données **honnêtes**. »

> **→ Transition :** « Données dédoublonnées, oui — mais une anomalie bien plus sournoise m'attendait dans ces images. »

---

## Slide 5 — EDA : Biais protocole d'acquisition
**[À l'écran]** *Moyenne cancer / moyenne normal / carte de différence (bleu-rouge).*

« Voici une expérience toute simple mais qui change tout. Je calcule l'**image moyenne** des cancers, celle des normaux, et je les **soustrais**. Si la différence venait de la tumeur, elle devrait s'allumer **au centre**, dans le cerveau. Or regardez : elle s'allume **sur les bords**, sur le pourtour de la tête.

Traduction : ce qui sépare le mieux mes deux classes, à ce stade, ce n'est pas la pathologie — c'est le **cadrage, l'intensité, le protocole d'acquisition**. Mes images "cancer" et "normal" ne viennent probablement pas des mêmes machines. **Le risque, c'est qu'un modèle apprenne ce biais au lieu d'apprendre la tumeur.** Je note ce soupçon, et je continue. »

> **→ Transition :** « Gardez cette carte en tête — on va y revenir d'une façon spectaculaire. Pour l'instant, transformons ces images en quelque chose d'exploitable. »

---

## Slide 6 — Extraction de features (Transfer Learning)
**[À l'écran]** *IRM → ResNet50 gelé → vecteur 2048 · normalisation · PCA.*

« Je n'ai ni le budget ni assez de données pour entraîner un réseau de zéro — ce serait du sur-apprentissage garanti. J'utilise donc le **transfer learning** : un **ResNet50 pré-entraîné** sur ImageNet, dont je **gèle les couches** et **retire la tête**. Chaque IRM devient un **vecteur de 2 048 caractéristiques**. Je normalise, je réduis la dimension par **PCA**. C'est rapide, ça ne coûte **aucune annotation**, et ça réutilise un savoir visuel généraliste — retenez ce point, il sera décisif pour le passage à l'échelle. »

> **→ Transition :** « Avec ces vecteurs, je pose la question naturelle : les images se regroupent-elles spontanément par pathologie ? »

---

## Slide 7 — Clustering & pseudo-labels par consensus
**[À l'écran]** *t-SNE (RAW2048, Agglo k=2) · consensus · pseudo-labellisation · expertise humaine ciblée.*

« J'ai testé plusieurs algorithmes de clustering. Le meilleur sépare *en partie* les deux groupes, mais c'est **trop bruité** pour étiqueter aveuglément un millier d'images. Plutôt que de me fier à un seul algorithme, j'ai construit un **consensus** de trois modèles — SVC, ExtraTrees et LabelSpreading — évalué proprement, **hors échantillon**. Ce consensus filtré atteint un **ARI d'environ 0,875**, bien plus fiable.

Ma règle devient simple et honnête : je **garde** les pseudo-labels sur lesquels les trois modèles sont **d'accord et confiants**, et j'envoie les cas **ambigus** vers une **annotation humaine ciblée**. On ne force jamais un cas douteux. »

> **→ Transition :** « J'avais donc des features et des pseudo-labels. J'entraîne un premier modèle complet… et là, surprise. »

---

## Slide 8 — Un peu trop beau pour être vrai
**[À l'écran]** *AUC ~0,98 (30 test / 68 train) · le bord seul prédit le cancer à AUC ~0,79 · dataset biaisé.*

« Mon modèle obtient une **AUC de 0,98**. Évalué proprement, en validation croisée. Sur 68 images d'entraînement, pour une tâche que l'œil humain ne tranche pas facilement… **c'est trop beau.** Et quand c'est trop beau, je me méfie.

Alors je fais le test décisif. Je masque tout le cerveau, je ne garde que le **bord de l'image** — une zone qui ne contient **aucun tissu cérébral** — et je demande : ce bord seul prédit-il le cancer ? Réponse : **AUC 0,79.** C'est **cliniquement impossible**. Ça prouve que mes classes diffèrent par leur **protocole d'acquisition**, pas par la tumeur. Mon "quasi-parfait" était **gonflé par des métadonnées techniques.**

C'est un résultat *négatif* — et pourtant c'est ma plus belle découverte : **une grosse performance ne garantit pas la validité clinique.** »

> **→ Transition :** « Restait la vraie question d'ingénieur : ce biais, peut-on le neutraliser ? »

---

## Slide 9 — Neutraliser le biais
**[À l'écran]** *Grille : brut / zone retirée (rouge) / cerveau masqué / masqué + CLAHE.*

« J'ai tenté trois approches, de la plus douce à la plus radicale. La plus radicale, la voici : **ne garder que le cerveau.** J'isole la tête, je mets tout le reste à zéro et je recadre. Le **rouge**, sur cette image, c'est exactement ce que je **supprime** : bordure, cadrage, fond. J'ajoute du CLAHE pour égaliser l'intensité, et des augmentations intensives pour casser les corrélations parasites.

Et soyons honnêtes sur le résultat : le biais **grossier recule** — l'AUC tombe de 0,98 vers 0,90–0,95 — **mais ne disparaît pas**. Un signal résiduel persiste dans le tissu lui-même, et là, **on ne peut pas trancher** entre un confondant d'acquisition et une vraie différence pathologique. Conséquence méthodologique forte : **à partir d'ici, je benchmarke uniquement sur les données traitées.** »

> **→ Transition :** « Sur ces données assainies, je peux enfin répondre à la question du projet : le semi-supervisé apporte-t-il quelque chose ? »

---

## Slide 10 — Supervisé vs Semi-supervisé
**[À l'écran]** *F1 : 0,82 → 0,91 · Recall : 0,79 → 0,95.*

« Deux modèles, **exactement le même jeu de test, jamais vu**. Le supervisé n'apprend que sur mes images annotées. Le semi-supervisé se **pré-entraîne sur les pseudo-labels**, puis s'affine.

Le verdict est net, surtout sur ce qui compte pour nous. Le supervisé : **F1 0,82, recall 0,79 — mais instable** d'un tirage à l'autre. Le semi-supervisé : **F1 0,91, recall 0,95 — et stable.** Autrement dit, une fois le raccourci neutralisé, **le semi-supervisé redevient utile** : il rate moins de cancers et il est plus régulier. C'est exactement l'inverse de ce qu'on voyait sur les données brutes, où le biais rendait tout le monde "parfait" — la valeur des pseudo-labels était **masquée**. »

> **→ Transition :** « Et si le pré-entraînement aide déjà, jusqu'où peut-on pousser l'exploitation des images non annotées ? »

---

## Slide 11 — Self-training itératif
**[À l'écran]** *Schéma boucle · performance optimisée · couverture étendue · approche conservatrice · bénéfice opérationnel.*

« J'ai testé le **self-training** : pseudo-labéliser le pool par **paliers de confiance décroissante**, en réentraînant à chaque tour — le tout **sans fuite**, avec un test sanctuarisé à chaque fold. Résultat : on couvre de **93 % à 97 % du pool** sans dégrader la performance. La double valeur est claire : **prédictive**, modeste mais réelle ; et surtout **opérationnelle** — on réduit drastiquement le volume à annoter à la main. Et toujours la même prudence : **les derniers cas, les plus ambigus, on ne les force pas** — ils partent à l'expert. »

> **→ Transition :** « Et c'est précisément ce mécanisme qui répond à la question finale de l'entreprise : le passage à l'échelle. »

---

## Slide 12 — Passage à l'échelle : stratégie proposée
**[À l'écran]** *Pyramide 4 étages · objectif 4 M d'images pour 5 000 €.*

« Le défi posé : **4 millions d'images, 5 000 €.** D'abord la réalité : annoter tout à la main, même au tarif le plus optimiste, c'est **des millions d'euros** — des centaines de fois le budget. **Exclu.** Ma stratégie ne paie donc l'humain que là où ça compte vraiment. Quatre étages.

**Un — uncertainty sampling.** Je dépense le **gros du budget** sur quelques centaines de labels **cliniques**, en priorité sur les **cas ambigus** de mon étude. On passe d'une centaine à **500–1 000 annotations fortes**, mais bien choisies.

**Deux — self-training progressif :** ces graines propagent les labels et couvrent **~97 % du volume** automatiquement, pour quelques euros de calcul.

**Trois — labellisation résiduelle :** une seconde passe ciblée sur les cas encore ambigus, notamment les négatifs douteux.

**Quatre — un nouveau self-training**, qui produit le **livrable final**.

À l'arrivée : **4 millions d'images étiquetées**, chacune avec un **score de fiabilité**, la grande majorité en haute fiabilité. L'humain n'aura touché que **~0,02 % des images**. Et je reste honnête : ce livrable, c'est des **pseudo-labels tiérés par fiabilité**, pas 4 millions de vérités cliniques. »

> **→ Transition :** « Ce qui me permet de conclure sur ce que ce projet a vraiment livré. »

---

## Slide 13 — Conclusion & Recommandation
**[À l'écran]** *Pipeline robuste · Neutralisation des biais · Validation du semi-supervisé · Recommandation de passage à l'échelle.*

« Quatre choses. Un : un **pipeline reproductible et sans fuite**. Deux : un **biais d'acquisition démontré et neutralisé** — la rigueur scientifique avant le score flatteur. Trois : une **validation honnête du semi-supervisé**, qui apporte de la **stabilité et du recall** une fois le biais écarté. Quatre : une **recommandation de passage à l'échelle chiffrée et réaliste**.

En une phrase : **oui, on peut exploiter les images non annotées — à condition de ne jamais confondre un pseudo-label avec une vérité clinique.** Merci de votre attention, je suis prêt pour vos questions. »

---

## Réserve de réponses (questions probables)
- **« 0,98, c'est un échec ? »** → Non : c'est un **diagnostic**. Le score était réel mais dû au protocole. L'avoir prouvé (bord à 0,79) m'évite de livrer un modèle qui s'effondrerait sur un nouveau scanner.
- **« Test à 30 images, fiable ? »** → Petit, donc je ne lis jamais un seul split : CV répétée, moyenne ± écart-type, baselines.
- **« Pourquoi le recall et pas l'accuracy ? »** → Rater un cancer (faux négatif) est l'erreur grave ; l'accuracy masquerait ces faux négatifs.
- **« 4 M labellisées à 5 000 €, vraiment ? »** → Pas 4 M de labels cliniques : **~1 000 labels experts + propagation**, livraison **tiérée par fiabilité**, l'humain ne touche que ~0,02 % des images.
- **« Pourquoi pas K-Means seul ? »** → Trop bruité pour pseudo-labéliser ; je garde le **consensus filtré (ARI ~0,875)**, baseline conservée.
