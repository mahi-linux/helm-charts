# helm-charts


##### How to create custom helm charts

```
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
```
[root@k8s-master01 ~]# helm list -A
NAME    NAMESPACE       REVISION        UPDATED STATUS  CHART   APP VERSION

[root@k8s-master01 ~]# helm install custom-nginx nginx/
NAME: custom-nginx
LAST DEPLOYED: Tue Feb 14 15:04:27 2023
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=nginx,app.kubernetes.io/instance=custom-nginx" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT

[root@k8s-master01 ~]# helm list -A
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
custom-nginx    default         1               2023-02-14 15:04:27.663026 +0530 IST    deployed        nginx-0.1.0     1.16.0   
```

##### Notice custom deployed nginx pod is running
```
[root@k8s-master01 ~]# kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
custom-nginx-59855d95fb-njbfk   1/1     Running   0          91s

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
[root@k8s-master01 ~]# helm repo index --url https://mahi-linux.github.io/helm-chart/ .

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

  => Go to  https://github.com/mahi-linux/helm-charts/settings/pages
  => On the Branch dropdown and select master and root directory then click on save.
  => This GitHub page will take few min to publish. Validate the pipeline status: heck the workflow: https://github.com/mahi-linux/helm-charts/actions
  => commit and push the code: git add . && git commit -am "nginx chart" && git push

```
##### Web validation
```
Once the webpage is published validate: https://mahi-linux.github.io/helm-charts/index.yaml
and  https://mahi-linux.github.io/helm-charts/

[root@k8s-master01 ~]# helm repo add myrepo https://mahi-linux.github.io/helm-charts

[root@k8s-master01 ~]# helm repo list
NAME    URL                                     
myrepo  https://mahi-linux.github.io/helm-charts

* To remove the repository
[root@k8s-master01 ~]# helm repo remove myrepo
"myrepo" has been removed from your repositories
```
