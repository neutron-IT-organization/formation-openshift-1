# Ressources OpenShift & Kubernetes

Référence complète des ressources (objets API) disponibles dans OpenShift / Kubernetes, organisées par domaine fonctionnel.

---

## Workloads

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **Pod** | `po` | Unité de base : un ou plusieurs conteneurs partageant IP et volumes. Rarement créé directement. |
| **Deployment** | `deploy` | Déploiement d'applications sans état. Gère un ReplicaSet et orchestre les rolling updates. |
| **ReplicaSet** | `rs` | Maintient N réplicas de pods. Géré automatiquement par un Deployment. |
| **StatefulSet** | `sts` | Déploiement d'applications avec état. Pods nommés séquentiellement, volumes dédiés par réplica. |
| **DaemonSet** | `ds` | Un pod par nœud (ou sous-ensemble). Utilisé pour les agents système (monitoring, logs, réseau). |
| **Job** | - | Exécute une tâche ponctuelle jusqu'à complétion. Crée un ou plusieurs pods en parallèle. |
| **CronJob** | `cj` | Planifie des Jobs selon une expression cron. |
| **ReplicationController** | `rc` | Ancêtre du ReplicaSet. Déprécié, remplacé par Deployment. |

```bash
oc get pods
oc get deployments
oc get statefulsets
oc get daemonsets
oc get jobs
oc get cronjobs
```

---

## Réseau

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **Service** | `svc` | Abstraction réseau stable exposant des pods via un sélecteur. Types : ClusterIP, NodePort, LoadBalancer, ExternalName. |
| **Route** | - | *(OpenShift)* Expose un Service via une URL HTTP(S) gérée par le routeur HAProxy. Supporte TLS Edge/Passthrough/Reencrypt. |
| **Ingress** | `ing` | *(Kubernetes)* Règles de routage HTTP(S) vers des Services. Implémenté par un Ingress Controller (nginx, HAProxy…). |
| **NetworkPolicy** | `netpol` | Politique de pare-feu au niveau pods. Contrôle les flux entrants (ingress) et sortants (egress). |
| **EndpointSlice** | `endpointslice` | Liste les IPs des pods derrière un Service. Remplace l'ancien objet Endpoints. |
| **IngressClass** | - | Définit quel contrôleur implémente les Ingress. |

```bash
oc get services
oc get routes
oc get networkpolicies
```

---

