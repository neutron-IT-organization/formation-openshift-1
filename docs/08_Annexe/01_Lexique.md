# Lexique OpenShift & Kubernetes

Définitions des termes clés utilisés tout au long de la formation.

---

## A

**Annotation**
Métadonnée arbitraire attachée à une ressource Kubernetes (paire clé/valeur). Contrairement aux labels, les annotations ne sont pas utilisées pour sélectionner des ressources mais pour stocker des informations à destination d'outils ou d'opérateurs.

**API Server (kube-apiserver)**
Point d'entrée central du control plane Kubernetes. Toutes les interactions avec le cluster (CLI, dashboard, opérateurs) passent par l'API server via des appels REST. Il valide et persiste l'état dans etcd.

**ApplicationSet**
Ressource ArgoCD permettant de générer automatiquement plusieurs Applications ArgoCD à partir de modèles (ex : une application par namespace utilisateur).

**ArgoCD**
Outil de livraison continue GitOps qui synchronise l'état du cluster avec un dépôt Git. Il est intégré dans OpenShift via l'Operator OpenShift GitOps.

---

## C

**CGroupes (Control Groups)**
Mécanisme du noyau Linux permettant de limiter et d'isoler les ressources (CPU, mémoire, I/O) utilisées par un groupe de processus. Fondement de l'isolation des conteneurs.

**CNI (Container Network Interface)**
Interface standardisée permettant aux plugins réseau (OVN-Kubernetes, Calico, Flannel…) de configurer le réseau des pods dans Kubernetes.

