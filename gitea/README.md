Gitea is a community managed lightweight code hosting solution written in Go.
First we'll be creating a dedicated namespace in which we will be installing gitea
https://docs.gitea.io/en-us/install-on-kubernetes/

```
kubectl create namespace gitea
```

### Add gitea helm chart
Next we will be adding the helm charts
```
helm repo add gitea-charts https://dl.gitea.io/charts/
```
### Export the gitea helm values so we can update them.
```
helm show values gitea-charts/gitea > gitea-values.yaml
```

### Create a generic nginx ingress
```
kubectl apply --filename https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```

### Edit gitea chart values file
Edit the gitea-values.yaml file to turn on ingress ( example ingress section shown below )
```
ingress:
  enabled: true 
  annotations: 
    kubernetes.io/ingress.class: public 
    # kubernetes.io/tls-acme: "true"
  hosts:
    - git.example.com
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - git.example.com
```
### Apply the updated gitea-values.yaml file
```
helm install -n gitea gitea gitea-charts/gitea -f gitea-values.yaml
```

### Check if gitea is running
```
kubectl get pods -n gitea
```

### Now that we have gitea running
Let's create an ingress controller so that we can access it.

First we need to get the name of the service we're adding to the ingress controller
```
kubectl get svc -n gitea
```

Apply the ingress-gitea.yaml file
```
kubectl apply -f ingress-gitea.yaml
```
