### How do I find the join command for kubeadm on the master?

    kubeadm token create --print-join-command

#### Here is a bash script that automate this task

    read -p 'master ip address : ' ipaddr
    sha_token = "$(openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //')"
    token = "$(kubeadm token list | awk '{print $1}' | sed -n '2 p')"
    echo "kubeadm join $ipaddr:6443 --token=$token --discovery-token-ca-cert-hash sha256:$sha_token"

#
#### Check API Version of Kubernetes

    kubectl api-resources | grep pods


#### Continuously watching kubernetes all services

    watch kubectl get all -o wide

#### List of Kubernetes Nodes:

    kubectl get nodes
#### View cluster events
    kubectl get events

#### View the kubectl configuration
    kubectl config view

#

### Accessing Kubernetes Dashboard

#### Installing the Kubernetes Dashboard

    kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml 

#### List of Namespaces

    kubectl get services --all-namespaces

#### Check all the resources

    kubectl get all -n kubernetes-dashboard

#### Accessing the Kubernetes Dashboard from Outside

#### Easy Ways

#### Port Forwarding for kubernetes-dashboard

    kubectl port-forward -n kubernetes-dashboard service/kubernetes-dashboard 10443:443 --address 0.0.0.0

#### Browsing Kubernetes-Dashboard

#### https://192.168.0.89:10443

#### Another Ways

<b>We will have to change the type of service from ClusterIp to NodePort. So, give the following command to edit the service and make the following changes.</b>

    kubectl edit service/kubernetes-dashboard -n kubernetes-dashboard

<b>After: You can give the IP of your wish if 32321 is occupied</b>

<b>Now, check if the service was changed successfully by giving the following command:</b>

    kubectl get svc

<b>It will open the Kubernetes dashboard in the web browser.</b>

> kubectl proxy --address='0.0.0.0' --disable-filter=true <br>
> kubectl proxy --address='0.0.0.0' --accept-hosts='^*$' <br>
> kubectl proxy --address='0.0.0.0' --port=8001 --accept-hosts='^*$' <br>

    kubectl -n kubernetes-dashboard describe service kubernetes-dashboard
#

#

#### Deploy a simple Nginx POD

    kubectl create deployment nginx-web --image=nginx
####
    kubectl expose deployment nginx-web --type NodePort --port=80
####
    kubectl get deployment,pod,svc
#

#### Create Nginx POD using YAML file

    nano nginx.yaml
####
    apiVersion: v1 
    kind: Pod 
    metadata:
      name: nginx-pod 
    spec:
      containers:
      - name: nginx 
        image: nginx:latest 
        ports:
        - containerPort: 80 
####
    kubectl apply -f nginx.yaml

#

#### Kubectl Port-Forwarding Syntax

<b>Port-Forward Syntax</B>

    kubectl port-forward POD_NAME LOCAL_PORT:REMOTE_POD_PORT

<b>Command Description</b> 
> POD_NAME: Give our pod name. <br>
> LOCAL_PORT: Give a port which we access from the internet. <br>
> REMOTE_POD_PORT: Give our pods default port. <br>

<b>Example:</b>

    kubectl port-forward nginx-pod 8085:80 --address 0.0.0.0

#### Kubectl port-forward in background

    kubectl port-forward nginx-pod 8085:80 &
####
    kubectl port-forward nginx-pod 8085:80 --address 0.0.0.0 &

#### Port-Forwarding a local port to a pod with address binding:

#### kubectl port-forward --address <address> <pod-name> <local-port>:<pod-port>

#### Example command:

    kubectl port-forward --address 192.168.0.100 my-pod 8080:80

#### Forwarding a local port to a service with address binding:

#### kubectl port-forward --address <address> service/<service-name> <local-port>:<service-port>

#### Example command:

    kubectl port-forward --address 192.168.0.100 service/my-service 8080:80

#### Listen on a random port locally, forwarding to 80 in the pod

    kubectl port-forward pod/nginx :80

#### Check port-forwarding services 

    ps -ef | grep port-forward

#### Analyze Network Connections

    netstat -antp

#


#### Kubectl Basic Operation in Kubernetes

#### Run a Pod

    kubectl run nginx -–image nginx

#### Port-forwarding the pod
    
    kubectl port-forward nginx 8080:80

#### View the Pod Logs

    kubectl logs nginx

#### View the Pod Logs Continuously

    kubectl logs -f pod-name

#### Run a Pod with Shell Exection  

    kubectl run myshell -it –image busybox –sh

#### Run a Pod with Shell Exection and auto delete pods

    kubectl run myshell —rm –it –image busybox –sh

#### Execute command from outside to the inside pods

    kubectl exec pod-name date

#### Execute command from outside to the inside pods

    kubectl exec pod-name -c container-name date

#### Execute pods

    kubectl exec -it pod-name /bin/bash

#### Delete Pods

    kubectl delete pod myportfolio

#### Delete all Pods

    kubectl delete pods –all

#### Delete pods & services using label name

    kubectl delete pods,services -l name=label-name

