# Exercice Guidé : Les Réseaux de Pods et de Services

## Ce que vous allez apprendre

Dans cet exercice, vous allez explorer les trois couches de connectivité dans OpenShift :
1.  **IP de Pod** : Communication directe entre conteneurs (instable).
2.  **ClusterIP** : IP de service stable à l'intérieur du cluster.
3.  **NodePort** : Accès par le noeud (IP du serveur).
4.  **Route HTTPS** : Exposition sécurisée sur Internet.

Vous apprendrez également à lever l'isolation réseau entre les namespaces via une **NetworkPolicy**.

---

## Objectifs

- [ ] Déployer une application web
- [ ] Configurer une **NetworkPolicy** pour autoriser le trafic entre namespaces
- [ ] Tester la connectivité via **Pod IP**, **ClusterIP**, **NodePort** et **Route**

---

## Étape 1 : Déployer l'application

Créez le fichier `welcome-deployment.yaml` :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: welcome-app
  namespace: <CITY>-user-ns
spec:
  replicas: 1
  selector:
    matchLabels:
      app: welcome-app
  template:
    metadata:
      labels:
        app: welcome-app
    spec:
      containers:
      - name: welcome-app
        image: quay.io/neutron-it/welcome-app:latest
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: "1m"
            memory: "64Mi"
          limits:
            cpu: "100m"
            memory: "128Mi"
```

```bash
oc apply -f welcome-deployment.yaml
```

---

## Étape 2 : Configurer la NetworkPolicy

Par défaut, dans l'environnement de formation, les namespaces sont isolés. Pour que vous puissiez effectuer des tests de communication (via `curl` depuis votre terminal web ou entre vos namespaces), il faut autoriser le trafic entrant vers votre application.

Créez le fichier `welcome-policy.yaml` :

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-multi-namespace-ingress
  namespace: <CITY>-user-ns
spec:
  podSelector:
    matchLabels:
      app: welcome-app
  ingress:
  - from:
    - namespaceSelector: {} # Autorise le trafic provenant de TOUS les namespaces (utile pour le terminal web et les tests croisés)
  policyTypes:
  - Ingress
```

```bash
oc apply -f welcome-policy.yaml
```

---

## Étape 3 : Communication via l'IP du Pod (Interne)

### 3.1 Récupérer l'IP du Pod
```bash
oc get pods -o wide
```
*Notez l'IP dans la colonne `IP`.*

### 3.2 Tester la connexion
Lancez un `curl` directement dans votre terminal web :
```bash
# Remplacez <POD_IP> par l'IP notée précédemment
curl -s <POD_IP>:8080 | grep "Bienvenue"
```

---

## Étape 4 : Créer et tester le Service ClusterIP (Stable)

### 4.1 Créer le service
Créez le fichier `welcome-clusterip.yaml` :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: welcome-svc
  namespace: <CITY>-user-ns
spec:
  selector:
    app: welcome-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

```bash
oc apply -f welcome-clusterip.yaml
```

### 4.2 Tester via l'IP du Service
```bash
oc get svc welcome-svc
```
*Notez la `CLUSTER-IP`.*

```bash
# Remplacez <CLUSTER_IP> par l'IP du service
curl -s <CLUSTER_IP>:80 | grep "Bienvenue"
```

---

## Étape 5 : Créer et tester le Service NodePort

Le NodePort expose l'application sur un port fixe de votre serveur (le noeud).

### 5.1 Créer le service NodePort
Créez le fichier `welcome-nodeport.yaml` :

```yaml
apiVersion: v1
kind: Service
metadata:
  name: welcome-svc-nodeport
  namespace: <CITY>-user-ns
spec:
  selector:
    app: welcome-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080
  type: NodePort
```

```bash
oc apply -f welcome-nodeport.yaml
```

### 5.2 Tester via l'IP du Noeud
Utilisez l'IP de votre serveur OpenShift (**192.168.0.251**) :
```bash
curl -s http://192.168.0.251:30080 | grep "Bienvenue"
```

---

## Étape 6 : Créer la Route HTTPS (Exposition Publique)

### 6.1 Créer la Route Edge
Créez le fichier `welcome-route.yaml` :

```yaml
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: welcome-route
  namespace: <CITY>-user-ns
spec:
  to:
    kind: Service
    name: welcome-svc
  port:
    targetPort: 80
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Redirect
```

```bash
oc apply -f welcome-route.yaml
```

### 6.2 Tester dans le navigateur
```bash
oc get route welcome-route -o jsonpath='https://{.spec.host}{"\n"}'
```
Ouvrez l'URL générée dans votre navigateur.

---

## Récapitulatif

| Méthode | Portée | Stabilité | Usage principal |
|---|---|---|---|
| **Pod IP** | Interne | **Instable** | Debugging |
| **ClusterIP** | Interne | **Stable** | Inter-services |
| **NodePort** | Bureau local | **Stable** | Accès technique |
| **Route HTTPS** | **Internet (Public)** | **Stable** | Accès utilisateurs |

---

## Étape 7 : Nettoyage

```bash
oc delete deployment welcome-app
oc delete svc welcome-svc welcome-svc-nodeport
oc delete route welcome-route
oc delete networkpolicy allow-multi-namespace-ingress
```
