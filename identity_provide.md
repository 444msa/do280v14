<pre>
<h2> Configure Identity Providers  </h2>

[mahsan@r9 ~]$ htpasswd -c -B -b /tmp/htpasswd new_admin redhat
Adding password for user new_admin
[mahsan@r9 ~]$ htpasswd  -B -b /tmp/htpasswd new_developer redhat
Adding password for user new_developer
[mahsan@r9 ~]$ cat /tmp/htpasswd
new_admin:$2y$05$wJGHFLJ7hZeQptZaxEkS8.uimhmw75XztA61SYXyGM7PbCfA9u3hu
new_developer:$2y$05$A0lAXJskhiPajy7KcCbRMukX6D/FL90P6ulZqb0chMgVxQ80dm8eu
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc create secret generic localusers --from-file htpasswd=/tmp/htpasswd -n openshift-config
secret/localusers created
[mahsan@r9 ~]$ oc get secrets -n openshift-config
NAME                                      TYPE                             DATA   AGE
builder-dockercfg-7r7fz                   kubernetes.io/dockercfg          1      210d
default-dockercfg-676lc                   kubernetes.io/dockercfg          1      210d
deployer-dockercfg-566hs                  kubernetes.io/dockercfg          1      210d
etcd-client                               kubernetes.io/tls                2      211d
ex280-secue                               Opaque                           1      8h
htpass-secret                             Opaque                           1      210d
initial-service-account-private-key       Opaque                           1      211d
localusers                                Opaque                           1      19s
login-template                            Opaque                           1      210d


[mahsan@r9 ~]$ oc adm policy add-cluster-role-to-user cluster-admin new_admin
Warning: User 'new_admin' not found
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "new_admin"
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc adm policy add-cluster-role-to-user cluster-admin new_admin
Warning: User 'new_admin' not found
clusterrole.rbac.authorization.k8s.io/cluster-admin added: "new_admin"
[mahsan@r9 ~]$ oc get oauth cluster -o yaml > /tmp/oauth.yaml
[mahsan@r9 ~]$


[mahsan@r9 ~]$ cat /tmp/oauth.yaml
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  annotations:
    include.release.openshift.io/ibm-cloud-managed: "true"
    include.release.openshift.io/self-managed-high-availability: "true"
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"config.openshift.io/v1","kind":"OAuth","metadata":{"annotations":{},"name":"cluster"},"spec":{"identityProviders":[{"htpasswd":{"fileData":{"name":"htpass-secret"}},"mappingMethod":"claim","name":"developer","type":"HTPasswd"}],"templates":{"login":{"name":"login-template"}},"tokenConfig":{"accessTokenMaxAgeSeconds":0}}}
    release.openshift.io/create-only: "true"
  creationTimestamp: "2024-12-03T08:02:20Z"
  generation: 2
  name: cluster
  ownerReferences:
  - apiVersion: config.openshift.io/v1
    kind: ClusterVersion
    name: version
    uid: 1de19896-228b-4a9d-913f-0d775bc4bebc
  resourceVersion: "20146"
  uid: 34972387-914d-47a6-8643-c440983c06f2
spec:
  identityProviders:
  - htpasswd:
      fileData:
     <b>   name: localusers
    mappingMethod: claim
    name: myusers
    type: HTPasswd </b>
  templates:
    login:
      name: login-template
  tokenConfig:
    accessTokenMaxAgeSeconds: 0
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc replace -f /tmp/oauth.yaml
oauth.config.openshift.io/cluster replaced

[mahsan@r9 ~]$  oc get all -n openshift-authentication
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                                  READY   STATUS    RESTARTS   AGE
pod/oauth-openshift-898f86859-9kpg4   1/1     Running   3          2d16h

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/oauth-openshift   ClusterIP   10.217.4.49   <none>        443/TCP   211d

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/oauth-openshift   1/1     1            1           211d

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/oauth-openshift-6fcc9dd448   0         0         0       211d
replicaset.apps/oauth-openshift-78df6fd57d   0         0         0       211d
replicaset.apps/oauth-openshift-7fcd5d96ff   0         0         0       211d
replicaset.apps/oauth-openshift-898f86859    1         1         1       2d16h
replicaset.apps/oauth-openshift-b468b8d84    0         0         0       189d
replicaset.apps/oauth-openshift-d494d9776    0         0         0       211d
replicaset.apps/oauth-openshift-d66d84594    0         0         0       210d

NAME                                       HOST/PORT                          PATH   SERVICES          PORT   TERMINATION            WILDCARD
route.route.openshift.io/oauth-openshift   oauth-openshift.apps-crc.testing          oauth-openshift   6443   passthrough/Redirect   None
[mahsan@r9 ~]$

