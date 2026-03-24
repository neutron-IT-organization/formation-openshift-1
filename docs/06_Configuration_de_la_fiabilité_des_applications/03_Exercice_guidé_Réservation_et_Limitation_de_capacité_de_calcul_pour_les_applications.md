# Exercice Guidé : Gestion des Requests, Limites et Quotas dans OpenShift

Cet exercice vous guidera dans la configuration des *requests* et *limites* de ressources pour un déploiement, puis dans la création et le test d'un *ResourceQuota*.

## Objectifs de l'exercice

1. Configurer les *requests* et *limites* pour un déploiement.
2. Créer un *ResourceQuota* pour restreindre la consommation globale du namespace.
3. Observer le comportement d'OpenShift lorsque le quota est dépassé.
4. Analyser les consommations via les quotas (**used** vs **hard**).

---

## Étape 1 : Créer un Déploiement avec Requests et Limites

Créez un fichier nommé `deployment-limite.yaml` :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-limite
spec:
  replicas: 1
  selector:
    matchLabels:
      app: test-limite
  template:
    metadata:
      labels:
        app: test-limite
    spec:
      containers:
      - name: limite-container
        image: registry.access.redhat.com/ubi8/ubi:latest
        command: ["sh", "-c", "while true; do echo Hello OpenShift; sleep 5; done"]
        resources:
          requests:
            memory: "128Mi"
            cpu: "300m"
          limits:
            memory: "256Mi"
            cpu: "600m"
```

Appliquez le déploiement :

```bash
oc apply -f deployment-limite.yaml
```

Vérifiez que le pod est Running :

```bash
oc get pods -l app=test-limite
```

Consultez les *requests* et *limites* appliquées au pod :

```bash
oc describe pod -l app=test-limite | grep -A 6 "Limits:"
```

---

## Étape 2 : Configurer un ResourceQuota

Créez un fichier `quota.yaml` pour définir un quota de ressources dans votre namespace :

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-formation
spec:
  hard:
    requests.cpu: "900m"       # Max 900 millicores en requests (3 pods × 300m)
    requests.memory: "512Mi"   # Max 512 Mi en requests
    limits.cpu: "2"            # Max 2 CPU en limits
    limits.memory: "1Gi"       # Max 1 Gi en limits
    pods: "5"                  # Max 5 pods
```

:::info Calcul du quota
Avec 1 pod existant (300m CPU request) et un quota de 900m :
- 1 pod : 300m utilisés (33% du quota)
- 2 pods : 600m utilisés (67% du quota)
- 3 pods : 900m utilisés (100% du quota — limite atteinte)
- 4 pods : 1200m nécessaires → **REFUSÉ** (dépasse 900m)
:::

Appliquez le quota :

```bash
oc apply -f quota.yaml
```

Vérifiez que le quota est bien configuré :

```bash
oc describe resourcequota quota-formation
```

```
Name:              quota-formation
Namespace:         prague-user-ns
Resource           Used    Hard
--------           ----    ----
limits.cpu         600m    2
limits.memory      256Mi   1Gi
pods               1       5
requests.cpu       300m    900m
requests.memory    128Mi   512Mi
```

---

## Étape 3 : Tester le Dépassement du Quota

Essayez de scaler le déploiement au-delà du quota :

```bash
oc scale deployment/test-limite --replicas=4
```

Vérifiez les événements pour comprendre pourquoi OpenShift refuse de créer les pods supplémentaires :

```bash
oc get events --sort-by='.lastTimestamp' | tail -5
```

Vous devriez voir un événement similaire à :

```
Warning  FailedCreate  replicaset-controller  Failed to create pod: exceeded quota: quota-formation,
requested: requests.cpu=300m, used: requests.cpu=900m, limited: requests.cpu=900m
```

**Décryptage du message :**
- `FailedCreate` : OpenShift n'a pas pu créer de nouveaux pods
- `exceeded quota: quota-formation` : le quota `quota-formation` a été dépassé
- `requested: requests.cpu=300m` : le 4ème pod demande 300m CPU supplémentaires
- `used: requests.cpu=900m` : les 3 pods existants utilisent déjà les 900m autorisés
- `limited: requests.cpu=900m` : la limite du quota est 900m

Vérifiez l'état réel des pods :

```bash
oc get pods -l app=test-limite
```

```
NAME                           READY   STATUS    RESTARTS   AGE
test-limite-xxx-yyy1           1/1     Running   0          5m
test-limite-xxx-yyy2           1/1     Running   0          1m
test-limite-xxx-yyy3           1/1     Running   0          30s
```

Seulement 3 pods sont Running (la limite du quota est 900m = 3 × 300m).

---

## Étape 4 : Analyser les Consommations des Quotas

Consultez l'état détaillé du quota :

```bash
oc describe resourcequota quota-formation
```

```
Name:              quota-formation
Namespace:         prague-user-ns
Resource           Used    Hard
--------           ----    ----
limits.cpu         1800m   2
limits.memory      768Mi   1Gi
pods               3       5
requests.cpu       900m    900m
requests.memory    384Mi   512Mi
```

Pour surveiller les ressources consommées par chaque pod :

```bash
oc adm top pod -l app=test-limite
```

---

## Étape 5 : Nettoyer l'Environnement

```bash
oc delete deployment test-limite
oc delete resourcequota quota-formation
```

Vérifiez que les ressources sont supprimées :

```bash
oc get deployment,resourcequota
```

---

## Conclusion

Vous avez appris à :
1. Configurer les *requests* et *limites* pour contrôler la consommation par pod
2. Définir des *ResourceQuotas* pour limiter la consommation globale d'un namespace
3. Observer et interpréter les erreurs de dépassement de quota
4. Analyser les consommations actuelles vs les limites

**Bonne pratique production** : En production, définissez toujours des quotas sur vos namespaces et configurez des *requests*/*limits* sur tous vos conteneurs pour éviter qu'une application ne consomme toutes les ressources du cluster.
