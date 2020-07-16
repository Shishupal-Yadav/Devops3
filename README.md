# Launching a webpage on top of Kubernetes
**Project Description**
- Create container image that’s has Jenkins installed  using dockerfile  Or You can use the Jenkins Server on RHEL 8/7
-  When we launch this image, it should automatically starts Jenkins service in the container.
-  Create a job chain of job1, job2, job3 and  job4 using build pipeline plugin in Jenkins 
-  Job1 : Pull  the Github repo automatically when some developers push repo to Github.
- Job2 : 
    - By looking at the code or program file, Jenkins should automatically start the respective language interpreter installed image container to deploy code on top of Kubernetes ( eg. If code is of  PHP, then Jenkins should start the container that has PHP already installed )
    - Expose your pod so that testing team could perform the testing on the pod
    - Make the data to remain persistent ( If server collects some data like logs, other user information )
-  Job3 : Test your app if it  is working or not.
-  Job4 : if app is not working , then send email to developer with error messages and redeploy the application after code is being edited by the developer

# Let's understand some of the basic things

# What is Kubernetes?

![alt text](https://upload.wikimedia.org/wikipedia/commons/3/39/Kubernetes_logo_without_workmark.svg)

-Kubernetes is a portable, extensible, open-source platform for managing containerized workloads and services, that facilitates both declarative configuration and automation. It has a large, rapidly growing ecosystem. Kubernetes services, support, and tools are widely available.

-The name Kubernetes originates from Greek, meaning helmsman or pilot. Google open-sourced the Kubernetes project in 2014. Kubernetes combines over 15 years of Google's experience running production workloads at scale with best-of-breed ideas and practices from the community.

# Why you need Kubernetes and what it can do
-Containers are a good way to bundle and run your applications. In a production environment, you need to manage the containers that run the applications and ensure that there is no downtime. For example, if a container goes down, another container needs to start. Wouldn't it be easier if this behavior was handled by a system?

-That's how Kubernetes comes to the rescue! Kubernetes provides you with a framework to run distributed systems resiliently. It takes care of scaling and failover for your application, provides deployment patterns, and more. For example, Kubernetes can easily manage a canary deployment for your system.

Kubernetes provides you with:

-Service discovery and load balancing
-Storage orchestration
-Automated rollouts and rollbacks
-Automatic bin packing
-Self-healing
-Secret and configuration management

# What is Jenkins and why we use it?

![alt text](https://www.hopetutors.com/wp-content/uploads/2019/05/jenkins_new_0.jpg)


-Jenkins is an open-source automation tool written in Java with plugins built for Continuous Integration purposes. Jenkins is used to build and test your software projects continuously making it easier for developers to integrate changes to the project, and making it easier for users to obtain a fresh build. It also allows you to continuously deliver your software by integrating with a large number of testing and deployment technologies.

-With Jenkins, organizations can accelerate the software development process through automation. Jenkins integrates development life-cycle processes of all kinds, including build, document, test, package, stage, deploy, static analysis, and much more.

-Jenkins achieves Continuous Integration with the help of plugins. Plugins allows the integration of Various DevOps stages. If you want to integrate a particular tool, you need to install the plugins for that tool. For example: Git, Maven 2 project, Amazon EC2, HTML publisher etc.

**After Knowing these technologies Let's start our project**

**Step 1**
*Initially , first we have to create the Docker Image  using Dockerfile , so that when we run container using this docker Image  it should automatically start jenkins. I have completed this task by using the Jenkins Server on my base Redhat but you can use this Dockerfile to create your Image *

![Dockerfile](/Images/1.png)

*After creating this file run the following command  :*
**Command to be run in the directory where your Dockerfile present**
```
docker build -t Name:Tag .
```
**Command to be run in /root directory or other directory**
```
docker build -t Name:Tag /Dockerfile path/
```

**Step 2**
*Then we have to create our code and push to our github account , I have uploaded my codes as index.html and index.php in this github repository*

**Step 3**
*JOB 1*
- This Job will pull the code from github and copy to /devops3 directory in my base Redhat8
- In this Job I have Poll SCM trigger , so that when any changes are made in Code then it will automatically downlaod the new code and copy it to directory
- This Job will also trigger next Job

![Job1](/Images/Job1.jpg/)

![Job1.](/Images/Job1..jpg)

**Step 4**
*JOB 2*
- Now this Job will launch the pods using the httpd and php(vimal3/apache-webserver-php) images , it will also create pvc , and also  service so that pods can be exposed to outside world
- This Job would also delete all the resources if they are already running
- And also trigger the next Job

![Job2](/Images/Job2.jpg)

![Job2.](/Images/Job2..jpg)

*Output of Job*

![Job 2](/Images/Job 2 Output.jpg)

***html-pod.yml***
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: htmlpvc
spec:
   accessModes:
     - ReadWriteOnce
   resources:
       requests:
          storage: 10Gi
 
---
apiVersion: v1
kind: Pod
metadata:
    name: html-pod
    labels:
      app: html

spec:
      containers:
      - image: httpd
        name: html-cont
        
        
        volumeMounts:
        - name: html-persistent-storage
          mountPath: /usr/local/apache2/htdocs
      volumes:
      - name: html-persistent-storage
        persistentVolumeClaim:
          claimName: htmlpvc


---
apiVersion: v1
kind: Service
metadata:
  name: html-service
spec:
  type: NodePort
  selector:
    app: html
  ports:
    - nodePort: 31990
      port: 80
      targetPort: 80
```

***php-pod.yml***

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: phppvc
spec:
   accessModes:
     - ReadWriteOnce
   resources:
       requests:
          storage: 10Gi
 
---
apiVersion: v1
kind: Pod
metadata:
    name: php-pod
    labels:
      app: php

spec:
      containers:
      - image: vimal13/apache-webserver-php
        name: php-cont
        
        
        volumeMounts:
        - name: php-persistent-storage
          mountPath: /var/www/html
      volumes:
      - name: php-persistent-storage
        persistentVolumeClaim:
          claimName: phppvc


---
apiVersion: v1
kind: Service
metadata:
  name: php-service
spec:
  type: NodePort
  selector:
    app: php
  ports:
    - nodePort: 31991
      port: 80
      targetPort: 80

```
**Step 5**
*Job 3*
- This Job will  copy the code downloaded from github to containers 
- This Job will also  check the status code of our running  application 
- It will fail the build if status code is other than 200
- It will aslo  send an email always either bulid failed or success
- It will trigger the next Job

![Job3](/Images/Job3.jpg)

![Job3..jpg](/Images/Job3..jpg)

***Mail for build***

![Mail](/Images/InkedMail_LI.jpg)


***Now let's use browser to see how out webpage look ,Aloha! here is the ouput***
-Html Output
![Output](/Images/31990.jpg)
-Php Output
![Output](/Images/31991..jpg)

**Step 6**
*Job 4*
- This Job check whethet our pod is running or not , it not then it will trigger the JOB 1 again
- It will also send an email if Job failed

![Output](/Images/Job 4..jpg)
![Output](/Images/Job 4...jpg) 

