# Déploiement WordPress sur Docker Swarm

## 1. Préparation des trois VM

Sur chaque VM (Alpine Linux), mise à jour du système puis installation du daemon Docker :

```bash
apk update && apk upgrade
apk add docker
```

Démarrage du service Docker :

```bash
rc-service docker start
```

## 2. Initialisation du cluster Swarm

Sur la VM manager, création du cluster. L'adresse annoncée doit être l'IP du manager, seule interface par laquelle les workers le joindront :

```bash
docker swarm init --advertise-addr 10.228.242.240
```

La commande retourne un token de connexion. Sur chaque worker, exécuter la commande `docker swarm join` fournie avec ce token :

```bash
docker swarm join --token <TOKEN_WORKER> 10.228.242.240:2377
```

Depuis le manager, contrôler que les trois nœuds sont bien enregistrés avec le statut `Ready` :

```bash
docker node ls
```

## 3. Réseaux overlay

Deux réseaux de type overlay sont créés sur le manager. Swarm les réplique automatiquement sur tous les nœuds afin que les conteneurs puissent communiquer entre hôtes :

```bash
# Réseau exposé : Nginx y contacte les répliques WordPress
docker network create --driver overlay frontend

# Réseau isolé : seule la liaison WordPress ↔ MariaDB y est possible
docker network create --driver overlay backend
```

La base de données n'étant attachée qu'à `backend`, elle reste inaccessible depuis `frontend`, donc depuis le monde extérieur.

## 4. Secrets Docker

Les identifiants de MariaDB sont stockés dans les secrets Swarm plutôt qu'en variables d'environnement en clair. Chaque secret est injecté dans les conteneurs concernés sous `/run/secrets/` :

```bash
# Mot de passe root de MariaDB
echo "mdp_root" | docker secret create db_root_password -

# Mot de passe de l'utilisateur WordPress
echo "mdp_user" | docker secret create db_password -
```

Lister les secrets créés :

```bash
docker secret ls
```

## 5. Arborescence du projet et configuration Nginx

Sur le manager, création du dossier de travail et du fichier de configuration du reverse proxy :

```bash
mkdir -p /root/wordpress-swarm/nginx
touch /root/wordpress-swarm/nginx/default.conf
```

Contenu de `default.conf`. Nginx sert les fichiers statiques depuis le volume partagé WordPress et transmet les requêtes `.php` aux conteneurs PHP-FPM via le nom de service `wordpress`, résolu par le DNS interne de Swarm :

Voir fichier `default.conf`

Important : le montage `./nginx/default.conf` est un chemin local de l'hôte. Comme Nginx tourne sur les workers, ce fichier doit exister au même emplacement sur `worker1` et `worker2` :

```bash
# À exécuter sur worker1 et worker2
mkdir -p /root/wordpress-swarm/nginx
# Copier default.conf depuis le manager vers /root/wordpress-swarm/nginx/
```

Sans cette copie, la tâche Nginx reste bloquée en erreur `mounting` lors du déploiement.
