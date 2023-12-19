# Two-Tier-flask-App-Deployment-Kubernetes   - PART-2

                               TWO-TIER_ARCHITECTURE    PATR-2 :-


                    Process up to deployment in Kubernetes 

        ![image](https://github.com/DineshA055/Two-Tier-flask-App-Deployment-Kubernetes/assets/101075223/eedfb965-e9c5-4504-a5f3-20849f028151)



In previous part we have completed up to docker compose

In same way our image is available in Docker-Hub so we can use image anytime by pull the image from docker hub

In kubernetes am going to use same image to run the container PODS

Kubernetes have more advantage on auto-healing and auto-scaling the pod when there huge traffic ,
management on worker nodes

Kubernetes have master node and worker node

Master node (Control plane)
1. API server
2. Scheduler – to schedule the pod in correct node
3. etcd     - key value store for cuurent state to restore
4. control manager – have control on all pod are running with good state
5. Cloud-contol-manager – required using cloud provider ex:- aws

Worker-Node (Data plane):-
1. Kubelet – make sure Pod is Up and running with no issues
2. Kubeproxy – follow the rules for connecting the network anf follow traffic between pod

Master node have  the control of worker node to give instruction perform the action.

ABOVE FIND THE ARCHITECTURE OF KUBERNETES  :-
![image](https://github.com/DineshA055/Two-Tier-flask-App-Deployment-Kubernetes/assets/101075223/9ec8ee2e-fdd4-4be9-a370-196620693d61)


NEXT :-

CONTINUE DEPLOYMENT WITH TWO_TIER_APP WITH KUBERNETES


Practise we required a mater node and worker node 

Requirement :- 2 Instance 1 for master and 1 for worker node

After creating cluster in 1st Instance will init master node in cluster 
and worker node will be available in 2nd Instance 

Using join command generate token form master node and paster the join command in 2nd instance where worker node is running will join with master node

Now Master node have control of worker node and master node can give instruction to perform action indside worker node.

First step :- Install kubernetes in 1st instance 

This guide outlines the steps needed to set up a Kubernetes cluster using kubeadm.
Pre-requisites

* Ubuntu OS (Xenial or later)
* sudo privileges
* Internet access
* t2.medium instance type or higher

## Both Master & Worker Node

Run the following commands on both the master and worker nodes to prepare them for kubeadm.

```bash
# using 'sudo su' is not a good practice.
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl
sudo apt install docker.io -y

sudo systemctl enable --now docker # enable and start in single command.

# Adding GPG keys.
curl -fsSL "https://packages.cloud.google.com/apt/doc/apt-key.gpg" | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/kubernetes-archive-keyring.gpg

# Add the repository to the sourcelist.
echo 'deb https://packages.cloud.google.com/apt kubernetes-xenial main' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update 
sudo apt install kubeadm=1.20.0-00 kubectl=1.20.0-00 kubelet=1.20.0-00 -y
```
---

## Master Node

1. Initialize the Kubernetes master node.

    ```bash
    sudo kubeadm init
    ```
  
    After succesfully running, your Kubernetes control plane will be initialized successfully.

3. Set up local kubeconfig (both for root user and normal user):

    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```
4. Apply Weave network:

    ```bash
    kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
    ``

5. Generate a token for worker nodes to join:

    ```bash
    sudo kubeadm token create --print-join-command
    ```

6. Expose port 6443 in the Security group for the Worker to connect to Master Node



---

## Worker Node

1. Run the following commands on the worker node.

    ```bash
    sudo kubeadm reset pre-flight checks
    ```
  
2. Paste the join command you got from the master node and append `--v=5` at the end.
*Make sure either you are working as sudo user or use `sudo` before the command*

  
   After succesful join->
   
--

## Verify Cluster Connection

On Master Node:

```bash
kubectl get nodes
```

---


Installation completed based on above procedure given for master node and worker node 
In Mater Node
CMD :- sudo kubeadmin init           # initialize the creation of cluster and master node components and worker node components with cretificate and key will be added 

In Worker node :-     CMD :- sudo kubeadm reset pre-flight checks

# to reset the kubeadm access because it will act has worker node has data plane 

Next :- Generate join token from master node and paster in worker node to the worker node in master node to control the action 

Master node :- CMD :-  sudo kubeadm token create –print-join-command

     o/p-----> kubeadm join and token ..........................................

Worker Node :- paste the Output from master provided

cmd : sudo with paste the kubeadm join command  and last add   --v=5

make sure 6443 Port was open in Security group on Master instance

So worker node will join the cluster 

Now worker node joined the cluster with kubeadm join cmd with token 

##### All setup is completed now we can start creating the YAML file to run the POD 

Here we use  POD.YAML / DEPLOYMENT.YAML (FOR auto healing ) / SERVICE.YAML (To expose the service to connect ( Node port / Cluster IP / Load Balancer ) 


ONCE CREATED REQUIRED YAML FILE IN GITHUB REPOSITORY 

Clone the repo in master terminal for apply 

CMD :- Git clone Htpps:   ......Git-repo......

After the clone the repo in terminal 

Cd to navigate to the path in directoy where Kubernetes YAML manifest file where created 

CD flask-app2

ls

Here we required to execute the deployment.yaml file for auto scaling and SVC to expose 

can run at same time 

CMD :- kubectl apply  f      flask – app2-deployment.yml   - f      flask-app2-svc.yml

you can see result of both are created for conformation go with cmd 

CMD :- Kubectl get pods          # it will show only pod was running state

IF WANT TO KNOW ALL 

CMD :- kubectl get all 

this cmd will show PODS running status first 

Second Kubernetes service ->type ---- Cluster IP with port 
Service app exposed with---> type ---- Load balancer IP  with port it was exposed to access

Deployment.apps
Replicaset.apps

If you want to the detail of pods and status you can use commnad

CMD :- kubectl describe pod podname              #you can get first line shows the pod name copy only pod name paste in cmd

Now it will describe everything of POD in details 
If you want to know the service detail completely 

CMD :- Kubeclt describe service sevicename       #ex:- service name app2-service  

if you want to know the replicaer detail compelety 

CMD :- kubectl describe replicaset name         #replicaset name you can see app2-f7adnbd



you can expose port using Load balancer exposed the pod with in-nodeport out-side expse port 

make sure port was open in wokrer node instance 

Once open in worker instance now you can access the application in web-page with worker public ip

IP:port-number --------------> page will be rendered in browser


Next :- YOU CAN SEE ERROR IN PAGE IN APPLICATION STILL WE HAVE CREATED THE SQL for CONNECTION 

We required to a create pod for SQL 

We required YAML FILE FOR (mysql-deployment.yml  /mysql-service.yml / mysql-secrets.yml/ mysql-configmap.yml )

deployment for auto-heling of replicaset
service to expose pod with node connect to backend-port 
Secrets have encrypted password of Mysql is important 
configmap used to initdb     -> initialize the DB when creating the POD also in entry point itself create the table automatically 

CMD :- to execute in same time 

kubectl -f mysql-deployment.yml       -f     mysql-service.yml     ........continue adding all mysql.yml files

Now you can see created sql.app -----created

check for pod is running    CMD :- kubeclt get pods

you can see mysql pod is running status

If want to know all related to mysql      CMD :- kubectl get all

If want to describe all details of pods   CMD :- kubectl describe pods mysql-6747dfeh5

IF WANT TO CHECK THE LOG FOR CONTAINER ---> GO TO --> WORKER INSTANCE------TERMINAL 

CMD :- sudo docker ps

# show the container id for app and mysql and up time

# TO CHECK THE LOG OF CONTAINER 

CMD :- sudo docker logs container ID                   #container ID will be sql or app container ID 

it will show the logs for the container with full we can find out any error as well



**************** DONE WITH KUBERNETES TWO_TIER ARCHITECTURE (CAN BE ADDED IN FUTURE ANYTHING NEW UPDATE OR CHANGES ******************************************************************************


NEXT ------ PART3 -------Continue with HELM

how to package all yaml file with proper template 

HELM provides deafult template like Yaml format just required to add values and details certain places 

next 

Using HELM------> create a chart for app and my sql ------> 

Package the chart ------ and execute helm using install the package was done on deployment completed

Now you can see application will be render in browser 

HELM have proper TEMPLATE to add the values / deployment / service given with default 

It solve the problem of keeping the YAML file here and there insteasd of using HELM template keep it execute and change anytime for application


*******************************************************************************


 







