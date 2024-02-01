# argocd-demo-install

## Bootstrapping

- Create a dir to work in
```
$ mkdir -p charts/argo-cd
```

- Make the Chart in `charts/argo-cd/Chart.yaml`

- Override the chart values with `charts/argo-cd/values.yaml`

- Make the Chart lock and the tgz necessary for the bootstrap installation
```
$ helm repo add argo-cd https://argoproj.github.io/argo-helm
$ helm dep update charts/argo-cd/
```

- Exclude that from repo
```
$ echo "charts/**/charts" >> .gitignore
```

- Create a namespace for argocd
```
$ kubectl create ns myargo
```

- Bootstrap: First installation manual
```
$ helm install argo-cd charts/argo-cd/ -n argocd
```

- Port-forward to access the web ui
This is just for demos. In a real env, hook it to an ALB, give it SSL/TLS termination, and set DNS for it
```
$ kubectl port-forward svc/argo-cd-argocd-server 8080:443 -n argocd
```

- Username is admin. Get password:
```
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Have ArgoCD manage itself

- Now we prepare the root-app
```
$ mkdir -p charts/root-app/templates
$ touch charts/root-app/values.yaml
```

- We make the chart: `charts/root-app/Chart.yaml`

- We make the app manifest for our root app in `charts/root-app/templates/root-app.yaml`

- The first time we manually apply the manifest. Later it will be automatic
```
$ helm template charts/root-app/ | kubectl apply -f -
```

- In the Web ui we verify that the root-app was created

- We make a manifest for argocd so that it can manage itself: `charts/root-app/templates/argo-cd.yaml`

- In the web ui we verify that argocd now appears as an app

## Use ArgoCD

- Now we can add all the apps we want to `charts/root-app/templates/`

![Screenshot of the final result](/docs/assets/images/final_result.png)

- The apps in the screenshot above are from: https://github.com/k-candidate/hello-k8s and https://github.com/k-candidate/hello-prod

## TO DO:
- [ ] Project templates to group apps depending on teams, repos, destinations (cluster/ns), object restriction, roles (rbac).
- [ ] Use a policy engine like OPA Gatekeeper (does not allow resource mutation) or Kyverno (does allow for mutation, but still incubating)
- [x] Add KEDA for Autoscaling.
- [x] Add KEDA HTTP Add-on.
- [ ] Add Prometheus. WIP...
- [ ] Add Grafana.
- [ ] Add OpenCost.
- [ ] Migrate to a cluster instead of Minikube as I already hit the ceiling (of default minikube) after adding KEDA, and Prometheus.
- [ ] Use HA for argocd. Requires 3 nodes.