[mahsan@r9 ~]$  oc get all -n openshift-authentication
Warning: apps.openshift.io/v1 DeploymentConfig is deprecated in v4.14+, unavailable in v4.10000+
NAME                                   READY   STATUS    RESTARTS   AGE
pod/oauth-openshift-6bf4f7d788-jkx7t   1/1     Running   0          3m

NAME                      TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
service/oauth-openshift   ClusterIP   10.217.4.49   <none>        443/TCP   211d

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/oauth-openshift   1/1     1            1           211d

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/oauth-openshift-6bf4f7d788   1         1         1       3m1s
replicaset.apps/oauth-openshift-6fcc9dd448   0         0         0       211d
replicaset.apps/oauth-openshift-78df6fd57d   0         0         0       211d
replicaset.apps/oauth-openshift-7fcd5d96ff   0         0         0       211d
replicaset.apps/oauth-openshift-898f86859    0         0         0       2d16h
replicaset.apps/oauth-openshift-b468b8d84    0         0         0       189d
replicaset.apps/oauth-openshift-d494d9776    0         0         0       211d
replicaset.apps/oauth-openshift-d66d84594    0         0         0       210d

NAME                                       HOST/PORT                          PATH   SERVICES          PORT   TERMINATION            WILDCARD
route.route.openshift.io/oauth-openshift   oauth-openshift.apps-crc.testing          oauth-openshift   6443   passthrough/Redirect   None
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc login -u new_admin -p redhat
Login successful.

You have access to 67 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "storage-volume".
[mahsan@r9 ~]$ oc get node
NAME   STATUS   ROLES                         AGE    VERSION
crc    Ready    control-plane,master,worker   211d   v1.30.6
[mahsan@r9 ~]$ oc login -u new_developer -p redhat
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

[mahsan@r9 ~]$ oc get node
Error from server (Forbidden): nodes is forbidden: User "new_developer" cannot list resource "nodes" in API group "" at the cluster scope
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get users
NAME            UID                                    FULL NAME   IDENTITIES
developer       6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin       42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
new_admin       fc391888-06b5-4b1b-bdf5-f67f9f00abcf               myusers:new_admin
new_developer   e26ff1fc-3316-4d63-863c-81c54c274c48               myusers:new_developer
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get identity
NAME                    IDP NAME    IDP USER NAME   USER NAME       USER UID
developer:developer     developer   developer       developer       6e16cbbe-41aa-49cd-8158-5aab2967cadf
developer:kubeadmin     developer   kubeadmin       kubeadmin       42c8dcb9-09ea-4b49-898c-4d90aad72acd
myusers:new_admin       myusers     new_admin       new_admin       fc391888-06b5-4b1b-bdf5-f67f9f00abcf
myusers:new_developer   myusers     new_developer   new_developer   e26ff1fc-3316-4d63-863c-81c54c274c48
[mahsan@r9 ~]$


<h2> change new user  </h2>

[mahsan@r9 ~]$ oc extract secret/localusers -n openshift-config --to /tmp/auth-providers --confirm
/tmp/auth-providers/htpasswd
[mahsan@r9 ~]$ ls -ltr /tmp
total 12
drwx------ 2 root   root      6 Jun 28 15:53 vmware-root_1024-2965448061
drwx------ 2 root   root      6 Jun 29 09:52 vmware-root_1048-2966103395
drwx------ 2 root   root      6 Jun 29 10:17 vmware-root_1037-4256676136
drwx------ 2 mahsan mahsan   24 Jun 29 10:35 ssh-XXXX9N1dkm
drwx------ 2 mahsan mahsan    6 Jun 29 10:48 Temp-41e4a8f1-dd39-4418-b843-55ade9bbea54
drwx------ 2 root   root      6 Jun 29 15:58 vmware-root_1039-4248221866
drwx------ 2 mahsan mahsan   24 Jun 29 16:22 ssh-XXXXoKbL0M
drwx------ 2 root   root      6 Jun 30 15:21 vmware-root_1044-2991203048
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-dbus-broker.service-DvW5IO
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-bluetooth.service-MhzCmr
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-chronyd.service-Jlb1er
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-irqbalance.service-YtFp8E
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-rtkit-daemon.service-aX7u1N
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-switcheroo-control.service-doAb4g
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-systemd-logind.service-1mAApE
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-upower.service-ufzlOH
drwx------ 2 root   root      6 Jul  1 15:40 vmware-root_1041-4248746163
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-ModemManager.service-f8udmJ
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-kdump.service-h1hBtA
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-power-profiles-daemon.service-wNgXwX
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-colord.service-twFlyA
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-fwupd.service-rnRjxL
-rw-r--r-- 1 mahsan mahsan  541 Jul  2 11:08 passwd
drwx------ 2 mahsan mahsan   24 Aug  6 09:29 ssh-XXXX1GNoFt
-rw-r--r-- 1 mahsan mahsan  146 Aug  6 09:32 htpasswd
-rw-r--r-- 1 mahsan mahsan 1184 Aug  6 09:41 oauth.yaml
drwxr-xr-x 2 mahsan mahsan   22 Aug  6 10:02 auth-providers


