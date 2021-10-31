# Automatic Deployment with Argo

## Deploying Argo
First, we create Argo's namspace and deploy everything it needs.
```
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/core-install.yaml
```

## Accessing the UI
To gain access to the UI, we can run this command to expose the `argocd-server` so we can access it.
```
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
```

Next, run `kubectl get services -n argocd` and copy paste the `External-IP` for `argocd-server`. Keep this URI handy!

To login, we will use `admin` as the username. To get the password, run:
```
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

## Time to login
Replace `<ARGOCD_SERVER>` with the URI you obtained in the previous step.
```
argocd login <ARGOCD_SERVER>
```

When prompted for username and password, input the correct values that we received. 

> If asked whether to continue, input `yes`

Finally, copy paste the URI into your browser to access the UI.