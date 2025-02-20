
# Services & Networking

## Create an Ingress using the hello service on port 5396.
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /hello
        pathType: Prefix
        backend:
          service:
            name: hello
            port:
              number: 5396
```

### Verify
```sh
kubectl get ingress
curl [Internal-IP]:port/hello
hello
```

## Modify a Pod and add port 80, also expose it with a ClusterIP service TCP in the port 80, additionally expose it using NodePort.
### Pod Definition
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp
spec:
  containers:
  - name: myapp
    image: alpine:latest
    command: ['sh', '-c', 'while true; do echo "logging" >> /opt/logs.txt; sleep 1; done']
    ports:
    - containerPort: 80
```

### Service Definition
```yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  selector:
    app: myapp
  ports:
    - protocol: TCP
      port: 80
```
```sh
kubectl apply -f myapp-service.yaml
kubectl expose service myapp-service --type=NodePort --port 80 --name=myapp-nodeport-service
```

## Create a NetworkPolicy in the echo namespace to allow traffic to the internal namespace on port 9200/tcp, ensuring that the echo namespace only accepts traffic on port 9200/tcp from the internal namespace.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-echo-to-internal
  namespace: echo
spec:
  podSelector: {}
  policyTypes:
  - Egress
  - Ingress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: internal
    ports:
    - protocol: TCP
      port: 9200
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: internal
    ports:
    - protocol: TCP
      port: 9200
```

    