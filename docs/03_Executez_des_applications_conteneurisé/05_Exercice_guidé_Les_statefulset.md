# Exercice Guidé : Les StatefulSets (Version Visuelle avec Nginx)

## Ce que vous allez apprendre

Dans cet exercice, vous allez visualiser concrètement les deux piliers d'un **StatefulSet** :
1.  **L'Identité Stable** : Chaque pod a un nom fixe (`web-0`, `web-1`).
2.  **Le Stockage Dédié** : Contrairement à un Deployment classique, chaque pod du StatefulSet possède **son propre disque dur persistant**, totalement indépendant des autres.

Nous allons utiliser un serveur web **Nginx** pour afficher le contenu de ces disques directement dans votre navigateur.

---

## Objectifs

A la fin de cet exercice, vous serez capable de :

- [ ] Déployer un **StatefulSet** avec un template de volume
- [ ] Vérifier que chaque pod a son **propre volume (PVC)** distinct
- [ ] Personnaliser le contenu de chaque volume via le navigateur
- [ ] Prouver que les données survivent au redémarrage d'un pod spécifique

---

## Étape 1 : Créer le manifeste YAML

Nous allons créer un StatefulSet avec **2 réplicas**, un **Service Headless** (obligatoire pour l'identité réseau) et une **Route** pour accéder à l'interface.

Créez un fichier nommé `stateful-visual.yaml` :

```bash
vi stateful-visual.yaml
```

Contenu du fichier :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
  namespace: <CITY>-user-ns
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None # <-- Ceci définit un service "Headless"
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
  namespace: <CITY>-user-ns
spec:
  serviceName: "nginx-headless"
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: registry.access.redhat.com/ubi9/nginx-122:latest
        ports:
        - containerPort: 8080
          name: web
        resources:
          requests:
            cpu: "10m"
            memory: "64Mi"
          limits:
            cpu: "100m"
            memory: "128Mi"
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-public
  namespace: <CITY>-user-ns
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: nginx-route
  namespace: <CITY>-user-ns
spec:
  to:
    kind: Service
    name: nginx-public
  tls:
    termination: edge
```

---

## Étape 2 : Déploiement et observation

Appliquez le fichier :

```bash
oc apply -f stateful-visual.yaml
```

### 2.1 Observer l'ordre de démarrage
Le StatefulSet démarre les pods **un par un**. Vérifiez cela :
```bash
oc get pods -l app=nginx
```
*Le pod `web-1` ne démarrera que lorsque `web-0` sera prêt.*

### 2.2 Vérifier les volumes (PVC)
C'est le point clé : vous allez voir **deux volumes distincts**, un pour chaque pod.
```bash
oc get pvc
```
| Nom du PVC | Lié au Pod |
|---|---|
| `www-web-0` | `web-0` |
| `www-web-1` | `web-1` |

---

## Étape 3 : Personnaliser les données (La partie visuelle)

Actuellement, les volumes sont vides. Nous allons écrire un message différent dans chaque pod pour prouver qu'ils ne partagent pas le même disque.

### Écrire dans le Pod 0 :
```bash
oc exec web-0 -- bash -c 'echo "<html><body style=\"background-color:#ADD8E6\"><h1>Je suis le POD 0</h1><p>Mon stockage est unique et persistant.</p></body></html>" > /usr/share/nginx/html/index.html'
```

### Écrire dans le Pod 1 :
```bash
oc exec web-1 -- bash -c 'echo "<html><body style=\"background-color:#90EE90\"><h1>Je suis le POD 1</h1><p>J ai mon propre disque, different du Pod 0 !</p> bodies></html>" > /usr/share/nginx/html/index.html'
```

---

## Étape 4 : Tester dans le navigateur

Récupérez l'URL de votre route :
```bash
oc get route nginx-route
```

1.  Ouvrez l'URL dans votre navigateur (en `https://`).
2.  Actualisez la page plusieurs fois.
3.  **Observation** : Vous verrez alterner la page **Bleue (Pod 0)** et la page **Verte (Pod 1)**.

:::info Pourquoi ?
Le Service répartit votre trafic entre les deux pods. Comme chaque pod a son propre volume, vous voyez bien deux contenus différents. C'est la preuve que **StatefulSet != Deployment** (où les données seraient souvent partagées ou identiques).
:::

---

## Étape 5 : Prouver la persistance

Nous allons "tuer" le Pod 0 et vérifier qu'il retrouve bien son propre disque bleu au redémarrage.

1.  Supprimez le pod 0 :
    ```bash
    oc delete pod web-0
    ```
2.  Attendez qu'il revienne en `Running`.
3.  Actualisez votre navigateur jusqu'à retomber sur le **Pod 0**.
4.  **Résultat** : La page est toujours **bleue** avec le texte "Je suis le POD 0". 

**Conclusion** : Le pod a retrouvé son identité (`web-0`) et a été automatiquement reconnecté à son volume dédié (`www-web-0`).

---

## Étape 6 : Nettoyage

```bash
oc delete -f stateful-visual.yaml
# N'oubliez pas les PVC qui restent par sécurité !
oc delete pvc www-web-0 www-web-1
```
