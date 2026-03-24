# Exercice Guidé : Utilisation des ConfigMaps et Secrets dans OpenShift

Cet exercice vous guidera à travers la création, la gestion et l'injection de *ConfigMaps* et de *Secrets* dans une application déployée dans OpenShift.

## Objectifs de l'Exercice

- Créer des *ConfigMaps* pour stocker des données de configuration.
- Créer des *Secrets* pour gérer des données sensibles.
- Injecter les *ConfigMaps* et les *Secrets* dans un déploiement via des variables d'environnement.
- Mettre à jour un *ConfigMap* et observer l'impact sur l'application.

## Prérequis

Une application **Welcome App** est déjà déployée dans votre namespace. Elle affiche un message de bienvenue configurable. Vérifiez qu'elle est en cours d'exécution :

```bash
oc get deployment welcome-app
```

```
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
welcome-app   2/2     2            2           5m
```

---

## Étape 1 : Créer un ConfigMap

Un *ConfigMap* stocke des données de configuration non sensibles.

Créez un fichier nommé `welcome-config.yaml` :

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: welcome-config
data:
  welcome_message: "Bienvenue sur notre site de démonstration !"
  app_mode: "production"
```

Appliquez le *ConfigMap* :

```bash
oc apply -f welcome-config.yaml
```

Vérifiez sa création :

```bash
oc get configmap welcome-config -o yaml
```

---

## Étape 2 : Créer un Secret

Un *Secret* stocke des données sensibles (mots de passe, tokens) encodées en base64.

Créez un fichier nommé `welcome-secret.yaml` :

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: welcome-secret
type: Opaque
data:
  api_token: d2VsY29tZVRva2VuMTIz
```

:::note
`d2VsY29tZVRva2VuMTIz` est l'encodage base64 de `welcomeToken123`.
Vous pouvez encoder vos propres valeurs avec : `echo -n "votre-valeur" | base64`
:::

Appliquez le *Secret* :

```bash
oc apply -f welcome-secret.yaml
```

Vérifiez sa création (les données sont masquées) :

```bash
oc get secret welcome-secret -o yaml
```

---

## Étape 3 : Injecter le ConfigMap et le Secret dans l'Application

Modifiez le déploiement `welcome-app` pour injecter les variables d'environnement depuis le *ConfigMap* et le *Secret* :

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: welcome-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: welcome-app
  template:
    metadata:
      labels:
        app: welcome-app
    spec:
      containers:
        - name: welcome-app-container
          image: quay.io/neutron-it/welcome-app:latest
          ports:
            - containerPort: 8080
          env:
            - name: WELCOME_MESSAGE
              valueFrom:
                configMapKeyRef:
                  name: welcome-config
                  key: welcome_message
            - name: APP_MODE
              valueFrom:
                configMapKeyRef:
                  name: welcome-config
                  key: app_mode
            - name: API_TOKEN
              valueFrom:
                secretKeyRef:
                  name: welcome-secret
                  key: api_token
```

Sauvegardez ce contenu dans `welcome-app-updated.yaml` et appliquez :

```bash
oc apply -f welcome-app-updated.yaml
```

Vérifiez que les pods sont redémarrés avec la nouvelle configuration :

```bash
oc get pods -l app=welcome-app
```

Vérifiez les variables d'environnement dans un pod :

```bash
oc exec deployment/welcome-app -- env | grep -E "WELCOME|APP_MODE|API_TOKEN"
```

Sortie attendue :
```
WELCOME_MESSAGE=Bienvenue sur notre site de démonstration !
APP_MODE=production
API_TOKEN=welcomeToken123
```

---

## Étape 4 : Mettre à Jour le ConfigMap

Modifiez le fichier `welcome-config.yaml` pour changer le message :

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: welcome-config
data:
  welcome_message: "Bienvenue à notre nouvelle application déployée avec OpenShift !"
  app_mode: "development"
```

Réappliquez le *ConfigMap* :

```bash
oc apply -f welcome-config.yaml
```

:::warning Les variables d'environnement ne se mettent pas à jour automatiquement
Contrairement aux volumes, les variables d'environnement injectées depuis un ConfigMap ne sont **pas mises à jour dynamiquement**. Il faut redémarrer les pods.
:::

Redémarrez les pods pour qu'ils récupèrent la nouvelle configuration :

```bash
oc rollout restart deployment welcome-app
```

Vérifiez la mise à jour :

```bash
oc exec deployment/welcome-app -- env | grep WELCOME_MESSAGE
```

---

## Étape 5 : Nettoyage

Supprimez les ressources créées pendant cet exercice :

```bash
oc delete configmap welcome-config
oc delete secret welcome-secret
oc delete deployment welcome-app
```

---

## Conclusion

Vous avez appris à :
- Créer des *ConfigMaps* pour externaliser la configuration des applications
- Créer des *Secrets* pour gérer les données sensibles
- Injecter ces ressources comme variables d'environnement dans un déploiement
- Mettre à jour un *ConfigMap* et redémarrer l'application pour appliquer les changements

Ces pratiques permettent de **découpler la configuration du code** et de gérer les données sensibles de manière centralisée et sécurisée.
