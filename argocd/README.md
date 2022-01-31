# Setup Argo

# Install
Use the approriate install manifest based on the targeted kubernetes platform.  If running on Openshift, use the install-openshift.yaml.  Otherwise use the install.yaml.

```
kubectl create namespace argocd
kubectl apply -f install.yaml -n argocd
```

# Modify Service Type
To reach the ArgoCD UI, you need to modify the default service type.  Either NodePort or LoadBalancer can be used depending on your capabilities.

```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```


# Get Initial Admin Password
Get the initial admin password.

```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d; echo
```
