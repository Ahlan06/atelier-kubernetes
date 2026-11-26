# Rendu — Atelier Kubernetes

> Complétez ce fichier au fil de l'atelier, puis faites un commit et un push (voir la section « Rendu » du README).

**Nom / Prénom : MIRA Ahlan**
**Groupe :**

---

## Exercice 1 — Cluster

1. Combien de nœuds affiche `kubectl get nodes` ? Quel est le rôle de chacun ?
   > 3 nœuds : `k3d-lab-server-0` est le control plane, et `k3d-lab-agent-0` et `k3d-lab-agent-1` sont les deux workers.
2. Que montre `docker ps` ? Qu'est-ce qu'un « nœud » dans k3d, concrètement ?
   > `docker ps` liste les conteneurs Docker en cours d'exécution. Dans k3d, chaque nœud Kubernetes est concrètement un conteneur Docker qui exécute k3s : le server héberge le control plane et les agents servent de workers.

## Exercice 2 — Premier déploiement

3. Quels objets Kubernetes ont été créés par `kubectl create deployment` ? (indice : `kubectl get all`)
   > `kubectl create deployment` crée un Deployment, qui crée un ReplicaSet chargé de maintenir un pod nginx en fonctionnement. La commande `kubectl expose` a ensuite créé le Service `web`.
4. Vous avez supprimé le pod `web-...` : que s'est-il passé, et pourquoi ?
   > Le pod supprimé a été remplacé par un nouveau pod avec un nom différent. Le ReplicaSet du Deployment maintient le nombre désiré de réplicas (ici 1) et recrée un pod quand celui-ci est supprimé.

## Exercice 3 — Modifier un conteneur en direct

5. Après suppression du pod, votre message « Bienvenue... » a-t-il survécu ? Pourquoi ?
   > Non. Le nouveau pod repart de l'image nginx d'origine : les modifications faites directement dans le système de fichiers du conteneur sont éphémères et disparaissent avec le pod.
6. Pourquoi le `port-forward` a-t-il cessé de fonctionner après la suppression du pod ?
   > Le port-forward était relié à un pod précis. Quand ce pod a été supprimé, le tunnel a été interrompu, même si le Service `web` a ensuite un nouveau pod comme cible.

## Exercice 4 — Déclaratif

7. Quelle différence entre `kubectl create` et `kubectl apply` ?
   > `kubectl create` crée une ressource de façon impérative et échoue généralement si elle existe déjà. `kubectl apply` applique un état déclaratif décrit dans un manifeste YAML : il crée la ressource si nécessaire et la met à jour sinon.
8. À quoi sert le `selector` d'un Service ? Que se passe-t-il s'il ne correspond à aucun pod ?
   > Le `selector` du Service choisit les pods recevant le trafic selon leurs labels. S'il ne correspond à aucun pod, le Service n'a aucun endpoint et aucune requête ne peut être acheminée vers un pod.

## Exercice 5 — Scaling & mises à jour

9. Sur quels nœuds se trouvent vos 3 réplicas ? Collez la sortie de `kubectl get pods -l app=site -o wide`.
   ```
   NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE
   site-54dbc4f754-72vdb   1/1     Running   0          11m   10.42.0.6    k3d-lab-server-0
   site-54dbc4f754-h855c   1/1     Running   0          10m   10.42.2.9    k3d-lab-agent-1
   site-54dbc4f754-w4wh7   1/1     Running   0          11m   10.42.1.6    k3d-lab-agent-0
   ```
10. Avec l'image `nginx:9.99`, quel statut avait le pod ? Quelle commande vous a donné la cause ?
    > Le nouveau pod était en `ErrImagePull`, puis pouvait passer en `ImagePullBackOff`. `kubectl describe pod -l app=site` a donné la cause exacte : Docker Hub n'a pas trouvé l'image `nginx:9.99`.
11. Pendant la panne, le site restait-il accessible ? Pourquoi ?
    > Oui. Les trois réplicas n'étaient pas remplacées simultanément : les anciennes réplicas restaient disponibles pendant que le nouveau pod cassé était isolé par le déploiement rolling update.

## Exercice 6 — Portainer & Mario

12. URL publique de votre Mario :
    > https://humble-barnacle-x5q55754vx6jhpjw6-8585.app.github.dev/
13. Retrouvez le Deployment `mario` en ligne de commande. Portainer et `kubectl` parlent-ils au même composant ? Lequel ?
    > Oui. Le Deployment `mario` est visible avec `kubectl get deploy mario -o yaml` et Portainer/Kubernetes utilisent tous les deux l'API Server du même cluster local.

## Bonus (facultatif)

14. Bonus réalisés et remarques :
    > Bonus A réalisé : l'Ingress `site` expose le site sur le port 8081 via Traefik.
    > Bonus B réalisé : une `readinessProbe` HTTP sur `/` et des ressources CPU/mémoire ont été ajoutées au conteneur `nginx`.
    > Bonus C et D non réalisés.

---

**Score obtenu avec `./scripts/check.sh` :** 18 / 18
