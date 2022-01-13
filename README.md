# Kubernetes (K8s)

[![GoPkg Widget]][GoPkg] [![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/569/badge)](https://bestpractices.coreinfrastructure.org/projects/569)

<img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100">

----

Kubernetes, also known as K8s, is an open source system for managing [containerized applications]
across multiple hosts. It provides basic mechanisms for deployment, maintenance,
and scaling of applications.

Kubernetes builds upon a decade and a half of experience at Google running
production workloads at scale using a system called [Borg],
combined with best-of-breed ideas and practices from the community.

Kubernetes is hosted by the Cloud Native Computing Foundation ([CNCF]).
If your company wants to help shape the evolution of
technologies that are container-packaged, dynamically scheduled,
and microservices-oriented, consider joining the CNCF.
For details about who's involved and how Kubernetes plays a role,
read the CNCF [announcement].

----
$ kubectl config use-context mk8s
$ kubectl get node
$ kubectl cordon mk8s-master-0
$ kubectl drain mk8s-master-0 --ignore-daemonsets --force
$ ssh mk8s-master-0
$ sudo -i
# apt-get install -y kubeadm=1.22.0-00
# kubeadm version 
# kubeadm upgrade plan
# kubeadm upgrade apply v1.22.0 --etcd-upgrade=false
# apt-get install kubelet=1.22.0-00 kubectl=1.22.0-00
# kubelet version
# kubelet version
# systemctl status kubelet
# systemctl daemon-reload
$ exit
$ kubectl get node   #  ensure only the master node has been upgraded to 1.22

$ kubectl cordon ek8s-node-1 # Mark the node as unschedulable through the cordon subcommand
$ kubectl drain ek8s-node-1 --ignore-daemonsets --force # Safely eject all pods of a node

$ kubectl create clusterrole deployment-clusterrole --verb=create --resource=deployments,statefulsets,daemonsets
$ kubectl create namespace app-team1
$ kubectl -n app-team1 create serviceaccount cicd-token
$ kubectl -n app-team1 create clusterrolebinding cicd-token-binding --clusterrole=deployment-clusterrole --serviceaccount=app-team1:cicd-token
$ kubectl -n app-team1 describe clusterrolebindings.rbac.authorization.k8s.io cicd-token-binding

 export ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert=/opt/KUIN00601/ca.crt --cert=/opt/KUIN00601/etcd-client.crt --key=/opt/KUIN00601/etcd-client.key snapshot save /data/backup/etcd-snapshot.db

$ export ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 --cacert=/opt/KUIN00601/ca.crt --cert=/opt/KUIN00601/etcd-client.crt --key=/opt/KUIN00601/etcd-client.key snapshot restore /var/lib/backup/etcd-snapshot-previous.db

create new namespace internal

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace
  namespace: internal
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector: {}
    ports:
    - protocol: TCP
      port: 9200

kubectl describe ns corp-bar
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-port-from-namespace1
  namespace: internal
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          project: internal
    ports:
    - protocol: TCP
      port: 9200

$ kubectl config use-context k8s 
$ kubectl expose deployment front-end --port=80 --target-port=80 --protocol=TCP --type=NodePort --name=front-end-svc


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ping
  namespace: ing-internal
spec:
  rules:
  - http:
      paths:
      - path: /hi
        pathType: Prefix
        backend:
          service:
            name: hi
            port:
              number: 5678
$ kubectl get pod -n ing-internal -o wide   # fetch the IP address of ingress
$ curl -kL # returning hi means OK

$ kubectl config use-context k8s kubectl scale deployment webserver --replicas=6
$ kubectl config use-context k8s kubectl describe node | grep -i taints|grep -v -i noschedule echo $num > /opt/CKA0402/kucc1.txt

$ kubectl config use-context k8s
$ kubectl  top  pod -l name=cpu-user -A
    NAMAESPACE NAME        CPU   MEM
    delault    cpu-user-1  45m   6Mi
    delault    cpu-user-2  38m   6Mi
    delault    cpu-user-3  35m   7Mi
    delault    cpu-user-4  32m   10Mi
$ echo 'cpu-user-1' >> /opt/CKA00321/CKA00123.txt


JSONPATH='{range .items[*]}{@.metadata.name}:{range
@.status.conditions[*]}{@.type}={@.status};{end}{end}' \
&& kubectl get nodes -o jsonpath="$JSONPATH" | grep
"Ready=True" > /opt/node-status
//Verify
cat /opt/node-status

