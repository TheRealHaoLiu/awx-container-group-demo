# AWX Container Group Manual Instruction

## Prepare your kubernetes cluster for AWX Container Group

### Create a Namespace for the Container Group

This namespace is created to contain pods that get created when running join on the container group

```bash
kubectl create namespace awx-cg-demo-manual
```

For more information about the namespace, please refer to [Kubernetes Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

### Create a ServiceAccount for the Container Group

In order for AWX to communicate with the Kube API server to run jobs, a credential is needed. You can use token for any credential that you like so long as it has the correct permissions.

In this example, we will create a new ServiceAccount and use the token for the ServiceAccount as the credential.

```bash
kubectl create serviceaccount awx-cg-demo-manual -n awx-cg-demo-manual
```

For more information about the ServiceAccount, please refer to [Kubernetes ServiceAccount](https://kubernetes.io/docs/reference/access-authn-authz/service-accounts-admin/)

### Create Role for the ServiceAccount

The credential that AWX use to communicate with the Kube API Server require certain permissions. Before we can assign the permission to the ServiceAccount, we need to create a Role that contains the permissions.

```bash
kubectl apply -f - << EOF
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: awx-cg-demo-manual-role
  namespace: awx-cg-demo-manual
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["pods/attach"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
EOF
```

For more information about the Role, please refer to [Kubernetes Role](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#role-and-clusterrole)

### Create RoleBinding for the ServiceAccount

RoleBinding is used to assign the Role to the ServiceAccount. In this example, we will create a RoleBinding that assign the Role we created in the previous step to the ServiceAccount we created in the previous step, which grants the ServiceAccount the permissions we defined in the Role.

```bash
kubectl apply -f - << EOF
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: awx-cg-demo-manual-rolebinding
  namespace: awx-cg-demo-manual
subjects:
- kind: ServiceAccount
  name: awx-cg-demo-manual
  namespace: awx-cg-demo-manual
roleRef:
  kind: Role
  name: awx-cg-demo-manual-role
  apiGroup: rbac.authorization.k8s.io
EOF
```

For more information about the RoleBinding, please refer to [Kubernetes RoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)

### Creating a ServiceAccount token

Since AWX will be using the ServiceAccount as it's identity to authenticate with the Kube API Server, we need to create long lived token for the ServiceAccount.

```bash
kubectl apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: awx-cg-demo-manual
  namespace: awx-cg-demo-manual
  annotations:
    kubernetes.io/service-account.name: awx-cg-demo-manual
type: kubernetes.io/service-account-token
EOF

```

For more information about anually create a long-lived API token for a ServiceAccount, please refer to [Manually create a long-lived API token for a ServiceAccount](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-a-long-lived-api-token-for-a-serviceaccount)

### Retrive the generated ServiceAccount token

IMPORTANT: this command will print out the ServiceAccount token in plain text! Make sure you keep the token secret.

```bash
kubectl get secret awx-cg-demo-manual-token -n awx-cg-demo-manual -o jsonpath={.data.token} | base64 -d
```

### (Optional) Create image pull secrets

If you are using a private image registry to host your Execution Environment images please make sure you create a image pull secret in this namespace that the container group can refrence.

For more information about image pull secrets, please refer to [Kubernetes ImagePullSecrets](https://kubernetes.io/docs/concepts/containers/images/#specifying-imagepullsecrets-on-a-pod)

## Setting up Container Group in AWX UI
TODO
