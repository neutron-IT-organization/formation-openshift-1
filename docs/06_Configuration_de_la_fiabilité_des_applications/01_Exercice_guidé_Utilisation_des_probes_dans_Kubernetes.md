# Exercice Guidé : Readiness et Liveness Probes dans OpenShift

Cet exercice vous guidera à travers l'ajout de sondes de santé (*probes*) sur une application et la simulation de défaillances pour observer leur comportement.

## Objectifs de l'Exercice

1. Ajouter une **liveness probe** pour redémarrer automatiquement un conteneur défaillant.
2. Ajouter une **readiness probe** pour empêcher le trafic d'atteindre un pod non prêt.
3. Simuler des défaillances et observer les comportements.

## Prérequis

Une application **probes-app** est déjà déployée dans votre namespace. Elle expose trois endpoints :
- `/healthz` : endpoint de liveness (retourne 200 si le pod est "vivant")
- `/readyz` : endpoint de readiness (retourne 200 si le pod est "prêt")
- `/toggle-live` : bascule l'état de liveness (simulation de panne)
- `/toggle-ready` : bascule l'état de readiness (simulation de non-disponibilité)

Vérifiez que l'application est en cours d'exécution :

```bash
oc get deployment probes-app
```

```
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
probes-app   1/1     1            1           5m
```

---

## Étape 1 : Ajouter les Probes via la Console OpenShift

1. Accédez à la console OpenShift > **Workloads** > **Deployments** > `probes-app`.

2. Dans le menu **Actions** (en haut à droite), sélectionnez **"Add Health Checks"**.

3. **Ajoutez une Liveness Probe** :
   - Type : **HTTP GET**
   - Path : `/healthz`
   - Port : `8080`
   - Initial Delay : `10` secondes
   - Period : `5` secondes
   - Failure Threshold : `3`

![liveness probe](./images/liveness-probe.png)

4. **Ajoutez une Readiness Probe** :
   - Type : **HTTP GET**
   - Path : `/readyz`
   - Port : `8080`
   - Initial Delay : `5` secondes
   - Period : `5` secondes
   - Failure Threshold : `3`

![readiness probe](./images/readiness-probe.png)

5. Cliquez sur **"Save"** pour appliquer les probes.

Attendez que le pod redémarre avec les nouvelles probes :

```bash
oc get pods -l app=probes-app -w
```

---

## Étape 2 : Tester la Readiness Probe

### 2.1. Désactiver la Readiness

Ouvrez un shell dans le pod probes-app :

```bash
oc rsh deployment/probes-app
```

Depuis le shell dans le pod, basculez l'état de readiness :

```bash
curl http://localhost:8080/toggle-ready
```

Attendez quelques secondes (le temps que la probe détecte l'échec), puis vérifiez l'état :

```bash
curl http://localhost:8080/readyz
```

Vous devriez recevoir une réponse indiquant que le pod n'est pas prêt.

### 2.2. Observer dans la Console

- Retournez dans la console OpenShift et ouvrez les **Events** du pod `probes-app`.
- Vous devriez voir un événement indiquant que le pod est marqué comme "Non Prêt" (`Readiness probe failed`).

![readiness probe events](./images/not-ready-event.png)

Le pod n'est plus inclus dans les endpoints du service : **aucun trafic ne lui est envoyé**.

### 2.3. Réactiver la Readiness

```bash
curl http://localhost:8080/toggle-ready
```

Vérifiez que le pod est de nouveau prêt :

```bash
curl http://localhost:8080/readyz
```

Réponse attendue : `200 OK` — le pod reçoit à nouveau du trafic.

Sortez du shell :
```bash
exit
```

---

## Étape 3 : Tester la Liveness Probe

### 3.1. Simuler une Panne

Ouvrez à nouveau un shell dans le pod :

```bash
oc rsh deployment/probes-app
```

Basculez l'état de liveness :

```bash
curl http://localhost:8080/toggle-live
```

### 3.2. Observer le Redémarrage Automatique

Attendez 15-20 secondes (3 échecs × 5 secondes), puis observez dans la console ou en ligne de commande :

```bash
oc get pods -l app=probes-app
```

```
NAME                          READY   STATUS    RESTARTS   AGE
probes-app-xxx-yyy            1/1     Running   1          10m
```

Le compteur **RESTARTS** est passé à 1 — OpenShift a redémarré automatiquement le conteneur.

:::info Liveness vs Readiness
- **Readiness probe** : si elle échoue, le pod est retiré des endpoints du service (plus de trafic) mais n'est **pas redémarré**.
- **Liveness probe** : si elle échoue (au-delà du `failureThreshold`), OpenShift **redémarre automatiquement** le conteneur.
:::

---

## Étape 4 : Vérifier la Configuration des Probes

Affichez la configuration complète des probes dans le déploiement :

```bash
oc get deployment probes-app -o jsonpath='{.spec.template.spec.containers[0].livenessProbe}' | python3 -m json.tool
oc get deployment probes-app -o jsonpath='{.spec.template.spec.containers[0].readinessProbe}' | python3 -m json.tool
```

---

## Étape 5 : Nettoyage

Supprimez le déploiement :

```bash
oc delete deployment probes-app
```

---

## Résumé

Dans cet exercice, vous avez pu observer :
- **Readiness Probe** : empêche le trafic d'atteindre le pod quand il n'est pas prêt, sans le redémarrer
- **Liveness Probe** : redémarre automatiquement le conteneur quand il détecte une panne, garantissant la résilience

Ces sondes sont essentielles pour maintenir des applications robustes et disponibles sur OpenShift.
