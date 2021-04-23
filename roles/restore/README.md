Restore Role
=========

The purpose of this role is to restore your AWX deployment from an existing PVC backup. The backup includes:
  - custom deployment specific values in the spec section of the AWX custom resource object
  - backup of the postgresql database
  - secret_key, admin_password, and broadcast_websocket secrets
  - database configuration



Requirements
------------

This role assumes you are authenticated with an Openshift or Kubernetes cluster:
  - The awx-operator has been deployed to the cluster
  - AWX is deployed to via the operator
  - An AWX backup is available on a PVC in your cluster (see the backup [README.md](../backup/README.md))


Usage
----------------

Then create a file named `restore-awx.yml` with the following contents:

```yaml
---
apiVersion: awx.ansible.com/v1beta1
kind: AWXRestore
metadata:
  name: restore1
  namespace: my-namespace
spec:
  tower_name: mytower
  tower_backup_pvc: awxbackup1-backup-claim
  tower_backup_pvc_namespace: 'old-awx-namespace'
  tower_backup_dir: /backups/tower-openshift-backup-2021-04-02-03:25:08
```

Note that the `tower_name` above is the name of the AWX deployment you intend to create and restore to.  

The namespace specified is the namespace the resulting AWX deployment will be in.  The namespace you specified must be pre-created.  

```
kubectl create ns my-namespace
```

Finally, use `kubectl` to create the restore object in your cluster:

```bash
$ kubectl apply -f restore-awx.yml
```

This will create a new deployment and restore your backup to it.  


Role Variables
--------------

The name of the backup directory can be found as a status on your AWXBackup object.  This can be found in your cluster's console, or with the client as shown below.  

```bash
$ kubectl get awxbackup awxbackup1 -o jsonpath="{.items[0].status.towerBackupDirectory}"
/backups/tower-openshift-backup-2021-04-02-03:25:08
```

```
tower_backup_dir: '/backups/tower-openshift-backup-2021-04-02-03:25:08'
```


The name of the PVC can also be found by looking at the backup object.  

```bash
$ kubectl get awxbackup awxbackup1 -o jsonpath="{.items[0].status.towerBackupClaim}"
awx-backup-volume-claim
```

```
tower_backup_pvc: 'awx-backup-volume-claim'
```

By default, the backup pvc will be created in the same namespace the awxbackup object is created in. This namespace must be specified using the `tower_backup_pvc_namespace` variable.

```
tower_backup_pvc_namespace: 'custom-namespace'
```

If a custom postgres configuration secret was used when deploying AWX, it must be set:

```
tower_postgres_configuration_secret: 'awx-postgres-configuration'
```


Testing
----------------

You can test this role directly by creating and running the following playbook with the appropriate variables:

```
---
- name: Restore AWX
  hosts: localhost
  gather_facts: false
  roles:
    - restore
```

License
-------

MIT