### Creamos carpeta para demo

~~~
mkdir -p ./clusters/demo/ejemplo2
~~~

### Creamos namespace ejemplo2

~~~
cat <<EOF > ./clusters/demo/ejemplo2/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ejemplo2
EOF
~~~

### Subimos cambios a github

~~~
{
  git add .
  git commit -m 'Añado namespace ejemplo1'
  git push origin main
}
~~~

### Creamos componentes necesarios

~~~
cat <<EOF > ./clusters/demo/ejemplo2/ingress.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: ingress-letschat
  namespace: ejemplo2
spec:
  rules:
  - host: www.letschat.com
    http:
      paths:
      - path: "/"
        backend:
          serviceName: letschat
          servicePort: 8080
EOF
~~~
~~~
cat <<EOF > ./clusters/demo/ejemplo2/letschat-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: letschat
  labels:
    name: letschat
  namespace: ejemplo2
spec:
  replicas: 1 
  selector:
    matchLabels:
      name: letschat
  template:
    metadata:
      labels:
        name: letschat
    spec:
      containers:
      - name: letschat
        image: sdelements/lets-chat
        ports:
          - name: http-server
            containerPort: 8080
EOF
~~~
~~~
cat <<EOF > ./clusters/demo/ejemplo2/letschat-srv.yaml
apiVersion: v1
kind: Service
metadata:
  name: letschat
  namespace: ejemplo2
spec:
  type: NodePort
  ports:
  - name: http
    port: 8080
    targetPort: http-server
  selector:
    name: letschat
EOF
~~~
~~~
cat <<EOF > ./clusters/demo/ejemplo2/mongo-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
  labels:
    name: mongo
  namespace: ejemplo2
spec:
  replicas: 1
  selector:
    matchLabels:
      name: mongo
  template:
    metadata:
      labels:
        name: mongo
    spec:
      containers:
      - name: mongo
        image: mongo
        ports:
          - name: mongo
            containerPort: 27017
EOF
~~~
~~~
cat <<EOF > ./clusters/demo/ejemplo2/mongo-srv.yaml
apiVersion: v1
kind: Service
metadata:
  name: mongo
  namespace: ejemplo2
spec:
  ports:
  - name: mongo
    port: 27017
    targetPort: mongo
  selector:
    name: mongo
EOF
~~~

### Comprobamos creacción

~~~
tree -L 4
~~~

### Vemos componentes clúster en tiempo real

~~~
watch -n1 kubectl get all --namespace=ejemplo2
~~~

### Subimos a git

~~~
{
  git add .
  git commit -m 'Subimos ejemplo2'
  git push origin main
}
~~~

### Forzamos reconciliación

~~~
flux reconcile kustomization flux-system --with-source
~~~

### Editamos /etc/hosts

~~~
sudo echo "localhost www.letschat.com" >> /etc/hosts
~~~