## Configuration

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **ConfigMap** | `cm` | Données de configuration non sensibles (fichiers, variables d'env). |
| **Secret** | - | Données sensibles encodées en base64 (mots de passe, tokens, certificats). Types : Opaque, kubernetes.io/tls, kubernetes.io/dockerconfigjson… |

```bash
oc get configmaps
oc get secrets
oc get configmap <name> -o yaml
oc get secret <name> -o jsonpath='{.data.<key>}' | base64 -d
```

---

## Stockage

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **PersistentVolume** | `pv` | Volume de stockage provisionné dans le cluster (ressource cluster-scoped). |
| **PersistentVolumeClaim** | `pvc` | Demande de stockage émise par un pod. Liée à un PV. |
| **StorageClass** | `sc` | Définit un type de stockage et son provisioner dynamique. |
| **VolumeSnapshot** | `vs` | Capture instantanée d'un PVC pour sauvegarde ou clonage. |
| **VolumeSnapshotClass** | `vsc` | Définit le provisioner pour les snapshots. |

```bash
oc get pvc
oc get pv
oc get storageclass
```

---

## Autoscaling

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **HorizontalPodAutoscaler** | `hpa` | Ajuste le nombre de réplicas selon des métriques CPU/mémoire/custom. |
| **VerticalPodAutoscaler** | `vpa` | Ajuste les requests/limits des conteneurs selon leur consommation réelle. |
| **PodDisruptionBudget** | `pdb` | Garantit un nombre minimal de pods disponibles lors d'opérations de maintenance. |

```bash
oc get hpa
oc autoscale deployment <name> --min=2 --max=10 --cpu-percent=70
```

---

## RBAC & Sécurité

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **ServiceAccount** | `sa` | Identité d'un pod pour interagir avec l'API Kubernetes. |
| **Role** | - | Ensemble de permissions dans un namespace. |
| **ClusterRole** | `cr` | Ensemble de permissions au niveau cluster. |
| **RoleBinding** | `rb` | Lie un sujet (user/group/SA) à un Role dans un namespace. |
| **ClusterRoleBinding** | `crb` | Lie un sujet à un ClusterRole au niveau cluster. |
| **SecurityContextConstraints** | `scc` | *(OpenShift)* Politique de sécurité contrôlant ce qu'un pod peut faire (UID, capabilities, volumes…). Remplace les PodSecurityPolicies de Kubernetes vanilla. |

```bash
oc get serviceaccounts
oc get roles -n <namespace>
oc get clusterroles
oc get rolebindings -n <namespace>
oc get clusterrolebindings
oc get scc
oc adm policy add-role-to-user edit <user> -n <namespace>
```

---

## Namespaces & Projets

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **Namespace** | `ns` | Isolation logique des ressources dans Kubernetes. |
| **Project** | - | *(OpenShift)* Namespace enrichi avec des fonctionnalités supplémentaires (quotas, annotations, RBAC automatique). |
| **ResourceQuota** | `quota` | Limite la consommation totale de ressources dans un namespace. |
| **LimitRange** | `limits` | Définit les valeurs par défaut et limites min/max de ressources par conteneur/pod. |

```bash
oc get namespaces
oc get projects
oc get resourcequota
oc get limitrange
oc describe resourcequota
```

---

## Nœuds & Infrastructure

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **Node** | `no` | Machine (physique ou VM) dans le cluster. |
| **MachineSet** | - | *(OpenShift)* Gère le cycle de vie des machines workers (création/suppression). |
| **Machine** | - | *(OpenShift)* Représente une machine individuelle (VM, bare-metal). |
| **MachineConfig** | `mc` | *(OpenShift)* Configure l'OS des nœuds (fichiers, systemd, kubelet…). |
| **MachineConfigPool** | `mcp` | *(OpenShift)* Regroupe des nœuds avec la même configuration et gère le rollout. |

```bash
oc get nodes
oc get machinesets -n openshift-machine-api
oc get machines -n openshift-machine-api
oc get machineconfig
oc get machineconfigpool
oc adm cordon <node>
oc adm drain <node> --ignore-daemonsets
```

---

## Monitoring & Alerting

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **PrometheusRule** | - | Définit des règles d'alerte et d'enregistrement Prometheus. |
| **ServiceMonitor** | - | Configure Prometheus pour scraper les métriques d'un Service. |
| **PodMonitor** | - | Configure Prometheus pour scraper les métriques directement sur des pods. |
| **AlertmanagerConfig** | - | Configure les routes et récepteurs Alertmanager. |

```bash
oc get prometheusrule -n openshift-monitoring
oc get servicemonitor -n openshift-monitoring
oc get route -n openshift-monitoring | grep prometheus
oc get route -n openshift-monitoring | grep grafana
```

---

## Cluster OpenShift

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **ClusterVersion** | `cv` | Version et état de mise à jour du cluster OpenShift. |
| **ClusterOperator** | `co` | Opérateurs intégrés d'OpenShift gérant les composants du cluster (réseau, stockage, monitoring…). |
| **ClusterNetwork** | - | Configuration réseau globale du cluster. |
| **DNSRecord** | - | Enregistrements DNS gérés par le DNS Operator. |
| **OAuth** | - | Configuration de l'authentification OpenShift (fournisseurs d'identité). |
| **User** | - | Utilisateur OpenShift (admin seulement). |
| **Group** | - | Groupe d'utilisateurs OpenShift. |
| **Identity** | - | Identité provenant d'un fournisseur d'authentification. |

```bash
oc get clusterversion
oc get clusteroperators
oc get co | grep -v "True.*False.*False"
oc get users
oc get groups
oc get oauth cluster -o yaml
```

---

## GitOps (ArgoCD / OpenShift GitOps)

| Ressource | Abréviation | Description |
|-----------|-------------|-------------|
| **Application** | - | *(ArgoCD)* Définit une application GitOps : source Git, destination cluster, sync policy. |
| **ApplicationSet** | - | *(ArgoCD)* Génère plusieurs Applications à partir de templates (générateurs : list, git, cluster…). |
| **AppProject** | - | *(ArgoCD)* Projet ArgoCD définissant le périmètre de déploiement (repos, clusters, namespaces autorisés). |

```bash
oc get applications -n openshift-gitops
oc get applicationsets -n openshift-gitops
oc get appprojects -n openshift-gitops
```

---

## API Discovery

Pour explorer toutes les ressources disponibles dans votre cluster :

```bash
# Lister toutes les ressources API disponibles
oc api-resources

# Lister avec les groupes et versions API
oc api-resources -o wide

# Voir la définition d'une ressource
oc explain deployment
oc explain deployment.spec.template.spec.containers.resources

# Voir toutes les versions d'une API
oc api-versions
```
