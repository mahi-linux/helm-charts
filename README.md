# helm-charts

##### How to create custom helm charts
```
Reference: https://medium.com/@mattiaperi/create-a-public-helm-chart-repository-with-github-pages-49b180dbb417

# Create new github repository
 => https://github.com/new
 => Clone to your local machine: git clone https://github.com/mahi-linux/helm-charts.git
 => navigate to the dir:  cd helm-charts/
```
##### To create custom helm chart
```
[root@k8s-master01 ~]# helm create nginx
Creating nginx
```
##### To view the chart
```
[root@k8s-master01 ~]# tree nginx/
nginx/
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

4 directories, 10 files
```

##### Make required changes
```
* For example if you are using Minikube change "service type" from ClusterPort to NodePort in values.yaml file
```

##### `helm list` runs a series of tests to verify that the chart is well-formed:
```
[root@k8s-master01 ~]# helm lint nginx
==> Linting nginx
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

##### Package your chart
```
[root@k8s-master01 ~]# helm package nginx
Successfully packaged chart and saved it to: /Users/maheshwar.thumkuntla/kubernets/helm-charts/nginx-1.0.0.tgz

helm package nginx -u => This will download the dependencies at the latest version and package your chart.
helm package nginx -d ./helm-charts => This will package to under /helm-charts directory
```

##### Publish custom repository to GitHub Pages
```
[root@k8s-master01 ~]# helm repo index --url https://mahi-linux.github.io/helm-charts/ .

[root@k8s-master01 ~]# cat index.yaml 
apiVersion: v1
entries:
  nginx:
  - apiVersion: v2
    appVersion: 1.16.0
    created: "2023-02-14T15:50:01.928787+05:30"
    description: A Helm chart for Kubernetes
    digest: 3bad3b83dfc1f92d4b927bd834eef0ca265c6b48ae021911af66296ea060d276
    name: nginx
    type: application
    urls:
    - https://mahesh-linux.github.io/helm-chart/nginx-1.0.0.tgz
    version: 1.0.0
generated: "2023-02-14T15:50:01.927468+05:30"

```
##### Go to:  https://github.com/mahi-linux/helm-charts/settings/pages

* On the Branch dropdown and select master and root directory then click on save.
* This GitHub page will take few min to publish. Validate the pipeline status:
* Check the github workflow: https://github.com/mahi-linux/helm-charts/actions

#### commit and push the code
```
 git add . && git commit -am "nginx chart" && git push
```
##### Web validation
```
Once the webpage is published validate index page:
https://mahi-linux.github.io/helm-charts/index.yaml (and)
https://mahi-linux.github.io/helm-charts/
```
##### Add helm custom repository
```
[root@k8s-master01 ~]# helm repo add myrepo https://mahi-linux.github.io/helm-charts
[root@k8s-master01 ~]# helm repo list
NAME    URL                                     
myrepo  https://mahi-linux.github.io/helm-charts
```
##### search for nginx chart
```
[root@k8s-master01 ~]# helm search repo nginx
NAME            CHART VERSION   APP VERSION     DESCRIPTION                
myrepo/nginx    0.1.0           1.16.0          A Helm chart for Kubernetes
```
##### To get the latest updates with helm
```
maheshwar.thumkuntla@MHYD789329BA helm-charts % helm repo update      
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "myrepo" chart repository
Update Complete. ⎈Happy Helming!⎈

maheshwar.thumkuntla@MHYD789329BA helm-charts % helm search repo nginx
NAME        	CHART VERSION	APP VERSION	DESCRIPTION                
myrepo/nginx	1.0.0        	1.16.0     	A Helm chart for Kubernetes
```
##### Install nginx chart
```
[root@k8s-master01 ~]# helm install nginx myrepo/nginx
NAME: nginx
LAST DEPLOYED: Tue Feb 14 19:14:59 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services nginx)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT
```

##### Get all the services installed by helm nginx chart
```
[root@k8s-master01 ~]# kubectl get all
NAME                      READY   STATUS    RESTARTS   AGE
pod/nginx-c9dd94d-gt22l   1/1     Running   0          9s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        14d
service/nginx        NodePort    10.110.13.169   <none>        80:31630/TCP   9s

NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx   1/1     1            1           9s

NAME                            DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-c9dd94d   1         1         1       9s
```
##### Validate with minikube
```
[root@k8s-master01 ~]# minikube service nginx
```

##### To remove the repository
[root@k8s-master01 ~]# helm repo remove myrepo
"myrepo" has been removed from your repositories
```
