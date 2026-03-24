# Exercice Guidé : Les Déploiements et Rolling Updates dans OpenShift

Dans cet exercice, vous allez créer un déploiement dans OpenShift et tester une stratégie de mise à jour en Rolling Updates, puis effectuer un rollback.

## Objectifs de l'Exercice

- Créer un déploiement avec plusieurs réplicas.
- Observer le comportement du Rolling Update lors d'une mise à jour d'image.
- Effectuer un rollback en cas de problème.
- Vérifier l'historique des déploiements.

---

### Étape 1 : Créer un Déploiement

Créez un fichier nommé `my-deployment.yaml` avec le contenu suivant :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
  labels:
    app: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: registry.access.redhat.com/ubi9/httpd-24:1-3
        ports:
        - containerPort: 8080
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "500m"
```

Appliquez ce déploiement :

```bash
oc apply -f my-deployment.yaml
```

---

### Étape 2 : Vérifier le Déploiement

Vérifiez que le déploiement et les pods sont en cours d'exécution :

```bash
oc get deployments
oc get pods
```

Sortie attendue :
```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE
my-deployment   3/3     3            3           30s
```

```
NAME                             READY   STATUS    RESTARTS   AGE
my-deployment-xxx-yyy1           1/1     Running   0          30s
my-deployment-xxx-yyy2           1/1     Running   0          30s
my-deployment-xxx-yyy3           1/1     Running   0          30s
```

---

### Étape 3 : Mettre à Jour l'Application et Observer le Rolling Update

Avant la mise à jour, ouvrez la console OpenShift dans **"Workloads" > "Deployments"** et cliquez sur `my-deployment` pour observer le Rolling Update en temps réel.

Déclenchez la mise à jour de l'image :

```bash
oc set image deployment/my-deployment my-container=registry.access.redhat.com/ubi9/httpd-24:1-325
```

Observez le rollout dans le terminal :

```bash
oc rollout status deployment/my-deployment
```

```
Waiting for deployment "my-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "my-deployment" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "my-deployment" rollout to finish: 2 of 3 updated replicas are available...
deployment "my-deployment" successfully rolled out
```

:::info Rolling Update
Le Rolling Update remplace les pods un par un (selon `maxUnavailable` et `maxSurge`), ce qui garantit une **disponibilité continue** de l'application pendant la mise à jour.
:::

---

### Étape 4 : Consulter l'Historique des Déploiements

Consultez l'historique des révisions pour voir les versions précédentes :

```bash
oc rollout history deployment/my-deployment
```

```
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
```

---

### Étape 5 : Effectuer un Rollback

Si la mise à jour pose un problème, vous pouvez revenir à la version précédente :

```bash
oc rollout undo deployment/my-deployment
```

Cette commande déclenche un rollback vers la révision précédente.

---

### Étape 6 : Vérifier le Rollback

1. **Vérifiez le statut du déploiement** :

   ```bash
   oc rollout status deployment/my-deployment
   ```

2. **Vérifiez que tous les pods sont en cours d'exécution** :

   ```bash
   oc get pods
   ```

3. **Vérifiez l'image actuellement utilisée** :

   ```bash
   oc get deployment my-deployment -o jsonpath='{.spec.template.spec.containers[0].image}'
   ```

   L'image doit être revenue à `registry.access.redhat.com/ubi9/httpd-24:1-3`.

---

### Étape 7 : Nettoyage

Supprimez le déploiement créé lors de cet exercice :

```bash
oc delete -f my-deployment.yaml
```

---

### Conclusion

Vous avez appris à :
- Créer un déploiement avec une stratégie Rolling Update
- Mettre à jour l'image d'un déploiement et observer le rollout
- Consulter l'historique des révisions
- Effectuer un rollback vers une version précédente
