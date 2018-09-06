# infra-kubernetes
for CoreOS

## init cluster
`sudo kubeadm init --pod-network-cidr=192.168.0.0/16`


## store config
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


## network (calico v3.2)
https://docs.projectcalico.org/v3.2/getting-started/kubernetes/

```
kubectl apply -f https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/hosted/etcd.yaml
kubectl apply -f https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/rbac.yaml
kubectl apply -f https://docs.projectcalico.org/v3.2/getting-started/kubernetes/installation/hosted/calico.yaml
watch kubectl get pods --all-namespaces
```


## untainted
`kubectl taint nodes --all node-role.kubernetes.io/master-`


## storage rook.io (release-0.8)
https://rook.io/

```
git clone https://github.com/rook/rook.git
cd rook
git checkout remotes/origin/release-0.8
cd cluster/examples/kubernetes/ceph
```

### edit env from `operator.yaml`
coreos specificity: https://rook.github.io/docs/rook/master/flexvolume.html

```
- name: FLEXVOLUME_DIR_PATH
  value: "/var/lib/kubelet/volumeplugins"
```

```
kubectl create -f operator.yaml
watch kubectl -n rook-ceph-system get pods
```

```
kubectl create -f cluster.yaml
kubectl create -f storageclass.yaml
```


## helm (v2.9.1)
### install local client
`curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get | bash`

```
kubectl --namespace kube-system create serviceaccount tiller
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller
helm init --service-account tiller
```

### securing
`kubectl patch deployment tiller-deploy --namespace=kube-system --type=json --patch='[{"op": "add", "path": "/spec/template/spec/containers/0/command", "value": ["/tiller", "--listen=localhost:44134"]}]'`


## ingress-nginx
https://kubernetes.github.io/ingress-nginx/
`wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml`

#### edit bloc for controller part
```
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Equal
        effect: NoSchedule
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: node-role.kubernetes.io/master
                operator: Exists
      hostNetwork: true
      terminationGracePeriodSeconds: 60
```

`kubectl create -f mandatory.yaml`


## istio
https://istio.io

###download local client
`curl -L https://git.io/getLatestIstio | sh -`

### deploy with helm
```
kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml
kubectl apply -f install/kubernetes/helm/istio/charts/certmanager/templates/crds.yaml

kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
helm init --service-account tiller
helm install install/kubernetes/helm/istio --name istio --namespace istio-system --set gateways.istio-ingressgateway.type=NodePort --set gateways.istio-egressgateway.type=NodePort
```

`kubectl get svc -n istio-system`


## cert-manager
`helm install --name cert-manager --namespace kube-system --set ingressShim.extraArgs='{--default-issuer-name=letsencrypt-staging,--default-issuer-kind=ClusterIssuer}' stable/cert-manager`

### deploy issuers + clusterissuers


## monitoring stack
https://github.com/stouff-capital/com-stouffcapital-monitoring



