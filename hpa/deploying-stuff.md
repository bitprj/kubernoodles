# HPA (Horizontal Pod Autoscaling)

## Configuring ngnix

Name this YAML file `deployment-nginx.md` and place it in your `kube` folder:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-conf
  namespace: default
data:
  nginx.conf: |
    user nginx;
    worker_processes  3;
    error_log  /dev/stdout;
    events {
      worker_connections  10240;
    }
    http {
      access_log	/dev/stdout;
      server {
          listen       80;
          server_name  _;
          location / {
              root   html;
              index  index.html index.htm;
          }
      }
      include /etc/nginx/virtualhost/virtualhost.conf;
    }
  virtualhost.conf: | # where we say where to horizontal autoscale (manipulate's ip)
    upstream app {
      server manipulation-service:80; 
      keepalive 1024;
    }
    server {
      listen 80 default_server;
      access_log /dev/stdout;
      error_log /dev/stdout;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
      location /nginx_status {
        stub_status;
      }
    }
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /etc/nginx # mount nginx-conf volumn to /etc/nginx
          readOnly: true
          name: nginx-conf
        - mountPath: /var/log/nginx
          name: log
      volumes:
      - name: nginx-conf
        configMap:
          name: nginx-conf # place ConfigMap `nginx-conf` on /etc/nginx
          items:
            - key: nginx.conf
              path: nginx.conf
            - key: virtualhost.conf
              path: virtualhost/virtualhost.conf # dig directory
      - name: log
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  namespace: default
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
```

Notice this specific section:
```yaml
 virtualhost.conf: | # where we say where to horizontal autoscale (manipulate's ip)
    upstream app {
      server manipulation-service:80; 
      keepalive 1024;
    }
```
We placed `manipulation-service:80`, the address of the server we wish to autoscale, in this configuration.

## Configuring the sending server
In our case, the `fetch-service` will send requests to the manipulation service, so we'll make `nginx` run as the "middle man." The final step in configuring this is adding the `nginx` address in the fetch service's environment variables.
```
- name: MANIPULATE_ENDPOINT
    value: nginx:80
```

## Applying HPA to the Service
*Make sure to apply the new kube files and restart the deployments.*
Run the below command to start HPA, making sure to change `manipulation-service` to the name of the deployment that you want to autoscale.
```
kubectl autoscale deployment manipulation-service --cpu-percent=50 --min=1 --max=10
horizontalpodautoscaler.autoscaling/manipulation-service autoscaled
```

## Testing it out

```
➜  tinyhats-hpa git:(master) ✗ kubectl get deployment manipulation-service
NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
manipulation-service   1/1     1            1           26m
```
