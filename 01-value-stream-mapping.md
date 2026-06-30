# Value Stream Mapping : cartographier le flux de l'idée à la prod

## Objectif

À la fin de cet atelier, vous savez cartographier sur papier le parcours complet d'une
modification, de l'idée jusqu'à la production. Vous distinguez le temps de travail du temps
d'attente, vous calculez l'efficacité du flux, et vous identifiez le goulot d'étranglement, c'est
à dire l'étape qui ralentit tout le reste. C'est le diagnostic qui justifiera, dans les jours
suivants, pourquoi on automatise telle ou telle étape.

## Contexte

Avant d'automatiser quoi que ce soit, il faut voir ou part le temps. Le DevOps ne consiste pas a
empiler des outils, mais à raccourcir le délai entre "on a une idée" et "c'est en prod, ça marche".
Le Value Stream Mapping (cartographie de la chaîne de valeur) est l'outil qui rend ce délai visible.
Aujourd'hui, pas de code : papier, feutres, et un cas concret tiré de notre fil rouge. On produit
une carte qu'on gardera comme référence pour mesurer nos progrès quand on mettra en place la CI, le
conteneur et le déploiement.

Travaillez en binôme ou en trinôme.

## Consignes

### 1. Choisir et borner le cas

Le cas à cartographier : livrer un correctif sur l'appli fil rouge. Scénario concret : un
utilisateur signale que `POST /tasks` accepte un titre fait uniquement d'espaces. Vous devez
corriger ce comportement et le livrer en production.

Bornez clairement la carte :

- Point de départ : le bug est signale et vous décidez de le traiter.
- Point d'arrivée : le correctif tourne en production et un utilisateur peut le constater.

Tout ce qui est avant (découverte du bug) ou après (autre demande) sort de la carte.

### 2. Lister les étapes dans l'ordre

Sur un outil pour faire des schemas gauche à droite, posez une carte par étape, dans l'ordre réel du flux.
Listez ce qui se passe vraiment chez vous, pas un monde idéal. Une liste typique pour ce cas :

- Le ticket attend dans la file avant d'être pris.
- Reproduction du bug et écriture du correctif.
- Le code attend une revue (la PR est ouverte mais personne ne regarde encore).
- Revue de la PR par un collègue.
- Lancement et attente des tests.
- Le correctif attend un créneau de déploiement.
- Déploiement en production.
- Vérification que c'est bon en prod.

N'hésitez pas à ajouter ou retirer des étapes selon votre organisation. L'important est que la
suite soit fidèle à la réalité.

### 3. Chronométrer chaque étape : travail et attente

Pour chaque étape, estimez deux durées distinctes :

- Le temps de travail (process time) : la durée pendant laquelle quelqu'un agit réellement sur la
  tâche.
- Le temps d'attente (wait time / lead time) : la durée pendant laquelle la tâche ne bougé pas
  (elle attend dans une file, une validation, un créneau).

Notez ces deux nombres sous chaque carte, dans la même unité (minutes ou heures, choisissez et
tenez vous y). Soyez honnêtes : une revue qui prend 10 minutes mais qui attend 1 jour avant d'être
faite, c'est 10 minutes de travail et beaucoup d'attente.

### 4. Calculer le total et l'efficacité

Additionnez séparément :

- La somme de tous les temps de travail.
- La somme de tous les temps d'attente.
- Le temps total (lead time complet) = travail + attente, du tout premier au tout dernier instant.

Puis calculez l'efficacité du flux :

```
efficacite = temps de travail total / temps total
```

Exprimez le résultat en pourcentage. Ne soyez pas surpris si c'est bas : sur beaucoup de chaînes
réelles, l'efficacité est sous les 15 pour cent. C'est tout l'intérêt de la mesure.

### 5. Identifier LE goulot

Repérez l'unique étape qui contribue le plus au délai total. Le goulot, ce n'est pas forcément
l'étape la plus longue en travail : c'est le plus souvent une grosse attente. Entourez la sur la
carte.

Questions guides pour conclure :

- Quelle étape pèse le plus lourd dans le temps total ? Est-ce du travail ou de l'attente ?
- Si vous ne pouviez améliorer qu'une seule étape, laquelle, et pourquoi celle la ?
- Cette étape est-elle un problème d'outillage, de disponibilité des personnes, ou de politique
  interne (par exemple un seul créneau de déploiement par semaine) ?
- Que faudrait il pour faire tomber cette attente a presque zéro ?

## Livrable

Vous avez réussi si :

- Votre carte montre toutes les étapes, du signalement du bug à la vérification en prod, dans
  l'ordre.
- Chaque étape porte un temps de travail et un temps d'attente, dans la même unité.
- Les trois totaux sont calculés : travail total, attente totale, temps total.
- L'efficacité du flux est calculée en pourcentage avec la formule donnée.
- UN goulot est entoure et vous savez l'expliquer en une phrase (sa nature et sa cause).
- Vous pouvez nommer une action concrète qui réduirait ce goulot.

## Aide

### La formule, au clair

```
temps total      = temps de travail total + temps d'attente total
efficacite (%)   = (temps de travail total / temps total) x 100
```

Une efficacité de 100 pour cent voudrait dire qu'on travaille en continu sans aucune attente. En
pratique c'est impossible et ce n'est pas le but : le but est de voir ou se cache l'attente pour la
réduire.

### Exemple de cartographie résolu

Cas : livrer le correctif "titre vide refuse" sur le fil rouge. Estimations en minutes.

- Ticket en file d'attente : travail 0, attente 480 (il attend une demi-journée).
- Reproduction et correctif : travail 40, attente 0.
- PR en attente de revue : travail 0, attente 240.
- Revue de la PR : travail 15, attente 0.
- Tests (lances à la main, on attend le résultat) : travail 5, attente 10.
- Attente d'un créneau de déploiement : travail 0, attente 600.
- Déploiement manuel : travail 20, attente 0.
- Vérification en prod : travail 10, attente 0.

Calculs :

- Temps de travail total : 40 + 15 + 5 + 20 + 10 = 90 minutes.
- Temps d'attente total : 480 + 240 + 10 + 600 = 1330 minutes.
- Temps total : 90 + 1330 = 1420 minutes.
- Efficacité : 90 / 1420 = 0,063, soit environ 6,3 pour cent.

Lecture : on ne "travaille" que 90 minutes sur un délai de presque 24 heures. Le goulot n'est pas
le code (40 minutes) mais l'attente d'un créneau de déploiement (600 minutes) et l'attente de revue
(240 minutes). L'action la plus rentable n'est pas de coder plus vite : c'est de supprimer le
créneau unique de déploiement (déployer à la demande) et de raccourcir le délai de revue. C'est
exactement ce qu'on adressera avec la chaîne d'intégration et un déploiement automatisé dans les
jours suivants.

### Pièges à éviter

- Confondre travail et attente. Si personne ne touche la tâche, c'est de l'attente, même si "ça
  avance" dans votre tête.
- Mélanger les unités (des minutes ici, des heures la). Choisissez une seule unité et convertissez
  tout.
- Designer comme goulot l'étape la plus visible plutôt que la plus coûteuse. Laissez les chiffres
  décider.
- Vouloir corriger plusieurs étapes à la fois. On cible UN goulot : une fois leve, un autre
  apparaîtra, et on recommencera.

### Pour aller plus loin dans la discussion

- Comparez vos cartes entre groupes : les goulots se ressemblent souvent (revue, créneau de
  déploiement, environnement de test indisponible).
- Gardez une photo de votre carte. On la ressortira en fin de cours pour estimer combien
  l'automatisation a fait gagner sur l'efficacité du flux.
