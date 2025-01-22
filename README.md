# Set up a GKE multi node shared volume with Filestore
```bash
kubectl create ns my-ns
```
```bash
gcloud container clusters update my-cluster \
   --update-addons=GcpFilestoreCsiDriver=ENABLED --zone=us-central1-c
```
```bash
gcloud filestore instances list --project=my-project --zone=us-central1-c
```
```bash
FILESTORE_INSTANCE_NAME=$(gcloud filestore instances list --project=my-project --zone=us-central1-c --format="value(name)" --limit=1)
FILESTORE_SHARE_NAME=$(gcloud filestore instances list --project=my-project --zone=us-central1-c --format="value(fileShares[0].name)" --limit=1)
FILESTORE_INSTANCE_IP=$(gcloud filestore instances list --project=my-project --zone=us-central1-c --format="value(networks[0].ipAddresses[0])" --limit=1)
```
```bash
# Print the values
echo "FILESTORE_INSTANCE_NAME: $FILESTORE_INSTANCE_NAME"
echo "FILESTORE_SHARE_NAME: $FILESTORE_SHARE_NAME"
echo "FILESTORE_INSTANCE_IP: $FILESTORE_INSTANCE_IP"
# FILESTORE_INSTANCE_NAME: pvc-2195c1a0-a80c-405e-91a1-2186c3cc0bbb
# FILESTORE_SHARE_NAME: vol1
# FILESTORE_INSTANCE_IP: 10.73.118.242
```
```bash
kubectl apply -f pv.yaml
```
```bash
kubectl apply -f pvc.yaml -n my-ns
```
```bash
kubectl get pv,pvc -n my-ns 
```
```bash
# NAME                     CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM          STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
# persistentvolume/my-pv   1Ti        RWX            Retain           Bound    my-ns/my-pvc                  <unset>                          28m
# NAME                           STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
# persistentvolumeclaim/my-pvc   Bound    my-pv    1Ti        RWX            standard-rwx   <unset>                 26m
```
```bash
kubectl get nodes | grep dataplane-pool | cut -d' ' -f1
```
```bash
# gke-my-cluster-dataplane-pool-85bedef0-8wfb
# gke-my-cluster-dataplane-pool-85bedef0-pd5t
```
```bash
kubectl label node gke-my-cluster-dataplane-pool-85bedef0-8wfb label1=True
```
```bash
# node/gke-my-cluster-dataplane-pool-85bedef0-8wfb labeled
```
```bash
kubectl label node gke-my-cluster-dataplane-pool-85bedef0-pd5t label2=True
```
```bash
# node/gke-my-cluster-dataplane-pool-85bedef0-pd5t labeled
```
```bash
kubectl apply -f dep-label1.yaml -f dep-label2.yaml
```
```bash
kubectl get pods,deployments -n my-ns
```
```bash
# NAME                         READY   STATUS    RESTARTS   AGE
# pod/dep-1-5c4dcfb84d-29jz4   1/1     Running   0          47s
# pod/dep-2-858b96f697-7kmxr   1/1     Running   0          47s A
# NAME                    READY   UP-TO-DATE   AVAILABLE   AGE
# deployment.apps/dep-1   1/1     1            1           48s
# deployment.apps/dep-2   1/1     1            1           47s
```
```bash
kubectl exec -it -n my-ns pod/dep-1-5c4dcfb84d-29jz4 -- /bin/sh
```
```bash
cd usr/myshare/
```
```bash
echo "hello from node 1" > hello.txt
```
```bash
cat hello.txt
```
```bash
# hello from node 1
```
```bash
kubectl exec -it -n my-ns pod/dep-2-858b96f697-7kmxr -- /bin/sh
```
```bash
cd usr/myshare
```
```bash
cat hello.txt 
```
```bash
# hello from node 1
```