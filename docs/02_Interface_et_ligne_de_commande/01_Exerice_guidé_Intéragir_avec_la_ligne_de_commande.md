# Exercice Guidé : Interaction avec OpenShift via la Ligne de Commande

### Objectif

Cet exercice vous guidera à travers les étapes de base pour se connecter à un cluster OpenShift, explorer les commandes disponibles, et gérer une application simple.

Toutes les commandes doivent être exécutées dans le **terminal web d'OpenShift**.

---

## Étape 1 : Connexion au Cluster OpenShift

Pour commencer, vous devez vous connecter à votre cluster OpenShift :

1. Accédez à la console Web OpenShift.
2. Cliquez sur votre **nom d'utilisateur** en haut à droite.
3. Sélectionnez **"Copy login command"**.

    ![Copy login command](./images/copy-login-command.svg)

4. Cliquez sur **"Display Token"** pour afficher le token.

    ![Display Token](./images/display-token.png)

5. Copiez la commande de connexion affichée.

6. Ouvrez le **terminal web OpenShift** en haut à droite de la console.

![Open web terminal](./images/open-web-terminal.svg)

7. Cliquez sur **"Open terminal in a new tab"** et sélectionnez votre projet `<VOTRECITY>-user-ns`. Cliquez sur **"Start"**. Le premier démarrage peut prendre quelques secondes.

![new-onglet terminal](./images/new-onglet-web-terminal.svg)

![select project terminal](./images/select-project.svg)

8. **Collez et exécutez la commande de connexion** dans votre terminal web OpenShift.

```bash
oc login --token=<votre_token> --server=https://api.neutron-sno-office.neutron-it.fr:6443
```

Vous devriez voir un message confirmant la connexion réussie :
```
Login successful.

You have access to the following projects and can switch between them with 'oc project <projectname>':

  * prague-user-ns

Using project "prague-user-ns".
```

---

## Étape 2 : Exploration des Commandes Disponibles

Maintenant que vous êtes connecté, explorons les commandes disponibles :

1. **Listez les commandes disponibles avec `kubectl`** :
```bash
kubectl help
```

2. **Listez les commandes disponibles avec `oc`** :
```bash
oc help
```

La commande `oc` étend `kubectl` avec des fonctionnalités spécifiques à OpenShift comme `new-app`, `new-project`, `start-build`, etc.

---

## Étape 3 : Gestion du Namespace

Vérifiez que vous êtes dans le bon namespace :

```bash
oc project
```

Sortie attendue :
```
Using project "prague-user-ns" from context named "prague-user-ns/..." on server "https://api.neutron-sno-office.neutron-it.fr:6443".
```

---

## Étape 4 : Création d'une Nouvelle Application

Déployez une application simple à partir d'une image de conteneur :

```bash
oc new-app --image=quay.io/neutron-it/p02l01-go-app
```

Sortie attendue :
```
--> Found container image ec997ee from quay.io for "quay.io/neutron-it/p02l01-go-app"

    Go 1.21.11
    ----------
    ...

--> Creating resources ...
    deployment.apps "p02l01-go-app" created
    service "p02l01-go-app" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing:
     'oc expose service/p02l01-go-app'
    Run 'oc status' to view your app.
```

---

## Étape 5 : Description et Vérification de l'Application

1. **Attendez que le pod soit en état Running** :
```bash
oc get pods
```

```
NAME                                         READY   STATUS    RESTARTS   AGE
p02l01-go-app-6c457d7469-vhl2q               1/1     Running   0          2m33s
```

2. **Décrivez le déploiement pour obtenir les détails** :
```bash
oc describe deployment/p02l01-go-app
```

---

## Étape 6 : Affichage des Logs de l'Application

```bash
oc logs deployment/p02l01-go-app
```

Sortie attendue :
```
Bravo, vous êtes dans Exercice Guidé : Interaction avec OpenShift via la Ligne de Commande
```

Les logs sont essentiels pour diagnostiquer les problèmes et vérifier que l'application fonctionne correctement.

---

## Étape 7 : Exécution de Commandes dans un Pod

Pour interagir directement avec les conteneurs de votre application :

1. **Obtenez la liste des pods** :
```bash
oc get pods
```
```
NAME                             READY   STATUS    RESTARTS   AGE
p02l01-go-app-7f5b7fdfd6-4rfsl   1/1     Running   0          2m
```

2. **Obtenez un shell interactif dans le pod** :
```bash
oc exec -it deployment/p02l01-go-app -- /bin/sh
```

3. **Exécutez des commandes dans le pod** :
```bash
ps aux
```

```
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
1001540+       1  0.0  0.0  12004  2496 ?        Ss   10:09   0:00 sh -c ./main && while true; do sleep 86400; done
1001540+      12  0.0  0.0  23148  1528 ?        S    10:09   0:00 /usr/bin/sleep 86400
1001540+      20  1.5  0.0  12136  3232 pts/0    Ss   10:20   0:00 /bin/sh
1001540+      26  0.0  0.0  44784  3436 pts/0    R+   10:20   0:00 ps aux
```

4. **Sortez du shell** :
```bash
exit
```

---

## Étape 8 : Suppression de l'Application

Nettoyez les ressources créées :

```bash
oc delete all -l app=p02l01-go-app
```

Confirmation de suppression :
```
service "p02l01-go-app" deleted
deployment.apps "p02l01-go-app" deleted
imagestream.image.openshift.io "p02l01-go-app" deleted
```

---

### Conclusion

En suivant ces étapes, vous avez appris à :
- Vous connecter à un cluster OpenShift via le terminal web
- Comparer les commandes `kubectl` et `oc`
- Créer et gérer une application avec `oc new-app`
- Afficher les logs et exécuter des commandes dans un pod
- Nettoyer les ressources créées
