# Before to start
	Review "vi/vim" editor shortcuts 
	Revisar la Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
	kubectl config get-contexts
	k  get all
	k api-resources.   Muestra el listao de recuros incluso con una columan de la versión de cada uno de ellos. Util para la actualizacion de recursos (o yaml de los mismos). Asi mismo el uso de "k explain <RECURSO>"  (ej.: k explain deploy)
	Revisar el parámetro --schedule de los cronjob (minute, day, day of month, month, day of week)
	Obtener el yaml de un recurso ya generado excluyendo todo aquello no relevante:
		 kubectl get serviceaccount backend-team -o yaml
	Buscar entre todos los pods (en formato yaml) la existencia de pod con nombre XXX
		k -n saturn get pod -o yaml | grep XXX -A10
	"port" es el visible hacia el exterior del pod/deploy/service/…. !!!!! "targetPort" es el interno (80 en este caso) y se especifica antes con el formato por ejemplo: 3333:80.  TargetPort (80) debe coincidir por containerPort (80) para poder hacer la comunicación interna. Y 3333 es la comunicación externa.
	Al hacer un script con "kubectl" incluir siempre el namespace aunque sea "default", puesto que el mismo podría ejecutarse desde cualquier otro namespace y no obtener el resultado esperado.
	Si nos posicionamos en alguna carpeta concreta con cd /xxx/xxx/ejercicio1, al cambiar de ejercicio recordar volver a la ruta original (o de usuario) ej.: cd ~
	En los NetworkPolicy, ojo para un Ingress o un Egress, se especifican diferenes reclas con un "-" lo que implica un OR, es decir o una cosa o la otra, en caso contrario se interpreta como un AND en cuyo caso estaría indentado (y sin "-"), perteneciendo a un bloque superior del yaml.  --> Ejercicio 20. TestKiller !
	Si hay que exponer un Deployment a traves de un servicio:
		Crear el Deploy: "k create deploy xxx --image=  ……" -n <NAMESPACE>
		Utilizar a continuación "k expose --port=3333--target-port=80 -n <NAPESPACE>" 
	kubectl run nginx --image=nginx --restart=Never --env=var1=val1
	ResourceQuouta: Asociada al namespace sobre el que se crea. "k create quota myrq  --hard=cpu=1,memory=1G,pods=2 --dry-run -o yaml"
	To terminate a JOB: 
		activeDeadlineSeconds:  n.  -> when it's successfully started but take more than n  seconds to complete execution
		backOffLimit: n. --> Number or retries after fail
		startingDeadlineSeconds: n. -> when job take more than n seconds 
	Al crear un ServiceAccount se crea automáticamente un Secret con un Token
	En el pod se puede especificar tanto el atributo serviceAccount como el serviceAccountName, para que dicho pod tal Service Account.
	Para Depurar: Si el pod se ha iniciado usar "logs" y, sino hay logs (es que el pod no se ha iniciado), usar "describe" en su lugar.
	"expose" . Crear un Servicio a partir de un deploy o un pod. Cuando se crear un servicio a partir de deployment se especifica:
	    --port y 
	    --target-port
	Comando "cut" -f -> (field) -d (delimitador). Ej: cat xxxx\yyy\source.txt | cut -f 1 -d ":" > xxx\yyy\target.txt.  ---> crea un fichero target.txt con el resultado de la primera columna(field) del fichero source.txt
	Comando "awk" -F "<separador>" '{print $<N>}' -> N: número columna. 
	kubectl get-context // kubectl use-context xxxx // kubectl set-context --current --namespace xx
	A ReadinessProbe will be executed periodically all the time, not just during start or until a Pod is ready
	CRD (Custom Resources Definition) : k get crd 
	Get a list of all DbBackup objects : k get db-backups -A


## CORE
	Create a temporary (--rm) pod tu run "env"
	Note: --restart=Never is necesary to avoid continuous restarting implies "env" doesn't executed
	
	Get name and namespaces values for a pod
		○ [Option1] k run busybox --image=busybox --dry-run=client --restart=Never -n ns -o jsonpath="{['.metadata.name','.metadata.namespace']}"
		○ [Option 2] k get pods -o jsonpath="{.items[*]['metadata.name','metadata.namespace']}" --all-namespaces
		○ k get secret neptune-sa-v2-token-2mwl9  -n neptune -o jsonpath='{.data.token}{"\n"}'.   // Last {"\n"} remove char % at the end of the line.

	## LABELS & ANNOTATIONS
		○ nodeSelector: Include in pod at "containers"  level
	
		### Annotations
			§ k annotate po nginx{1..3} --list
			§ k describe po nginx{1..3} | grep -<N> -i annotation.    
				□  -i = ignore case. 
				□ - <N> -> num. lines before and after  
				□ -A <N>  --> lines after
				□ -B <N>  --> lines before
		○ Listado especificos
			§ kubectl get pods --sort-by=.metadata.name
			§ kubectl get po -o=custom-columns="POD_NAME:.metadata.name, POD_STATUS:.status.containerStatuses[].state"
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
		○ The cron job should be terminated if it takes more than X seconds to start execution after its scheduled time
			§ startingDeadlineSeconds: x
		
		○ # ┌───────────── minute (0 - 59)
