# Kubernetes kind
kind is a tool for running local Kubernetes clusters using Docker container “nodes”.
kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI

## Install kind using go modules
```
go install sigs.k8s.io/kind@v0.11.1
```
## Create a simple kubernetes cluster
```
kind create cluster
```
## Delete kubernetes cluster
```
kind delete cluster
```
## Create your kubernetes cluster with ingress
Create a kind cluster with extraPortMappings and node-labels.

extraPortMappings allow the local host to make requests to the Ingress controller over ports 80/443
node-labels only allow the ingress controller to run on a specific node(s) matching the label selector
```
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

## Create nginx ingress
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

## Install kubernetes dashboard
Add kubernetes-dashboard repository
```
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
```

## Export the kubernetes-dashboard helm values so we can update them.
```
helm show values kubernetes-dashboard/kubernetes-dashboard > dashboard-values.yaml
```

Update the dashboard-values.yaml file to enable the ingress ( example below )
```ingress:
  enabled: true
  annotations:
    kubernetes.io/ingress.class: nginx
    nginx.ingress.kubernetes.io/backend-protocol: "HTTPS"
  hosts:
    - kubernetes-dashboard.domain.com
```

Deploy a Helm Release named "dashboard" using the kubernetes-dashboard chart
```
helm install dashboard kubernetes-dashboard/kubernetes-dashboard -f dashboard-values.yaml
```

## Get kubernetes dashboard secure token
```
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | awk '/^deployment-controller-token-/{print $1}') | awk '$1=="token:"{print $2}'
```

## Access the kubernetes dashboard
Set your hosts file to the domain specified above to point to your laptop's IP.
You should now be able to access the dashboard by going to the host you configure in the previous step
```
https://kubernetes-dashboard.domain.com
```

At this point you should have a fully working bubernetes cluster using kind.
