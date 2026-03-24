# Exercice Guidé : Les Services et les Routes dans OpenShift

Dans cet exercice, vous allez créer différents types de services pour exposer votre application et configurer des routes pour l'accès externe.

## Objectifs de l'Exercice

- Créer un service `ClusterIP` pour l'accès interne.
- Configurer une route HTTP et HTTPS pour l'accès externe.
- Tester les différentes configurations.

## Prérequis

Une application **Olympic Medals** est déjà déployée dans votre namespace par le formateur. Vérifiez qu'elle est bien en cours d'exécution :

```bash
oc get deployment olympic-medals-app
```

```
NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
olympic-medals-app   2/2     2            2           5m
```

```bash
oc get pods -l app=olympic-medals-app
```

```
NAME                                  READY   STATUS    RESTARTS   AGE
olympic-medals-app-xxx-yyy1           1/1     Running   0          5m
olympic-medals-app-xxx-yyy2           1/1     Running   0          5m
```

Si l'application n'est pas encore disponible, attendez quelques secondes et réessayez.

---

## Étape 1 : Créer un Service ClusterIP

Un service `ClusterIP` expose l'application **uniquement à l'intérieur du cluster**.

Créez un fichier nommé `clusterip-service.yaml` :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: olympic-medals-svc
spec:
  selector:
    app: olympic-medals-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
  type: ClusterIP
```

Appliquez le service :

```bash
oc apply -f clusterip-service.yaml
```

Vérifiez que le service est créé et qu'il pointe bien vers les pods :

```bash
oc get svc olympic-medals-svc
oc describe svc olympic-medals-svc
```

Observez les **Endpoints** — ils correspondent aux IPs des pods de l'application.

---

## Étape 2 : Créer une Route HTTP

Une `Route` expose l'application à l'extérieur du cluster via le nom de domaine géré par OpenShift.

Créez un fichier nommé `http-route.yaml` :

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: olympic-medals-http
spec:
  to:
    kind: Service
    name: olympic-medals-svc
  port:
    targetPort: 80
```

Appliquez la route :

```bash
oc apply -f http-route.yaml
```

Récupérez l'URL de la route :

```bash
oc get route olympic-medals-http -o jsonpath='http://{.spec.host}'
```

Ouvrez l'URL dans votre navigateur pour accéder à l'application Olympic Medals.

![route access](./images/route-access.png)

---

## Étape 3 : Créer une Route HTTPS (TLS Edge)

Une route TLS en mode `edge` chiffre le trafic entre le client et l'OpenShift router.

Créez un fichier nommé `https-route.yaml` :

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: olympic-medals-https
spec:
  to:
    kind: Service
    name: olympic-medals-svc
  port:
    targetPort: 80
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

Appliquez la route :

```bash
oc apply -f https-route.yaml
```

Récupérez l'URL HTTPS :

```bash
oc get route olympic-medals-https -o jsonpath='https://{.spec.host}'
```

Testez l'accès HTTPS dans votre navigateur.

![route https access](./images/route-https-access.png)

:::info TLS Edge Termination
En mode `edge`, le chiffrement TLS est terminé au niveau du router OpenShift. Le trafic entre le router et les pods peut être en HTTP non chiffré. C'est le mode le plus courant pour les applications web simples.
:::

---

## Étape 4 : Inspecter la Configuration Réseau

Listez toutes les ressources réseau de votre namespace :

```bash
oc get svc,route
```

```
NAME                      TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
service/olympic-medals-svc  ClusterIP   172.30.x.x     <none>        80/TCP    5m

NAME                                    HOST/PORT                                             ...
route.route.openshift.io/olympic-medals-http    olympic-medals-http-prague-user-ns.apps...
route.route.openshift.io/olympic-medals-https   olympic-medals-https-prague-user-ns.apps...
```

---

## Étape 5 : Nettoyage

Supprimez les services et routes créés pendant cet exercice (le déploiement de l'application sera conservé pour les exercices suivants) :

```bash
oc delete svc olympic-medals-svc
oc delete route olympic-medals-http olympic-medals-https
```

---

## Conclusion

Vous avez appris à exposer une application avec différentes méthodes :
- **Service ClusterIP** : accès interne uniquement (pod-to-pod, service-to-service)
- **Route HTTP** : accès externe en HTTP
- **Route HTTPS (TLS edge)** : accès externe chiffré

Ces concepts sont fondamentaux pour comprendre comment les applications communiquent dans OpenShift.
