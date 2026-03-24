# Exercice Guidé : Examen des Ressources Kubernetes

Dans cet exercice, nous allons explorer comment examiner et manipuler les ressources Kubernetes en utilisant `oc`. Nous nous concentrons sur les sorties personnalisées, l'extraction de manifestes YAML et leur modification.

## Objectifs de l'exercice

1. Afficher des ressources avec des vues personnalisées.
2. Extraire un manifeste YAML d'un déploiement existant.
3. Modifier le manifeste pour créer une nouvelle ressource.
4. Appliquer les modifications au cluster.

---

### Étape 1 : Affichage des Pods avec une Vue Personnalisée

Le namespace `l03p02` contient une application de démonstration déployée en avance par le formateur. Vous y avez accès en lecture.

Affichez les pods dans ce namespace :

```bash
oc get pods -n l03p02
```

**Exemple de sortie :**
```
NAME                          READY   STATUS    RESTARTS   AGE
l03p02-app-75bb5d5698-c7dzj   1/1     Running   0          10m
```

Maintenant, affinez l'affichage avec des colonnes personnalisées pour ne montrer que le nom et le statut :

```bash
oc get pods --custom-columns=NAME:.metadata.name,STATUS:.status.phase -n l03p02
```

**Résultat attendu :**
```
NAME                          STATUS
l03p02-app-75bb5d5698-c7dzj   Running
```

:::tip Colonnes personnalisées
La syntaxe `--custom-columns=NOM:.chemin.jsonpath` permet d'afficher uniquement les champs qui vous intéressent. Très utile pour créer des vues concises de vos ressources.
:::

---

### Étape 2 : Extraction d'un Manifeste YAML

Extrayez le manifeste du déploiement `l03p02-app` du namespace partagé au format YAML :

```bash
oc get deployment l03p02-app -n l03p02 -o yaml > deployment.yaml
```

Observez le contenu du fichier généré :

```bash
cat deployment.yaml
```

**Extrait du YAML généré :**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: l03p02-app
    app.kubernetes.io/instance: l03p02
  name: l03p02-app
  namespace: l03p02
spec:
  replicas: 1
  selector:
    matchLabels:
      app: l03p02-quarkus-aap
  template:
    metadata:
      labels:
        app: l03p02-quarkus-aap
    spec:
      containers:
      - args:
        - -c
        - echo hello from neutron IT; while true; do sleep 10; done
        command:
        - /bin/bash
        image: registry.access.redhat.com/ubi8/ubi:latest
        name: quarkus-container
status:
  availableReplicas: 1
  ...
```

---

### Étape 3 : Modification du Manifeste YAML

Ouvrez le fichier `deployment.yaml` dans l'éditeur :

```bash
vi deployment.yaml
```

Apportez les modifications suivantes :

1. **Supprimez tout le bloc `status`** (tout ce qui est sous `status:`)
2. **Gardez uniquement `name` et `namespace`** dans `metadata` (supprimez `labels`, `annotations`, `creationTimestamp`, etc.)
3. **Changez le nom** : `l03p02-app` → `<VOTRECITY>-l03p02-app`
4. **Changez le namespace** : `l03p02` → `<VOTRECITY>-user-ns`

**Manifeste YAML modifié (exemple pour prague) :**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prague-l03p02-app
  namespace: prague-user-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: l03p02-quarkus-aap
  template:
    metadata:
      labels:
        app: l03p02-quarkus-aap
    spec:
      containers:
      - args:
        - -c
        - echo hello from neutron IT; while true; do sleep 10; done
        command:
        - /bin/bash
        image: registry.access.redhat.com/ubi8/ubi:latest
        name: quarkus-container
```

:::info Pourquoi supprimer le status et les metadata ?
Le champ `status` est géré par Kubernetes et ne doit pas être fourni lors de la création de ressources. Les métadonnées comme `resourceVersion`, `uid`, `creationTimestamp` sont aussi spécifiques à l'instance existante et doivent être retirées.
:::

---

### Étape 4 : Application du Manifeste Modifié

Appliquez le fichier modifié dans votre namespace :

```bash
oc apply -f deployment.yaml
```

Vérifiez que le déploiement a été créé :

```bash
oc get deployment -n <VOTRECITY>-user-ns
```

```
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
prague-l03p02-app     1/1     1            1           30s
```

---

### Étape 5 : Nettoyage des Ressources

Supprimez le déploiement créé lors de cet exercice :

```bash
oc delete -f deployment.yaml
```

---

### Conclusion

Dans cet exercice, vous avez appris à :
- Utiliser des colonnes personnalisées avec `oc get`
- Extraire un manifeste YAML avec `-o yaml`
- Modifier un manifeste pour l'adapter à votre namespace
- Appliquer et supprimer des ressources avec `oc apply` et `oc delete`

Ces compétences sont essentielles pour gérer et réutiliser efficacement des ressources Kubernetes.
