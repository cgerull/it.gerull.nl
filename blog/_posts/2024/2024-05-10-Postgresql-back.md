---
layout: post
title: PostgreSQL Backup
description: "Examples for PostgreSQL backup scripts and jobs."
tag: kubernetes linux postgresql
category: Database
date: 2024-05-10 10:10:23
---

## Backup alle databases met pg_dumpall

Consistente, logical backup van alle database in het DBMS waarbij afhankelijkheden en constrains bewaard blijven.

Deze backup maakt een snapshot en is niet voor Point-In-Time (PTR) recovery geschikt.

### k8s cronjob

```yaml
---
apiVersion: batch/v1
kind: CronJob
metadata:
  name: postgresql-pgdumpall
  labels:
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: postgresql
    app.kubernetes.io/version: 16.3.0
    helm.sh/chart: postgresql-15.5.7
    app.kubernetes.io/component: pg_dumpall
spec:
  schedule: "0 23 * * *"
  timeZone: "CET"
  concurrencyPolicy: Allow
  failedJobsHistoryLimit: 1
  successfulJobsHistoryLimit: 3
  startingDeadlineSeconds: 600
  jobTemplate:
    spec:
      ttlSecondsAfterFinished: 86400
      template:
        metadata:
          labels:
            app.kubernetes.io/managed-by: Helm
            app.kubernetes.io/name: postgresql
            app.kubernetes.io/version: 16.3.0
            helm.sh/chart: postgresql-15.5.7
            app.kubernetes.io/component: pg_dumpall
          annotations:
            backup.velero.io/backup-volumes: datadir
        spec:
          imagePullSecrets:
            - name: harbor-puller
          containers:
          - name: postgresql-pgdumpall
            image: docker.io/bitnami/postgresql:15.6.0-debian-12-r20
            imagePullPolicy: "IfNotPresent"
            env:
              - name: PGUSER
                value: postgres
              - name: PGPASSWORD
                valueFrom:
                  secretKeyRef:
                    name: postgres-users
                    key: postgres-password
              - name: PGHOST
                value: postgresql
              - name: PGPORT
                value: "5432"
              - name: PGDUMP_DIR
                value: /backup/pgdump
            command:
              - /bin/sh
              - -c
              - |
                echo "Starting PostgreSql dump of all databases."
                pg_dumpall --clean --if-exists --load-via-partition-root --quote-all-identifiers \
                  --no-password --file=${PGDUMP_DIR}/pg_dumpall-$(date '+%Y-%m-%d-%H-%M').pgdump
                echo "PostgreSql dump finished, taking a nap."
                sleep 10800
                echo "Terminate job after 3h waiting for Velero"
            volumeMounts:
              - name: datadir
                mountPath: /backup/pgdump
                subPath:
              - name: empty-dir
                mountPath: /tmp
                subPath: tmp-dir
            securityContext:
              allowPrivilegeEscalation: false
              capabilities:
                drop:
                - ALL
              privileged: false
              readOnlyRootFilesystem: true
              runAsGroup: 0
              runAsNonRoot: true
              runAsUser: 1001
              seLinuxOptions: {}
              seccompProfile:
                type: RuntimeDefault
            resources:
              limits:
                cpu: 150m
                ephemeral-storage: 1024Mi
                memory: 192Mi
              requests:
                cpu: 100m
                ephemeral-storage: 50Mi
                memory: 128Mi
          restartPolicy: OnFailure
          volumes:
            - name: datadir
              persistentVolumeClaim:
                claimName: postgresql-pgdumpall
            - name: empty-dir
              emptyDir: {}
```

### k8s job

```yaml
---
apiVersion: batch/v1
kind: Job
metadata:
  name: postgresql-pgdumpall-init
spec:
  template:
    metadata:
      labels:
        helm.sh/chart: postgresql-0.1.0
        app.kubernetes.io/name: postgresql
        app.kubernetes.io/instance: pro
        app.kubernetes.io/version: "15.4"
        app.kubernetes.io/managed-by: Helm
        app.kubernetes.io/component: pg_dumpall
    spec:
      containers:
        - name: pgdumpall-init
          image: docker.io/bitnami/postgresql:15.6.0-debian-12-r20
          imagePullPolicy: "IfNotPresent"
          env:
            - name: PGUSER
              value: postgres
            - name: PGPASSWORD
              valueFrom:
                secretKeyRef:
                  name: postgres-users
                  key: postgres-password
            - name: PGPORT
              value: "5432"
            - name: PGHOST
              value: pro-postgresql
            - name: PGDUMP_DIR
              value: /backup/pgdump
          command:
            - /bin/sh
            - -c
          args:
            - |
              [ $(ls -1 ${PGDUMP_DIR}/pg_dumpall-*.pgdump 2>/dev/null | wc -l) -gt 0 ] && exit 0
              sleep 30
              pg_dumpall --clean --if-exists --load-via-partition-root --quote-all-identifiers \
                --no-password --file=${PGDUMP_DIR}/pg_dumpall-init-$(date '+%Y-%m-%d-%H-%M').pgdump
          volumeMounts:
            - name: datadir
              mountPath: /backup/pgdump
              subPath:
            - name: empty-dir
              mountPath: /tmp
              subPath: tmp-dir
          securityContext: {}
      restartPolicy: OnFailure
      volumes:
        - name: datadir
          persistentVolumeClaim:
            claimName: postgresql-pgdumpall
        - name: empty-dir
          emptyDir: {}
```

