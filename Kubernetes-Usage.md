# Kubernetes Usage

## 0 Setup

### 0.1 Installation

* MacOS

    ```
    brew install kubernetes-cli
    ```
    
* Download [Docker-Edge](https://download.docker.com/mac/edge/Docker.dmg)

### 0.2 Config file

* `~/kube/config`
* Use `kubectl config view` to view the config file.

### 0.3 Example Dockerfile

```dockerfile
FROM ubuntu:16.04
MAINTAINER Qhan<qhan@ailabs.tw>
RUN apt-get update
RUN apt-get install -y \
    vim \
    tree \
    htop \
    python3-pip
WORKDIR /app
COPY src /app
CMD ["sleep", "infinity"]
```

* `src` has `test.py`

    ```python
    for i in range(10): print(i)
    ```

## 1 *Minikube Usage (skip)

* Start minikube 

    ```
    minikube start
    ```

* Set docker env

    ```
    eval $(minikube docker-env)
    ```

* Build image

    ```
    docker build -t test:0.0.2
    ```

* Run in minikube

    ```
    kubectl run test-pod --image=test:0.0.2 --image-pull-policy=Never python3 test.py
    ```

* Check that it's running

    ```
    kubectl get pods
    ```

## 2 Commands

### 2.1 Clusters

* List all clusters
    
    ```
    kubectl config get-contexts
    ```
    
* Show current selected cluster
    
    ```
    kubectl config current-context
    ```
    
* Select a cluster

    ```
    kubectl config use-context [context name]
    ```

### 2.2 Pods

* List all pods

    ```
    kubectl get pods
    ```

* **Apply a config file to the current cluster**

    ```
    kubectl apply -f [config file]
    ```

  * Config file example `ailabs.yaml` (see 3.1)

* Run a command in the specific container of a pod

    ```
    kubectl exec -it [pod name] -c [container name][cmd]
    ```

  * Example: 

      ```
      kubectl exec -it test-pod -c test-container python3 test.py
      ```

* Attach to a specific container of a pod

    ```
    kubectl attach [pod name] -c [container name]
    ```

* Create and run a particular image as a deployment

    ```
    kubectl run -it [--rm][pod name] --image=[registry]/[repository]:[tag] --image-pull-policy=[Never|Always|IfNotPresent][cmd]
    ```

* Delete a pod

    ```
    kubectl delete pod [pod name]
    ```

* Delete a deployment (if the pod can't be deleted)

    ```
    kubectl delete deploy [deploy name]
    ```

## 3 Config File Examples

### 3.1 `ailabs.yaml`([referenced dockerfile](#03-example-dockerfile))

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
    - name: test-container
      image: registry.corp.ailabs.tw/lab/test:0.0.2
      imagePullPolicy: Always
  imagePullSecrets:
    - name: ailabs-registry-secret
```

### 3.2 [`pod.yaml`](https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/)

```yaml
apiVersion: v1
 kind: Pod
 metadata:
   name: rss-site
   labels:
     app: web
 spec:
   containers:
     - name: front-end
       image: nginx
       ports:
         - containerPort: 80
     - name: rss-reader
       image: nickchase/rss-php-nginx:v1
       ports:
         - containerPort: 88
```

### 3.3 [`deployment.yaml`](https://www.mirantis.com/blog/introduction-to-yaml-creating-a-kubernetes-deployment/)

```yaml
apiVersion: extensions/v1beta1
 kind: Deployment
 metadata:
   name: rss-site
 spec:
   replicas: 2
   template:
     metadata:
       labels:
         app: web
     spec:
       containers:
         - name: front-end
           image: nginx
           ports:
             - containerPort: 80
         - name: rss-reader
           image: nickchase/rss-php-nginx:v1
           ports:
             - containerPort: 88
```

## 4 Ingress

* Kubernetes pods and external world are seperated, therefore we need Ingress to help us to forward outside ports to every pods' ports.

### 4.1 Setup Helm

* Install from homebrew

    ```
    brew install kubernetes-helm
    ```

* Install Tiller and deploy

    ```
    helm init
    ```

* Check if Tiller is running

    ```
    kubectl get pods --namespace kube-system
    ```

  * The output should be like this

        ```
        NAME                                         READY     STATUS    RESTARTS   AGE
        ...
        tiller-deploy-7ccf99cd64-6xzrw               1/1       Running   0          36m	
        ```

* Check if Helm is properly setup

    ```
    helm version
    ```

  * The output should be like this

        ```
        Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
        Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
        ```

* If want to reset Tiller, use these commands

    ```
    kubectl delete deployment tiller-deploy --namespace kube-system
    ```

    ```
    helm reset
    ```

  * And re-install Tiller again

    ```
    helm init
    ```

### 4.2 Install Ingress

* Install Ingress by Helm

    ```
    helm install stable/nginx-ingress
    ```

  * The output should have an example of yaml config for Ingress

        ```yaml
        NAME: nginx-ingress
        LAST DEPLOYED: Fri Oct 27 18:26:58 2017
        NAMESPACE: default
        STATUS: DEPLOYED
        ...
        An example Ingress that makes use of the controller:
        
        kind: Ingress
            metadata:
            annotations:
            kubernetes.io/ingress.class: nginx
            name: example
            namespace: foo
        ```

* Copy the line after `annotation:` and paste it onto your own yaml file.

    ```yaml
    kubernetes.io/ingress.class: nginx
    ```
