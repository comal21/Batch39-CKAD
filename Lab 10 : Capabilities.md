### Task : Set capabilities for a Container
With Linux capabilities, you can grant certain privileges to a process without granting all the privileges of the root user. To add or remove Linux capabilities for a Container, include the capabilities field in the securityContext section of the Container manifest.

Lets check, what happens when you don't include a capabilities field.
```
vi security-context-3.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod3
spec:
  containers:
  - name: sec-ctx-3
    image: gcr.io/google-samples/node-hello:1.0
```
Create the Pod:
```
kubectl apply -f security-context-3.yaml
```
Verify that the Pod's Container is running:
```
kubectl get pod security-context-pod3
```
Get a shell into the running Container:
```
kubectl exec -it security-context-pod3 -- sh
```
In your shell, list the running processes:
```
ps aux
```
In your shell, view the status for process 1:
```
cd /proc/1 && cat status
```
The output shows the capabilities bitmap for the process. Make a note of the capabilities bitmap. 

Inside the pod, you can try to manipulate network-related settings, which require NET_ADMIN capability.
```
ip link set dev eth0 down
```
If you're able to execute this command without any errors, it indicates that the NET_ADMIN capability is working, allowing you to manipulate network interfaces.

You can also try to manipulate system time, which requires SYS_TIME capability.
```
date -s "2024-02-07 12:00:00"
```
If you're able to execute this command without any errors, it indicates that the SYS_TIME capability is working, allowing you to manipulate system time.
```
exit
```
Now, run a Container that is the same as the preceding container, except that it has additional capabilities set.

```
vi security-context-4.yaml
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: security-context-pod4
spec:
  containers:
  - name: sec-ctx-4
    image: gcr.io/google-samples/node-hello:1.0
    securityContext:
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
```
```
kubectl apply -f security-context-4.yaml
```
```
kubectl exec -it security-context-pod4 -- sh
```
In your shell, view the capabilities for process 1:
```
cd /proc/1 && cat status
```
Compare the capabilities of the two Containers:

In the capability bitmap of the first container, bits 12 and 25 are clear. In the second container, bits 12 and 25 are set. Bit 12 is CAP_NET_ADMIN, and bit 25 is CAP_SYS_TIME. 

Note: Linux capability constants have the form CAP_XXX. But when you list capabilities in your container manifest, you must omit the CAP_ portion of the constant. For example, to add CAP_SYS_TIME, include SYS_TIME in your list of capabilities.

Inside the pod, you can try to manipulate network-related settings, which require NET_ADMIN capability.
```
ip link set dev eth0 down
```
If you're able to execute this command without any errors, it indicates that the NET_ADMIN capability is working, allowing you to manipulate network interfaces.

You can also try to manipulate system time, which requires SYS_TIME capability.
```
date -s "2024-02-07 12:00:00"
```
If you're able to execute this command without any errors, it indicates that the SYS_TIME capability is working, allowing you to manipulate system time.
```
exit
```

