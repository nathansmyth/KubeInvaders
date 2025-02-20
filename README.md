# KubeInvaders

 KubeInvaders is a gamified chaos engineering tool for Kubernetes Clusters. **It is like Space Invaders but the pods are alien ships**

![Alt Text](https://github.com/lucky-sideburn/KubeInvaders/blob/master/kubeinvaders.gif)

**jump between namespaces pressing 'n' !**
![Alt Text](https://github.com/lucky-sideburn/KubeInvaders/blob/master/kubeinvaders-switch-namespace.gif)


![Alt Text](https://github.com/lucky-sideburn/KubeInvaders/blob/master/logo.png)

### Description

KubeInvaders has been written with Defold (https://www.defold.com/).

Through KubeInvaders you can stress your Openshift cluster in a fun way and check how it is resilient.


### Special Input Keys

| Input           | Action                                                                                   |
|-----------------|------------------------------------------------------------------------------------------|
|     n           | Change namespace (you should define namespaces list. Ex: TARGET_NAMESPACE=foo1,foo2,foo3)|
|     a           | Switch to automatic mode                                                                 |
|     m           | Switch to manual mode                                                                    |


### Install KubeInvaders on Openshift

To Install KubeInvaders on your Openshift Cluster clone this repo and launch the following commands:

```bash
TARGET_NAMESPACE=foobar
## You can define multiple namespaces ex: TARGET_NAMESPACE=foobar,foobar2

# Choose route host for your kubeinvaders instance.
ROUTE_HOST=kubeinvaders.org

# Please add your source ip IP_WHITELIST. This will add haproxy.router.openshift.io/ip_whitelist in KubeInvaders route
# https://docs.openshift.com/container-platform/3.9/architecture/networking/routes.html#whitelist
IP_WHITELIST="93.44.96.4"

oc new-project kubeinvaders --display-name='KubeInvaders'
oc create sa kubeinvaders -n kubeinvaders
oc create sa kubeinvaders -n $TARGET_NAMESPACE
oc adm policy add-role-to-user edit -z kubeinvaders -n $TARGET_NAMESPACE

TOKEN=$(oc describe secret -n $TARGET_NAMESPACE $(oc describe sa kubeinvaders -n $TARGET_NAMESPACE | grep Tokens | awk '{ print $2}') | grep 'token:'| awk '{ print $2}')
oc process -f openshift/KubeInvaders.yaml -p ROUTE_HOST=$ROUTE_HOST -p TARGET_NAMESPACE=$TARGET_NAMESPACE -p TOKEN=$TOKEN | oc create -f -
```

### Install KubeInvaders on Kubernetes

```bash
#Change with the namespace you want to stress
TARGET_NAMESPACE='foobar'
## You can define multiple namespaces ex: TARGET_NAMESPACE=foobar,foobar2

#Change with the URL of your Kubeinvaders
ROUTE_HOST=kubeinvaders.org

kubectl apply -f kubernetes/kubeinvaders-namespace.yml
kubectl apply -f kubernetes/kubeinvaders-deployment.yml -n kubeinvaders
kubectl expose deployment  kubeinvaders --type=NodePort --name=kubeinvaders -n kubeinvaders --port 8080
kubectl apply -f kubernetes/kubeinvaders-ingress.yml -n kubeinvaders
kubectl create sa kubeinvaders -n foobar
kubectl apply -f kubernetes/kubeinvaders-role.yml
kubectl apply -f kubernetes/kubeinvaders-rolebinding.yml

TOKEN=`kubectl describe secret $(kubectl get secret -n foobar | grep 'kubeinvaders-token' | awk '{ print $1}') -n foobar | grep 'token:' | awk '{ print $2}'`

kubectl set env deployment/kubeinvaders TOKEN=$TOKEN -n kubeinvaders
kubectl set env deployment/kubeinvaders NAMESPACE=$TARGET_NAMESPACE -n kubeinvaders
kubectl set env deployment/kubeinvaders ROUTE_HOST=$ROUTE_HOST -n kubeinvaders
```