#### View Description

    kubectl describe pod nginx 

#### Create a Deployment

    kubectl create deployment nginx --image nginx:latest --replicas 3

#### List of Deployments

    kubectl get deployment

#### Scale a Deployment

    kubectl scale deployment nginx --replicas 5

#### Delete Deployment

    kubectl delete deploy myshell

#### Create a pod using local docker images using image-policy

    kubectl run myportfolio --image=myportfolio:latest --image-pull-policy=Never --restart=Never

#### Expose a Service

    kubectl expose deployment nginx –type NodePort –port 80

#### View Service Description

    kubectl describe svc ngnix

#### List of pods widely

    kubectl get pods -o wide

#### Create a Namespace

    kubectl create namespace devops-tool-suite
#
### Kubernetes Persistent Volumes using HostPath

#### Check your cluster’s storage classes

    kubectl get storageclass

#### Create a Persistent Volume

    nano nginx-hostpath.yaml
####
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: nginxpv
    spec:
      accessModes:
        - ReadWriteOnce
      capacity:
        storage: 1Gi
      storageClassName: standard
      hostPath:
        path: /nfsstorage/nginxdata
    
    ---
    #Create a Persistent Volume Claim

    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: nginxpvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
      storageClassName: standard
      volumeName: nginxpv
      
    ---
    #Attach PVCs to Pods
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          volumeMounts:
              name: data
            - mountPath: /usr/share/nginx/html
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: nginxpvc

####
    kubectl create -f nginx-hostpath.yaml
####
    kubectl get pv,pvc,pod
        
#
#
### Kubernetes Persistent Volumes using NFS

    nano nginx-nfs.yaml
####
    apiVersion: v1
    kind: PersistentVolume
    metadata:
      name: mynfspv
    spec:
      capacity:
        storage: 2Gi
      accessModes:
        - ReadWriteOnce
      nfs:
        server: 192.168.0.96
        path: /nfsstorage/jenkins_home
      persistentVolumeReclaimPolicy: Retain
    
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mynfspvc
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
    ---
    #Attach PVCs to Pods
    
    apiVersion: v1
    kind: Pod
    metadata:
      name: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          volumeMounts:
              name: data
            - mountPath: /usr/share/nginx/html
      volumes:
        - name: data
          persistentVolumeClaim:
            claimName: mynfspvc

####
    kubectl create -f nginx-nfs.yaml
####
    kubectl get pv,pvc,pod
#
#
### Jenkins Deployment in Kubernetes

    nano jenkins-k8s.yaml
####
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: myjenkins
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: myjenkins
      template:
        metadata:
          labels:
            app: myjenkins
        spec:
          containers:
          - name: myjenkins
            image: jenkins/jenkins:lts
            ports:
              - name: http-port
                containerPort: 8080
              - name: jnlp-port
                containerPort: 50000
            #command: ["sh", "-c", "chown -R 777:777 /var/jenkins_home"]
            #command: ["sh", "-c", "chown root:root /var/jenkins_home"]
            volumeMounts:
              - name: jenkins-home
                mountPath: /var/jenkins_home
          volumes:
              - name: jenkins-home
                persistentVolumeClaim:
                 claimName: pvcnfs
####
    kubectl create -f jenkins-k8s.yaml
####
    kubectl get pod
#
#
### Verifying Kubernetes Volume 

#### Verifying Storage Class List
    kubectl get sc
#### Describe Specefic Storage Class
    kubectl describe sc StorageClass-Name
#### Verifying Persistent Volume List
    kubectl get pv
#### Verifying Persistent Volume Claim List
    kubectl get pvc
#### Describe Specefic Persistent Volume
    kubectl describe pv pv-name
#### Describe Specefic Persistent Volume Claim
    kubectl describe pvc pvc-name
#### Delete Specefic Storage Class
    kubectl delete sc sc-name
#### Delete Specefic PVC
    kubectl delete pvc pvc-name
#### Delete Specefic PV
    kubectl delete pv pv-name
#
#
### Run Nginx Pod using NFS Data Persistent 
   nano nginx-nfs.yaml
####
    apiVersion: v1 
    kind: Pod 
    metadata:
      name: web1 
    spec:
      containers:
      - name: nginx 
        image: nginx:latest 
        ports:
        - containerPort: 80 
        volumeMounts:
          - mountPath: /usr/share/nginx/html
            name: nginxstorage
      volumes:
        - name: nginxstorage
          nfs:
            server: 192.168.0.96
            path: /nfs-share/kubernetes
#
#
### Create yml file using image-pull-policy=Never

    nano pod-localimage.yaml
####
    apiVersion: v1
    kind: Pod
    metadata:
      name: web1
    spec:
      containers:
      - name: myportfolio
        image: myportfolio:latest
        imagePullPolicy: Never
        ports:
        - containerPort: 80
        volumeMounts:
          - mountPath: /usr/share/nginx/html
            name: nginxstorage
      volumes:
        - name: nginxstorage
          nfs:
            server: 192.168.0.96
            path: /nfs-share/kubernetes
        
#