### docker (scheduled) container

```yaml

```

### pg_dumpall script

```bash
#!/bin/sh
#

PGDUMP_DIR="${BACKUP_DIR:-/backup}"
export PGDUMP_DIR

TS="$(date '+%Y-%m-%d-%H-%M')"
export TS

echo "$(date '+%Y-%m-%d-%H-%M') -- Starting PostgreSql dump of all databases." > "${PGDUMP_DIR}/${TS}"-dump.log
pg_dumpall --clean \
  --if-exists \
  --load-via-partition-root \
  --quote-all-identifiers \
  --no-password \
  --verbose 2>>"${PGDUMP_DIR}/${TS}"-dump.log | gzip - > "${PGDUMP_DIR}/${TS}"-pg_dumpall.sql.gz \
&& echo "$(date '+%Y-%m-%d-%H-%M') -- PostgreSql dump finished." >> "${PGDUMP_DIR}/${TS}"-dump.log
```

## PostgreSql alle databases met pg_dumpall

Met dit script kunnen specifieke database in een centrale database gexporteerd worden. De uitvoering kubernetes als (cron)job werkt zoals boven bij pg_dumpall beschreven.

In dit voorbeeld word de database dump direct naar een S3 buckert (MinIO) gestuurd.

Deze backup maakt een snapshot en is niet voor PTR recovery geschikt.

```bash
#!/bin/sh
#
# Check if environment variables exist
if [ -z "$MINIO_ACCESS_KEY" ] || [ -z "$MINIO_SECRET_KEY" ] || [ -z "$MINIO_ENDPOINT" ] || [ -z "$BUCKET_NAME" ] || [ -z "$BACKUP_DIR" ]; then
    echo "Error: Missing required environment variables."
    exit 1
fi

PGDUMP_DIR="${BACKUP_DIR:-/backup}"
export PGDUMP_DIR
TS="$(date '+%Y-%m-%d-%H-%M')"
export TS
MC_HOST_MINIO="http://${MINIO_ACCESS_KEY}:${MINIO_SECRET_KEY}@${MINIO_ENDPOINT#http://}"
export MC_HOST_MINIO

# Dump databases seperatly
DATABASES=$(psql -t -A -c "SELECT datname FROM pg_database WHERE datname <> ALL ('{template0,template1,postgres}')")
for DB in $DATABASES; do
    echo "$(date '+%Y-%m-%d-%H-%M') -- Starting PostgreSql dump of database ${DB}." >> "${PGDUMP_DIR}/${TS}-${DB}-dump.log"
    pg_dump --clean --if-exists --create --load-via-partition-root --quote-all-identifiers \
        --no-password --verbose --dbname="${DB}" 2>>"${PGDUMP_DIR}/${TS}-${DB}-dump.log" | gzip - >  "${PGDUMP_DIR}/${TS}-${DB}.sql.gz" \
        && echo "$(date '+%Y-%m-%d-%H-%M') -- PostgreSql dump of database ${DB} finished." >> "${PGDUMP_DIR}/${TS}-${DB}-dump.log"
    echo "Saving ${TS}-${DB}-dump to MinIO."
    mcli -C /tmp cp "${PGDUMP_DIR}/${TS}-${DB}-dump.log"  "MINIO/${BUCKET_NAME}/${TS}-${DB}-dump.log"
    mcli -C /tmp cp "${PGDUMP_DIR}/${TS}-${DB}.sql.gz"  "MINIO/${BUCKET_NAME}/${TS}-${DB}.sql.gz"
done
```

## Save to S3 / MinIO

Save pg_dumpall backups to an offsite datastore.

### k8s/cronjob