# │ ┌───────────── hour (0 - 23)
# │ │ ┌───────────── day of the month (1 - 31)
# │ │ │ ┌───────────── month (1 - 12)
# │ │ │ │ ┌───────────── day of the week (0 - 6) (Sunday to Saturday;
# │ │ │ │ │                                   7 is also Sunday on some systems)
# │ │ │ │ │                                   OR sun, mon, tue, wed, thu, fri, sat
# │ │ │ │ │
# * * * * *

	## CONFIG MAGPS
		○ When several literals, we have to use --from-literal, for each one
		○ Create and display a configmap from a file, giving the key 'special'
			§ k create cm config2 --from-file=special=config.txt --dry-run=client -o yaml
		○ When a pod create some keys from a config map
			env:
			    - name: option # name of the env variable
			      valueFrom:
			        configMapKeyRef:
			          name: options # name of config map
			          key: var5 # name of the entity in config map
		○ When a pod create all keys from a config map:
			envFrom: # different than previous one, that was 'env'
			    - configMapRef: # different from the previous one, was 'configMapKeyRef'
			        name: anotherone # the name of the config map
		○ When a pod create a volume from a config map:
		  volumes: # add a volumes list
		  - name: myvolume # just a name, you'll reference this in the pods
		    configMap:
		      name: cmvolume # name of your configmap
	## SECURITY CONTEXT
		spec:
		  securityContext: # insert this line
		    runAsUser: 101 # UID for the user
		
		    securityContext:
		      capabilities:
		        add: ["NET_ADMIN", "SYS_TIME"]
	
	## REQUESTS AND LIMIT
		○ [DEPRECATED] kubectl run nginx --image=nginx --restart=Never --requests='cpu=100m,memory=256Mi' --limits='cpu=200m,memory=512Mi'
		○ k run nginx --image=nginx --restart=Never --dry-run=client -o yaml | kubectl set resources -f - --requests=cpu=100m,memory=256Mi --limits=cpu=200m,memory=512Mi --local -o yaml
	

	## SECRETS
		○ kubectl create secret generic mysecret --from-literal=password=mypass
	
	  volumes: # specify the volumes
	  - name: foo # this name will be used for reference inside the container
	    secret: # we want a secret
	      secretName: mysecret2 # name of the secret - this must already exist on pod creation
	
	    ---
	
	    env:
	      - name: SECRET_USERNAME
	        valueFrom:
	          secretKeyRef:
	            name: mysecret
	            key: username
	            optional: false # same as default; "mysecret" must exist
	                            # and include a key named "username"
	
	## SERVICE ACCOUNT
		○ New service account:
			§ kubectl get sa default -o yaml > sa.yaml
		○ In the pod:
			§ serviceAccountName: myuser # we use pod.spec.serviceAccountName
		○ Print out the token of the service account
		kubectl exec -it backend -- /bin/sh
		cat /var/run/secrets/kubernetes.io/serviceaccount/token
	
  ## BSERVAVILITY
	   livenessProbe:
	      exec:
	        command:
	        - cat
	        - /tmp/healthy
	      initialDelaySeconds: 5
	      periodSeconds: 5
	
	Lots of pods are running in qa,alan,test,production namespaces. All of these pods are configured with liveness probe. Please list all pods whose liveness probe are failed in the format of <namespace>/<pod name> per line.
	
		○ kubectl get ns # check namespaces
		○ kubectl -n qa get events | grep -i "Liveness probe failed"
		○ kubectl -n alan get events | grep -i "Liveness probe failed"
		○ kubectl -n test get events | grep -i "Liveness probe failed"
		○ kubectl -n production get events | grep -i "Liveness probe failed"
	
	
	Get CPU/memory utilization for nodes (metrics-server must be running)
		○ kubectl top nodes
	
	
	## SERVICES AND NETWORKING
	
		○ kubectl run nginx --image=nginx --restart=Never --port=80 --expose
		# observe that a pod as well as a service are created
		
		○ IMPORTANTE A un Servicio con NodePort, se puede acceder desde el PC indicando el puerto (posiblemente NO el 80 que es el usado internamente)
	
	
	Create service
		○ Option 1: kubectl expose deploy foo --port=6262 --target-port=8080
		○ Option 2: k create svc clusterip foo --tcp=6262:8080 
		
## HELM
		• helm create chart-test 	• this would create a helm 
		• helm install -f myvalues.yaml my redis ./redis 	• Running helm chart
		• helm list --pending -A	• Find pending helm deployment on all namespaces
		• helm list -n <NAMESPACE> -a	Show all to find and delete the broken release
		• helm uninstall -n namespace release_name	• Uninstall a release
		• helm upgrade -f myvalues.yaml -f override.yaml redis ./redis	• Upgrading helm chart 
		• helm repo add [NAME] [URL]  [flags]	• Add, list, remove, update and index chart repos
		• helm repo list / helm repo ls	        
		• helm repo remove [REPO1] [flags]	        
		• helm repo update / helm repo up
		• helm repo update [REPO1] [flags]
		• helm repo index [DIR] [flags]
		• helm pull [chart URL | repo/chartname] [...] [flags] # this would download a helm, not install 	Download a Helm chart from a repository
		• helm pull --untar [rep/chartname] # untar the chart after downloading it 
		• ➜ helm -n mercury install internal-issue-report-apache bitnami/apache --set replicaCount=2	Set replicas to 2
		heml show values [Chart] [flags]	
		
		Nota: Execute 'helm repo update' previously to upgrade a chart to a new version
	○ 
