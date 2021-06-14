### Instalación de Fluxcd

~~~
sudo curl -sL https://toolkit.fluxcd.io/install.sh | sudo bash
~~~

### Para ver su versión

~~~
flux --version
~~~

### Ver si cumple requisitos el cúster

~~~
flux check --pre
~~~

### Añadimos token de Github y usuario

~~~
export GITHUB_TOKEN=**************************
export GITHUB_USER=acabe10
~~~

### Creamos repositorio

~~~
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux-demo \
  --branch=main \
  --path=./clusters/demo \
  --personal \
  --private=false \
  --interval=1m
~~~

### Obtenemos namespaces y pods de flux-system

{
  kubectl get namespaces
  echo
  kubectl get pods --namespace flux-system
}

---------- Clonamos repositorio ----------

git clone git@github.com:acabe10/flux-demo.git
tree -L 4

---------- Creamos carpeta para demo ----------

mkdir -p ./clusters/demo/ejemplo1
tree -L 4

---------- Creamos namespace ejemplo1 ----------

cat <<EOF > ./clusters/demo/ejemplo1/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: ejemplo1
EOF

---------- Observamos en tiempo real los namespaces del clúster ----------

watch -n1 kubectl get ns

---------- Subimos cambios a github ----------

{
  git add .
  git commit -m 'Añado namespace ejemplo1'
  git push origin main
}

---------- Forzamos reconciliación ----------

flux reconcile kustomization flux-system --with-source

---------- Creamos nginx de ejemplo ----------

cat <<EOF > ./clusters/demo/ejemplo1/nginx.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: ejemplo1
  labels:
    app: nginx
    service: web
spec:
  containers:
    - image:  acabe10/web-prueba:v0.0.1
      name:  nginx
EOF

---------- Vemos componentes clúster en tiempo real ----------

watch -n1 kubectl get all --namespace=ejemplo1

---------- Subimos a git ----------

{
  git add .
  git commit -m 'Subimos nginx de ejemplo'
  git push origin main
}

flux reconcile kustomization flux-system --with-source

---------- Redirigimos puerto para poder ver el nginx ----------

kubectl port-forward nginx 8080:80 --namespace=ejemplo1
curl localhost:8080

---------- Borramos para ver que se crea de nuevo ----------

kubectl delete pod/nginx --namespace=ejemplo1
flux reconcile kustomization flux-system --with-source

---------- Desinstalamos flux ----------

flux uninstall

---------- Comprobamos que el namespace se borra ----------

NOTA!: Podriamos borrar el repositorio antes de eliminar flux

kubectl get ns

---------- Volvemos a sincronizar con repositorio ----------

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux-demo \
  --branch=main \
  --path=./clusters/demo \
  --personal \
  --private=false \
  --interval=1m

---------- Git pull ----------

git pull
