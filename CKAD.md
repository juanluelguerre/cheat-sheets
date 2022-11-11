# CKAD - Certification Kubernetes Application Development

## Before to start
Review "vi/vim" editor shortcuts 
Revisar la Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
kubectl config get-contexts
- k  get all
- k api-resources.   Muestra el listao de recuros incluso con una columan de la versión de cada uno de ellos. Util para la actualizacion de recursos (o yaml de los mismos). Asi mismo el uso de "k explain <RECURSO>"  (ej.: k explain deploy)
- Revisar el parámetro --schedule de los cronjob (minute, day, day of month, month, day of week)
- Obtener el yaml de un recurso ya generado excluyendo todo aquello no relevante:
   - kubectl get serviceaccount backend-team -o yaml
- Buscar entre todos los pods (en formato yaml) la existencia de pod con nombre XXX
`k -n saturn get pod -o yaml | grep XXX -A10`
- Parameter "port" es el visible hacia el exterior del pod/deploy/service/…. !!!!! "targetPort" es el interno (80 en este caso) y se especifica antes con el formato por ejemplo: 3333:80.  TargetPort (80) debe coincidir por containerPort (80) para poder hacer la comunicación interna. Y 3333 es la comunicación externa.
- Al hacer un script con "kubectl" incluir siempre el namespace aunque sea "default", puesto que el mismo podría ejecutarse desde cualquier otro namespace y no obtener el resultado esperado.
- Si nos posicionamos en alguna carpeta concreta con cd /xxx/xxx/ejercicio1, al cambiar de ejercicio recordar volver a la ruta original (o de usuario) ej.: cd ~
- En los NetworkPolicy, ojo para un Ingress o un Egress, se especifican diferenes reclas con un "-" lo que implica un OR, es decir o una cosa o la otra, en caso contrario se interpreta como un AND en cuyo caso estaría indentado (y sin "-"), perteneciendo a un bloque superior del yaml.  --> Ejercicio 20. TestKiller !
- Si hay que exponer un Deployment a traves de un servicio:
   - Crear el Deploy: `k create deploy xxx --image=  ……" -n <NAMESPACE>`
   - Utilizar a continuación `k expose --port=3333--target-port=80 -n <NAPESPACE>`
- **ResourceQuouta:** Asociada al namespace sobre el que se crea. "k create quota myrq  --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml"
- To **terminate a JOB**: 
  - activeDeadlineSeconds:  n.  -> when it's successfully started but take more than n  seconds to complete execution
  - backOffLimit: n. --> Number or retries after fail
  - startingDeadlineSeconds: n. -> when job take more than n seconds 
- Al crear un **ServiceAccount** se crea automáticamente un Secret con un Token
- En el pod se puede especificar tanto el atributo serviceAccount como el serviceAccountName, para que dicho pod tal Service Account.

- Para **Depurar**: Si el pod se ha iniciado usar "logs" y, sino hay logs (es que el pod no se ha iniciado), usar "describe" en su lugar.
- **expose** . Crear un Servicio a partir de un deploy o un pod. Cuando se crear un servicio a partir de deployment se especifica:
   - --port y  --target-port
- Comando `cut -f -> (field) -d (delimitador)`. Ej: `cat xxxx\yyy\source.txt` o `cut -f 1 -d ":" > xxx\yyy\target.txt`  --> crea un fichero target.txt con el resultado de la primera columna(field) del fichero source.txt
- Comando "awk" -F "<separador>" '{print $\<N\>}' -> N: número columna. 
- kubectl get-context // kubectl use-context xxxx // kubectl set-context --current --namespace xx
- A ReadinessProbe will be executed periodically all the time, not just during start or until a Pod is ready
- **CRD** (Custom Resources Definition) : k get crd 
- Get a list of all DbBackup objects : k get db-backups -A

	
## CORE
Create a temporary (--rm) pod tu run "env"
**Note:** --restart=Never is necesary to avoid continuous restarting implies "env" doesn't executed
	
Get name and namespaces values for a pod
- [Option1] k run busybox --image=busybox --dry-run=client --restart=Never -n ns -o jsonpath="{['.metadata.name','.metadata.namespace']}"
- [Option 2] k get pods -o jsonpath="{.items[*]['metadata.name','metadata.namespace']}" --all-namespaces
- k get secret neptune-sa-v2-token-2mwl9  -n neptune -o jsonpath='{.data.token}{"\n"}'.   // Last {"\n"} remove char % at the end of the line.

## LABELS & ANNOTATIONS
- nodeSelector: Include in pod at "containers"  level
	
