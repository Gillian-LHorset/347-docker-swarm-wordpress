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