[mahsan@r9 ~]$ htpasswd -b /tmp/htpasswd manager redhat
Adding password for user manager
[mahsan@r9 ~]$ cat /tmp/htpasswd
new_admin:$2y$05$wJGHFLJ7hZeQptZaxEkS8.uimhmw75XztA61SYXyGM7PbCfA9u3hu
new_developer:$2y$05$A0lAXJskhiPajy7KcCbRMukX6D/FL90P6ulZqb0chMgVxQ80dm8eu
manager:$apr1$jp6GDIBj$wuYDU1gCi/JveXyf8kK7b1
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc set data secret/localusers --from-file htpasswd=/tmp/htpasswd -n openshift-config
secret/localusers data updated
[mahsan@r9 ~]$

[mahsan@r9 ~]$  oc get pods -n openshift-authentication
NAME                              READY   STATUS    RESTARTS   AGE
oauth-openshift-b64954fb5-q2sgk   1/1     Running   0          3m27s
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc login -u manager -p redhat
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

[mahsan@r9 ~]$ oc get node
Error from server (Forbidden): nodes is forbidden: User "manager" cannot list resource "nodes" in API group "" at the cluster scope
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc new-project auth-providers
Now using project "auth-providers" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[mahsan@r9 ~]$ oc login -u new_developer -p redhat
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>

[mahsan@r9 ~]$ oc delete project auth-providers
Error from server (Forbidden): projects.project.openshift.io "auth-providers" is forbidden: User "new_developer" cannot delete resource "projects" in API group "project.openshift.io" in the namespace "auth-providers"


<h2> Change Password </h2>

[mahsan@r9 ~]$ oc login -u new_admin -p redhat
Login successful.

You have access to 68 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
[mahsan@r9 ~]$ oc extract secret/localusers -n openshift-config --to /tmp/auth-providers --confirm
/tmp/auth-providers/htpasswd
[mahsan@r9 ~]$ ls -ltr /tmp
total 12
drwx------ 2 root   root      6 Jun 28 15:53 vmware-root_1024-2965448061
drwx------ 2 root   root      6 Jun 29 09:52 vmware-root_1048-2966103395
drwx------ 2 root   root      6 Jun 29 10:17 vmware-root_1037-4256676136
drwx------ 2 mahsan mahsan   24 Jun 29 10:35 ssh-XXXX9N1dkm
drwx------ 2 mahsan mahsan    6 Jun 29 10:48 Temp-41e4a8f1-dd39-4418-b843-55ade9bbea54
drwx------ 2 root   root      6 Jun 29 15:58 vmware-root_1039-4248221866
drwx------ 2 mahsan mahsan   24 Jun 29 16:22 ssh-XXXXoKbL0M
drwx------ 2 root   root      6 Jun 30 15:21 vmware-root_1044-2991203048
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-dbus-broker.service-DvW5IO
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-bluetooth.service-MhzCmr
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-chronyd.service-Jlb1er
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-irqbalance.service-YtFp8E
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-rtkit-daemon.service-aX7u1N
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-switcheroo-control.service-doAb4g
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-systemd-logind.service-1mAApE
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-upower.service-ufzlOH
drwx------ 2 root   root      6 Jul  1 15:40 vmware-root_1041-4248746163
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-ModemManager.service-f8udmJ
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-kdump.service-h1hBtA
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-power-profiles-daemon.service-wNgXwX
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-colord.service-twFlyA
drwx------ 3 root   root     17 Jul  1 15:40 systemd-private-9f3ccb2271f34375a27badf71520b2d7-fwupd.service-rnRjxL
-rw-r--r-- 1 mahsan mahsan  541 Jul  2 11:08 passwd
drwx------ 2 mahsan mahsan   24 Aug  6 09:29 ssh-XXXX1GNoFt
-rw-r--r-- 1 mahsan mahsan 1184 Aug  6 09:41 oauth.yaml
drwxr-xr-x 2 mahsan mahsan   22 Aug  6 10:02 auth-providers
-rw-r--r-- 1 mahsan mahsan  192 Aug  6 10:03 htpasswd

