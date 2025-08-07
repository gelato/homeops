## Restoring DB:
```
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: foob2
spec:
  replicas: 2
  secretName: the-secret

  backupSchedule: "0 1 1 * * *"
  backupURL: gs://bucket_name/path/
  backupSecretName: backup-secret

  initBucketURL: gs://bucket_name/path/to/backup.xbackup.gz
  initBucketSecretName: backup-secret
```
## Backup secret:
```
apiVersion: v1
kind: Secret
metadata:
  name: my-cluster-backup-secret
type: Opaque
data:
    # AWS_ACCESS_KEY_ID: #
    # AWS_SECRET_ACCESS_KEY: #
    # AWS_REGION: us-east-1
    # AWS_ACL: ?
    # S3_PROVIDER: AWS
    # S3_ENDPOINT: ?

    # GCS_SERVICE_ACCOUNT_JSON_KEY: ?
    # GCS_PROJECT_ID: ?
    # GCS_OBJECT_ACL: ?
    # GCS_BUCKET_ACL: ?
    # GCS_LOCATION: ?
    # GCS_STORAGE_CLASS: MULTI_REGIONAL

    # only for read-only purpose
    # HTTP_URL: ?

    # AZUREBLOB_ACCOUNT: ?
    # AZUREBLOB_KEY: ?

# for more details check docker entrypoint: hack/docker/mysql-helper/docker-entrypoint.sh
# and rclone documentation: https://rclone.org/
```
## Backup:
```
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlBackup
metadata:
  name: my-cluster-backup

spec:
  # this field is required
  clusterName: my-cluster

  ## if backupURL is specified then the backup will be put
  ## at this path, else the backup URL will be filled with
  ## the cluster preset backupURL and a random name
  # backupURL: gs://bucket_name/path/to/backup.xtrabackup.gz

  ## specify a secret where to find credentials to access the
  ## bucket
  # backupSecretName: backup-secret

  ## specify the remote deletion policy. It can be on of ["retain", "delete"]
  # remoteDeletePolicy: retain
  ```
## DB secret:
```
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  # root password is required to be specified
  ROOT_PASSWORD: bXlwYXNz
  ## application credentials that will be created at cluster bootstrap
  # DATABASE:
  # USER:
  # PASSWORD:
```
## Cluster:
```
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlCluster
metadata:
  name: my-cluster
spec:
  replicas: 2
  secretName: my-secret

  ## For setting custom docker image or specifying mysql version
  ## the image field has priority over mysqlVersion.
  # image: percona:5.7
  # mysqlVersion: 5.7

  # initBucketURL: gs://bucket_name/backup.xtrabackup.gz
  # initBucketSecretName:

  ## PodDisruptionBudget
  # minAvailable: 1

  ## For recurrent backups set backupSchedule with a cronjob expression with seconds
  # backupSchedule:
  # backupURL: s3://bucket_name/
  # backupSecretName:
  # backupScheduleJobsHistoryLimit:
  # backupRemoteDeletePolicy:

  ## Custom Server ID Offset for replication
  # serverIDOffset: 100

  ## Configs that will be added to my.cnf for cluster
  mysqlConf:
  #   innodb-buffer-size: 128M


  ## Specify additional pod specification
  # podSpec:
  #   imagePullSecrets: []
  #   labels: {}
  #   annotations: {}
  #   affinity:
  #     podAntiAffinity:
  #       preferredDuringSchedulingIgnoredDuringExecution:
  #         weight: 100
  #         podAffinityTerm:
  #           topologyKey: "kubernetes.io/hostname"
  #           labelSelector:
  #             matchlabels: <cluster-labels>
  #   backupAffinity: {}
  #   backupNodeSelector: {}
  #   backupPriorityClassName:
  #   backupTolerations: []
  #   # Override the default preStop hook with a custom command/script
  #   mysqlLifecycle:
  #     preStop:
  #       exec:
  #         command:
  #           - /scripts/demote-if-master
  #   nodeSelector: {}
  #   resources:
  #     requests:
  #       memory: 1G
  #       cpu:    200m
  #   tolerations: []
  #   priorityClassName:
  #   serviceAccountName: default
  #   # Use a initContainer to fix the permissons of a hostPath volume.
  #   initContainers:
  #     - name: volume-permissions
  #       image: busybox
  #       securityContext:
  #         runAsUser: 0
  #       command:
  #         - sh
  #         - -c
  #         - chmod 750 /data/mysql; chown 999:999 /data/mysql
  #       volumeMounts:
  #         - name: data
  #           mountPath: /data/mysql

  ## Specify additional volume specification
  # volumeSpec:
  #   # https://godoc.org/k8s.io/api/core/v1#EmptyDirVolumeSource
  #   emptyDir: {}

  #   # https://godoc.org/k8s.io/api/core/v1#HostPathVolumeSource
  #   hostPath:
  #     path:
  #     type:

  #   # https://godoc.org/k8s.io/api/core/v1#PersistentVolumeClaimSpec
  #   persistentVolumeClaim:
  #     accessModes: [ "ReadWriteOnce" ]
  #     resources:
  #       requests:
  #         storage: 1Gi

  ## Specify service objectives
  ## If thoses SLO are not fulfilled by cluster node then that node is
  ## removed from scheme
  # targetSLO:
  #   maxSlaveLatency: 10s

  ## You can use custom volume for /tmp partition if needed.
  ## Is disabled by default
  # tmpfsSize: 1Gi

  ## Set cluster in read only
  # readOnly: false

  ## Use `pigz` for parallel compression/decompression of backups
  ## Or specify any arbitrary compress/decompress commands with args
  # backupCompressCommand:
  #   - pigz
  #   - --stdout
  #
  # backupDecompressCommand:
  #   - pigz
  #   - --decompress

  ## Add metrics exporter extra arguments
  # metricsExporterExtraArgs:
  #   - --collect.info_schema.userstats
  #   - --collect.perf_schema.file_events

  ## Add extra arguments to rclone
  # rcloneExtraArgs:
  #   - --buffer-size=1G
  #   - --multi-thread-streams=8
  #   - --retries-sleep=10s
  #   - --retries=10
  #   - --transfers=8

  ## Add extra arguments to xbstream
  # xbstreamExtraArgs:
  #   - --parallel=8

  ## Add extra arguments to xtrabackup
  # xtrabackupExtraArgs:
  #   - --parallel=8

  ## Add extra arguments to xtrabackup during --prepare
  # xtrabackupPrepareExtraArgs:
  #   - --use-memory=5G

  ## Set xtrabackup target directory (the directory needs to exist)
  # xtrabackupTargetDir: /var/lib/mysql/.tmp/xtrabackup/

  # Add additional SQL commands to run during init of mysql
  # initFileExtraSQL:
  #   - "CREATE USER test@localhost"
```
## DB:
```
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlDatabase
metadata:
  name: my-database
spec:
  database: db-name-in-mysql
  clusterRef:
    name: my-cluster
    namespace: default
```
## User:
```
apiVersion: v1
kind: Secret
metadata:
  name: my-user-password
data:
  PASSWORD: bXlzcWwtcGFzc3dvcmQtZm9yLXVzZXI=

---
apiVersion: mysql.presslabs.org/v1alpha1
kind: MysqlUser
metadata:
  name: my-user
spec:
  user: user-name-in-mysql
  clusterRef:
    name: my-cluster
    namespace: default
  password:
    name: my-user-password
    key: PASSWORD
  allowedHosts:
    - localhost
  permissions:
    - schema: db-name-in-mysql
      tables: ["table1", "table2"]
      permissions:
        - SELECT
        - UPDATE
        - CREATE
```