### Annotations
- k annotate po nginx{1..3} --list
- k describe po nginx{1..3} | grep -<N> -i annotation.    
  - -i = ignore case. 
  - <N> -> num. lines before and after  
  - -A <N>  --> lines after
  - -B <N>  --> lines before

## Listado especificos
- kubectl get pods --sort-by=.metadata.name
- kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"
		○ Show Metrics for busybox's pod's containers
			§ kubectl top pod busybox --containers
		○ List pods when label is dev or prod
			§ kubectl get pods -l 'env in (dev,prod)'
		○ To be sure a pod run under an specific Cluster Node.
			§ k run xxxx --dry-run=client --restart=never > pod.yaml
			§ Edit pod.yaml and include
			spec:
			  nodeSelector:
			    nodeName: <CLUSTER-NODE-NAME>
		○ Delete all pods
			§ kubectl delete po --all
		○ Set a new image para el pod "mypod" en el namespace "ckad-prep"
			§ k set image pod mypod mypod=nginx:1.15.12 -n ckad-prep
	
		○ Get the label 'app' for the pods (show a column with APP labels)
			§ kubectl get po -L app
			or
			§ kubectl get po --label-columns=app
		○ List of annotations, en lugar de usar el "describe xxxx | grep -i yyyyy "
			§ kubectl annotate pod nginx1 --list
		○ Alternative to list annotation formatted to columns:
			§ kubectl get po nginx1 -o custom-columns=Name:metadata.name,ANNOTATIONS:metadata.annotations.description
	
## DEPLOYMENTS
		○ kubectl get rs -l run=nginx
		○ Rollouts:
			§ k rollout status deploy nginx1
			§ k rollout history deploy nginx1 --revision 2
			§ k get rs --> get all replicasets
			§ K rollout undo deploy nginx --to-revision=1
		○ Scale:
			§ k scale deploy nginx1 --replicas=5
			§ k autoscale deploy nginx1 --min=5 --max=10 --cpu-percent=80
			§ kubectl get hpa nginx.  // View Horizontal Pod Autoscaling
			§ kubectl delete hpa nginx
		
## JOBS
		○ k create job busybox --image=busybox --dry-run=client -o yaml -- /bin/sh -c 'echo hello;sleep 30;echo world' > job.yaml
			§ -- /bin/sh -c help here to avoit wait until complete creation
		○ Teminate a job if it take more than x secs
			§ activeDeadlineSeconds: x 
		○ Run a Job X times:
			§ completions : x
		○ The value of the attribute successfulJobsHistoryLimit defines how many executions are kept in the history
			§ kubectl get cronjobs current-date -o yaml | grep successfulJobsHistoryLimit:
		○ Follow the logs in a pod/job or whatever other resource
			§ k logs busybox--1-2pz2c -f.  ------> "-f"
			
## CRON JOBS
- The cron job should be terminated if it takes more than X seconds to start execution after its scheduled time `startingDeadlineSeconds: X`
```
# ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# * * * * *
```
## CONFIG MAGPS
- When several literals, we have to use --from-literal, for each one
- Create and display a configmap from a file, giving the key 'special'
	§ k create cm config2 --from-file=special=config.txt --dry-run=client -o yaml
- When a pod create some keys from a config map
	env:
	    - name: option # name of the env variable
	      valueFrom:
		configMapKeyRef:
		  name: options # name of config map
		  key: var5 # name of the entity in config map
- When a pod create all keys from a config map:
	envFrom: # different than previous one, that was 'env'
	    - configMapRef: # different from the previous one, was 'configMapKeyRef'
		name: anotherone # the name of the config map
- When a pod create a volume from a config map:
  volumes: # add a volumes list
  - name: myvolume # just a name, you'll reference this in the pods
    configMap:
      name: cmvolume # name of your configmap

## SECURITY CONTEXT
```
spec:
  securityContext: # insert this line
    runAsUser: 101 # UID for the user

    securityContext:
      capabilities:
	add: ["NET_ADMIN", "SYS_TIME"]
```

## REQUESTS AND LIMIT
- [DEPRECATED] `kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'`
- `k run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi --local -o yaml`
	

## SECRETS
- kubectl create secret generic mysecret --from-literal=password=mypass

```
  volumes: # specify the volumes
  - name: foo # this name will be used for reference inside the container
    secret: # we want a secret
      secretName: mysecret2 # name of the secret - this must already exist on pod creation
```

```
env:
- name: SECRET_USERNAME
valueFrom:
  secretKeyRef:
    name: mysecret
    key: username
    optional: false # same as default; "mysecret" must exist
		    # and include a key named "username"
```
	
