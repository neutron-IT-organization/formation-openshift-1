# Exercice Guidé : Exploration de la console OpenShift

Dans cet exercice, vous allez apprendre à naviguer dans la console web d'OpenShift et à déployer votre première application. Suivez les étapes ci-dessous pour vous familiariser avec l'interface utilisateur et ses fonctionnalités.

## Objectifs de l'exercice

- Accéder à la console web d'OpenShift
- Naviguer entre les perspectives Developer et Administrator
- Déployer une application à partir d'une image de conteneur
- Accéder à l'application via une Route
- Superviser les ressources du projet

## Étape 1 : Accéder à la console web

1. Ouvrez votre navigateur web.
2. Entrez l'URL de la console web d'OpenShift :
```shell
https://console-openshift-console.apps.neutron-sno-office.neutron-it.fr/
```
3. Cliquez sur `Neutron Guest Identity Management`
4. Connectez-vous avec vos identifiants :
   - **Utilisateur** : `<VOTRECITY>-user` (ex: `prague-user`)
   - **Mot de passe** : `OpenShift4formation!`

## Étape 2 : Créer un projet et déployer une application

Basculez vers la **perspective Developer** (menu en haut à gauche) :

1. Cliquez sur votre **Project** en haut à gauche, puis sur **"Create project"**.

![Create Project](./images/create_project.png)

Créez un projet nommé **`console-exploration-<VOTRECITY>`** (ex: `console-exploration-prague`) et ajoutez une brève description.

2. Cliquez sur **"Create"** pour finaliser la création du projet.

![Create Project](./images/assistant_create_project.png)

3. Cliquez sur **"+Add"** pour déployer une application.

4. Sélectionnez **"Container images"**.

![Basic quarkus](./images/container_image.png)

5. Dans le champ **"Image name from external registry"**, entrez :
```
docker.io/openshift/hello-openshift
```
Examinez les valeurs par défaut, puis cliquez sur **"Create"** en bas de la page.

![Create application](./images/create_application.png)

## Étape 3 : Accéder à l'application via la Route

1. Rendez-vous sur la page **"Topology"** qui affiche maintenant votre déploiement `hello-openshift`.

2. Cliquez sur l'icône du déploiement pour ouvrir le panneau de détails.

![Topology view](./images/topology_view.png)

3. Dans la section **"Routes"**, cliquez sur le lien pour accéder à l'application dans votre navigateur.

![Get route](./images/get_route.png)

Vous devriez voir un message de bienvenue de l'application `hello-openshift`.

![Result](./images/result_hello_world.png)

## Étape 4 : Inspecter les ressources en vue Administrator

Basculez vers la **perspective Administrator** (menu en haut à gauche) :

1. Accédez à **"Home" > "Projects"** pour voir la liste des projets, y compris `console-exploration-<VOTRECITY>`.

![Global](./images/administrator_global_view.png)

2. Accédez à **"Workloads" > "Pods"** pour voir les pods de `hello-openshift`.

![pod](./images/pod_view.png)

3. Accédez à **"Workloads" > "Deployments"** pour voir les détails du déploiement.

![deployment](./images/deployments_view.png)

4. Accédez à **"Networking" > "Services"** pour voir le service associé.

![Rservice](./images/service_view.png)

## Étape 5 : Supprimer le projet

1. Retournez à **"Home" > "Projects"**.

2. Dans le menu contextuel (⋮) du projet `console-exploration-<VOTRECITY>`, sélectionnez **"Delete Project"**.

![project](./images/project_delete.png)

3. Saisissez le nom du projet pour confirmer la suppression, puis cliquez sur **"Delete"**.

## Conclusion

Vous avez maintenant une bonne compréhension de la console web d'OpenShift. Vous savez comment créer des projets, déployer des applications depuis une image de conteneur, accéder aux applications via une Route et naviguer entre les perspectives Developer et Administrator.

---

Dans la prochaine section, nous aborderons l'architecture d'OpenShift et Kubernetes.
