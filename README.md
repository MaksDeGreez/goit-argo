# goit-argo

GitOps repository for the GoIT MLOps course. Argo CD watches this repository
and keeps the Kubernetes cluster in the state described here. Nothing in this
repository is applied by hand.

Argo CD itself is installed by Terraform, in the `goit-mlops` repository,
branch `lesson-7`, folder `lesson-7/terraform/argocd/`.

## Layout

```
namespace/
├── application/          -> Argo CD Application "application"
│   ├── ns.yaml           namespace
│   ├── demo-nginx.yaml   demo Deployment + Service
└── infra-tools/          -> Argo CD Application "infra-tools"
    └── ns.yaml           namespace where Argo CD runs
```

One `ApplicationSet` watches the pattern `namespace/*`. Every folder that
matches becomes one Argo CD Application:

* the Application is named after the folder, for example `application`;
* it deploys everything inside that folder;
* it deploys into the namespace with the same name.

So adding a new folder under `namespace/` is enough to get a new Application.
No change to the Terraform code and no new Argo CD object are needed.

## Sync settings

The ApplicationSet turns on automatic sync:

* `prune: true` — an object deleted from Git is deleted from the cluster;
* `selfHeal: true` — an object changed by hand in the cluster is put back;
* `CreateNamespace=true` — the namespace is created if it is missing.

Argo CD checks this repository every 60 seconds, so a push shows up in the
cluster in about a minute.

## Checking a change

```bash
kubectl get applications -n infra-tools
kubectl get deploy,pods -n application
kubectl -n application port-forward deployment/demo-nginx 8081:80
```

Then open <http://localhost:8081>.
