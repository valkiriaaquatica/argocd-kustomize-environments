## Kustomize + Argo CD Multi-Environment Cluster Bootstrapping

This repo uses `kustomize build --enable-helm --load-restrictor=LoadRestrictionsNone` to render Helm charts inside Kustomize overlays.
This an effective way of **bootsrapping** you different clusters in different environemtns with similar configurations and not repeting too many files.

It is think on having 3 different clusters (one per environemt) and applying each applicationset in the folder /applicationsets in each cluster, thinking of 1 ArgoCD * 1 Cluster(environment). If you are using 1 ArgoCD who controlls all the environments, you will need just to change the name in the applications inside the generators of each applicationset to not have the same apps name.

### Structure

* `applicationsets/`: Argo CD `ApplicationSet` manifests (`dev.yaml`, `stg.yaml`, `prod.yaml`).
* `apps/`:

  * `cert-manager/`: Helm chart with base config + per-environment overrides.
  * `istiod/`: Helm chart with shared values, version changes per environment.

---

### App Logic

#### `cert-manager`

* Uses **value merging**:

  * `base/values.yaml`: shared defaults
  * `overlays/<env>/values.yaml`: overrides per environment

#### `istiod`

* Uses a **single shared `values.yaml`**
* Only the **chart version** changes per environment.

---

### Important for Argo CD

To make this work, Argo CD must allow loading values from outside the overlay directory.
You must set this in the `argocd-cm` ConfigMap:

```yaml
data:
  kustomize.buildOptions: --enable-helm --load-restrictor LoadRestrictionsNone
```

### Create the Applications
To deploy to a specific environment, simply apply the corresponding `ApplicationSet` file in each cluster:

```bash
kubectl apply -f applicationsets/dev.yaml     # for the dev cluster
kubectl apply -f applicationsets/stg.yaml     # for the staging cluster
kubectl apply -f applicationsets/prod.yaml    # for the production cluster
```


Reference: [Argo CD Kustomize Guide](https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/)
