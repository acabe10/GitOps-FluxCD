### Instalamos componentes para actualizar imágenes

~~~
flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=flux-demo \
  --branch=main \
  --path=./clusters/demo \
  --personal \
  --private=false \
  --interval=1m \
  --read-write-key \
  --components-extra=image-reflector-controller,image-automation-controller
~~~

### Comprobamos que se ha creado

~~~
{
  kubectl get namespaces
  echo
  kubectl get pods --namespace flux-system
}
~~~

### Build de imagen a subir

~~~
FROM debian:buster-slim
    MAINTAINER Alejandro Cabezas "alejandrocabezab@gmail.com"
    RUN apt update && apt install -y apache2
    COPY index.html /var/www/html/index.html
    ENTRYPOINT ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
~~~

Creación de imágenes para dockerhub:

~~~
docker build -t acabe10/web-prueba:0.0.1 .
docker push acabe10/web-prueba:0.0.1
~~~

~~~
docker build -t acabe10/web-prueba:0.0.2 .
docker push acabe10/web-prueba:0.0.2
~~~

### Obtengo versión actual imagen

~~~
kubectl get pod nginx -o jsonpath="{..image}" --namespace ejemplo1
~~~

### Creamos directorio para crear los objetos

~~~
mkdir -p clusters/demo/automation
~~~

### Creamos objeto ImageRepository para realizar el seguimiento

~~~
flux create image repository web-prueba \
  --image=acabe10/web-prueba \
  --interval=1m \
  --namespace=flux-system \
  --export > clusters/demo/automation/web-prueba-registry.yaml
~~~

### Subimos a GIT

~~~
{
  git add .
  git commit -m 'Add imagerepository web-prueba'
  git push origin main
}
~~~

### Forzamos la reconciliación

~~~
flux reconcile kustomization flux-system --with-source
~~~

### Comprobamos objeto creado

~~~
flux get image repository --all-namespaces
~~~

### Creamos el objeto imagepolicy para aplicar el criterio

~~~
flux create image policy web-prueba \
  --namespace=flux-system \
  --image-ref=web-prueba \
  --select-semver='>=0.0.1 <2.0.0' \
  --export > clusters/demo/automation/web-prueba-policy.yaml
~~~

### Subimos a GIT

~~~
{
  git add .
  git commit -m 'Add web-prueba image policy'
  git push origin main
}
~~~

### Forzamos la reconciliación

~~~
flux reconcile kustomization flux-system --with-source
~~~

### Obtenemos los objetos con la política aplicada

~~~
flux get image policy --all-namespaces
~~~

### Decimos a flux qué se va a actualizar

~~~
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
    - image:  acabe10/web-prueba:v0.0.1  # {"\$imagepolicy": "flux-system:web-prueba"}
      name:  nginx
EOF
~~~

### Creamos objeto ImageUpdateAutomation

~~~
flux create image update web-prueba \
  --namespace=flux-system \
  --git-repo-ref=flux-system \
  --checkout-branch=main \
  --push-branch=main \
  --author-name=fluxbot \
  --author-email=fluxbot@web-prueba.com \
  --commit-template="{{ range .Updated.Images -}}
[demo] Automated image update **{{ \$.AutomationObject }}** to **{{ .Identifier }}**
{{ end -}}
Automation name: {{ .AutomationObject }}

Files:
{{ range \$filename, \$_ := .Updated.Files -}}
- {{ \$filename }}
{{ end -}}

Objects:
{{ range \$resource, \$_ := .Updated.Objects -}}
- {{ \$resource.Kind }} {{ \$resource.Name }}
{{ end -}}

Images:
{{ range .Updated.Images -}}
- {{.}}
{{ end -}}" \
  --export > clusters/demo/automation/web-prueba-automation.yaml
~~~

### Subimos a GIT

~~~
{
  git add .
  git commit -m 'Add ImageUpdateAutomation web-prueba'
  git push origin main
}
~~~

### Forzamos reconciliación

~~~
flux reconcile kustomization flux-system --with-source
~~~

### Obtenemos objetos update images

~~~
flux get image update --all-namespaces
~~~

### Obtengo versión actual imagen


~~~
{
  flux get image policy --all-namespaces
  echo
  kubectl get pod nginx -o jsonpath="{..image}" --namespace ejemplo1
}
~~~