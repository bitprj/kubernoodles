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

```
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)                      AGE
argocd-dex-server       ClusterIP      10.100.133.80    <none>                                                                   5556/TCP,5557/TCP,5558/TCP   2d13h
argocd-metrics          ClusterIP      10.100.137.12    <none>                                                                   8082/TCP                     2d13h
argocd-redis            ClusterIP      10.100.220.82    <none>                                                                   6379/TCP                     2d13h
argocd-repo-server      ClusterIP      10.100.226.5     <none>                                                                   8081/TCP,8084/TCP            2d13h
argocd-server           LoadBalancer   10.100.251.172   xxxxxxxxxxxxxxxxx-xxxxxxxxxxxx.us-west-2.elb.amazonaws.com   80:32440/TCP,443:30601/TCP   2d13h
argocd-server-metrics   ClusterIP      10.100.201.179   <none>                                                                   8083/TCP                     2d13h
```

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