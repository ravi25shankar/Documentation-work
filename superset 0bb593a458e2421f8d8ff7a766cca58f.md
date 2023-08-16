# Superset

## 1. Task requirement

To run the **superset** with a **podman play kube** which is generated with the helm chart of the
superset.

## What is **Superset ?**

Superset is a modern data exploration and data visualisation platform. Superset can replace
or augment proprietary business intelligence tools for many teams. Superset integrates well
with a variety of data sources. Superset provides: A no-code interface for building charts
quickly.

## What is podman play kube ?

Podman play kube will be read in a structured file of Kubernetes YAML. It will then recreate
the containers, pods or volumes described in the YAML.

# 2 . Environment Details

OS ubuntu - 20.04

# 3 . List of tools and technologies

Podman version 3.4.2

Helm version 3.12.1 

### other :

vim 

Bash

# 4 . Definition of tools

Podman - It is an open source tool for developing, managing, and running containers
on your Linux systems.

Helm - It is a tool that streamlines installing and managing Kubernetes applications.

### other :

Vim - It is a highly configurable text editor built to make creating and changing any
kind of text very efficient .

Bash - It is the command line shell that you encounter when you open the terminal on
most Unix operating systems, like MacOS and Linux.

# 5. Command for the setup or configuration

## Download Helm from below commands on ubuntu

****************Step 1:**************** 

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
```

- **`curl https://baltocdn.com/helm/signing.asc`**: This uses the **`curl`** command-line tool to download the contents of the specified URL, which appears to be the GPG (GNU Privacy Guard) signing key for Helm, a package manager for Kubernetes.

- **`gpg --dearmor`**: This uses the **`gpg`** command to dearmor the GPG key. In GPG, "dearmor" means to convert a GPG public or private key from the binary format to a text-based format. This is often done for distribution or storage purposes.

- **`sudo tee /usr/share/keyrings/helm.gpg > /dev/null`**: This part of the command uses **`tee`** to write the output of the previous **`gpg`** command to the file **`/usr/share/keyrings/helm.gpg`**.
- The **`> /dev/null`** part at the end redirects the standard output (stdout) to the "null" device .

****************Step 2:**************** 

```bash
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
```

