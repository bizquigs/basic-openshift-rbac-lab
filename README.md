# basic-openshift-rbac-lab
This deploys the ***sample rbac lab application*** from [RedHatDemos OpenShift Security Lab 3: RBAC](https://github.com/RedHatDemos/SecurityDemos/blob/master/2021Labs/OpenShiftSecurity/documentation/lab3.adoc) into your own cluster. You won't need that link unless you want to see the full narrative with diagrams and screenshots.

The instructions below will enable you to run the exercise and understand the basics of what is happening.

<u>[Gist containing instructions](https://gist.github.com/bizquigs/06ff9635b4b78753d03d05b98b476804) for running the lab (also available in included [Instructions.txt](https://github.com/bizquigs/basic-openshift-rbac-lab/blob/main/Instructions.txt) file):</ul>

***Get started:***
```
git clone https://github.com/bizquigs/basic-openshift-rbac-lab.git

cd basic-openshift-rbac-lab

```
Then follow the instructions in the above file, or follow along below if you want to use the copy button on the code blocks.

Full lab instructions and narrative:
https://github.com/bizquigs/SecurityDemos/blob/master/2021Labs/OpenShiftSecurity/documentation/lab3.adoc

Below has condensed instructions for deploying on _your_ cluster.


###########################
# Install the rbac-lab app
# 
```
git clone https://github.com/bizquigs/basic-openshift-rbac-lab.git
cd basic-openshift-rbac-lab
```

# log into to your cluster with oc
#
```
oc new-project rbac-lab
```
```
oc create -f ./deployment.yaml
```
```
oc create -f ./service.yaml
```
```
oc create -f ./route.yaml
```

#####################################################
# Orient and see resources & behavior 
#
Home->Projects->rbac-lab
Workloads->Pods
Workloads->Deployments
Networking->Services
Networking->Routes

Developer view->Topology

```
oc get all -n rbac-lab
```

######################################################
# Fix the first error
#

Access the webapp from the route by either going to:

_Networking->Routes "Location" field_

 - or -

_Developer view->Topology_
and selecting either the Go To Url icon at the 1 o'clock position on the app icon
 - or -
expand the Deployment pane by selecting the openshift logo and then clicking on the Route 


# Execute the following command to create a new Role called pod-lister that grants access to list all pods.
```
oc create role pod-lister --verb=list --resource=pods
```

  # view it
```
  oc get role pod-lister -o yaml
```
# Execute the following command to create a new RoleBinding called pod-listers:
```
oc create rolebinding pod-listers --role=pod-lister --serviceaccount=rbac-lab:default
```
  # view it
```
  oc get rolebinding pod-listers -o yaml
```
  
# With the Role and RoleBinding now created in order for the default Service Account to list pods in the rbac-lab namespace, 
# return to the application in the web browser and refresh the page to confirm that a valid response is now being displayed 
# for the first query.

Should see "Number of Pods in Namespace 'rbac-lab': 1


###################################################
# Fix the second error
#

# Execute the following command to create a new ClusterRole called namespace-lister that grants access 
# to list all namespaces in the cluster:
```
oc create clusterrole namespace-lister --verb=list --resource=namespace
```
  # View it
```
  oc get clusterrole namespace-lister -o yaml
```  
# Now, create a ClusterRoleBinding to associate the pod-lister ClusterRole to the default Service Account 
# in the rbac-lab namespace:
```
oc create clusterrolebinding namespace-listers --clusterrole=namespace-lister --serviceaccount=rbac-lab:default
```
  # View it
```
  oc get clusterrolebinding namespace-listers -o yaml
```

# With the ClusterRole and ClusterRoleBinding created, return once again to the application in the web browser 
# and refresh the page. The second query should now display a valid response.
# You should see something like
Number of Namespaces: 75



###################################################
# Fix the third error
#

# See main doc for discussion of API groups vs core group.
# The first two fixes above were for the core group.
# Now we'll use an API group that deals with users.

# Check access (impersonate)
```
oc auth can-i list users
```
```
oc auth can-i list users --as=system:serviceaccount:rbac-lab:default
```
# Creating Policies for Resources Outside the Core API
#
# Because Users is cluster-scoped rather than namespace-scoped, we need to make a ClusterRole and a ClusterRoleBinding

```
oc api-resources | grep users
```

# Now that we know the API Group users are part of, we can create a ClusterRole called 
# user-lister using the following command:
```
oc create clusterrole user-lister --verb=list --resource=users.user.openshift.io
```

# Finally, create a ClusterRoleBinding to grant access to the default Service Account to the newly created ClusterRole
```
oc create clusterrolebinding user-listers --clusterrole=user-lister --serviceaccount=rbac-lab:default
```

# While we could confirm that the application can now query for users, 
# let’s use User Impersonation to determine ahead of time whether the default Service Account has the appropriate rights.
```
oc auth can-i list users --as=system:serviceaccount:rbac-lab:default
```

# Now let's go refresh the webapp
Number of Users: 1
