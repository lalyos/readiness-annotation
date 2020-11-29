This repo demonstarates how to implement a readiness probe based on an annotation?
This just for demonstrational purpose, but use it in production if you want ;)

## Demo

Deploy the app
```
kubectl apply -f https://raw.githubusercontent.com/lalyos/readiness-annotation/master/back-depl.yaml
```

switch the first pod (replication is 2) to `ready=false`
```
k annotate pod $(k get po -l app=myapp -ojsonpath='{.items[0].metadata.name}')  --overwrite ready=false
```

swith all pods to `ready=true`
```
k annotate pod -l app=myapp --overwrite ready=true # TRUE
```

## tl;dr

The trick is to map a specific annotation `ready` into the filesystem, using the downwardsApi
```
    spec:
      volumes:
      - name: podinfo
        downwardAPI:
          defaultMode: 420
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.annotations['ready']
            path: ready
    ...
      containers:
      - name: myapp
        volumeMounts:
          - name: podinfo
            mountPath: /etc/podinfo
```

The readiness probe is an **exec** type, instead of the boring **httpGet**:
```
        readinessProbe:
            initialDelaySeconds: 2
            periodSeconds: 3
            failureThreshold: 1
            exec:
              command:
                - "sh"
                - "-c"
                - "grep true /etc/podinfo/ready"
```