[mahsan@r9 ~]$ MANAGER_PASSWD="$(openssl rand -hex 15)"
[mahsan@r9 ~]$ echo $MANAGER_PASSWD
78da790952dad955cb59652f3e0bb5

[mahsan@r9 ~]$ htpasswd -b /tmp/htpasswd manager ${MANAGER_PASSWD}
Updating password for user manager
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc set data secret/localusers --from-file htpasswd=/tmp/htpasswd -n openshift-config
secret/localusers data updated
[mahsan@r9 ~]$

[mahsan@r9 ~]$  oc get pods -n openshift-authentication
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-5f557f77bd-fwbkf   1/1     Running   0          68s

[mahsan@r9 ~]$ oc login -u manager -p ${MANAGER_PASSWD}
Login successful.

You have one project on this server: "auth-providers"

Using project "auth-providers".
[mahsan@r9 ~]$ oc whoami
manager


<h2> Delete the User </h2>

[mahsan@r9 ~]$ oc extract secret/localusers -n openshift-config --to /tmp/auth-providers --confirm
/tmp/auth-providers/htpasswd
[mahsan@r9 ~]$ htpasswd -D /tmp/htpasswd manager
Deleting password for user manager
[mahsan@r9 ~]$ oc set data secret/localusers --from-file htpasswd=/tmp/htpasswd -n openshift-config
secret/localusers data updated
[mahsan@r9 ~]$  oc get pods -n openshift-authentication
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-5f557f77bd-fwbkf   1/1     Running   0          10m


[mahsan@r9 ~]$  oc get pods -n openshift-authentication
NAME                               READY   STATUS    RESTARTS   AGE
oauth-openshift-5c6487c686-jppck   1/1     Running   0          4m30s
[mahsan@r9 ~]$


[mahsan@r9 ~]$ oc login -u manager -p ${MANAGER_PASSWD}
Login failed (401 Unauthorized)
Verify you have provided the correct credentials.
[mahsan@r9 ~]$ oc get users
NAME            UID                                    FULL NAME   IDENTITIES
developer       6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin       42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
manager         27bfa975-40d6-4477-b949-d2b18300af29               myusers:manager
new_admin       fc391888-06b5-4b1b-bdf5-f67f9f00abcf               myusers:new_admin
new_developer   e26ff1fc-3316-4d63-863c-81c54c274c48               myusers:new_developer
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc get identity
NAME                    IDP NAME    IDP USER NAME   USER NAME       USER UID
developer:developer     developer   developer       developer       6e16cbbe-41aa-49cd-8158-5aab2967cadf
developer:kubeadmin     developer   kubeadmin       kubeadmin       42c8dcb9-09ea-4b49-898c-4d90aad72acd
myusers:manager         myusers     manager         manager         27bfa975-40d6-4477-b949-d2b18300af29
myusers:new_admin       myusers     new_admin       new_admin       fc391888-06b5-4b1b-bdf5-f67f9f00abcf
myusers:new_developer   myusers     new_developer   new_developer   e26ff1fc-3316-4d63-863c-81c54c274c48
[mahsan@r9 ~]$ oc delete identity "myusers:manager"
identity.user.openshift.io "myusers:manager" deleted
[mahsan@r9 ~]$ oc get identity
NAME                    IDP NAME    IDP USER NAME   USER NAME       USER UID
developer:developer     developer   developer       developer       6e16cbbe-41aa-49cd-8158-5aab2967cadf
developer:kubeadmin     developer   kubeadmin       kubeadmin       42c8dcb9-09ea-4b49-898c-4d90aad72acd
myusers:new_admin       myusers     new_admin       new_admin       fc391888-06b5-4b1b-bdf5-f67f9f00abcf
myusers:new_developer   myusers     new_developer   new_developer   e26ff1fc-3316-4d63-863c-81c54c274c48
[mahsan@r9 ~]$

[mahsan@r9 ~]$ oc delete user manager
user.user.openshift.io "manager" deleted
[mahsan@r9 ~]$ oc get users
NAME            UID                                    FULL NAME   IDENTITIES
developer       6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin       42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
new_admin       fc391888-06b5-4b1b-bdf5-f67f9f00abcf               myusers:new_admin
new_developer   e26ff1fc-3316-4d63-863c-81c54c274c48               myusers:new_developer
[mahsan@r9 ~]$




</pre>
