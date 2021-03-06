# 03. 建立 kubeconfig 文件

## Table of Contents
- [前言](#%E5%89%8D%E8%A8%80)
- [建立 kubelets 客戶端的 TLS Bootstrap]()
- [建立 admin.conf 的 kubeconfig]()
- [建立 controller-manager.conf 的 kubeconfig]()
- [建立 scheduler.conf 的 kubeconfig]()
- [建立 kubectl 的 kubeconfig 文件](#%E5%BB%BA%E7%AB%8B-kubectl-kubeconfig-%E6%96%87%E4%BB%B6)
  * [Master]()
  * [workernode1]()
  * [workernode2]()
- [建立 kube-proxy kubeconfig 文件](#%E5%BB%BA%E7%AB%8B-kube-proxy-kubeconfig-%E6%96%87%E4%BB%B6)

## 前言
在上一步已經建立好 kubelet, admin, kube-controller-manager, kube-scheduler, kube-proxy 的憑證和私鑰。  
而 kubeconfig 是 Kubernetes client 端和 API Server 認證的保證。  

## 建立 kubelets 客戶端的 TLS Bootstrap
set up TLS client certificate bootstrapping for kubelets
ref: https://kubernetes.io/docs/admin/kubelet-tls-bootstrapping/#kubelet-configuration
```sh
$ export KUBE_APISERVER="10.140.0.2"
$ cd /etc/kubernetes/ssl

# generate tokens
$ export BOOTSTRAP_TOKEN=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ')

$ cat <<EOF > /etc/kubernetes/token.csv
${BOOTSTRAP_TOKEN},kubelet-bootstrap,10001,"system:kubelet-bootstrap"
EOF


# bootstrap set-cluster
$ kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBE_APISERVER}:6443 \
    --kubeconfig=../bootstrap.kubeconfig

# bootstrap set-credentials
$ kubectl config set-credentials kubelet-bootstrap \
    --token=${BOOTSTRAP_TOKEN} \
    --kubeconfig=../bootstrap.kubeconfig

# bootstrap set-context
$ kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=kubelet-bootstrap \
   --kubeconfig=../bootstrap.kubeconfig


# bootstrap set default context
$ kubectl config use-context default --kubeconfig=../bootstrap.kubeconfig
```

## 建立 admin.conf 的 kubeconfig
建立 admin.conf 配置對集群的訪問的文件
```
$ cd /etc/kubernetes/ssl/
# admin set-cluster
$ kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBE_APISERVER}:6443 \
    --kubeconfig=../admin.conf

# admin set-credentials
$ kubectl config set-credentials kubernetes-admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=../admin.conf

# admin set-context
$ kubectl config set-context kubernetes-admin@kubernetes \
    --cluster=kubernetes-the-hard-way \
    --user=kubernetes-admin \
    --kubeconfig=../admin.conf

# admin set default context
$ kubectl config use-context kubernetes-admin@kubernetes \
    --kubeconfig=../admin.conf
```

## 建立 controller-manager.conf 的 kubeconfig

```sh
$ cd /etc/kubernetes/ssl/

# controller-manager set-cluster
$ kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBE_APISERVER}:6443 \
    --kubeconfig=../controller-manager.conf

# controller-manager set-credentials
$ kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=controller-manager.pem \
    --client-key=controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=../controller-manager.conf

# controller-manager set-context
$ kubectl config set-context system:kube-controller-manager@kubernetes \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-controller-manager \
    --kubeconfig=../controller-manager.conf

# controller-manager set default context
$ kubectl config use-context system:kube-controller-manager@kubernetes \
    --kubeconfig=../controller-manager.conf
```

## 建立 scheduler.conf 的 kubeconfig
```
$ cd /etc/kubernetes/ssl/

# scheduler set-cluster
$ kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBE_APISERVER}:6443 \
    --kubeconfig=../scheduler.conf

# scheduler set-credentials
$ kubectl config set-credentials system:kube-scheduler \
    --client-certificate=scheduler.pem \
    --client-key=scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=../scheduler.conf

# scheduler set-context
$ kubectl config set-context system:kube-scheduler@kubernetes \
    --cluster=kubernetes-the-hard-way \
    --user=system:kube-scheduler \
    --kubeconfig=../scheduler.conf

# scheduler set default context
$ kubectl config use-context system:kube-scheduler@kubernetes \
    --kubeconfig=../scheduler.conf
```

## 建立 kubectl kubeconfig 文件

### Master
```sh
$ export KUBE_APISERVER="10.140.0.2"
$ cd /etc/kubernetes/ssl

# kubelet set-cluster
$ kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBE_APISERVER}:6443 \
    --kubeconfig=../kubelet.conf

# kubelet set-credentials
$ kubectl config set-credentials system:node:master \
    --client-certificate=kubeletmaster.pem \
    --client-key=kubeletmaster-key.pem \
    --embed-certs=true \
    --kubeconfig=../kubelet.conf

# kubelet set-context
$ kubectl config set-context system:node:master@kubernetes \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:master \
    --kubeconfig=../kubelet.conf

# kubelet set default context
$ kubectl config use-context system:node:master@kubernetes \
    --kubeconfig=../kubelet.conf
```

### workernode1
> 以下動作先在 A 進行，之後再複製到 D

```sh
$ export KUBE_APISERVER="10.140.0.2"
$ cd /etc/kubernetes/ssl

# 設置 cluster 的參數(worknode1)
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBE_APISERVER}:6443 \
    --kubeconfig=workernode1.kubeconfig

# 設定 client 端的認證參數 (worknode1)
  kubectl config set-credentials system:node:workernode1 \
    --client-certificate=workernode1.pem \
    --client-key=workernode1-key.pem \
    --embed-certs=true \
    --kubeconfig=workernode1.kubeconfig

# 設置上下文參數(worknode1)
  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:workernode1 \
    --kubeconfig=workernode1.kubeconfig

# 設置默認的上下文
kubectl config use-context default --kubeconfig=workernode1.kubeconfig
```

### workernode2
> 以下動作先在 A 進行，之後再複製到 E

```sh
$ export KUBE_APISERVER="10.140.0.2"
$ cd /etc/kubernetes/ssl

# 設置 cluster 的參數(worknode2)
  kubectl config set-cluster kubernetes-the-hard-way \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBE_APISERVER}:6443 \
    --kubeconfig=workernode2.kubeconfig

# 設定 client 端的認證參數 (worknode2)
  kubectl config set-credentials system:node:workernode2 \
    --client-certificate=workernode2.pem \
    --client-key=workernode2-key.pem \
    --embed-certs=true \
    --kubeconfig=workernode2.kubeconfig

# 設置上下文參數(worknode2)
  kubectl config set-context default \
    --cluster=kubernetes-the-hard-way \
    --user=system:node:workernode2 \
    --kubeconfig=workernode2.kubeconfig

# 設置默認的上下文
kubectl config use-context default --kubeconfig=workernode2.kubeconfig
```

- 將 workernode1.kubeconfig 複製到 D 的 `/etc/kubernetes`, workernode2.kubeconfig 複製到 E 的 `/etc/kubernetes`

## 建立 kube-proxy kubeconfig 文件
> 以下動作先在 A 進行，之後再複製到 D, E

```sh
$ cd /etc/kubernetes/ssl

# 設置 cluster 的參數
kubectl config set-cluster kubernetes-the-hard-way \
  --certificate-authority=ca.pem \
  --embed-certs=true \
  --server=https://${KUBE_APISERVER}:6443 \
  --kubeconfig=kube-proxy.kubeconfig

# 設定 client 端的認證參數 
kubectl config set-credentials kube-proxy \
  --client-certificate=kube-proxy.pem \
  --client-key=kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig

# 設置上下文參數
kubectl config set-context default \
  --cluster=kubernetes-the-hard-way \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig

# 設置默認的上下文
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

- 將 kube-proxy.kubeconfig 複製到 D, E 的 `/etc/kubernetes`