cat >> config.txt << EOF
key1=value1
key2=value2
EOF
cat config.txt
// Create configmap from "config.txt" file
kubectl create cm keyvalcfgmap --from-file=config.txt
//Verify
kubectl get cm keyvalcfgmap -o yaml

kubectl create secret generic my-secret --fromliteral=username=user --from-literal=password=mypassword
// Verify
kubectl get secret --all-namespaces
kubectl get secret generic my-secret -o yaml

kubectl get nodes -o jsonpath="{range
.items[*]}{.metadata.name}
{.spec.taints[?(@.effect=='NoSchedule')].effect}{\"\n\"}{end}"
| awk 'NF==1 {print $0}' > /opt/schedulable-nodes.txt
// Verify
cat /opt/schedulable-nodes.txt

kubectl run nginx --image=nginx --restart=Never --env=var1=val1
# then
kubectl exec -it nginx -- env
# or
kubectl exec -it nginx -- sh -c 'echo $var1'
# or
kubectl describe pod nginx | grep val1
# or
kubectl run nginx --restart=Never --image=nginx --env=var1=val1
-it --rm - env

kubectl create job hello-job --image=busybox --dry-run -o yaml
-- echo "Hello I am from job" > hello-job.yaml
// edit the yaml file to add completions: 10 and
kubectl create -f hello-job.yaml
YAML File:
apiVersion: batch/v1
kind: Job
metadata:
name: hello-job
spec:
completions: 10
parallelism: 5
template:
metadata:
spec:
containers:
- command:
- echo
- Hello I am from job
image: busybox
name: hello-job
restartPolicy: Never

kubectl run nginx --image=nginx --dry-run=client -o yaml > nodeselector-pod.yaml
// add the nodeSelector like below and create the pod
vi nodeselector-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  nodeSelector:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
kubectl apply -f nodeselector-pod.yaml
//Verify
kubectl get no -show-labels
kubectl get pod
kubectl describe pod nginx | grep Node-Selectors

Kubectl get nodes
// Check which node shows a not ready
kubectl describe nodes "node-name"
// Login to the node which shows as not ready and check the
process for kubelet, docker , kube-proxy.
// systemctl status kubelet (or) ps -aux | grep -i "processname"
// If the process is not started, then start using
systemctl start kubelet / docker
// Verify
ps -auxxww | grep -i "process-name"
kubectl get nodes

kubectl create namespace development kubectl run nginx --image=nginx --restart=Never -n development
kubectl autoscale deploy webapp --min=10 --max=20 --cpu percent=85 kubectl get hpa kubectl get pod -l app=webapp
kubectl get pods -l 'env in (dev,prod)' --show-labels


apiVersion: v1
kind: Pod
metadata:
  name: 11-factory-app
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
      i=0;
      while true;
      do
        echo "$i: $(date)" >> /var/log/11-factory-app.log;
        sleep 1;
      done
    volumeMounts:
    - name: logs
      mountPath: /var/log
  - name: busybox
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/11-factory-app.log']
    volumeMounts:
    - name: logs
      mountPath: /var/log
  volumes:
  - name: logs

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pv-volume
spec:
  storageClassName: csi-hostpath-sc
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Mi
 
---
apiVersion: v1
kind: Pod
metadata:
  name: web-server
spec:
  volumes:
    - name: task-pv-storage
      persistentVolumeClaim:
        claimName: pv-volume
  containers:
    - name: web-server
      image: nginx
      ports:
        - containerPort: 80
          name: "http-server"
      volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: task-pv-storage
  nodeSelector:
    disk: ssd

disk=spinning
apiVersion: v1
kind: Pod
metadata:
  name: nginx-kusc00401
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  nodeSelector:
    disk: ssd
	
kubectl get pod
kubectl set resources pod nginx-prod --
limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi
kubectl scale deploy webapp --replicas=20 kubectl get deploy webapp kubectl get pod -l app=webapp
kubectl get pv --sort-by=.spec.capacity.storage > /opt/pvlist.txt

curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl

apiVersion: v1
kind: Pod
metadata:
  name: kucc1
spec:
  containers:
  - name: nginx
    image: nginx
  - name: consul
    image: consul

sudo cp /etc/kubernetes/admin.conf /home/vagrant/config
mkdir .kube
mv config .kube/
sudo chown $(id -u):$(id -g ) $HOME/.kube/config
kubectl version

kubectl top -l name=cpu-user -A
echo 'pod name' >> /opt/KUT00401/KUT00401.txt

kubectl get po -all-namespaces > /opt/pods-list.yaml

kubectl get deploy front-end
kubectl edit deploy front-end -o yaml
#port specification named http
#service.yaml

apiVersion: v1
kind: Service
metadata:
  name: front-end-svc
spec:
  ports:
    - protocol: TCP
      port: 80
  selector:
    app: nginx
  type: NodePort
# kubectl create -f service.yaml
# kubectl get svc
# port specification named http
kubectl expose deployment front-end --name=front-end-svc --port=80 --tarport=80 --type=NodePort

## To start using K8s

See our documentation on [kubernetes.io].

Try our [interactive tutorial].

Take a free course on [Scalable Microservices with Kubernetes].

To use Kubernetes code as a library in other applications, see the [list of published components](https://git.k8s.io/kubernetes/staging/README.md).
Use of the `k8s.io/kubernetes` module or `k8s.io/kubernetes/...` packages as libraries is not supported.

## To start developing K8s

The [community repository] hosts all information about
building Kubernetes from source, how to contribute code
and documentation, who to contact about what, etc.

If you want to build Kubernetes right away there are two options:

##### You have a working [Go environment].

```
mkdir -p $GOPATH/src/k8s.io
cd $GOPATH/src/k8s.io
git clone https://github.com/kubernetes/kubernetes
cd kubernetes
make
```

##### You have a working [Docker environment].

```
git clone https://github.com/kubernetes/kubernetes
cd kubernetes
make quick-release
```

For the full story, head over to the [developer's documentation].

## Support

If you need support, start with the [troubleshooting guide],
and work your way through the process that we've outlined.

That said, if you have questions, reach out to us
[one way or another][communication].

[announcement]: https://cncf.io/news/announcement/2015/07/new-cloud-native-computing-foundation-drive-alignment-among-container
[Borg]: https://research.google.com/pubs/pub43438.html
[CNCF]: https://www.cncf.io/about
[communication]: https://git.k8s.io/community/communication
[community repository]: https://git.k8s.io/community
[containerized applications]: https://kubernetes.io/docs/concepts/overview/what-is-kubernetes/
[developer's documentation]: https://git.k8s.io/community/contributors/devel#readme
[Docker environment]: https://docs.docker.com/engine
[Go environment]: https://golang.org/doc/install
[GoPkg]: https://pkg.go.dev/k8s.io/kubernetes
[GoPkg Widget]: https://pkg.go.dev/badge/k8s.io/kubernetes.svg
[interactive tutorial]: https://kubernetes.io/docs/tutorials/kubernetes-basics
[kubernetes.io]: https://kubernetes.io
[Scalable Microservices with Kubernetes]: https://www.udacity.com/course/scalable-microservices-with-kubernetes--ud615
[troubleshooting guide]: https://kubernetes.io/docs/tasks/debug-application-cluster/troubleshooting/