## SERVICE ACCOUNT
- New service account: `kubectl get sa default -o yaml > sa.yaml`
- In the pod: _serviceAccountName_: myuser # we use pod.spec.serviceAccountName
- Print out the token of the service account
`kubectl exec -it backend -- /bin/sh 'cat /var/run/secrets/kubernetes.io/serviceaccount/token'`
	
## OBSERVAVILITY
```
   livenessProbe:
      exec:
	command:
	- cat
	- /tmp/healthy
      initialDelaySeconds: 5
      periodSeconds: 5
```	
Lots of pods are running in qa,alan,test,production namespaces. All of these pods are configured with liveness probe. Please list all pods whose liveness probe are failed in the format of <namespace>/<pod name> per line.	
- kubectl get ns \# check namespaces
- kubectl -n qa get events | grep -i "Liveness probe failed"
- kubectl -n alan get events | grep -i "Liveness probe failed"
- kubectl -n test get events | grep -i "Liveness probe failed"
- kubectl -n production get events | grep -i "Liveness probe failed"
	
Get CPU/memory utilization for nodes (metrics-server must be running)
- `kubectl top nodes`	


## SERVICES AND NETWORKING
- `kubectl run nginx --image=nginx --restart=Never --port=80 --expose \# observe that a pod as well as a service are created`
- **Importante:** A un Servicio con **NodePort**, se puede acceder desde el PC indicando el puerto (posiblemente NO el 80 que es el usado internamente)
	
Create service
- Option 1: `kubectl expose deploy foo --port=6262 --target-port=8080`
- Option 2: `k create svc clusterip foo --tcp=6262:8080`

## HELM
- helm create chart-test 	• this would create a helm 
- helm install -f myvalues.yaml my redis ./redis 	• Running helm chart
- helm list --pending -A	• Find pending helm deployment on all namespaces
- helm list -n <NAMESPACE> -a	Show all to find and delete the broken release
- helm uninstall -n namespace release_name	• Uninstall a release
- helm upgrade -f myvalues.yaml -f override.yaml redis ./redis	• Upgrading helm chart 
- helm repo add [NAME] [URL]  [flags]	• Add, list, remove, update and index chart repos
- helm repo list / helm repo ls	        
- helm repo remove [REPO1] [flags]	        
- helm repo update / helm repo up
- helm repo update [REPO1] [flags]
- helm repo index [DIR] [flags]
- helm pull [chart URL | repo/chartname] [...] [flags] # this would download a helm, not install 	Download a Helm chart from a repository
- helm pull --untar [rep/chartname] # untar the chart after downloading it 
- helm -n mercury install internal-issue-report-apache bitnami/apache --set replicaCount=2	Set replicas to 2
		
- heml show values [Chart] [flags]			
**Note:** Execute 'helm repo update' previously to upgrade a chart to a new version

## Varios

`--restart=Never --rm -i --  curl -m http://xxx:<PUERTO>`. // OJO inlcuir namespace -n <xxx> o bien como sufijo de la url: http://xxx.<namespace>:<PUERTO>

**Shells**:
- IMAGE BUSYBOX: ---> SH
- IMAGE NGINX: --> BASH !

k get all -n <xxx>

**Curl with**
- Option1: `k -n pluto expose pod project-plt-6cc-api --nameproject-plt-6cc-svc --port 3333 --target-port 80`
- Option2 : `k -n pluto create service clusterip project-plt-6cc-svc --tcp3333:80 $do`

**Important:** al escribir un "exit" en la consola ---> se cierra la sessión --> hablar con el examinador

Restart un deploy: k -n moon rollout restart deploy web-moon  --> despues de hacer cambios para aplicarlos     !!!!

"Al consultar los logs hay que especificar el contenedor:  "-c xxxx"

Al incluir un comando linux ej: echo > "/…./../index.html" este path debe ser el mismo indicado en el volumen. No se puede incluir unicamente el nombre del fichero (.html)

Al tratar de identificar si un SVC seleccional (selector) o no los pods usar k get pods -o wide en la columna "SELECTOR" se puede apreciar esto y solucionar rapidamente el problema

Revisar: Understand Rolling Update Deployment including maxSurge and maxUnavailable !!!!

NodePort: Para acceder desde fuera (ej: usando curl) , conecer las Ips previamente mediante un "k get nodes -o wide". Se ejectua el curl en cada ip del cluster y así conocemos en cual de ellos funciona :) 

