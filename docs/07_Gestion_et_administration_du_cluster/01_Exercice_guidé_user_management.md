# Exercice Guidé : Gestion des Utilisateurs et RBAC dans OpenShift

Cet exercice est réalisé en **mode démonstration par le formateur**. Vous allez observer comment configurer des rôles différents pour des utilisateurs et vérifier leurs accès respectifs.

---

## Objectifs de l'Exercice

- Créer deux utilisateurs avec des identifiants distincts (déjà configurés sur le cluster).
- Attribuer des rôles différents dans un namespace.
- Vérifier que chaque utilisateur a des accès spécifiques.
- Comprendre la différence entre les rôles `view`, `edit` et `admin`.

---

## Prérequis

- Un cluster OpenShift opérationnel avec des droits administratifs.
- OpenShift CLI (`oc`) configuré et connecté avec un compte admin.

:::info Contexte de la formation
Les utilisateurs de la formation (`paris-user`, `tokyo-user`, etc.) sont déjà configurés avec le provider `htpasswd`. Chaque utilisateur a le rôle `admin` dans son propre namespace `<city>-user-ns`.

Dans cet exercice, nous allons créer de nouveaux utilisateurs temporaires pour démontrer la gestion des rôles.
:::

---

## Étapes de l'Exercice

### 1. Créer le Fichier HTPasswd pour les Nouveaux Utilisateurs

Sur votre machine, générez un fichier htpasswd avec deux utilisateurs de test :

```bash
# Créer le fichier avec user1
htpasswd -c -B -b /tmp/htpasswd-demo user1 password1

# Ajouter user2 au fichier
htpasswd -B -b /tmp/htpasswd-demo user2 password2

# Vérifier le contenu
cat /tmp/htpasswd-demo
```

Sortie attendue :
```
user1:$2y$05$...
user2:$2y$05$...
```

### 2. Créer le Secret HTPasswd dans OpenShift

```bash
oc create secret generic htpasswd-demo \
  --from-file=htpasswd=/tmp/htpasswd-demo \
  -n openshift-config
```

### 3. Configurer le Provider d'Identité

Ajoutez le provider au fichier OAuth du cluster :

```bash
oc edit oauth cluster
```

Ajoutez dans `spec.identityProviders` :
```yaml
  - htpasswd:
      fileData:
        name: htpasswd-demo
    mappingMethod: claim
    name: demo-htpasswd
    type: HTPasswd
```

Sauvegardez et fermez l'éditeur. Attendez que les pods oauth-openshift redémarrent :

```bash
oc get pods -n openshift-authentication -w
```

### 4. Créer un Namespace et Attribuer des Rôles Différents

```bash
oc new-project rbac-demo
```

Donnez à `user1` le rôle `view` (lecture seule) :
```bash
oc policy add-role-to-user view user1 -n rbac-demo
```

Donnez à `user2` le rôle `edit` (lecture + écriture) :
```bash
oc policy add-role-to-user edit user2 -n rbac-demo
```

Vérifiez les rolebindings créés :
```bash
oc get rolebinding -n rbac-demo
```

### 5. Vérifier les Accès de Chaque Utilisateur

**Connexion en tant que `user1` (rôle view)** :
```bash
oc login -u user1 -p password1
```

```bash
# Autorisé : lister les pods
oc get pods -n rbac-demo
# No resources found in rbac-demo namespace.

# Interdit : créer un pod
oc run nginx --image=nginx -n rbac-demo
# Error from server (Forbidden): pods is forbidden: User "user1" cannot create resource...
```

**Connexion en tant que `user2` (rôle edit)** :
```bash
oc login -u user2 -p password2
```

```bash
# Autorisé : lister les pods
oc get pods -n rbac-demo

# Autorisé : créer un pod
oc run nginx --image=nginx -n rbac-demo
# pod/nginx created
```

### 6. Vérifier les Permissions sans Se Connecter

OpenShift permet de tester les permissions sans changer d'utilisateur avec `oc auth can-i` :

```bash
# Revenir en tant qu'admin
oc login -u <votre-user> -p <votre-password>

# Vérifier si user1 peut créer des pods dans rbac-demo
oc auth can-i create pods --as=user1 -n rbac-demo
# no

# Vérifier si user2 peut créer des pods dans rbac-demo
oc auth can-i create pods --as=user2 -n rbac-demo
# yes
```

---

## Nettoyage

```bash
# Supprimer le projet et les permissions
oc delete project rbac-demo

# Supprimer les utilisateurs temporaires
oc delete user user1 user2
oc delete identity demo-htpasswd:user1 demo-htpasswd:user2

# Supprimer le secret et le provider
oc delete secret htpasswd-demo -n openshift-config

# Éditer l'OAuth pour retirer le provider demo-htpasswd
oc edit oauth cluster
# Supprimez le bloc correspondant à demo-htpasswd
```

---

## Résumé des Rôles OpenShift

| Rôle | Get/List | Create | Update | Delete | Manage RBAC |
|------|----------|--------|--------|--------|-------------|
| `view` | ✅ | ❌ | ❌ | ❌ | ❌ |
| `edit` | ✅ | ✅ | ✅ | ✅ | ❌ |
| `admin` | ✅ | ✅ | ✅ | ✅ | ✅ (namespace) |
| `cluster-admin` | ✅ | ✅ | ✅ | ✅ | ✅ (cluster) |

---

## Conclusion

Vous avez appris à :
- Créer des utilisateurs avec `htpasswd` et les intégrer dans OpenShift
- Attribuer des rôles différents (`view`, `edit`) à des utilisateurs dans un namespace
- Vérifier les accès avec `oc auth can-i`
- Nettoyer les utilisateurs et configurations temporaires

La gestion fine des accès (RBAC) est essentielle pour la sécurité des clusters OpenShift en production.
