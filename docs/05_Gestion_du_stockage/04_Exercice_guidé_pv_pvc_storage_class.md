# Exercice Guidé : Persistent Volumes (PV), PVC et Storage Class

## Objectif

Démontrer la différence entre le stockage **éphémère** (`emptyDir`) et le stockage **persistant** (PVC) en observant ce qui se passe avec les données après un redémarrage de pod.

## Prérequis

Une application **Todo App** connectée à PostgreSQL est déjà déployée dans votre namespace. L'application utilise actuellement un stockage éphémère (`emptyDir`).

Vérifiez les déploiements disponibles :

```bash
oc get deployment
```

```
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
postgres    1/1     1            1           5m
todo-app    1/1     1            1           5m
```

Récupérez l'URL de la Todo App :

```bash
oc get route todo-route -o jsonpath='https://{.spec.host}'
```

Ouvrez l'URL dans votre navigateur et ajoutez quelques tâches à la liste.

---

## Étape 1 : Observer la Perte de Données avec le Stockage Éphémère

Vérifiez que PostgreSQL utilise bien `emptyDir` :

```bash
oc get deployment postgres -o jsonpath='{.spec.template.spec.volumes}' | python3 -m json.tool
```

Vous devriez voir `"emptyDir": {}` dans la configuration des volumes.

Ajoutez quelques tâches dans l'interface web de la Todo App, puis redémarrez le pod PostgreSQL :

```bash
oc delete pod -l app=postgres
```

Kubernetes recrée automatiquement le pod. Attendez qu'il soit de nouveau Running :

```bash
oc get pods -l app=postgres -w
```

Retournez sur l'interface de la Todo App et rafraîchissez la page. **Les tâches ont disparu** — c'est la limitation du stockage éphémère (`emptyDir`).

:::warning Stockage éphémère
Un volume `emptyDir` est créé vide à la création du pod et **détruit avec lui**. Toute donnée est perdue si le pod est supprimé ou redémarré.
:::

---

## Étape 2 : Passer au Stockage Persistant avec un PVC

Pour conserver les données même après un redémarrage de pod, nous allons modifier le déploiement PostgreSQL pour utiliser un *PersistentVolumeClaim* (PVC).

Créez d'abord le PVC. Créez un fichier `postgres-pvc.yaml` :

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Appliquez le PVC :

```bash
oc apply -f postgres-pvc.yaml
```

Vérifiez que le PVC est créé et lié à un volume :

```bash
oc get pvc postgres-pvc
```

```
NAME           STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
postgres-pvc   Bound    ...      1Gi        RWO            ...            10s
```

---

## Étape 3 : Modifier le Déploiement PostgreSQL

Créez un fichier `postgres-pvc-deployment.yaml` pour utiliser le PVC :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
      containers:
        - name: postgres
          image: registry.access.redhat.com/rhscl/postgresql-12-rhel7:latest
          ports:
            - containerPort: 5432
          env:
            - name: POSTGRESQL_USER
              valueFrom:
                secretKeyRef:
                  name: postgres-credentials
                  key: POSTGRES_USER
            - name: POSTGRESQL_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-credentials
                  key: POSTGRES_PASSWORD
            - name: POSTGRESQL_DATABASE
              valueFrom:
                secretKeyRef:
                  name: postgres-credentials
                  key: POSTGRES_DB
          volumeMounts:
            - name: postgres-storage
              mountPath: /var/lib/pgsql/data
```

Appliquez la mise à jour :

```bash
oc apply -f postgres-pvc-deployment.yaml
```

Attendez que le pod redémarre avec le nouveau volume :

```bash
oc rollout status deployment/postgres
```

---

## Étape 4 : Tester la Persistance des Données

1. Ajoutez de nouvelles tâches dans l'interface de la Todo App.

2. Redémarrez le pod PostgreSQL :

```bash
oc delete pod -l app=postgres
```

3. Attendez que le pod soit de nouveau Running :

```bash
oc get pods -l app=postgres -w
```

4. Rafraîchissez la page de la Todo App. **Les tâches sont toujours présentes !**

:::tip Stockage persistant
Le PVC survit à la suppression du pod. Lorsque Kubernetes recrée le pod, il remonte le même volume avec les données intactes.
:::

---

## Étape 5 : Inspecter le PVC et la Storage Class

Affichez les détails du PVC :

```bash
oc describe pvc postgres-pvc
```

```
Name:          postgres-pvc
Namespace:     prague-user-ns
Status:        Bound
Volume:        pvc-xxx-yyy
Capacity:      1Gi
Access Modes:  RWO
StorageClass:  thin-csi
...
```

Affichez les Storage Classes disponibles dans le cluster :

```bash
oc get storageclass
```

La Storage Class `(default)` est utilisée automatiquement si vous ne spécifiez pas de `storageClassName` dans votre PVC.

---

## Étape 6 : Nettoyage

Supprimez les ressources créées pendant cet exercice :

```bash
oc delete -f postgres-pvc-deployment.yaml
oc delete pvc postgres-pvc
```

---

## Conclusion

Vous avez démontré concrètement la différence entre :
- **`emptyDir`** : stockage temporaire, perdu à la suppression du pod
- **PVC** : stockage persistant, survit à la suppression du pod et permet aux applications stateful de conserver leurs données

En production, les bases de données et toute application stateful doivent toujours utiliser des PVCs pour garantir la durabilité des données.
