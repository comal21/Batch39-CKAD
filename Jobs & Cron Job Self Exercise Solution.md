apiVersion: batch/v1
kind: Job
metadata:
  name: ckad-job
spec:
  template:
    spec:
      containers:
      - name: job
        image: busybox
        command: ["echo", "CKAD Job Done"]
      restartPolicy: Never

kubectl get job ckad-job
kubectl get pods 
kubectl logs ckad-job
-------------------------------------------------
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ckad-cronjob
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: echo
            image: busybox
            command: ["echo", "Time for CKAD!"]
          restartPolicy: OnFailure

kubectl get cronjob ckad-job
kubectl get pods 
kubectl logs <podname>
--------------------------------------------------------
kubectl delete jobs --all
kubectl delete cronjobs --all
