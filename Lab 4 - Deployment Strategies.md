## Deployment Strategy
---
#### Self Exercise 
### Task 1: Recreate Strategy in Kubernetes 
```
vi recreate.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dep2
  name: dep2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: dep2
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: dep2
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
```
```
kubectl apply -f recreate.yaml
```
Add a watch on the pods in a new tab
```
kubectl get po -w
```
Set a new image for the deployment
```
kubectl set image deploy dep2 nginx=nginx:latest --record
```
Check how the pods are getting deleted and recreated. 


Check the rollout history
```
kubectl rollout history deployment dep2
```


Cross check if the image has been updated by executing the below command
```
kubectl describe deployments.apps dep2
```
Now lets rollback. In the below command  if no revison number is given, it rollbacks to the immediate previous one.
```
kubectl rollout undo deployment dep2 --to-revision 1           
```
Check the history
```
kubectl rollout history deployment dep2
```



### Task 2: Rolling Update in Kubernetes 

```
vi dep.yaml 
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: dep1
  name: dep1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: dep1
  strategy: {}
  template:
    metadata:
      labels:
        app: dep1
    spec:
      containers:
      - image: nginx
        name: nginx
        ports:
        - containerPort: 80
```

Note the strategy.Since nothing is given the default strategy comes into place which is the rolling update.

Apply the yaml file
```
kubectl apply -f dep.yaml
```
```
kubectl get deployments.apps,pod,rs
```

Note the Statgey and events by describing the deployment
```
kubectl describe deployments.apps dep1
```
Add a watch on the pods in a new tab
```
kubectl get po -w
```

Set a new image for the deployment
```
kubectl set image deploy dep1 nginx=nginx:latest --record
```
Check how the pods are getting deleted and recreated. 


Cross check if the image has been updated by executing the below command
```
kubectl describe deployments.apps dep1
```
To check the rollout history. The below command shows history of 10 versions.
```
kubectl rollout history deploy dep1
```

To Rollback
```
kubectl rollout undo deploy dep1 --to-revision 1
```
To check the history of a particular revision
```
kubectl rollout history deploy dep1 --revision=<revision-number>
```

---

### Task 3: Blue/Green Deployment in Kubernetes 
```
vi web-blue.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-blue
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: web-blue
    spec:
      containers:
      - image: mandarct/web-blue:v1
        name: web-blue
        ports:
        - containerPort: 80
          protocol: TCP
```
```		 
kubectl apply -f web-blue.yaml
```
```
kubectl get deploy,po
```


Now create NodePort service to access application
```		 
vi svc-web.yaml
```
```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-web
spec:
  ports:
  - port: 80        # Service port(internal)
    protocol: TCP
    targetPort: 80  # Container port
    nodePort: 32123 # Range from 30000 to 32767 . This is the external port number
  selector:
    app: web-blue
  type: NodePort

```
```	 
kubectl apply -f svc-web.yaml
```
```
kubectl get svc,ep
```

#### Access you application
http://workernodeip:32123
![image](https://github.com/user-attachments/assets/3620566e-51b1-48f6-9ae2-6308cbff5b1a)

![image](https://github.com/user-attachments/assets/ff04e08d-6848-467e-a863-9d923f8aad37)


Create another deployment (Green)
```
vi web-green.yaml
```
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-green
  strategy:
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: web-green
    spec:
      containers:
      - image: mandarct/web-green:v1
        name: web-green
        ports:
        - containerPort: 80
          protocol: TCP
```
```
kubectl apply -f web-green.yaml
```
```
kubectl get deployment,po
```


Access this application using same service that we created previously, by changing the selector in the Service yaml file.
![image](https://github.com/user-attachments/assets/5b825e53-4790-486e-b6ab-cbbf417df274)

Replace Selector `web-blue` by `web-green`
```	 
kubectl replace -f svc-web.yaml --force
```
Access you application on the port 32123

![image](https://github.com/user-attachments/assets/ac5ecd4b-098f-40b5-be92-660582f8bc9f)


### Task 4: Canary Deployment in Kubernetes 

Service and deployment should have a common label.
Replace selector->matchlabel & template->labels with `type: web-app` in the yaml file of service and both the deployments and replace all

```
kubectl replace -f web-green.yaml --force
```
```
kubectl replace -f web-blue.yaml --force
```
```
kubectl replace -f svc-web.yaml --force
```

Check all obejcts
```
kubectl get po,svc,ep
```

Describe the endpoint. You would notice all 6 pods Ip added to the end-point
```
kubectl describe ep svc-web
```

Access the application on port 32123. Refresh the page a few times. Sometime it will show Blue, other time it will show green. 







