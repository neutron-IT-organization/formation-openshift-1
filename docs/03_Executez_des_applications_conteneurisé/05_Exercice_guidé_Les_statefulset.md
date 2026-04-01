# Exercice Guidé : Les StatefulSets avec MySQL (Version Simplifiée)

## Ce que vous allez apprendre

Dans cet exercice, vous allez découvrir les **StatefulSets**, un type de ressource Kubernetes indispensable pour les bases de données. Vous allez apprendre comment un StatefulSet garantit une **identité stable** au pod et une **persistance des données** même après la suppression d'un pod.

---

## Objectifs

A la fin de cet exercice, vous serez capable de :

- [ ] Déployer un **StatefulSet** MySQL avec son stockage persistant
- [ ] Vérifier que le pod reçoit un **nom stable** (`mysql-0`)
- [ ] Prouver la **persistance des données** après la suppression et recréation automatique du pod
- [ ] Nettoyer proprement les ressources et leurs volumes

---

:::tip Terminal web OpenShift
Toutes les commandes `oc` de cet exercice sont à exécuter dans le **terminal web OpenShift**.
:::

## Étape 1 : Créer le fichier de configuration

Nous allons définir un StatefulSet MySQL avec **1 seul réplica** pour simplifier l'observation.

Créez un fichier nommé `mysql-app.yaml` :

```bash
vi mysql-app.yaml
```

Contenu du fichier :

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
  namespace: <CITY>-user-ns
spec:
  serviceName: "mysql"
  replicas: 1
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
            cpu: "50m"
            memory: "128Mi"
          limits:
            cpu: "200m"
            memory: "256Mi"
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

:::info Pourquoi 1 Gi ?
Le volume persistant (PVC) sera créé automatiquement par OpenShift grâce au **Dynamic Provisioning**. Le StatefulSet s'occupe de lier ce volume spécifiquement au pod `mysql-0`.
:::

---

## Étape 2 : Déployer et vérifier

Appliquez le fichier YAML et vérifiez la création du pod et du volume :

```bash
oc apply -f mysql-app.yaml
```

### Vérifier le pod (Nom stable)
```bash
oc get pods -l app=mysql
```
*Le nom doit être exactement `mysql-0`.*

### Vérifier le volume (PVC)
```bash
oc get pvc
```
*Vous devriez voir un PVC nommé `mysql-data-mysql-0` avec le statut `Bound`.*

---

## Étape 3 : Écrire des données

Nous allons utiliser une commande "one-liner" pour créer une table et insérer une donnée sans entrer manuellement dans le pod.

```bash
oc exec mysql-0 -- mysql -u user -ppassword mydb -e "CREATE TABLE test(msg VARCHAR(50)); INSERT INTO test VALUES('Le stockage fonctionne !'); SELECT * FROM test;"
```

**Sortie attendue :**
```
+---------------------------+
| msg                       |
+---------------------------+
| Le stockage fonctionne !   |
+---------------------------+
```

---

## Étape 4 : Tester la persistance (Le crash-test)

C'est ici que nous prouvons l'utilité du StatefulSet. Nous allons supprimer le pod. OpenShift va le recréer, et le nouveau pod doit retrouver ses données.

### 4.1 : Supprimer le pod
```bash
oc delete pod mysql-0
```

### 4.2 : Attendre le redémarrage
Attendez que le pod soit à nouveau `Running` :
```bash
oc get pods -l app=mysql -w
```
*(Appuyez sur Ctrl+C quand il est prêt)*

### 4.3 : Vérifier les données
Relancez la commande de lecture :
```bash
oc exec mysql-0 -- mysql -u user -ppassword mydb -e "SELECT * FROM test;"
```

**Si vous revoyez le message, c'est gagné !** Le nouveau pod `mysql-0` a automatiquement récupéré son volume `mysql-data-mysql-0`.

---

## Étape 5 : Nettoyage

Attention : Dans un StatefulSet, Kubernetes ne supprime **jamais** les volumes (PVC) automatiquement par sécurité. Il faut le faire à la main.

```bash
# 1. Supprimer le StatefulSet
oc delete statefulset mysql

# 2. Supprimer le volume persistant
oc delete pvc mysql-data-mysql-0
```

---

## Récapitulatif

| Concept | Ce qu'il faut retenir |
|---|---|
| **Identité stable** | Le pod s'appelle toujours `mysql-0`, ce qui facilite les sauvegardes et configurations. |
| **Persistance** | Les données survivent à la suppression du pod car elles sont stockées sur un volume externe lié à son identité. |
| **Usage** | Recommandé pour toutes les bases de données (MySQL, Postgres, MongoDB, etc.). |
