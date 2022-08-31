## CSR USER
### Step 1 : Create user and password

```
useradd myuser
passwd myuser
cd /home/myuser
```

### Step 2 : Generate key and csr file

```
openssl genrsa -out john.key 2048
openssl req -new -key myuser.key -subj "/CN=myuser/O=developers" -out myuser.csr
```

### Step 3 : Create CertificateSigningRequest

```
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: myuser
spec:
  groups:
  - system:authenticated
  request: $(cat myuser.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  usages:
  - client auth
EOF
```

```
kubectl certificate approve myuser
kubectl get csr myuser -o jsonpath='{.status.certificate}' | base64 --decode > myuser.crt
```
### Step 4 : Create role and rolebinding

```
kubectl create role developer --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
kubectl create rolebinding developer-binding-myuser --role=developer --user=myuser
```

### Step 5 : Add to kubeconfig

```
kubectl config set-credentials myuser --client-key=/home/myuser/myuser.key --client-certificate=/home/myuser/myuser.crt --embed-certs=true
kubectl config set-context myuser --cluster=kubernetes --user=myuser
```

### Step 6 : Use context check context and test

```
kubectl config use-context myuser
kubectl config view
kubectl get pods -A
```

### Step 7 add kubeconfig to path /home/myuser/.kube/config

```
cp -p /root/.kube/config /home/myuser/.kube/config
```

### Step 8 Config file kubeconfig to newfile

```
vi /home/myuser/.kube/config
```

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: (cat /etc/kubernetes/pki/ca.crt | base64 | tr -d "\n")
    server: https://192.0.2.10:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: thanakrit
  name: thanakrit
current-context: thanakrit
kind: Config
preferences: {}
users:
- name: thanakrit
  user:
    client-certificate-data: (cat myuser.crt | base64 | tr -d "\n")
    client-key-data: (cat myuser.key | base64 | tr -d "\n")

```

### Step 9 test use user to kubectl get pod

```
su - myuser
export KUBECONFIG=/home/myuser/.kube/config
kubectl config view
kubectl get pod
```