Nota: El type "NodePort" necesita también un número de puerto nodePort en la sección de ports del Service. 
  - Podemos hacer también un 	
        - "k -n jupiter get pod -o wide" y la columna "NODE" nos indica en el que se está ejecutando exactamente
        - O, un "k -n jupiter get pod jupiter-crew-deploy-8cdf99bc9-klwqt -o yaml | grep nodeName"

## VIM:
Create the file ~/.vimrc with the following content:
```
set tabstop=2
set expandtab
set shiftwidth=2
```
To indent multiple lines press Esc and type :set shiftwidth=2. First mark multiple lines using Shift v and the up/down keys. Then to indent the marked lines press > or <. You can then press . to repeat the action.

`~/.bashrc` o ejecutarlo directamente in linea de comandos
`alias kn='kubectl config set-context --current --namespace '`  // ojo al espacio final

Re-ejecutar el `~/.bashrc `, para aplicar cambios ====> `. ~/.bashrc`

/etc/hosts ---->  <IP> <name/domain>. Nos permitirá hacer un curo o un wget a dicho nombre/dominio en lugar de indicar la IP

	
## Varios 2
**Canary deploy**: https://phoenixnap.com/kb/kubernetes-canary-deployments
Docker image into jar
Update deployment from V1.15 -> v1.22:  --> usar k explain pods/deploy …. Para conocer la última versión, etc
Como asignar un % de memoria a un pod según el total de la memoria del cluster !!!
	k get node <NODE> -o yaml | grep -i memory 
	k get node controlplane -o jsonpath="['CPU={.status.capacity.cpu}','MEM={.status.capacity.memory}','PODs:{.status.capacity.pods}']{'\n'}"
	Ej. de salida: ['CPU=1','MEM=2035376Ki','PODs:110']
	
Si se dispone de la instalación del api de motitorización 
kubectl top nodes  /// kubectl top pod -A
Crear un Job a partir de un cronjob:  kubectl create job test-job --from=cronjob/a-cronjob


Network Policy: 
IMPORTANTE: De manera predeterminada los network policies no se encuentran activados en kubernetes y dependen de cada proveedor: Azure (AKS), …
There are existing Pods in Namespace space1 and space2 
We need a new NetworkPolicy named np that restricts all Pods in Namespace space1 to only have outgoing traffic to Pods in Namespace space2 .
The NetworkPolicy should still allow DNS traffic on port 53 TCP and UDP.
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: np
  namespace: space1
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - ports:
    - port: 53
      protocol: TCP
    - port: 53
      protocol: UDP
  - to:
     - namespaceSelector:
        matchLabels:
         kubernetes.io/metadata.name: space2

Create / Move from one namespace to another, …..
	• Change the Namespace to neptune, 
	• also remove the:
		○ status: section, 
		○ the token volume, 
		○ the token volumeMount and,
		○  the nodeName, 
	else the new Pod won't start !
```

In a POD:

```
…
spec:
      containers:
      - image: nginx:1.17.3-alpine
        name: holy-api-container
        securityContext:                            # add
          allowPrivilegeEscalation: false  # add
          privileged: false                           # add
…
```
En los servicios (SVC)
	- --port (ej: 80)--> puerto experto para usar con la ip del Endpoint (k get ep): <IP>:<PORT>
	- --target-port  (ej: 3333)-> puerto a usar como:  <nombre-servicio>:<TARGET_PORT>


Default directoty for HTML files uder nginx: /usr/share/nginx/html

Namespaces; <SERVICE_NAME>.<NAMESPACE>.svc.cluster.local

To know what is the cluster node when a pod is runing:
   k get pod test-init-container-9547bd6bf-vjgsh -o yaml | grep -i nodeName:

Ingres
	- Un ingress especifica en backend.service.port el puerto correspondiente con el targetPort del servicio
	- Fichero /etc/hots -> permite indicar <IP> <ESPACIO> <HOSTNAME> (hostname indicado en el Ingress)
	- This annotation removes the need for a trailing slash when calling urls but it is not necessary for solving this scenario
	    nginx.ingress.kubernetes.io/rewrite-target: /
	

ReadinessProbes and LivenessProbes will be executed periodically all the time.

If a StartupProbe is defined, ReadinessProbes and LivenessProbes won't be executed until the StartupProbe succeeds.

ReadinessProbe fails*: Pod won't be marked Ready and won't receive any traffic

LivenessProbe fails*: The container inside the Pod will be restarted

StartupProbe fails*: The container inside the Pod will be restarted

*fails: fails more times than configured with failureThreshold


Objetos deprecados
 k explain deploy|cronjob|….  --> se puede ver la versión y de ahí sustituir los yaml / recursos que pueda estar deprecado.

Rollout Gree-Blue
The idea is to have two Deployments running at the same time, one with the old and one with the new image.
But there is only one Service, and it only points to the Pods of one Deployment.
Once we point the Service to the Pods of the new Deployment, all new requests will hit the new image.

Pasos:
1. Crear nuevo deployment: misma etiqueta "app: <name>" y "version" pero valor "v2"
2. Una ver que todos los pods estan ejectuandose correctamente editar el SVC y cambiar de v1 a v2
3. Finalmente: kubectl scale deploy wonderful-v1 --replicas 0 
	
Rollout Canary
Pasos:
1. Crear nuevo deployment, SIN ninguna etiqueta de versión. Es decir la misma etiqueta "app" usada en el v1 (o deploy inicial)
2. El servicio siempre selecciona por dicha etiqutea
3. Crear el nuevo deploy
4. Y ajustar el escalado al deploy v1 y deploy v2


Crear un pod y expornerlo en el puerto 80:
• k run nginx --image=nginx --expose --port=80 --dry-run=client -o yaml

To watch a pod
• kubectl get po nginx -w #watch it


If pod crashed and restarted, get logs about the previous instance
• kubectl logs nginx -p
Or,
• kubectl logs nginx --previous
	
	
Deploy a pod in a particular Cluser Node
	
```
	apiVersion: v1
	kind: Pod
	metadata:
	  name: cuda-test
	spec:
	  containers:
	    - name: cuda-test
	      image: "k8s.gcr.io/cuda-vector-add:v0.1"
	  nodeSelector: # add this
	    accelerator: nvidia-tesla-p100 # the select
