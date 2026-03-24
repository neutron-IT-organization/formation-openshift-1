# Exercice Guidé : Les StatefulSets avec MySQL

Dans cet exercice, vous allez déployer une base de données MySQL à l'aide d'un StatefulSet dans OpenShift. Vous créerez deux réplicas et vérifierez que chaque réplica possède son propre Persistent Volume Claim (PVC) avec des données indépendantes.

## Objectifs de l'Exercice

- Déployer un StatefulSet MySQL avec 2 réplicas.
- Vérifier que chaque réplica a un PVC distinct.
- Écrire des données dans chaque instance et vérifier leur indépendance.

---

### Étape 1 : Créer le StatefulSet MySQL

Créez un fichier nommé `mysql-statefulset.yaml` avec le contenu suivant :

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: "mysql"
  replicas: 2
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: registry.access.redhat.com/rhscl/mysql-80-rhel7:latest
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "rootpassword"
        - name: MYSQL_DATABASE
          value: "mydb"
        - name: MYSQL_USER
          value: "user"
        - name: MYSQL_PASSWORD
          value: "password"
        ports:
        - containerPort: 3306
        resources:
          requests:
            cpu: "100m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql/data
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
```

Appliquez le StatefulSet :

```bash
oc apply -f mysql-statefulset.yaml
```

:::info StatefulSet vs Deployment
Contrairement à un Deployment, un StatefulSet donne à chaque pod :
- Un **nom stable et prévisible** (`mysql-0`, `mysql-1`)
- Un **PVC unique** (un volume de stockage par pod)
- Un **démarrage ordonné** (mysql-0 démarre avant mysql-1)
:::

---

### Étape 2 : Vérifier les Pods et les PVCs

1. **Listez les pods** :

```bash
oc get pod -l app=mysql
```

```
NAME      READY   STATUS    RESTARTS   AGE
mysql-0   1/1     Running   0          2m
mysql-1   1/1     Running   0          1m30s
```

2. **Listez les PVCs créés automatiquement** :

```bash
oc get pvc
```

```
NAME                 STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-data-mysql-0   Bound    ...      1Gi        RWO            ...            2m
mysql-data-mysql-1   Bound    ...      1Gi        RWO            ...            1m30s
```

Chaque pod a son propre PVC nommé `mysql-data-<pod-name>`.

![statefulset section](./images/statefulset-ui.png)
![pvc section](./images/pvc-ui.png)

---

### Étape 3 : Interagir avec les Bases de Données

#### 3.1. Se connecter au premier pod MySQL

```bash
oc exec -it mysql-0 -- bash
```

Une fois connecté, accédez à MySQL :

```bash
mysql -u user -ppassword mydb
```

#### 3.2. Écrire des données dans `mysql-0`

```sql
CREATE TABLE test_table (id INT PRIMARY KEY, data VARCHAR(50));
INSERT INTO test_table (id, data) VALUES (1, 'Data from mysql-0');
SELECT * FROM test_table;
```

Sortie attendue :
```
+----+-------------------+
| id | data              |
+----+-------------------+
|  1 | Data from mysql-0 |
+----+-------------------+
```

Sortez du pod :
```bash
exit
exit
```

#### 3.3. Se connecter au second pod MySQL

```bash
oc exec -it mysql-1 -- bash
```

```bash
mysql -u user -ppassword mydb
```

#### 3.4. Écrire des données dans `mysql-1`

```sql
CREATE TABLE test_table (id INT PRIMARY KEY, data VARCHAR(50));
INSERT INTO test_table (id, data) VALUES (2, 'Data from mysql-1');
SELECT * FROM test_table;
```

Sortie attendue :
```
+----+-------------------+
| id | data              |
+----+-------------------+
|  2 | Data from mysql-1 |
+----+-------------------+
```

---

### Étape 4 : Vérifier l'Indépendance des Données

Reconnectez-vous à `mysql-0` et vérifiez que ses données sont indépendantes de `mysql-1` :

```bash
oc exec -it mysql-0 -- bash
mysql -u user -ppassword mydb
SELECT * FROM test_table;
```

Vous devriez voir uniquement `Data from mysql-0` — les données de `mysql-1` ne sont pas visibles car chaque pod a son propre volume de stockage.

```
+----+-------------------+
| id | data              |
+----+-------------------+
|  1 | Data from mysql-0 |
+----+-------------------+
```

Sortez :
```bash
exit
exit
```

---

### Étape 5 : Nettoyage

Supprimez le StatefulSet et les PVCs associés :

```bash
oc delete statefulset mysql
oc delete pvc -l app=mysql
```

---

### Conclusion

Vous avez déployé un StatefulSet MySQL avec deux réplicas et vérifié que :
- Chaque pod a un **nom stable** (`mysql-0`, `mysql-1`)
- Chaque pod a un **PVC distinct** avec ses propres données
- Les données sont **indépendantes** entre les réplicas

C'est la valeur ajoutée des StatefulSets pour les applications stateful nécessitant de la persistance des données et une identité stable.
