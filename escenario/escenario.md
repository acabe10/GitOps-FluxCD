En la máquina que vayamos a hacer la instalación, debemos hacer lo siguiente para poder instalar Minikube:

~~~
sudo apt update && sudo apt upgrade -y && sudo apt install docker.io -y && sudo usermod -aG docker "debian"
exit
~~~

Hemos usado Minikube para crear el clúster de Kubernetes, para instalarlo:

~~~
sudo apt install curl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
minikube start
~~~

Para instalar `kubectl`, necesario para la administración del clúster:

~~~
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
~~~