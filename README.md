# argocd-demo-install

Create a dir to work in
$ mkdir -p charts/argo-cd

Make the Chart in charts/argo-cd/Chart.yaml

Override the chart values with charts/argo-cd/values.yaml

Make the Chart lock and the tgz necessary for the bootstrap installation
$ helm repo add argo-cd https://argoproj.github.io/argo-helm
$ helm dep update charts/argo-cd/

Exclude that from repo
$ echo "charts/**/charts" >> .gitignore

Create a namespace for argocd
$ kubectl create ns myargo

Bootstrap: First installation manual
$ helm install argo-cd charts/argo-cd/ -n argocd

Port-forward to access the web ui
$ kubectl port-forward svc/argo-cd-argocd-server 8080:443 -n argocd

Username is admin. Get password:
$ kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

Now we prepare the root-app
$ mkdir -p charts/root-app/templates
$ touch charts/root-app/values.yaml

We make the chart: charts/root-app/Chart.yaml

We make the app manifest for our root app in charts/root-app/templates/root-app.yaml

The first time we manually apply the manifest. Later it will be automatic
$ helm template charts/root-app/ | kubectl apply -f -

In the Web ui we verify that the root-app was created

We make a manifest for argocd so that it can manage itself: charts/root-app/templates/argo-cd.yaml

In the web ui we verify that argocd now appears as an app

Now we can add all the apps we want to charts/root-app/templates/



Things to do differently in a real env:
- Use HA for argocd.
- Project templates to group apps depending on teams, repos, destinations (cluster/ns), objects to restrict, roles (rbac).