```

And to use Node Affinity:
```
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: acceleratoroperator: Invalues:
            - nvidia-tesla-p100containers:
    ...
```
Autoscale the deployment, pods between 5 and 10, targetting CPU utilization at 80%
	kubectl autoscale deploy nginx --min=5 --max=10 --cpu-percent=80
	# view the horizontalpodautoscalers.autoscaling for nginx
	kubectl get hpa nginx
	
Delete the deployment and the horizontal pod autoscaler you created
	kubectl delete deploy nginx
	kubectl delete hpa nginx
	#Or
	kubectl delete deploy/nginx hpa/nginx
	
	
Ejemplo para un Canary Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-v2
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
      version: v2
  template:
    metadata:
      labels:
        app: my-app
        version: v2
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        volumeMounts:
        - name: workdir
          mountPath: /usr/share/nginx/html
      initContainers:
      - name: install
        image: busybox:1.28
        command:
        - /bin/sh
        - -c
        - "echo version-1 > /work-dir/index.html" -> En el segundo deploy "echo version-2 > /work-dir/index.html"
        volumeMounts:
        - name: workdir
          mountPath: "/work-dir"
      volumes:
      - name: workdir
        emptyDir: {}
```
Nota: Mas sencillo puede ser crear la v1 con la imagen httpd:alpine y la v2 con la imagen nginx:alpine !

Experimental: Wait for a specific condition on one or many resources.
	• K wait ….

Depurar directamente el JOB en lugar del pod que genera !
	k logs job/busybox 
	
Create a JOB or CRON JOB but ensure that it will be automatically terminated by kubernetes if it takes more than 30 seconds to execute:

```
…
spec:
  activeDeadlineSeconds: 30
  template:
    metadata:
      labels:
	run: busybox
    spec:
…
```
	
The CRON JOB should be terminated if it takes more than 17 seconds to start execution after its scheduled time (i.e. the job missed its scheduled time):
```
…
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: time-limited-job
spec:
  startingDeadlineSeconds: 17 
  jobTemplate:
    metadata:
      name: time-limited-job
    spec:
      template:
…
```
	
**Important:** la sección "securityContext" puede ir a nivel de POD o bien a nivel de Container . Asegurar bien lo que se pregunta.

Woriking with colums: Lots of pods are running in qa,alan,test,production namespaces. All of these pods are configured with liveness probe. Please list all pods whose liveness probe are failed in the format of <namespace>/<pod name> per line.
	k get events -n <NAMESPACE>| grep -i 'Liveness probe failed:' | awk '{print $4}'. # repeat for each namespace

Actualizar el tipo en un servicio
	• kubectl patch svc nginx -p '{"spec":{"type":"NodePort"}}'
	
	
Version de kubernetes:
- `k version`
- `k version --short`. -> {minor}.{mayor}.{patch}

Write the Api Group of Deployments into /root/group .
- k explain deploy ->  VERSION: {group}/{version} => "apps/v1".  Por tanto, el valor del Api group es  "apps"
	

