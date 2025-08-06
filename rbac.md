<pre>

<h1> Apply Permissions with RBAC               </h1>


[mahsan@r9 do280v14]$ oc get clusterrolebindings -o wide | grep -E 'ROLE|self-provisioner'
NAME                                                                        ROLE                                                                                    AGE    USE                  GROUPS                                         SERVICEACCOUNTS
self-provisioners                                                           ClusterRole/self-provisioner                                                            211d                        system:authenticated:oauth
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc describe clusterrolebindings self-provisioners
Name:         self-provisioners
Labels:       <none>
Annotations:  rbac.authorization.kubernetes.io/autoupdate: true
Role:
  Kind:  ClusterRole
  Name:  self-provisioner
Subjects:
  Kind   Name                        Namespace
  ----   ----                        ---------
  Group  system:authenticated:oauth
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
Warning: Your changes may get lost whenever a master is restarted, unless you prevent reconciliation of this rolebinding using the following command: oc annotate clusterrolebinding.rbac self-provisioners 'rbac.authorization.kubernetes.io/autoupdate=false' --overwrite
clusterrole.rbac.authorization.k8s.io/self-provisioner removed: "system:authenticated:oauth"
[mahsan@r9 do280v14]$ oc describe clusterrolebindings self-provisioners
Error from server (NotFound): clusterrolebindings.rbac.authorization.k8s.io "self-provisioners" not found
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc get clusterrolebindings -o wide | grep -E 'ROLE|self-provisioner'
NAME                                                                        ROLE                                                                                    AGE    USERS                                                            GROUPS                                         SERVICEACCOUNTS
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc get users
NAME            UID                                    FULL NAME   IDENTITIES
developer       6e16cbbe-41aa-49cd-8158-5aab2967cadf               developer:developer
kubeadmin       42c8dcb9-09ea-4b49-898c-4d90aad72acd               developer:kubeadmin
new_admin       fc391888-06b5-4b1b-bdf5-f67f9f00abcf               myusers:new_admin
new_developer   e26ff1fc-3316-4d63-863c-81c54c274c48               myusers:new_developer
[mahsan@r9 do280v14]$ oc login -u new_developer -p redhat
Login successful.

You don't have any projects. Contact your system administrator to request a project.
[mahsan@r9 do280v14]$ oc new-project test
Error from server (Forbidden): You may not request a new project via this API.
[mahsan@r9 do280v14]$


[mahsan@r9 do280v14]$ oc login -u new_developer -p redhat
Login successful.

You don't have any projects. Contact your system administrator to request a project.
[mahsan@r9 do280v14]$ oc new-project test
Error from server (Forbidden): You may not request a new project via this API.
[mahsan@r9 do280v14]$ oc login -u new_admin -p redhat
Login successful.

You have access to 68 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
[mahsan@r9 do280v14]$ oc new-project test
Now using project "test" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname

[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc new-project auth-rbac
Now using project "auth-rbac" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname



[mahsan@r9 do280v14]$ oc policy add-role-to-user admin new_developer
clusterrole.rbac.authorization.k8s.io/admin added: "new_developer"
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc policy add-role-to-user admin new_developer
clusterrole.rbac.authorization.k8s.io/admin added: "new_developer"
[mahsan@r9 do280v14]$ oc adm groups new dev-group
group.user.openshift.io/dev-group created
[mahsan@r9 do280v14]$ oc adm groups add-users dev-group developer
group.user.openshift.io/dev-group added: "developer"
[mahsan@r9 do280v14]$
[mahsan@r9 do280v14]$ oc adm groups new qa-group
group.user.openshift.io/qa-group created


[mahsan@r9 do280v14]$ oc adm groups add-users qa-group qa-engineer
group.user.openshift.io/qa-group added: "qa-engineer"
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc get groups
NAME        USERS
dev-group   developer
qa-group    qa-engineer
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc login -u new_developer -p redhat
Login successful.

You have one project on this server: "auth-rbac"

Using project "auth-rbac".
[mahsan@r9 do280v14]$ oc policy add-role-to-group edit dev-group
clusterrole.rbac.authorization.k8s.io/edit added: "dev-group"
[mahsan@r9 do280v14]$ oc policy add-role-to-group view  qa-group
clusterrole.rbac.authorization.k8s.io/view added: "qa-group"
[mahsan@r9 do280v14]$

mahsan@r9 do280v14]$ oc get rolebindings -o wide | grep -v '^system:'
NAME                    ROLE                               AGE   USERS           GROUPS                             SERVICEACCOUNTS
admin                   ClusterRole/admin                  11m   new_admin
admin-0                 ClusterRole/admin                  11m   new_developer
edit                    ClusterRole/edit                   98s                   dev-group
view                    ClusterRole/view                   78s                   qa-group
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc login -u developer
Logged into "https://api.crc.testing:6443" as "developer" using existing credentials.

You have access to the following projects and can switch between them with 'oc project <projectname>':

  * auth-rbac
    test-project

Using project "auth-rbac".
[mahsan@r9 do280v14]$ oc new-app --name http httpd:2.4
warning: Cannot check if git requires authentication.
--> Found container image 5bdbc62 (2 weeks old) from Docker Hub for "httpd:2.4"

    * An image stream tag will be created as "http:2.4" that will track this image

--> Creating resources ...
    imagestream.image.openshift.io "http" created
    deployment.apps "http" created
    service "http" created
--> Success
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/http'
    Run 'oc status' to view your app.
[mahsan@r9 do280v14]$


[mahsan@r9 do280v14]$ oc policy add-role-to-user edit qa-engineer
Error from server (Forbidden): rolebindings.rbac.authorization.k8s.io is forbidden: User "developer" cannot list resource "rolebindings" in API group "rbac.authorization.k8s.io" in the namespace "auth-rbac"

[mahsan@r9 do280v14]$ oc login -u new_developer -p redhat
Login successful.

You have one project on this server: "auth-rbac"

Using project "auth-rbac".
[mahsan@r9 do280v14]$ oc new-project test
Error from server (Forbidden): You may not request a new project via this API.
[mahsan@r9 do280v14]$ oc login -u new_developer -p redhat
Login successful.

[mahsan@r9 do280v14]$  oc adm policy add-cluster-role-to-group --rolebinding-name self-provisioners self-provisioner system:authenticated:oauth
Warning: Group 'system:authenticated:oauth' not found
clusterrole.rbac.authorization.k8s.io/self-provisioner added: "system:authenticated:oauth"
[mahsan@r9 do280v14]$

[mahsan@r9 do280v14]$ oc login -u developer
Logged into "https://api.crc.testing:6443" as "developer" using existing credentials.

You have access to the following projects and can switch between them with 'oc project <projectname>':

  * auth-rbac
    test-project

Using project "auth-rbac".
[mahsan@r9 do280v14]$ oc new-project test2
Now using project "test2" on server "https://api.crc.testing:6443".

You can add applications to this project with the 'new-app' command. For example, try:

    oc new-app rails-postgresql-example

to build a new example application in Ruby. Or use kubectl to deploy a simple Kubernetes application:

    kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.43 -- /agnhost serve-hostname








</pre>



