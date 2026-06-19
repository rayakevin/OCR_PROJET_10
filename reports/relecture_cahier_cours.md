# Relecture croisee - cahier des charges et ressources de cours

## Verdict

Le projet couvre le cahier des charges et va au-dela du minimum attendu sans sortir du cadre de la mission.

Le minimum demande : exploration image, extraction de features via modele pre-entraine, reduction de dimension, clustering K-Means/DBSCAN, ARI sur les images labellisees, approche semi-supervisee CNN, comparaison supervise vs semi-supervise, metriques pertinentes, definition of done et recommandation de passage a l'echelle.

Le projet ajoute : dedoublonnage MD5 + phash, benchmark clustering elargi, baseline RAW2048 sans PCA, pseudo-labellisation par consensus, self-training iteratif leakage-safe en outer CV 5 folds, baselines supervises sur embeddings, controle des raccourcis via features simples.

## Conformite cahier des charges

| Exigence | Couverture actuelle |
|---|---|
| Exploration visuelle, resolution, canaux, structure | Notebook 1, parties EDA et integrite |
| Nettoyage / coherence / biais | Doublons exacts + quasi-doublons, fuite labelise/sans_label retiree, analyse cadrage/intensite |
| Extraction ResNet ou equivalent | ResNet50 gele, embeddings 2048 sauvegardes |
| Preprocessing modele | Redimensionnement, normalisation ImageNet, standardisation features |
| Reduction dimension | PCA pour benchmark, t-SNE pour visualisation |
| Clustering K-Means / DBSCAN | Presents et conserves comme baselines |
| Tester plusieurs algorithmes | K-Means, DBSCAN, Agglomerative, Gaussian Mixture, Spectral, LabelSpreading, SVC, ExtraTrees, self-training |
| ARI sur images labellisees | ARI clustering + ARI out-of-fold pseudo-labelers |
| Jeux pseudo-labelise et labelise separes | CSV separes, test uniquement issu des images labellisees |
| CNN semi-supervise | Pre-entrainement pseudo-labelise puis affinage labelise |
| Comparaison supervise vs semi-supervise | Split principal + comparaison repetee |
| Test jamais vu | Split dedie, et exploration self-training en outer CV sanctuarisee |
| Metriques pertinentes | Recall cancer, precision, F1, ROC-AUC, balanced accuracy |
| Recommandation 4M images / 5000 euros | Guide presentation, slides 9-11 |

## Apports du repo de cours

### Pseudo-labeling

Le cours insiste sur les seuils de confiance et les iterations. Le projet l'integre avec :

- pseudo-labels filtres par consensus ;
- seuils de confiance ;
- self-training iteratif ;
- suivi du nombre d'images ajoutees ;
- outer CV sanctuarisee pour eviter la fuite.

C'est la piste la plus directement adaptee au sujet financier : labelliser moins a la main tout en controlant le bruit.

### Label propagation / Label spreading

Le cours propose LabelSpreading sur embeddings. Le projet l'integre dans le benchmark de pseudo-labellisation et dans le consensus final.

Resultat : LabelSpreading seul n'est pas le meilleur candidat, mais il apporte une famille de decision differente dans le consensus SVC + ExtraTrees + LabelSpreading.

### FixMatch / FlexMatch / MixMatch

Ces methodes sont pertinentes en SSL moderne, mais moins adaptees comme ajout de derniere passe dans ce projet :

- elles demandent une boucle deep learning plus longue ;
- les augmentations fortes doivent etre calibrees pour de l'IRM medicale, sinon elles peuvent detruire le signal clinique ;
- avec seulement 98 labels experts, leur evaluation robuste demanderait une outer CV couteuse ;
- le cahier des charges ne les impose pas.

Decision : les mentionner comme piste R&D de niveau suivant, pas les ajouter au benchmark principal. Gain espere : augmenter la couverture pseudo-labelisee et stabiliser le recall sans degrader la precision. Condition : valider par test sanctuarise, avec augmentations medicalement plausibles.

### Mean Teacher / regularisation par coherence

La logique student/teacher est pertinente, surtout pour industrialiser le SSL. Ici, elle serait plus lourde a implementer proprement qu'une boucle de pseudo-labeling sur embeddings et moins directement comparable au protocole demande.

Decision : piste future, a tester apres calibration d'augmentations medicalement plausibles. Gain espere : predictions plus stables sur les images non labellisees. Condition : comparer au self-training strict avec les memes folds et les memes metriques.

### Semi-supervised GANs

Les SGAN sont hors du coeur du cahier des charges et presentent un risque important de complexite / instabilite pour un gain incertain sur un petit set labellise.

Decision : ne pas retenir dans ce livrable ; mention possible en ouverture R&D uniquement. Gain espere : enrichir l'apprentissage via une perte adversariale sur le non labellise. Risque : instabilite forte et cout de validation superieur.

## Point critique a assumer

Le meilleur modele global reste tres proche du supervise sur embeddings. La vraie valeur de l'approche semi-supervisee n'est donc pas de "battre" le supervise partout, mais de construire une strategie de triage :

1. pseudo-labelliser automatiquement les cas tres confiants ;
2. ne pas forcer les cas ambigus ;
3. diriger les cas ambigus vers l'annotation experte ;
4. utiliser la validation sanctuarisee pour surveiller toute derive.

## Recommandation finale

Pour la soutenance, presenter le cahier des charges comme le socle minimum, puis expliquer l'extension :

- Minimum respecte : ResNet, PCA/t-SNE, K-Means/DBSCAN, ARI, CNN semi-supervise, comparaison.
- Extension justifiee : benchmark plus large, consensus, self-training iteratif leakage-safe.
- Ouverture R&D : FixMatch/FlexMatch/MixMatch/Mean Teacher/SGAN comme pistes a tester, avec gains attendus mais sans les presenter comme valides tant qu'un test sanctuarise ne les confirme pas.
- Conclusion metier : le passage a l'echelle n'est faisable que si l'on combine pseudo-labelisation prudente, active learning et annotation experte ciblee.
