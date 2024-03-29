# Helm Chart

## 通过Helm chart安装 hyperkuber
添加hyperkuber chart repo
```
helm repo add hyperkuber https://charts.sheencloud.com
```
查看repo是否添加成功
```
helm repo list
```
### Demo环境安装
demo环境安装，默认使用emptyDir存储
```
helm install hyperkuber hyperkuber/hyperkuber -n hyperkuber --create-namespace
```

注意：如果是Openshift平台，将Hyperkuber的ServiceAccount赋予anyuid的权限
```
oc adm policy add-scc-to-user anyuid -z hyperkuber
oc adm policy add-scc-to-user anyuid -z hyperkuber-mysql
oc adm policy add-scc-to-user anyuid -z hyperkuber-redis
```

### 生产环境安装
下载hyperkuber的chart安装包,修改values.yaml
```
# Default values for hyperkuber.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.
web:
  replicaCount: 1
  image:
    repository: sheencloud/hyperkuber-web
    pullPolicy: Always
    # Overrides the image tag whose default is the chart appVersion.
    tag: "latest"
  imagePullSecrets: []
  nameOverride: ""
  fullnameOverride: ""
  podAnnotations: {}
  podSecurityContext: {}
    # fsGroup: 2000
  securityContext: {}
    # capabilities:
    #   drop:
    #   - ALL
    # readOnlyRootFilesystem: true
    # runAsNonRoot: true
    # runAsUser: 1000
  service:
    type: ClusterIP
    port: 80
  ingress:
    ## Set to true to enable ingress record generation
    ##
    enabled: true

    ## Set this to true in order to add the corresponding annotations for cert-manager
    ##
    certManager: false

    ## When the ingress is enabled, a host pointing to this will be created
    ##
    hostname: console.hyperkuber.io

    ## Ingress annotations done as key:value pairs
    ## For a full list of possible ingress annotations, please see
    ## ref: https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md
    ##
    ## If certManager is set to true, annotation kubernetes.io/tls-acme: "true" will automatically be set
    ##
    annotations: {}


    ## If you're providing your own certificates, please use this to add the certificates as secrets
    ## key and certificate should start with -----BEGIN CERTIFICATE----- or
    ## -----BEGIN RSA PRIVATE KEY-----
    ##
    ## name should line up with a tlsSecret set further up
    ## If you're using cert-manager, this is unneeded, as it will create the secret for you if it is not set
    ##
    ## It is also possible to create and manage the certificates outside of this helm chart
    ## Please see README.md for more information
    ##
    secrets: []
    ## - name: hyperkuber.local-tls
    ##   key:
    ##   certificate:

  resources: {}
    # We usually recommend not to specify default resources and to leave this as a conscious
    # choice for the user. This also increases chances charts run on environments with little
    # resources, such as Minikube. If you do want to specify resources, uncomment the following
    # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
    # limits:
    #   cpu: 100m
    #   memory: 128Mi
    # requests:
    #   cpu: 100m
    #   memory: 128Mi

  nodeSelector: {}

  tolerations: []

  affinity: {}


server:
  replicaCount: 1
  image:
    repository: sheencloud/hyperkuber-server
    pullPolicy: Always
    # Overrides the image tag whose default is the chart appVersion.
    tag: "latest"
  imagePullSecrets: []
  nameOverride: ""
  fullnameOverride: ""
  persistence:
    enabled: false
    ## hyperkuber data Persistent Volume Storage Class
    ## If defined, storageClassName: <storageClass>
    ## If set to "-", storageClassName: "", which disables dynamic provisioning
    ## If undefined (the default) or set to null, no storageClassName spec is
    ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
    ##   GKE, AWS & OpenStack)
    ##
    # storageClass: "-"
    ##
    ## If you want to reuse an existing claim, you can pass the name of the PVC using
    ## the existingClaim variable
    # existingClaim: your-claim
    accessMode: ReadWriteOnce
    size: 20Gi
  podAnnotations: {}
  podSecurityContext: {}
    # fsGroup: 2000
  securityContext: {}
    # capabilities:
    #   drop:
    #   - ALL
    # readOnlyRootFilesystem: true
    # runAsNonRoot: true
    # runAsUser: 1000
  service:
    type: ClusterIP
    port: 8080
  # auto create tables
  db:
    migerate: true
  
  resources: {}
    # We usually recommend not to specify default resources and to leave this as a conscious
    # choice for the user. This also increases chances charts run on environments with little
    # resources, such as Minikube. If you do want to specify resources, uncomment the following
    # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
    # limits:
    #   cpu: 100m
    #   memory: 128Mi
    # requests:
    #   cpu: 100m
    #   memory: 128Mi

  nodeSelector: {}

  tolerations: []

  affinity: {}


mysql:
  ## Whether to deploy a mysql server to satisfy the applications database requirements. To use an external database set this to false and configure the externalDatabase parameters
  enabled: true
  ## https://raw.githubusercontent.com/bitnami/charts/master/bitnami/mysql/values.yaml
  primary:
    persistence:
      enabled: false
  auth:
    ## @param auth.rootPassword Password for the `root` user. Ignored if existing secret is provided
    ## ref: https://github.com/bitnami/bitnami-docker-mysql#setting-the-root-password-on-first-run
    ##
    rootPassword: "hyperkuber"
    ## @param auth.database Name for a custom database to create
    ## ref: https://github.com/bitnami/bitnami-docker-mysql/blob/master/README.md#creating-a-database-on-first-run
    ##
    database: hyperkuber
    ## @param auth.username Name for a custom user to create
    ## ref: https://github.com/bitnami/bitnami-docker-mysql/blob/master/README.md#creating-a-database-user-on-first-run
    ##
    username: "hyperkuber"
    ## @param auth.password Password for the new user. Ignored if existing secret is provided
    ##
    password: "hyperkuber"
    ## @param auth.replicationUser MySQL replication user
    ## ref: https://github.com/bitnami/bitnami-docker-mysql#setting-up-a-replication-cluster
    ##
  service:
    ## @param primary.service.type MySQL Primary K8s service type
    ##
    type: ClusterIP
    ## @param primary.service.ports.mysql MySQL Primary K8s service port
    ##
    ports:
      mysql: 3306
  
externalMysql:
  host: "mysql"
  port: 3306
  username: "hyperkuber"
  password: "hyperkuber"
  database: "hyperkuber"
  migerate : true


redis:
  ## Whether to deploy a redis server to satisfy the applications database requirements. To use an external database set this to false and configure the externalDatabase parameters
  ## https://raw.githubusercontent.com/bitnami/charts/master/bitnami/redis/values.yaml
  enabled: true
  ## @section Redis&reg; common configuration parameters
  ## https://github.com/bitnami/bitnami-docker-redis#configuration
  ##
  master:
    persistence:
      enabled: false
  ## @param architecture Redis&reg; architecture. Allowed values: `standalone` or `replication`
  ##
  architecture: standalone
  ## Redis&reg; Authentication parameters
  ## ref: https://github.com/bitnami/bitnami-docker-redis#setting-the-server-password-on-first-run
  ##
  auth:
    ## @param auth.enabled Enable password authentication
    ##
    enabled: false
    ## @param auth.sentinel Enable password authentication on sentinels too
    ##
    sentinel: false
    ## @param auth.password Redis&reg; password
    ## Defaults to a random 10-character alphanumeric string if not set
    ##
    password: ""
  service:
    ## @param master.service.type Redis&reg; master service type
    ##
    type: ClusterIP
    ## @param master.service.ports.redis Redis&reg; master service port
    ##
    ports:
      redis: 6379


externalRedis:
  # support redis, redis+sentinel
  # addr for redis: <host_redis>:<port_redis>
  # addr for redis+sentinel: <host_sentinel1>:<port_sentinel1>,<host_sentinel2>:<port_sentinel2>,<host_sentinel3>:<port_sentinel3>
  addr: "redis:6379"
  password: ""
  ## Additional deployment annotations

```

mysql,redis修改Values.yaml的参照bitnami配置
```
# 目前hyperkuber的chart依赖mysql，redis组件，chart来自bitnami，安装时请修改values的默认配置
# mysql：https://raw.githubusercontent.com/bitnami/charts/master/bitnami/mysql/values.yaml
# redis：https://raw.githubusercontent.com/bitnami/charts/master/bitnami/redis/values.yaml
```

安装Chart
```
helm install hyperkuber hyperkuber/hyperkuber -n hyperkuber --create-namespace -f values.yaml
```
