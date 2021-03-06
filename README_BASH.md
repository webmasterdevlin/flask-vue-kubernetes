# Running Flask on Kubernetes

## Want to learn how to build this?

Check out the [post](https://testdriven.io/running-flask-on-kubernetes).

## Want to use this project?

### Docker

Build the images and spin up the containers:

```sh
$ docker-compose up -d --build
```

Run the migrations and seed the database:

```sh
$ docker-compose exec server python manage.py recreate_db
$ docker-compose exec server python manage.py seed_db
```

Test it out at:

1. [http://localhost:8080/](http://localhost:8080/)
1. [http://localhost:5001/books/ping](http://localhost:5001/books/ping)
1. [http://localhost:5001/books](http://localhost:5001/books)

### Kubernetes

#### Minikube

Install and run [Minikube](https://kubernetes.io/docs/setup/minikube/):

1. Install a [Hypervisor](https://kubernetes.io/docs/tasks/tools/install-minikube/#install-a-hypervisor) (like [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [HyperKit](https://github.com/moby/hyperkit)) to manage virtual machines
1. Install and Set Up [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) to deploy and manage apps on Kubernetes
1. Install [Minikube](https://github.com/kubernetes/minikube/releases)

Toggle hypervisor using Powershell run as admin:

```sh
$ bcdedit /set hypervisorlaunchtype off
```

Start the cluster:

```sh
    - OSX
$ minikube config set driver hyperkit
or
    - Windows
$ minikube config set driver hyperv
or
    - Linux
$ minikube config set driver virtualbox

$ minikube start
$ minikube dashboard
```

#### Volume

Create the volume:

```sh
$ kubectl apply -f ./kubernetes/persistent-volume.yml
```

Create the volume claim:

```sh
$ kubectl apply -f ./kubernetes/persistent-volume-claim.yml
```

#### Secrets

Create the secret object:

```sh
$ kubectl apply -f ./kubernetes/secret.yml
```

#### Postgres

Create deployment:

```sh
$ kubectl apply -f ./kubernetes/postgres-deployment.yml
```

Create the service:

```sh
$ kubectl apply -f ./kubernetes/postgres-service.yml
```

Create the database:

```sh
$ kubectl get pods
$ POSTGRES_POD_NAME=$(kubectl get pod -l service=postgres -o jsonpath="{.items[0].metadata.name}")
$ kubectl exec $POSTGRES_POD_NAME --stdin --tty -- createdb -U postgres books
```

#### Flask

Build and push the image to Docker Hub:

```sh
$ DOCKERHUB_NAME=webmasterdevlin
$ docker build -t $DOCKERHUB_NAME/flask-kubernetes ./services/server
$ docker push $DOCKERHUB_NAME/flask-kubernetes
```

> Make sure to replace `webmasterdevlin` with your Docker Hub namespace in the above commands as well as in _kubernetes/flask-deployment.yml_

Create the deployment:

```sh
$ kubectl apply -f ./kubernetes/flask-deployment.yml
```

Create the service:

```sh
$ kubectl apply -f ./kubernetes/flask-service.yml
```

Apply the migrations and seed the database:

```sh
$ kubectl get pods
$ FLASK_POD_NAME=$(kubectl get pod -l app=flask -o jsonpath="{.items[0].metadata.name}")
$ kubectl exec $FLASK_POD_NAME --stdin --tty -- python manage.py recreate_db
$ kubectl exec $FLASK_POD_NAME --stdin --tty -- python manage.py seed_db
```

#### Ingress

Enable and apply:

```sh
$ minikube addons enable ingress
$ kubectl apply -f ./kubernetes/minikube-ingress.yml
```

Add entry to _/etc/hosts_ file:

```
$ echo "$(minikube ip) hello.world" | sudo tee -a /etc/hosts
```

Note: hello.world entry should only be mapped once to an IP. You can edit /etc/hosts anytime

Try it out:

1. [http://hello.world/books/ping](http://hello.world/books/ping)
1. [http://hello.world/books](http://hello.world/books)

#### Vue

Build and push the image to Docker Hub:

```sh
$ docker build -t $DOCKERHUB_NAME/vue-kubernetes ./services/client -f ./services/client/Dockerfile-minikube

$ docker push $DOCKERHUB_NAME/vue-kubernetes
```

> Again, replace `webmasterdevlin` with your Docker Hub namespace in the above commands as well as in _kubernetes/vue-deployment.yml_

Create the deployment:

```sh
$ kubectl apply -f ./kubernetes/vue-deployment.yml
```

Create the service:

```sh
$ kubectl apply -f ./kubernetes/vue-service.yml
```

Try it out at [http://hello.world/](http://hello.world/).

Add another Flask Pod to the cluster

```sh
$ kubectl scale deployment flask --replicas=2
$ kubectl get deployments flask
```

Make a few requests to the service:

```sh
$ for ((i=1;i<=10;i++)); do curl http://hello.world/books/ping; done
```
