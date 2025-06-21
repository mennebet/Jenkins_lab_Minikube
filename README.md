# Jenkins_lab_Minikube
Installing Jenkins using Minikube Cluster
Plan :
1.	Prerequisites
     - Install wsl2
     - Install minikube
     - Use Docker driver
3.	Start minikube
4.	Installing Helm
5.	Add the jenkins repo using Helm chart
6.	Create the following resources
    - jenkins-01-volume.yaml 
    - jenkins-02-sa.yaml 
7.	Create the file jenkins-values.yaml and make some modifications
8.	Install jenkins using helm and jenkins-values.yaml
9.	Get the admin password
10.	Forward the jenkins service
11.	Access jenkins from browser

Let’s start :
You need to have wsl2 installed within your machine, then you need to install minikube and have docker within your machine to use it as a driver.
 #minikube start --driver=docker --force
![image](https://github.com/user-attachments/assets/87aca934-aa13-4e7c-9bb7-d8d646cdece5)

--force to make use of docker driver with root privileges.

Now after starting minikube cluster you need to install helm following the documentation https://helm.sh/docs/intro/install/#from-script 
Then you need to add jenkins repo using helm chart 
![image](https://github.com/user-attachments/assets/239eedfe-0e9b-4703-9616-d7ffb980f857)

Now we need to create based on the manifest files the following resources :
- From file jenkins-01-volume.yaml
    - StorageClass
    - PersistentVolume
- From file jenkins-02-sa.yaml
    - ServiceAccount
    - ClusterRole
    - ClusterRoleBinding
      
#kubectl create namespace jenkins

#kubectl apply -f jenkins-01-volume.yaml

#kubectl apply -f jenkins-02-sa.yaml

Now we have to create the file jenkins-values.yaml and make some modifications, as we’re working on minikube here are the modifications to take into consideration :

- serviceType: NodePort
- nodePort : 32000
- storageClass: jenkins-pv
- serviceAccount:

     create: false

     name: jenkins

     annotations: {}

Now we can save the file and install jenkins using helm install command :

#chart=jenkinsci/jenkins

#helm install jenkins -n jenkins -f jenkins-values.yaml $chart

![image](https://github.com/user-attachments/assets/9df2c670-ee0c-4d22-aaf2-44de225c91c0)

Now before getting admin password and forwarding the service we need to verify the pods status

#kubectl get pods –n jenkins

![image](https://github.com/user-attachments/assets/a9268974-ee66-455f-b8c1-8e8c0797b3e0)

If you’re facing issues you can describe the pod and see the logs however you can verify the following 	also:
1.	Ssh minikube and make sure you’re able to rich the docker registry
![image](https://github.com/user-attachments/assets/484d1886-e1b2-4086-af1d-a2ca4f78d036)

2.	Change the permissions manually to allow jenkis account to write into /data folder
   ![image](https://github.com/user-attachments/assets/ed30801a-6ea8-40d5-a836-4cfbbd211688)

If everything is good and pods are in running status we can now get the admin password and ip forwad the service

#jsonpath="{.data.jenkins-admin-password}"
 
#secret=$(kubectl get secret -n jenkins jenkins -o jsonpath=$jsonpath)
 
#echo $(echo $secret | base64 --decode)

#minikube service jenkins -n jenkins

![image](https://github.com/user-attachments/assets/5b5693d5-affc-497b-b96e-e86078884412)

Now we can access to jenkins via the browser and authenticate using admin and the password we got from the secret in the previous step

![image](https://github.com/user-attachments/assets/96ba9f62-a962-4955-927a-b29b4d1a5288)

![image](https://github.com/user-attachments/assets/4c8ec773-c58f-4923-8dea-de46fcf9b1e1)

 # Useful links
- https://minikube.sigs.k8s.io/docs/tutorials/wsl_docker_driver/
- https://helm.sh/docs/intro/install/#from-script
- https://www.jenkins.io/doc/book/installing/kubernetes/
