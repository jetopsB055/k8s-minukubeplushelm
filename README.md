# k8sreadme
# k8sreadme for minikube installation and cluster setup

=============================================================
**Minikube Installation**
sudo apt update
sudo apt install docker.io
sudo snap install kubectl --classic
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
adduser newUser
usermod -aG sudo newUser
sudo usermod -aG docker $USER
su - newUser
minikube start
minikube status
kubectl version

** If you would like to setup a dashboard for the minikube, you would need to execute**
these commands, ensure you do it when you have a GUI based server available.
kubectl get-nodes
minikube addons list
minikube addons enable dashboard - http://127.0.0.0
===============================================================
**Helm Installation:**
sudo apt update
sudo apt install curl
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 > get_helm.sh
chmod +x get_helm.sh
sudo ./get_helm.sh
helm version
================================================================
**output of helm install my-redis bitnami/redis**

** Please be patient while the chart is being deployed **
Redis&reg; can be accessed on the following DNS names from within your cluster:
my-redis-master.default.svc.cluster.local for read/write operations (port 6379)
my-redis-replicas.default.svc.cluster.local for read-only operations (port 6379)
To get your password run:
export REDIS_PASSWORD=$(kubectl get secret --namespace default my-redis -o jsonpath="
{.data.redis-password}" | base64 -d) zyVviGViON
To connect to your Redis&reg; server:
1. Run a Redis&reg; pod that you can use as a client:
kubectl run --namespace default redis-client --restart='Never' --env REDIS_PASSWORD=$R
EDIS_PASSWORD --image docker.io/bitnami/redis:7.0.11-debian-11-r7 --command -- sleep infi
nity
Use the following command to attach to the pod:
kubectl exec --tty -i redis-client \
--namespace default -- bash
2. Connect using the Redis&reg; CLI:
REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h my-redis-master
REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h my-redis-replicas
To connect to your database from outside the cluster execute the following commands:
kubectl port-forward --namespace default svc/my-redis-master 6379:6379 &
REDISCLI_AUTH="$REDIS_PASSWORD" redis-cli -h 127.0.0.1 -p 6379
================================================================
**Deploy redis server helm chart on Minikube cluster:**
Setup a Redis Helm repository:
helm repo add bitnami https://charts.bitnami.com/bitnami
Deploy the Redis server:
helm install my-redis bitnami/redis
Verify the deployment:
K8S Assignment 3
kubectl get pods
Run the following command to create a local port forwarding to the Redis service:
kubectl port-forward svc/my-redis-master 6379:6379
You can connect to the Redis server using any Redis client.
redis-cli
**** To uninstall the Redis deployment, y
helm delete my-redis
================================================================
Deployment_with_a_single_container_allowing_it_to_connect_to_a_Redis_server:
Initialize a Helm Chart:
helm create my-redis-app
Modify the Chart Configuration: Open the values.yaml file within the my-redis-app
directory.
redis:
host: redis-service
port: 6379
Modify the Deployment Template: Open the templates/deployment.yaml file.
Replace the contents of the file with the following YAML definition:
apiVersion: apps/v1
kind: Deployment
metadata:
name: {{ include "my-redis-app.fullname" . }}
labels:
app.kubernetes.io/name: {{ include "my-redis-app.name" . }}
helm.sh/chart: {{ include "my-redis-app.chart" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
spec:
replicas: 1
selector:
matchLabels:
app.kubernetes.io/name: {{ include "my-redis-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
K8S Assignment 4
template:
metadata:
labels:
app.kubernetes.io/name: {{ include "my-redis-app.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
spec:
containers:
- name: my-redis-app
image: redis
ports:
- containerPort: 6379
command:
- "sleep"
- "infinity"
env:
- name: REDIS_HOST
value: {{ .Values.redis.host }}
- name: REDIS_PORT
value: "{{ .Values.redis.port }}"
resources: {}
Install the Helm Chart:
helm install my-redis-app ./my-redis-app
export POD_NAME=$(kubectl get pods --namespace default -l
"app.kubernetes.io/name=my-redis-app,app.kubernetes.io/instance=my-redis-app" -
o jsonpath="{.items[0].metadata.name}")
export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o
jsonpath="{.spec.containers[0].ports[0].containerPort}")
echo "Visit http://127.0.0.1:8080 to use your application"
kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
Connect to the Container:
kubectl exec -it <pod-name> -- /bin/bash
example: kubectl exec -it <update the master name> -- /bin/bash
kubectl exec -it <update the replica name> -- /bin/bash
Run Redis Commands:
 Once you are inside the container's shell, you can use the redis-cli command
K8S Assignment 5
 to execute Redis commands. For example, you can run the following command to 
 set a Redis key- value pair:
redis-cli SET mykey "Hello, World!”
You can now login to Master or the Replica pods and can validate the key values
redis-cli GET mykey
I have set the myname value to “Redis-key”
===============================================================
Script to “exec” to the deployment and set a new key in the redis server (OxKey)
with a
value (OxValue)
To execute a command within the container of the deployed Redis deployment and
set a new key-value pair in the Redis server, you can use the following script:
#!/bin/bash
# Get the pod name for the Redis deployment
POD_NAME=$(kubectl get pods -l app=my-redis-app -o jsonpath="{.items[0].metadata.name}")
# Execute the Redis command within the container
kubectl exec -it "$POD_NAME" -- redis-cli SET OxKey OxValue
echo "New key-value pair set in Redis server."
Save the script to a file, for example, set_redis_key.sh . Then, make it executable using
the following command:
chmod +x set_redis_key.sh
To run the script and set the new key-value pair in the Redis server, execute the
following command:
./set_redis_key.sh
The script uses kubectl to retrieve the pod name for the Redis deployment labeled as
app=my-redis-app . It then executes the redis-cli SET command within the container,
Script to “exec” to the deployment and set a new key in the redis server (OxKey) with a
value (OxValue) 2 setting the key "OxKey" with the value "OxValue" in the Redis server.
Finally, it displays
a message indicating the successful setting of the key-value pair.
K8S Assignment 6
===============================================================
Script to “exec” to the deployment and get the value of the key (OxKey) and print
it back out
#!/bin/bash
# Get the pod name for the Redis deployment
POD_NAME=$(kubectl get pods -l app=my-redis-app -o jsonpath="{.items[0].metadata.name}")
# Execute the Redis command within the container to get the value of the key
VALUE=$(kubectl exec -it "$POD_NAME" -- redis-cli GET OxKey)
# Print the value
echo "Value of OxKey: $VALUE"
Save the script to a file, for example, get_redis_key.sh . Then, make it executable using
the following command:
chmod +x get_redis_key.sh
To run the script and retrieve the value of the key from the Redis server, execute the
following command:
./get_redis_key.sh
The script uses kubectl to retrieve the pod name for the Redis deployment labeled as
app=my-redis-app . It then executes the redis-cli GET command within the container,
retrieving the value of the key "OxKey" from the Redis server. Finally, it prints out the
retrieved value
==============================================================
