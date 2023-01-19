## Adding a simple app + a load balancer

Sample deployment of a LB [here](https://docs.aws.amazon.com/eks/latest/userguide/sample-deployment.html)
Use the files shown in the link above.
eks-sample-deployment.yaml	
eks-sample-service.yaml

For convenience Here are the steps to deploy this app.

```
kubectl create namespace eks-sample-app
```

create a yaml file called eks-sample-deployment.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: eks-sample-linux-deployment
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: eks-sample-linux-app
  template:
    metadata:
      labels:
        app: eks-sample-linux-app
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values:
                - amd64
                - arm64
      containers:
      - name: nginx
        image: public.ecr.aws/nginx/nginx:1.21
        ports:
        - name: http
          containerPort: 80
        imagePullPolicy: IfNotPresent
      nodeSelector:
        kubernetes.io/os: linux
```

Deploy Yaml file
```
kubectl apply -f eks-sample-deployment.yaml
```
create eks-sample-service.yaml file.
```
apiVersion: v1
kind: Service
metadata:
  name: eks-sample-linux-service
  namespace: eks-sample-app
  labels:
    app: eks-sample-linux-app
spec:
  selector:
    app: eks-sample-linux-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

Now that we have it setup, lets expose it to the world.
```
kubectl expose deployment eks-sample-deployment --type=LoadBalancer --name=eks-sample-linux-service
```

you can check what that looks like here:
```
kubectl -n eks-sample-app describe service eks-sample-linux-service
```
can you tell where to go to access the newly built nginx server?

The nice thing about this is that it automatically configures an aws LB endpoint that can be used to access this without needing to configure it through the AWS API.

