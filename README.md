# Mini-projet-Kubernetes (en cours de rédaction, brouillon) :



 Deployment of files:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl apply -f ./
namespace/wordpress unchanged
secret/app-wordpress-secret unchanged
deployment.apps/wp-mysql unchanged
service/wp-mysql unchanged
service/wordpress unchanged
deployment.apps/wordpress unchanged
```

check the namespace:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl get namespaces 
NAME              STATUS   AGE
default           Active   4m1s
kube-flannel      Active   3m58s
kube-node-lease   Active   4m6s
kube-public       Active   4m6s
kube-system       Active   4m6s
wordpress         Active   61s
```

checking deployed objects in namespace wordpress, we noticed that the pod "wordpress" is not ready yet:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl get deployments.apps -n wordpress 
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
wordpress   0/1     1            0           104s
wp-mysql    1/1     1            1           104s
```

check once again and we noticed that the deployment of the pod wordpress is ready.
```bash
[node1 Mini-projet-Kubernetes]$ 
kubectl get deployments.apps -n wordpress 
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
wordpress   1/1     1            1           15m
wp-mysql    1/1     1            1           15m
```

Display replicasets in the namespace "wordpress":
```bash
[node1 Mini-projet-Kubernetes]$ kubectl get replicasets.apps -n wordpress 
NAME                  DESIRED   CURRENT   READY   AGE
wordpress-9c696ffbd   1         1         1       19m
wp-mysql-5f784bd86    1         1         1       19m
```

Display pod of the namespace "wordpress":
```bash
[node1 Mini-projet-Kubernetes]$ kubectl get pod -n wordpress 
NAME                        READY   STATUS    RESTARTS   AGE
wordpress-9c696ffbd-s8vfm   1/1     Running   0          31m
wp-mysql-5f784bd86-jjvc2    1/1     Running   0          31m
```

Check the state of the pod "wordpress-9c696ffbd-s8vfm" with docker command:
```bash
[node1 Mini-projet-Kubernetes]$ docker ps -a | grep s8vfm
47ad2179e19e   wordpress                    "docker-entrypoint.s…"   29 minutes ago  Up 29 minutes  k8s_wordpress_wordpress-9c696ffbd-s8vfm_wordpress_1efdbcbd-3f7c-44bc-b2b7-80f15b0bbf95_0
f1806efad5cc   k8s.gcr.io/pause:3.2         "/pause"                 32 minutes ago  Up 32 minutes  k8s_POD_wordpress-9c696ffbd-s8vfm_wordpress_1efdbcbd-3f7c-44bc-b2b7-80f15b0bbf95_0
```

Delete the pod:
```bash
[node1 Mini-projet-Kubernetes]$ docker rm -f 47ad2179e19e
47ad2179e19e
```

Kubernetes create a new pod after being removed:
```bash
[node1 Mini-projet-Kubernetes]$ docker ps -a | grep s8vfm
a534856a2143   wordpress                    "docker-entrypoint.s…"   4 minutes ago   Up 4 minutes   k8s_wordpress_wordpress-9c696ffbd-s8vfm_wordpress_1efdbcbd-3f7c-44bc-b2b7-80f15b0bbf95_0
f1806efad5cc   k8s.gcr.io/pause:3.2         "/pause"                 38 minutes ago  Up 38 minutes  k8s_POD_wordpress-9c696ffbd-s8vfm_wordpress_1efdbcbd-3f7c-44bc-b2b7-80f15b0bbf95_0
```

Ckech the number of restarts:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl get pod -n wordpress
NAME                        READY   STATUS    RESTARTS   AGE
wordpress-9c696ffbd-s8vfm   1/1     Running   1          45m
wp-mysql-5f784bd86-jjvc2    1/1     Running   0          45m
```


Display services:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl get service -n wordpress
NAME        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
wordpress   NodePort    10.107.207.246   <none>        80:30008/TCP   73m
wp-mysql    ClusterIP   10.98.149.222    <none>        3306/TCP       73m
```

Display service wordpress:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl describe service wordpress -n wordpress
Name:                     wordpress
Namespace:                wordpress
Labels:                   app=wordpress
Annotations:              Selector:  app=wordpress,tier=frontend
Type:                     NodePort
IP:                       10.107.207.246
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30008/TCP
Endpoints:                10.244.0.6:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Display service wp-mysql:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl describe service wp-mysql -n wordpress
Name:              wp-mysql
Namespace:         wordpress
Labels:            name=wordpress
Annotations:       Selector:  app=wordpress,tier=mysql
Type:              ClusterIP
IP:                10.98.149.222
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         10.244.0.4:3306
Session Affinity:  None
Events:            <none>
```

increase the number of pods to 2 :
```bash
[node1 Mini-projet-Kubernetes]$ kubectl scale deploy wordpress --replicas=2 -n wordpress 
deployment.apps/wordpress scaled
```
The number of pods has increased from 1 to 2 pods:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl get pod -n wordpress 
NAME                        READY   STATUS    RESTARTS   AGE
wordpress-9c696ffbd-mf46b   1/1     Running   0          46s
wordpress-9c696ffbd-s8vfm   1/1     Running   1          93m
wp-mysql-5f784bd86-jjvc2    1/1     Running   0          93m
```

So we can see that the service wordpress has now two IP adresses:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl describe service wordpress -n wordpress
Name:                     wordpress
Namespace:                wordpress
Labels:                   app=wordpress
Annotations:              Selector:  app=wordpress,tier=frontend
Type:                     NodePort
IP:                       10.107.207.246
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30008/TCP
Endpoints:                10.244.0.6:80,10.244.0.7:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

Display secrets:
```bash
[node1 Mini-projet-Kubernetes]$ kubectl get secrets -n wordpress
NAME                   TYPE                                  DATA   AGE
app-wordpress-secret   Opaque                                3      105m
default-token-cwh29    kubernetes.io/service-account-token   3      105m

[node1 Mini-projet-Kubernetes]$ kubectl get secrets -o yaml -n wordpress | grep data  
  data:
  metadata:
        {"apiVersion":"v1","data":{"mysql_password":"dG90bwo=","mysql_random_root_password":"MQo=","wordpress_db_password":"dG90bwo="},"kind":"Secret","metadata":{"annotations":{},"name":"app-wordpress-secret","namespace":"wordpress"},"type":"Opaque"}
        f:data:
        f:metadata:
  data:
  metadata:
        f:data:
        f:metadata:
metadata:
```

Open a browser and type the following address in the address bar:
```bash
http://<host ip address>:30008
```

![image](https://user-images.githubusercontent.com/72947514/230341975-018812de-0d29-47ea-a2da-8227c9044b24.png)