- This line tells the apt package manager to download Helm packages from the `https://baltocdn.com/helm/stable/debian/` repository.
- The `arch=$(dpkg --print-architecture)` part tells the apt package manager to download the correct package for your system architecture.
- The `signed-by=/usr/share/keyrings/helm.gpg` part tells the apt package manager to verify the authenticity of the packages using the GPG key that is stored in the file `/usr/share/keyrings/helm.gpg`.
- Creates a new file called `helm-stable-debian.list` in the directory `/etc/apt/sources.list.d/`.
- It adds the following line to the file:
deb [arch=(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main

****************Step 3: update**************** 

```bash
sudo apt update

```

**Step 4 : install helm**

```bash
sudo apt install helm
```
![image](https://github.com/ravi25shankar/Documentation-work/assets/141721174/c1f20af9-8316-46cc-a9f4-80dc240c70e6)


**Step 5 : create directory** 

```bash
mkdir superset-poc
```
```
cd superset-poc
```


**Step 6 : create yaml file  and add details in it** 

```bash
vim value-stage.yaml
```

```yaml
global:
  hosts:
    domain: example.com
postgresql:
  image:
    tag: 14.0
certmanager-issuer:
 email: me@example.com
```

**Step 7 : Generate Kubernetes manifests for deploying the Superset application.**

```bash
helm template -install podman-dry-run --debug superset/superset --generate-name --values values-stage.yaml > superset-kube.yaml
```

- `helm template`: This is the command that tells Helm to generate Kubernetes manifests based on a Helm chart.
- `--dry-run`: This  indicates  Helm will simulate the installation without actually deploying anything. This is useful for seeing what manifests would be generated without affecting the cluster.
- `--debug`: This enables debug output, providing more detailed information about the Helm template process.
- `superset/superset`: This specifies the Helm chart to use for generating the Kubernetes manifests. The chart name is in the format `repository/chart-name`.
- `--generate-name`: This generates a unique name for the release based on the chart's name. It's used when you want to create a new release without specifying a release name.
- `--values values-stage.yaml`: This  points to a values file (`values-stage.yaml`) that provides custom configuration values for the Helm chart.

- `> superset-kube.yaml`: This part of the command uses the output redirection (`>`) to save the generated Kubernetes manifests to a file named `superset-kube.yaml`.

In summary, this command generates Kubernetes manifests for deploying the Superset application using a specific Helm chart and custom configuration values from the `values-stage.yaml` file. The generated manifests are saved in the `superset-kube.yaml` file. The use of `--dry-run` and `--debug` ensures that we can preview the manifests before actually deploying the application, it helps to  verify that the configuration is correct.

![image](https://github.com/ravi25shankar/Documentation-work/assets/141721174/2fa222fc-f534-488e-a9de-aee30442ee3a)


**Step 8 : # Edit this  superset-kube.yaml and create a new final.yaml as per the requirement in podman play kube.**  


Add the below data -

```bash
cat superset-kube.yaml    # copy all data
```

```bash
vim final.yaml           # paste copied data
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubepostgresql
spec:
  replicas: 1
  template:
	metadata:
  	name: kubepostgresql
	spec:
  	serviceAccountName: default
  	securityContext:
    	fsGroup: 1001
  	hostNetwork: true
  	hostIPC: false
  	initContainers:
  	containers:
    	- name: postgresql
      	image: docker.io/bitnami/postgresql:14.6.0-debian-11-r13
      	imagePullPolicy: "IfNotPresent"
      	securityContext:
        	runAsUser: 1001
      	env:
        	- name: BITNAMI_DEBUG
          	value: "false"
        	- name: POSTGRESQL_PORT_NUMBER
          	value: "5432"
        	- name: POSTGRESQL_VOLUME_DIR
          	value: "/bitnami/postgresql"
        	- name: PGDATA
          	value: "/bitnami/postgresql/data"
        	- name: POSTGRES_USER
          	value: "superset"
        	- name: POSTGRES_POSTGRES_PASSWORD
          	value: superset
        	- name: POSTGRES_PASSWORD
          	value: superset
        	- name: POSTGRES_DB
          	value: "superset"
        	- name: POSTGRESQL_ENABLE_LDAP
          	value: "no"
        	- name: POSTGRESQL_ENABLE_TLS
          	value: "no"
        	- name: POSTGRESQL_LOG_HOSTNAME
          	value: "false"
        	- name: POSTGRESQL_LOG_CONNECTIONS
          	value: "false"
        	- name: POSTGRESQL_LOG_DISCONNECTIONS
          	value: "false"
        	- name: POSTGRESQL_PGAUDIT_LOG_CATALOG
          	value: "off"
        	# Others
        	- name: POSTGRESQL_CLIENT_MIN_MESSAGES
          	value: "error"
        	- name: POSTGRESQL_SHARED_PRELOAD_LIBRARIES
          	value: "pgaudit"
      	ports:
        	- name: tcp-postgresql
          	containerPort: 5432
      	resources:
        	limits: {}
        	requests:
          	cpu: 250m
          	memory: 256Mi
      	volumeMounts:
        	- name: data
          	mountPath: /bitnami/postgresql
  	volumes:
    	- name: data
      	hostPath:
        	path: ./postgres/
        	type: Directory
---
# Source: superset/charts/redis/templates/master/application.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kuberedis-master
spec:
  replicas: 1
  updateStrategy:
	type: RollingUpdate
  template:
	spec:
  	securityContext:
    	fsGroup: 1001
  	terminationGracePeriodSeconds: 30
  	containers:
    	- name: redis
      	image: docker.io/bitnami/redis:7.0.10-debian-11-r4
      	imagePullPolicy: "IfNotPresent"
      	securityContext:
        	runAsUser: 1001
      	command:
        	- /bin/bash
      	args:
        	- -c
        	- /opt/bitnami/scripts/start-scripts/start-master.sh
      	env:
        	- name: BITNAMI_DEBUG
          	value: "false"
        	- name: REDIS_REPLICATION_MODE
          	value: master
        	- name: ALLOW_EMPTY_PASSWORD
          	value: "yes"
        	- name: REDIS_TLS_ENABLED
          	value: "no"
        	- name: REDIS_PORT
          	value: "6379"
      	ports:
        	- name: redis
          	containerPort: 6379
          	hostPort: 6379
      	resources:
        	limits: {}
        	requests: {}
      	volumeMounts:
        	- name: start-scripts
          	mountPath: /opt/bitnami/scripts/start-scripts/start-master.sh
          	subPath: start-master.sh
        	- name: redis-data
          	mountPath: /data
        	- name: config
          	mountPath: /opt/bitnami/redis/mounted-etc
  	volumes:
    	- name: start-scripts
      	hostPath:
        	path: /home/harsh/superset/redis/start-master.sh
        	type: FileOrCreate
        	defaultMode: 0777
    	- name: config
      	hostPath:
        	path: ./redis/redis-conf
        	type: Directory
    	- name: redis-data
      	hostPath:
        	path: ./redis/redis-data
        	type: Directory
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubesuperset-worker
spec:
  replicas: 1
  template:
	spec:
  	securityContext:
    	runAsUser: 0
  	initContainers:
  	- command:
    	- /bin/sh
    	- -c
    	- dockerize -wait "tcp://$DB_HOST:$DB_PORT" -wait "tcp://$REDIS_HOST:$REDIS_PORT"
      	-timeout 120s
    	env:
      	- name: REDIS_HOST
        	value: 192.168.122.1
      	- name: REDIS_PORT
        	value: 6379
      	- name: DB_HOST
        	value: 192.168.122.1
      	- name: DB_PORT
        	value: 5432
      	- name: DB_USER
        	value: superset
      	- name: DB_PASS
        	value: superset
      	- name: DB_NAME
        	value: superset
    	image: 'apache/superset:dockerize'
    	imagePullPolicy: 'IfNotPresent'
    	name: wait-for-postgres-redis
  	containers:
    	- name: superset
      	image: "apachesuperset.docker.scarf.sh/apache/superset:2.1.0"
      	imagePullPolicy: IfNotPresent
      	command: ["/bin/sh","-c",". /app/pythonpath/superset_bootstrap.sh; celery --app=superset.tasks.celery_app:app worker"]
      	env:
        	- name: "SUPERSET_PORT"
          	value: "8088"
        	- name: REDIS_HOST
          	value: 192.168.122.1
        	- name: REDIS_PORT
          	value: 6379
        	- name: DB_HOST
          	value: 192.168.122.1
        	- name: DB_PORT
          	value: 5432
        	- name: DB_USER
          	value: superset
        	- name: DB_PASS
          	value: superset
        	- name: DB_NAME
          	value: superset
      	volumeMounts:
        	- name: superset-config
          	mountPath: "/app/pythonpath"
          	readOnly: true
      	livenessProbe:
        	exec:
          	command:
          	- sh
          	- -c
          	- celery -A superset.tasks.celery_app:app inspect ping -d celery@$HOSTNAME
        	failureThreshold: 3
        	initialDelaySeconds: 120
        	periodSeconds: 60
        	successThreshold: 1
        	timeoutSeconds: 60
      	resources:
        	{}
  	volumes:
    	- name: superset-config
      	hostPath:
        	path: ./superset/
        	type: Directory
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubesuperset
spec:
  replicas: 1
  template:
	spec:
  	securityContext:
    	runAsUser: 0
  	initContainers:
  	- command:
    	- /bin/sh
    	- -c
    	- dockerize -wait "tcp://$DB_HOST:$DB_PORT" -timeout 120s
    	env:
      	- name: REDIS_HOST
        	value: 192.168.122.1
      	- name: REDIS_PORT
        	value: 6379
      	- name: DB_HOST
        	value: 192.168.122.1
      	- name: DB_PORT
        	value: 5432
      	- name: DB_USER
        	value: superset
      	- name: DB_PASS
        	value: superset
      	- name: DB_NAME
        	value: superset
    	image: 'apache/superset:dockerize'
    	imagePullPolicy: 'IfNotPresent'
    	name: wait-for-postgres
  	containers:
    	- name: superset
      	image: "apachesuperset.docker.scarf.sh/apache/superset:2.1.0"
      	imagePullPolicy: IfNotPresent
      	command: ["/bin/sh","-c",". /app/pythonpath/superset_bootstrap.sh; /usr/bin/run-server.sh"]
      	env:
        	- name: "SUPERSET_PORT"
          	value: "8088"
        	- name: "SUPERSET_PORT"
          	value: "8088"
        	- name: REDIS_HOST
          	value: 192.168.122.1
        	- name: REDIS_PORT
          	value: 6379
        	- name: DB_HOST
          	value: 192.168.122.1
        	- name: DB_PORT
          	value: 5432
        	- name: DB_USER
          	value: superset
        	- name: DB_PASS
          	value: superset
        	- name: DB_NAME
          	value: superset
      	volumeMounts:
        	- name: superset-config
          	mountPath: "/app/pythonpath"
          	readOnly: true
      	ports:
        	- name: http
          	containerPort: 8088
          	hostPort: 8088
          	protocol: TCP
      	resources:
        	{}
  	volumes:
    	- name: superset-config
      	hostPath:
        	path: ./superset/
        	type: Directory
```

**Step 9 : Check list of file**

```bash
ls -lrth            **# it  will list all of the files in the current directory .**
```

**Step 10 :  create a script file and add details in it .**

```bash
vim pre-install-task.sh    
```

```bash
#!/bin/bash

mkdir superset
mkdir redis
mkdir postgres

cat <<- END > superset/superset_bootstrap.sh
	#!/bin/bash
	if [ ! -f ~/bootstrap ]; then echo "Running Superset with uid 0" > ~/bootstrap; fi

END

cat <<- END > superset/superset_config.py
import os
from cachelib.redis import RedisCache
    
def env(key, default=None):
	return os.getenv(key, default)
    
MAPBOX_API_KEY = env('MAPBOX_API_KEY', '')
CACHE_CONFIG = {
    	'CACHE_TYPE': 'RedisCache',
    	'CACHE_DEFAULT_TIMEOUT': 300,
    	'CACHE_KEY_PREFIX': 'superset_',
    	'CACHE_REDIS_HOST': env('REDIS_HOST'),
    	'CACHE_REDIS_PORT': env('REDIS_PORT'),
    	'CACHE_REDIS_PASSWORD': env('REDIS_PASSWORD'),
    	'CACHE_REDIS_DB': env('REDIS_DB', 1),
}
DATA_CACHE_CONFIG = CACHE_CONFIG
    
SQLALCHEMY_DATABASE_URI = f"postgresql+psycopg2://{env('DB_USER')}:{env('DB_PASS')}@{env('DB_HOST')}:{env('DB_PORT')}/{env('DB_NAME')}"
SQLALCHEMY_TRACK_MODIFICATIONS = True
SECRET_KEY = env('SECRET_KEY', 'thisISaSECRET_1234')
    
class CeleryConfig(object):
	CELERY_IMPORTS = ('superset.sql_lab', )
	CELERY_ANNOTATIONS = {'tasks.add': {'rate_limit': '10/s'}}
	BROKER_URL = f"redis://{env('REDIS_HOST')}:{env('REDIS_PORT')}/0"
	CELERY_RESULT_BACKEND = f"redis://{env('REDIS_HOST')}:{env('REDIS_PORT')}/0"
    
CELERY_CONFIG = CeleryConfig
RESULTS_BACKEND = RedisCache(
    	host=env('REDIS_HOST'),
    	port=env('REDIS_PORT'),
    	key_prefix='superset_results'
)
END
cat <<- END > superset/superset_init.sh
#!/bin/sh
set -eu
echo "Upgrading DB schema..."
superset db upgrade
echo "Initializing roles..."
superset init
    
echo "Creating admin user..."
superset fab create-admin \
           	--username admin \
                	--firstname Superset \
                	--lastname Admin \
                	--email admin@superset.com \
                	--password admin \
                	|| true
    
if [ -f "/app/configs/import_datasources.yaml" ]; then
   echo "Importing database connections.... "
   superset import_datasources -p /app/configs/import_datasources.yaml
fi
END
mkdir redis/redis-conf
cat <<- END > redis/redis-conf/master.conf
	dir /data
	# User-supplied master configuration:
	rename-command FLUSHDB ""
	rename-command FLUSHALL ""
	# End of master configuration
END

cat <<- END > redis/redis-conf/redis.conf
	appendonly yes
	save ""
END

cat <<- END > redis/redis-conf/replica.conf
    	dir /data
	rename-command FLUSHDB ""
	rename-command FLUSHALL ""
END

mkdir -p redis/redis-data
cat <<- END > redis/master-redis.sh
	#!/bin/bash
	[[ -f \$REDIS_PASSWORD_FILE ]] && export REDIS_PASSWORD="\$(< "\${REDIS_PASSWORD_FILE}")"
	if [[ -f /opt/bitnami/redis/mounted-etc/master.conf ]];then
    	cp /opt/bitnami/redis/mounted-etc/master.conf /opt/bitnami/redis/etc/master.conf
	fi
	if [[ -f /opt/bitnami/redis/mounted-etc/redis.conf ]];then
    	cp /opt/bitnami/redis/mounted-etc/redis.conf /opt/bitnami/redis/etc/redis.conf
	fi
	ARGS=("--port" "\${REDIS_PORT}")
	ARGS+=("--protected-mode" "no")
	ARGS+=("--include" "/opt/bitnami/redis/etc/redis.conf")
	ARGS+=("--include" "/opt/bitnami/redis/etc/master.conf")
	exec redis-server "\${ARGS[@]}"
END
chmod +x superset/superset_init.sh
chmod +x superset/superset_config.py
chmod +x superset/superset_bootstrap.sh
```

```bash
chmod +x pre-install-task.sh
```

- chmod : change the permissions of a file or directory.
- +x  : to add the execute permission to the file or directory.

**step 11 : Run the script** 

```bash
sh pre-install-task.sh
```

```bash
ls -lrth    # list all of the files in the current directory,
```
![image](https://github.com/ravi25shankar/Documentation-work/assets/141721174/a5aeab49-b7c2-495b-a972-ffc1a131dc2d)

**Step 12 : install podman** 

```bash
sudo apt install podman    # install podman       
```

       

**Step 13 : create the pods and containers** 

```bash
podman play kube final.yaml
```

- This command will create the pods and containers described in the file `final.yaml`. The containers within the pod will then be started, and the ID of the new Pod will be output.

![image](https://github.com/ravi25shankar/Documentation-work/assets/141721174/58fb7279-c062-4b11-8b17-204b0651a5d5)


**Step 13 : Test case list**  
![image](https://github.com/ravi25shankar/Documentation-work/assets/141721174/e7496bf1-9876-4243-9cb0-81615fd370a2)

**Screenshot with pass test cases result**

**Go to the browser.**
![image](https://github.com/ravi25shankar/Documentation-work/assets/141721174/eaaf3cbb-cea7-43ab-a0a4-d44ac39e8a31)



**Login with default credentials username: admin password: admin**
superset-kube.yaml
![image](https://github.com/ravi25shankar/Documentation-work/assets/141721174/e3820a18-bc22-4124-b4c7-a933a6434fc0)


## Reference link

**https://helm.sh/docs/intro/install/**  

**https://docs.podman.io/en/v4.2/markdown/podman-play-kube.1.html**

