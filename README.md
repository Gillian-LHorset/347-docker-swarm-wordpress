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