```yaml
---
apiVersion: batch/v1
kind: CronJob
metadata:
  labels:
    helm.sh/chart: postgresql-0.1.0
    app.kubernetes.io/name: postgresql
    app.kubernetes.io/version: "15.4"
    app.kubernetes.io/managed-by: Helm
  name: postgresql-save-dump-to-s3
spec:
  concurrencyPolicy: Allow
  failedJobsHistoryLimit: 1
  jobTemplate:
    metadata:
      creationTimestamp: null
    spec:
      template:
        metadata:
          creationTimestamp: null
          labels:
            app.kubernetes.io/name: postgresql
        spec:
          containers:
          - command:
            - /bin/sh
            - -c
            - |
              echo "Saving latest PostgreSql dump to MinIO."
              LATEST_DUMP=$(ls -1 ${PGDUMP_DIR}/pg_dumpall-*.pgdump | tail -n 2 | head -n 1)
              mc -C /tmp cp ${LATEST_DUMP}  MINIO/backups/$(basename ${LATEST_DUMP})
              echo "PostgreSql dump saved to MINIO/backups/{LATEST_DUMP}."
            env:
            - name: MC_HOST_MINIO
              valueFrom:
                secretKeyRef:
                  key: minio_alias
                  name: minio-for-backups

            - name: PGDUMP_DIR
              value: /backup/pgdump
            image: docker.io/minio/mc:RELEASE.2024-07-08T20-59-24Z #minio/mc
            imagePullPolicy: IfNotPresent
            name: postgresql-pg-save-s3
            resources:
              limits:
                cpu: 150m
                ephemeral-storage: 1Gi
                memory: 192Mi
              requests:
                cpu: 100m
                ephemeral-storage: 50Mi
                memory: 128Mi
            securityContext:
              allowPrivilegeEscalation: false
              capabilities:
                drop:
                - ALL
              privileged: false
              readOnlyRootFilesystem: true
              runAsNonRoot: true
              seccompProfile:
                type: RuntimeDefault
            terminationMessagePath: /dev/termination-log
            terminationMessagePolicy: File
            volumeMounts:
            - name: datadir
              mountPath: /backup/pgdump
              subPath:
            - mountPath: /tmp
              name: empty-dir
              subPath: tmp-dir
          dnsPolicy: ClusterFirst
          restartPolicy: OnFailure
          schedulerName: default-scheduler
          terminationGracePeriodSeconds: 30
          volumes:
          - name: datadir
            persistentVolumeClaim:
              claimName: postgresql-pgdumpall
          - name: empty-dir
      ttlSecondsAfterFinished: 86400
  schedule: 0 8 * * *
  timeZone: "CET"
  startingDeadlineSeconds: 600
  successfulJobsHistoryLimit: 3
  suspend: false
  ```

### Copy naar MinIO script

```bash
#!/bin/sh
#
# Check if environment variables exist
if [ -z "$MINIO_ACCESS_KEY" ] || [ -z "$MINIO_SECRET_KEY" ] || [ -z "$MINIO_ENDPOINT" ] || [ -z "$BUCKET_NAME" ] || [ -z "$BACKUP_DIR" ]; then
    echo "Error: Missing required environment variables."
    exit 1
fi

MC_HOST_MINIO="http://${MINIO_ACCESS_KEY}:${MINIO_SECRET_KEY}@${MINIO_ENDPOINT#http://}"
export MC_HOST_MINIO
PGDUMP_DIR="${BACKUP_DIR:-/backup}"
export PGDUMP_DIR
echo "Saving latest PostgreSql dump to MinIO."
LATEST_DUMP="$(ls -t "${PGDUMP_DIR}"/*.gz | head -n 1)"
mcli -C /tmp cp "${LATEST_DUMP}"  "MINIO/${BUCKET_NAME}/$(basename ${LATEST_DUMP})"
LATEST_LOG="$(ls -t "${PGDUMP_DIR}"/*.log | head -n 1)"
mcli -C /tmp cp "${LATEST_LOG}"  "MINIO/${BUCKET_NAME}/$(basename ${LATEST_LOG})"
echo "PostgreSql dump saved to MINIO/${BUCKET_NAME}/${LATEST_DUMP}."
```

## CNPG cluster backup met barman

Een cnpg cluster kan in zijn geheel inclusieve WAL archives met barman in een S3 bucket backuped worden.

Backups uit barman bevatten de WAL archives en zijn geschikt voor PTR.

