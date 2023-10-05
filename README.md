# cartov2-gitops

## Configure a run namespace

```
kubectl create namespace run-ns
```

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: run-ns
  name: app-package-and-pkgi-install-role
rules:
  - apiGroups: ["data.packaging.carvel.dev"]
    resources: ["packages"]
    verbs: ["get", "list", "create", "update", "delete", "patch"]
  - apiGroups: ["packaging.carvel.dev"]
    resources: ["packageinstalls"]
    verbs: ["get", "list", "create", "update", "delete", "patch"]
  - apiGroups: [""]
    resources: ["secrets"]
    verbs: ["get", "list", "create", "update", "delete", "patch"]
```

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: run-ns
  name: app-cr-role
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["configmaps", "services"]
  verbs: ["get", "list", "create", "update", "delete"]
- apiGroups: ["networking.k8s.io"]
  resources: ["ingresses"]
  verbs: ["get", "list", "create", "update", "delete"]
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-package-and-pkgi-install-rolebinding
  namespace: run-ns
subjects:
- kind: ServiceAccount
  name: default
  namespace: run-ns
roleRef:
  kind: Role
  name: app-package-and-pkgi-install-role
  apiGroup: rbac.authorization.k8s.io
```

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-cr-rolebinding
  namespace: run-ns
subjects:
- kind: ServiceAccount
  name: default
  namespace: run-ns
roleRef:
  kind: Role
  name: app-cr-role
  apiGroup: rbac.authorization.k8s.io
```

Validate the permissions of the service account using the `kubectl permissions` plugin.

```
❯ kubectl permissions default
ServiceAccount/default (run-ns)
├ RoleBinding/app-cr-rolebinding (run-ns)
│ └ Role/app-cr-role (run-ns)
│   ├ apps
│   │ └ deployments verbs=[get list create update delete] ✔
│   ├ core.k8s.io
│   │ ├ configmaps verbs=[get list create update delete] ✔
│   │ └ services verbs=[get list create update delete] ✔
│   └ networking.k8s.io
│     └ ingresses verbs=[get list create update delete] ✔
└ RoleBinding/app-package-and-pkgi-install-rolebinding (run-ns)
  └ Role/app-package-and-pkgi-install-role (run-ns)
    ├ core.k8s.io
    │ └ secrets verbs=[get list create update delete patch] ✔
    ├ data.packaging.carvel.dev
    │ └ packages verbs=[get list create update delete patch] ✔
    └ packaging.carvel.dev
      └ packageinstalls verbs=[get list create update delete patch] ✔
```

To install this package on a run cluster:

```
---
apiVersion: kappctrl.k14s.io/v1alpha1
kind: App
metadata:
 name: mypackage
 namespace: run-ns
spec:
 cluster:
   namespace: run-ns
 fetch:
 - git:
     url: https://github.com/garethjevans-test/cartov2-gitops
     ref: origin/main
     subPath: mypackage.example.com/packages
 - git:
     url: https://github.com/garethjevans-test/cartov2-gitops
     ref: origin/main
     subPath: mypackage.example.com/staging
 template:
 - ytt: {}

  deploy:
 - kapp:
     intoNs: run-ns
     rawOptions: ["--dangerous-allow-empty-list-of-resources=true"]
```
