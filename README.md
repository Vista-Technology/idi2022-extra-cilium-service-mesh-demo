# IDI2022 extra contents: Cilium Service Mesh DEMO
In this guide all the steps to reproduce the DEMO in a local environment.

## Technical prerequisites

* Operating System: Linux with Kernel >= 4.9.17 or MacOS
* Docker [installed](https://docs.docker.com/get-docker/)
* kubectl >= 1.22 [installed](https://kubernetes.io/docs/tasks/tools/#kubectl)
* Minikube installed
* Helm installed
* Minimum 8GB RAM
### Install Minikube

Check the [Minikube start Guide](https://minikube.sigs.k8s.io/docs/start/) for instructions on how to install minikube on your system. If you are using the provided virtual machine minikube is already installed.
### Install helm

For a complete overview refer to the helm installation [website](https://helm.sh/docs/intro/install/). If you have helm 3 already installed you can skip this step.

Use your package manager (`apt`, `yum`, `brew` etc), download the [latest Release](https://github.com/helm/helm/releases) or use the following command to install [helm](https://helm.sh/docs/intro/install/) helm:

```bash
curl -s https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```
## Step 0: Clone the repository
```bash
git clone https://github.com/Vista-Technology/idi2022-extra-cilium-service-mesh-demo.git

cd idi2022-extra-cilium-service-mesh-demo/manifests
```
## Step 1: Start a cluser with Minikube and install Cilium
[![asciicast](https://asciinema.org/a/xVLXUpRLDindSTV3OcfATU98J.svg)](https://asciinema.org/a/xVLXUpRLDindSTV3OcfATU98J)

```bash
minikube start --network-plugin=cni --cni=false --kubernetes-version=1.23.1 -p servicemesh

cilium install --version -service-mesh:v1.11.0-beta.1 --config enable-envoy-config=true --kube-proxy-replacement=probe
```

## Step 2: Deploy two sample backends
[![asciicast](https://asciinema.org/a/mpNqIfvK5JuJGC6Ii7zUJmmhe.svg)](https://asciinema.org/a/mpNqIfvK5JuJGC6Ii7zUJmmhe)

```bash
kubectl apply -f backend1.yaml

kubectl apply -f backend2.yaml
```
## Step 3: Test backends connectivity
[![asciicast](https://asciinema.org/a/K575C1qAEXgyWRRuas9SIYr5j.svg)](https://asciinema.org/a/K575C1qAEXgyWRRuas9SIYr5j)

```bash
for i in {1..10}; do
  kubectl run --rm=true -it --image=curlimages/curl --restart=Never curl -- curl  http://backend-1:8080/private
done

for i in {1..10}; do
  kubectl run --rm=true -it --image=curlimages/curl --restart=Never curl -- curl  http://backend-2:8080/private
done

for i in {1..10}; do
  kubectl run --rm=true -it --image=curlimages/curl --restart=Never curl -- curl  http://backend-1:8080/public
done

for i in {1..10}; do
  kubectl run --rm=true -it --image=curlimages/curl --restart=Never curl -- curl  http://backend-2:8080/public
done
```
## Step 4: Apply some NetworkPolicy
[![asciicast](https://asciinema.org/a/RCTl5ZIirq8W3SnZ5ZkTiHfyw.svg)](https://asciinema.org/a/RCTl5ZIirq8W3SnZ5ZkTiHfyw)

```bash
kubectl apply -f l7-network-policy.yaml

for i in {1..10}; do
  kubectl run --rm=true -it --image=curlimages/curl --restart=Never curl -- curl  http://backend-1:8080/public
done
```
## Step 5: Configure and test Service Mesh L7 balancing feature
[![asciicast](https://asciinema.org/a/OJtKf1LSYF0trAt4ewymfg9kE.svg)](https://asciinema.org/a/OJtKf1LSYF0trAt4ewymfg9kE)

```bash
kubectl apply -f envoyconfig.yaml
 
for i in {1..10}; do
  kubectl run --rm=true -it --image=curlimages/curl --restart=Never curl -- curl  http://backend-1:8080/private
done
```