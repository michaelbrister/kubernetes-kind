# Argo-cd up and running on Kubernetes kind

Argo CD is a declarative, GitOps continuous delivery tool for Kubernetes.
To get Argo-CD deployed on our Kubernetes cluster we will be utilizing the official helm chart

## Create the namespace where we will be installing argo-cd
```
kubectl create namespace argocd
```

## Add helm chart to helm repo
```
helm repo add argo https://argoproj.github.io/argo-helm
```

With that done we'll grab the values file so we can edit it to suite our kubernetes environment
```
helm show values argo/argo-cd > argocd-values.yaml
```

## Edit vaules.yaml file
With the values file created we can edit make some edits for our test environment.
Below is the ingress portion of the argocd-values.yaml file
```
  ingress:
    # -- Enable an ingress resource for the Argo CD server
    enabled: true
    # -- Additional ingress annotations
    annotations:
      kubernetes.io/ingress.class: nginx
    # -- Additional ingress labels
    labels: {}
    # -- Defines which ingress controller will implement the resource
    ingressClassName: ""

    # -- List of ingress hosts
    ## Argo Ingress.
    ## Hostnames must be provided if Ingress is enabled.
    ## Secrets must be manually created in the namespace
    hosts:
      - argocd.example.com

    # -- List of ingress paths
    paths:
      - /
```

For a test environment we'll need to enable insecure access as well
```
   # -- Additional command line arguments to pass to Argo CD server
    extraArgs:-
    - --insecure
 ```
 
With insecure access enabled and our ingress controller configured we can apply the helm chart

```
helm install -n argocd argocd argo/argo-cd -f argocd-values.yaml
```

With argo-cd install  we can now access the web interface at argocd.example.com


## Get default argo-cd login credentials
Default Username: admin
# Get the default argo password by running the command
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

Argo-cd should now be successfully installed and we can access it at the URL we specified with the default username and default password from the installed secret. 