**ConfigMap**
Ressource Kubernetes permettant de stocker des données de configuration non sensibles (fichiers de config, variables d'environnement) sous forme de paires clé/valeur. Peut être monté comme volume ou injecté comme variable d'environnement dans un pod.

**Conteneur**
Processus isolé s'exécutant sur un noyau Linux partagé grâce aux namespaces et cgroups. Contrairement à une VM, il ne virtualise pas le matériel mais partage le noyau de l'hôte, ce qui le rend beaucoup plus léger.

**Control Plane**
Ensemble des composants qui gèrent l'état du cluster : API server, etcd, scheduler, controller manager. Dans OpenShift, ces composants s'exécutent en haute disponibilité sur les nœuds masters.

**CRI (Container Runtime Interface)**
Interface standardisée entre Kubernetes et le runtime de conteneurs (CRI-O, containerd). Définit comment Kubernetes ordonne la création, l'arrêt et l'inspection des conteneurs.

**CRI-O**
Runtime de conteneurs léger développé spécifiquement pour Kubernetes, utilisé par défaut dans OpenShift 4. Implémente le CRI et utilise runc pour exécuter les conteneurs.

**CronJob**
Ressource Kubernetes permettant d'exécuter des Jobs de manière planifiée (syntaxe cron). Utile pour les tâches périodiques comme les sauvegardes ou les rapports.

**CRD (CustomResourceDefinition)**
Mécanisme permettant d'étendre l'API Kubernetes en définissant de nouveaux types de ressources. Les Operators utilisent des CRDs pour exposer leurs propres objets (ex : `EtcdCluster`, `PostgreSQL`).

---

## D

**DaemonSet**
Ressource garantissant qu'un pod et un seul s'exécute sur chaque nœud éligible du cluster (ou sur un sous-ensemble via nodeSelector/affinity). Utilisé pour les agents de monitoring, de logging ou les plugins réseau.

**Deployment**
Ressource déclarative pour déployer des applications sans état. Gère un ReplicaSet pour maintenir le nombre désiré de réplicas et orchestre les mises à jour progressives (RollingUpdate).

---

## E

**etcd**
Base de données clé/valeur distribuée et cohérente (consensus Raft) utilisée par Kubernetes pour stocker tout l'état du cluster. C'est la source de vérité du cluster.

---

## H

**HPA (HorizontalPodAutoscaler)**
Ressource qui ajuste automatiquement le nombre de réplicas d'un Deployment, StatefulSet ou ReplicaSet en fonction de métriques (CPU, mémoire, métriques custom).

**Helm**
Gestionnaire de paquets pour Kubernetes. Permet de déployer des applications complexes via des "charts" (templates paramétrables de manifestes YAML).

---

## I

**Image OCI**
Format standard d'image de conteneur défini par l'Open Container Initiative. Garantit la portabilité des images entre les différents runtimes (Docker, CRI-O, containerd).

**Ingress**
Ressource Kubernetes standard définissant des règles de routage HTTP(S) vers des Services. Dans OpenShift, les Routes sont l'équivalent natif avec des fonctionnalités supplémentaires.

---

## J

**Job**
Ressource Kubernetes pour l'exécution de tâches ponctuelles. Un Job crée un ou plusieurs pods et s'assure qu'ils se terminent avec succès avant d'être considéré comme complet.

---

## K

**kubelet**
Agent s'exécutant sur chaque nœud du cluster. Il communique avec l'API server, s'assure que les pods assignés au nœud s'exécutent correctement et rapporte leur état.

**kube-proxy**
Composant réseau s'exécutant sur chaque nœud. Maintient les règles iptables/ipvs pour implémenter les Services Kubernetes (load balancing vers les pods).

**Kustomize**
Outil de personnalisation de manifestes YAML Kubernetes sans templates. Intégré dans `kubectl` et `oc`. Permet de gérer des variantes (dev/staging/prod) d'une même configuration de base.

---

## L

**Label**
Paire clé/valeur attachée à une ressource Kubernetes (pod, node, service…). Utilisée pour identifier, organiser et sélectionner des ressources via des selectors. Fondamental pour le fonctionnement des Services et des Deployments.

**LimitRange**
Ressource définissant des contraintes sur les ressources (CPU, mémoire) par conteneur ou par pod dans un namespace. Définit des valeurs par défaut et des limites min/max.

---

## M

**MachineConfig**
Ressource OpenShift permettant de configurer les nœuds au niveau OS : fichiers de configuration, paramètres systemd, sysctl, kubelet config. Gérée par le Machine Config Operator.

**MachineConfigPool**
Regroupe des nœuds ayant la même configuration MachineConfig. Orchestre le rollout des configurations sur les nœuds (workers, masters, custom).

**MachineSet**
Ressource OpenShift gérant le cycle de vie d'un groupe de machines (nœuds workers). Similaire à un ReplicaSet pour les pods, mais pour les VMs/instances cloud.

---

## N

**Namespace**
Mécanisme d'isolation logique des ressources dans Kubernetes. Permet de séparer les environnements (dev, staging, prod) ou les équipes au sein d'un même cluster. Dans OpenShift, le concept de **Project** est un namespace enrichi.

**NetworkPolicy**
Ressource Kubernetes définissant les règles de pare-feu au niveau des pods. Permet de contrôler quel pod peut communiquer avec quel autre pod ou service.

**Node**
Machine (physique ou virtuelle) faisant partie d'un cluster Kubernetes. Les nœuds **masters** hébergent le control plane, les **workers** exécutent les workloads applicatifs.

---

## O

**OCI (Open Container Initiative)**
Organisation définissant les standards ouverts pour les formats d'images et les runtimes de conteneurs (specs `image-spec` et `runtime-spec`).

**Operator**
Logiciel qui automatise la gestion d'une application complexe dans Kubernetes en utilisant des CRDs et des contrôleurs personnalisés. Encode la connaissance opérationnelle d'un SRE/DBA dans du code.

**OVN-Kubernetes**
Plugin CNI utilisé par défaut dans OpenShift 4.x. Basé sur Open Virtual Network, il fournit le réseau de pods, les services et les NetworkPolicies.

---

## P

**PersistentVolume (PV)**
Ressource représentant un volume de stockage provisionné dans le cluster (NFS, AWS EBS, Ceph…). Il existe indépendamment des pods et a un cycle de vie propre.

**PersistentVolumeClaim (PVC)**
Demande de stockage émise par un pod. Le PVC est lié à un PV qui répond à ses critères (taille, mode d'accès, StorageClass). Le pod monte le PVC comme un volume.

**Pod**
La plus petite unité déployable dans Kubernetes. Un pod contient un ou plusieurs conteneurs partageant le même namespace réseau (IP) et des volumes communs. Les conteneurs d'un pod communiquent via `localhost`.

**Probe**
Mécanisme de vérification de la santé d'un conteneur :
- **Liveness** : redémarre le conteneur s'il est dans un état défaillant
- **Readiness** : retire le pod du Service s'il n'est pas prêt à recevoir du trafic
- **Startup** : laisse le temps à un conteneur lent à démarrer avant d'activer les autres probes

**Project**
Dans OpenShift, un Project est un namespace Kubernetes enrichi avec des fonctionnalités supplémentaires : quotas par défaut, annotations, accès via la console web.

---

## R

**RBAC (Role-Based Access Control)**
Système de gestion des accès basé sur des rôles. Définit **qui** (user, group, ServiceAccount) peut faire **quoi** (verbs : get, list, create, delete…) sur **quelles ressources** dans quel périmètre (namespace ou cluster).

**ReplicaSet**
Ressource garantissant qu'un nombre spécifié de réplicas de pods identiques s'exécutent à tout moment. Généralement géré par un Deployment et rarement utilisé directement.

**ResourceQuota**
Ressource définissant des limites globales de consommation de ressources (CPU, mémoire, nombre de pods, PVCs…) pour l'ensemble d'un namespace.

**Role / ClusterRole**
Objet RBAC définissant un ensemble de permissions. Un **Role** s'applique dans un namespace spécifique, un **ClusterRole** s'applique au niveau du cluster entier.

**RoleBinding / ClusterRoleBinding**
Lie un sujet (user, group, ServiceAccount) à un Role ou ClusterRole, lui accordant ainsi les permissions définies.

**Route**
Ressource OpenShift permettant d'exposer un Service via une URL HTTP(S) externe. Gérée par le HAProxy Router d'OpenShift. Supporte les terminaisons TLS Edge, Passthrough et Reencrypt.

---

## S

**Secret**
Ressource Kubernetes pour stocker des données sensibles (mots de passe, tokens, certificats) encodées en base64. Peut être monté comme volume ou injecté comme variable d'environnement. Doit être protégé par RBAC et chiffré au repos (encryption at rest).

**ServiceAccount**
Identité assignée à un pod pour interagir avec l'API Kubernetes. Permet de donner des permissions précises à une application via RBAC sans utiliser un compte utilisateur humain.

**StatefulSet**
Ressource pour les applications avec état (bases de données, queues de messages). Garantit des noms de pods stables et prévisibles, un ordre de démarrage/arrêt séquentiel, et des volumes persistants dédiés par réplica.

**StorageClass**
Définit un "type" de stockage disponible dans le cluster (SSD, HDD, NFS…). Utilisé par les PVCs pour le provisionnement dynamique de volumes.

---

## T

**Taint**
Propriété attachée à un nœud pour empêcher des pods de s'y scheduler, sauf si ces pods ont la **Toleration** correspondante. Permet de réserver des nœuds pour des workloads spécifiques.

**Toleration**
Propriété d'un pod lui permettant d'être schedulé sur un nœud ayant le Taint correspondant.

---

## V

**VPA (VerticalPodAutoscaler)**
Ajuste automatiquement les requests/limits de CPU et mémoire d'un pod en fonction de sa consommation réelle. Complémentaire au HPA.

**Volume**
Répertoire accessible aux conteneurs d'un pod. Peut être de type `emptyDir` (éphémère), `hostPath`, `configMap`, `secret`, `persistentVolumeClaim`, etc.
