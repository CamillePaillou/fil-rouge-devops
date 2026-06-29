# Mise en place : le fil rouge qui tourne et une première PR

## Objectif

À la fin de ce TP, vous avez le projet fil rouge dans votre propre dépôt Git, ses dépendances
installées, les tests qui passent au vert, l'API qui répond sur votre machine, et une première
pull request ouverte sur GitHub. Bref, tout le monde part de la même base saine et sait livrer
une modification par une PR.

## Contexte

Le fil rouge du cours est une petite API de tâches en Node.js (Express). Elle est volontairement
minimale : on lui ajoutera la chaîne d'intégration, le conteneur et l'observabilité au fil des
jours. Aujourd'hui on ne code presque rien, on s'assure juste que la boucle locale fonctionne de
bout en bout et que votre flux Git/GitHub est opérationnel. Vous connaissez déjà Git et GitHub,
donc on va vite : l'enjeu est que tout le monde ait exactement le même point de départ.

## Consignes

### 1. Récupérer le starter-code dans votre dépôt

Vous partez du dossier `starter-code/` fourni. Créez un dépôt vide sur GitHub (nom au choix, par
exemple `fil-rouge-devops`), puis copiez le contenu du starter dedans. Le plus simple :

```bash
# Copier le starter dans un nouveau dossier de travail (adaptez les chemins)
cp -R starter-code fil-rouge-devops
cd fil-rouge-devops

# Repartir d'un historique Git propre
rm -rf .git
git init
git add .
git commit -m "chore: point de depart fil rouge"

# Relier au depot distant cree sur GitHub (remplacez l'URL par la votre)
git branch -M main
git remote add origin git@github.com:VOTRE-COMPTE/fil-rouge-devops.git
git push -u origin main
```

Vérifiez sur GitHub que les fichiers sont bien la (`src/`, `test/`, `package.json`, `README.md`).

### 2. Installer les dépendances

```bash
npm install
```

Un dossier `node_modules/` apparaît et un `package-lock.json` est créé ou mis à jour. Le lockfile
fait partie du dépôt : il figé les versions, donc on le commit.

### 3. Lancer les tests et les voir verts

```bash
npm test
```

Les quatre tests doivent passer. Ils utilisent le lanceur intégré à Node (`node --test`), sans
dépendance en plus. Si c'est rouge, voir l'Aide avant d'aller plus loin : on ne construit rien
sur une base qui ne passe pas.

### 4. Démarrer l'API et l'appeler

Dans un premier terminal, lancez le serveur :

```bash
npm start
# API de taches a l'ecoute sur le port 3000
```

Dans un second terminal, appelez l'API avec curl :

```bash
# Page d'accueil : nom du service
curl http://localhost:3000/

# Liste des taches (vide au demarrage)
curl http://localhost:3000/tasks

# Creer une tache (corps JSON)
curl -X POST http://localhost:3000/tasks \
  -H "content-type: application/json" \
  -d '{"titre": "preparer la chaine CI"}'

# Relire la liste : la tache creee doit apparaitre
curl http://localhost:3000/tasks
```

Le POST doit renvoyer un objet avec un `id`, votre `titre` et `faite: false`. Arrêtez le serveur
avec Ctrl+C quand vous avez fini.

### 5. Une modification, une branche, un commit

On simule une vraie contribution. Créez une branche, faites un petit changement visible, commitez.
Exemple : ajoutez une ligne à la fin du `README.md` (votre prénom et la date du jour).

```bash
git switch -c mise-en-place

# ... editez README.md, ajoutez une ligne ...

git add README.md
git commit -m "docs: ajoute une trace de mise en place"
git push -u origin mise-en-place
```

### 6. Ouvrir la pull request

Ouvrez la PR de `mise-en-place` vers `main`. En ligne de commande avec GitHub CLI :

```bash
gh pr create --base main --head mise-en-place \
  --title "Mise en place du fil rouge" \
  --body "Installation OK, tests verts, API testee en local."
```

Sans la CLI, GitHub propose un bouton "Compare & pull request" après le push. Donnez un titre
clair et une description en une ou deux phrases. Ne la fusionnez pas tout de suite : on s'en sert
de support pour la suite.

## Livrable

Vous avez réussi si :

- Votre dépôt GitHub contient le fil rouge avec `package-lock.json` versionne.
- `npm test` affiche les quatre tests au vert sur votre machine.
- `npm start` démarre l'API et `curl http://localhost:3000/` renvoie le JSON du service.
- Un `POST /tasks` crée une tâche, et le `GET /tasks` suivant la liste bien.
- Une branche `mise-en-place` existe, avec au moins un commit distinct de `main`.
- Une pull request est ouverte sur GitHub, avec un titre et une description.

## Aide

### Rappels Git/GitHub

- Voir l'état courant : `git status`. Voir la branche : `git branch --show-current`.
- Changer de branche : `git switch nom`. En créer une : `git switch -c nom`.
- Lier un dépôt distant : `git remote add origin URL`, puis `git push -u origin nom-de-branche`.
- La première PR peut aussi se faire entièrement dans l'interface web : pousser la branche, puis
  cliquer sur "Compare & pull request".

### Vérifier votre environnement

```bash
node --version   # doit afficher v24.x (ou au minimum v22.x)
npm --version
```

Si `node` n'est pas en version 22 ou plus, mettez Node à jour avant de continuer. Le projet
déclare `"engines": { "node": ">=22" }` dans `package.json`.

### Si les tests sont rouges

- Refaites `npm install` : souvent le `node_modules/` est incomplet ou absent.
- Lisez le message d'erreur en entier. Une erreur d'import (`Cannot find module`) pointe vers une
  installation ratée, pas vers un bug du code.
- Vérifiez que vous êtes bien à la racine du projet (là où se trouve `package.json`).

### Si l'API ne répond pas

- `curl: connection refused` : le serveur n'est pas démarre, ou pas dans l'autre terminal. Relancez
  `npm start` et gardez ce terminal ouvert.
- `EADDRINUSE` (port déjà utilise) : un autre process écoute sur 3000. Changez de port avec une
  variable d'environnement : `PORT=3001 npm start`, puis adaptez vos commandes curl.
- Pour le POST, n'oubliez pas l'en-tête `-H "content-type: application/json"` : sans elle, le corps
  n'est pas parse et vous obtenez une erreur 400 "titre requis".

### Pièges courants

- Ne pas commiter `node_modules/` (il est volumineux et régénérable). Un `.gitignore` est fourni
  dans le starter : vérifiez qu'il liste bien `node_modules`.
- Commiter en revanche `package-lock.json` : il garantit que tout le monde installé les mêmes
  versions.
- Si `git push` est refuse, c'est souvent que le dépôt distant n'est pas le votre ou que la clé SSH
  n'est pas configurée. En HTTPS, GitHub vous demandera un token, pas votre mot de passe.