```yaml
# Backup sectie uit de cnpg values file

backup:
  enabled: true # Enable barman backup
  retentionPolicy: 30d # Retention policy in MinIO for backups
  barmanObjectStore:
    endpointURL: http://pro-minio-for-cnpg-backups.dpc-oa-pro-minio.svc.cluster.local:9000 # Cluster MinIO service endpoint
    bucketName: backups # MinIO bucket name  postfix, prefix is the release name
  s3CredentialSecret: minio-user # Name of the secret containing the MinIO credentials
  # Beware that this format accepts also the seconds field, and it is different from the crontab format in Unix/Linux systems.
  schedule: "0 0 1 * * *" # everyday at 01:00 UTC
  walArchive:
    enabled: true # Backup WAL archives, required for point-in-time recovery and object store backup
    compression: "gzip"
    maxParallel: 8
    encryption:
      enabled: false
```

```yaml
{{\* Fragment uit de cluster template */}}

{{- if and .Values.backup.enabled (not .Values.recovery.enabled) }}
  backup: # WAL-archives and base backups to Minio S3 - https://cloudnative-pg.io/documentation/current/backup_recovery/#other-s3-compatible-object-storages-providers
    retentionPolicy: {{ default "15d" .Values.backup.retentionPolicy }} # In S3 only by days, not by count - https://cloudnative-pg.io/documentation/current/backup_recovery/#retention-policies
    barmanObjectStore:
      endpointURL: {{ .Values.backup.barmanObjectStore.endpointURL }}  # Existing Minio - sp-demo/spri-minio-tst
      destinationPath: s3://{{ .Release.Name }}-{{ .Values.backup.barmanObjectStore.bucketName }}  #s3://{{ .Release.Name }}-backups # This bucket will be created automatically
      tags:
        backupRetentionPolicy: "expire" # https://cloudnative-pg.io/documentation/current/backup_barmanobjectstore/#tagging-of-backup-objects
      historyTags:
        backupRetentionPolicy: "keep"
      s3Credentials:
        inheritFromIAMRole: false # Use existing credentials - https://cloudnative-pg.io/documentation/current/api_reference/#s3credentials
        accessKeyId:
          key: userName
          name: {{ .Values.backup.s3CredentialSecret }}
        secretAccessKey:
          key: userPassword
          name: {{ .Values.backup.s3CredentialSecret }}
      {{- if .Values.backup.walArchive.enabled }}
      wal: # Parallel archiving with compression - https://cloudnative-pg.io/documentation/current/backup_recovery/#wal-archiving
        compression: {{ default "gzip" .Values.backup.walArchive.compression }}
        maxParallel: {{ default "8" .Values.backup.walArchive.maxParallel }}
        {{- if .Values.backup.walArchive.encryption.enabled }}
        encryption: {{ default "AES256" .Values.backup.walArchive.encryption.method }}
        {{- end }}

      {{- end }}
  {{- end }}
```

Door de cnpg cluster operator worden dit vertaalt in de volgende objecten.

```yaml
---
# Source: pro-cnpg/templates/schedule.yaml
#
# Backup schedule - https://cloudnative-pg.io/documentation/current/backup_recovery/#scheduled-backups
#
apiVersion: postgresql.cnpg.io/v1
kind: ScheduledBackup # https://cloudnative-pg.io/documentation/current/cloudnative-pg.v1
metadata:
  name: backup-schedule
spec:
  schedule: 0 0 0/2 * *
  backupOwnerReference: self
  cluster:
    name: pro-database-02
```

Deze cronjob maakt en start zijn beurt deze backup job.

```yaml
---
apiVersion: postgresql.cnpg.io/v1
kind: Backup
metadata:
  name: backup-schedule-20240821060000
  namespace: dpc-oa-pro-database-02
  labels:
    cnpg.io/cluster: pro-database-02
    cnpg.io/immediateBackup: 'false'
    cnpg.io/scheduled-backup: backup-schedule
spec:
  cluster:
    name: pro-database-02
  method: barmanObjectStore
status:
  stoppedAt: '2024-08-21T06:00:03Z'
  method: barmanObjectStore
  endWal: '000000010000000500000043'
  endLSN: 5/43009780
  backupName: backup-20240821060000
  serverName: pro-database-02
  instanceID:
    ContainerID: 'xxxxxx'
    podName: pro-database-02-2
  destinationPath: 's3://pro-database-02-backups'
  startedAt: '2024-08-21T06:00:01Z'
  beginWal: '000000010000000500000042'
  endpointURL: 'http://my-minio-service:9000'
  phase: completed
  s3Credentials:
    accessKeyId:
      key: userName
      name: minio-user
    secretAccessKey:
      key: userPassword
      name: minio-user
  backupId: 20240821T060001
  beginLSN: 5/4200B4E0
```
