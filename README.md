# Projet de Sécurisation Docker - Apache2 

Ce dépôt contient la configuration pour déployer un serveur web Apache (Httpd) durci sous Alpine Linux. 

## Mesures de Sécurité Appliquées

### Dockerfile (Durcissement de l'image)
* **Image de base :** `httpd:2.4-alpine` pour réduire la surface d'attaque (pas d'outils inutiles, image très légère de ~50 Mo).
* **Changement de Port :** Passage du port 80 au port **8080** pour permettre l'exécution sans privilèges root.
* **Utilisateur Non-Root :** Création de l'utilisateur `www-secure` (UID 10001). Le conteneur ne tourne pas en root.
* **Permissions strictes :** Le dossier `/htdocs` contenant la page `index.html` est verrouillé en lecture seule (`chmod 555`).

### Script Bash `deploy.sh` (Sécurité au Runtime)
* `--cap-drop=ALL` : Suppression de toutes les Linux Capabilities. Apache n'a besoin d'aucun droit système pour servir du HTML.
* `--read-only` : Le système de fichiers du conteneur est bloqué en lecture seule. Impossible pour un attaquant d'y injecter un script ou de modifier la configuration.
* `--tmpfs` : Seuls les répertoires de logs et de run d'Apache sont montés en mémoire volatile (RAM) pour permettre l'exécution, avec les options de sécurité `noexec` (interdiction d'exécuter des binaires) et `nosuid`.
* **Limitation des ressources :** RAM bridée à 64 Mo, CPU à 0.5 cœur et PIDs limités à 10 pour empêcher les attaques DoS (Déni de service) ou Fork Bomb.
* `--security-opt=no-new-privileges:true` : Interdiction stricte d'élévation de privilèges.

## Déploiement
Pour lancer un serveur, configurez le nom et le port dans `deploy.sh` puis exécutez :
```bash
./deploy.sh
