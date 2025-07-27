## Kustomize + Argo CD Multi-Environment Cluster Bootstrapping

This repository bootstraps multiple Kubernetes environments (dev, stg, prod) using `kustomize build --enable-helm --load-restrictor=LoadRestrictionsNone` along with Argo CD.

Each environment has a top-level `Application` (e.g., `applicationsets/dev-app/application.yaml`) that deploys an `ApplicationSet`. This `ApplicationSet` defines multiple apps, each tagged with a `sync-wave`, enabling controlled deployment order.

---

### Structure

* `applicationsets/dev-app/application.yaml`: Argo CD `Application` pointing to the `ApplicationSet` for the dev environment.
* `applicationsets/dev.yaml`: Defines the `ApplicationSet` with multiple apps using `sync-waves`.
* `apps/`: Application directories, each with a `base` and environment-specific `overlays`.

---

### Deploy to an Environment

Apply the `Application` for the desired environment:

```bash
kubectl apply -f applicationsets/dev-app/application.yaml
```

This deploys the `ApplicationSet`, which in turn creates individual applications in the order defined by `sync-waves`.

---

### Argo CD Configuration Required

Make sure Argo CD allows Kustomize to load values outside the overlay directory by setting this in the `argocd-cm` ConfigMap:

```yaml
data:
  kustomize.buildOptions: --enable-helm --load-restrictor LoadRestrictionsNone
```

---

### References

* [Argo CD Kustomize Guide](https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/)
* [Argo CD Sync Waves](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/)

