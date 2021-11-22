# rhossm-installation
Basic Openshift Service Mesh 2.x installation and configuration in a clean OCP platform.


## Prerequisites
 - OCP 4.6 or higher.
 - Cluster admin role access for installing the Openshift Service Mesh.
 - OC cli installed.

# OSSM 2.0.X

## Installation and basic configuration

Installing the operators:

Jaeger
```
oc apply -f 1-packagemanifests/1-jaeger-operator/subscription.yaml
```

Kiali
```
oc apply -f 1-packagemanifests/2-kiali-operator/subscription.yaml
```

OSSM
```
#For OSSM v2.0.8 oc apply -f 1-packagemanifests/3-ossm-operator/subscription-v2.0.8.yaml
#For OSSM v2.1.0 oc apply -f 1-packagemanifests/3-ossm-operator/subscription-v2.1.0.yaml
```

Now, the operators are installed and it is time to install the Service Mesh Control Plane with the configuration desired. For this, set the [SMCP Configuration](./2-smcp/basic.yaml) file up with your preferences and install it.

Create the istio-system namespace
```
oc new-project istio-system
```

Install the Service Mesh Control Plane
```
oc apply -f 2-smcp/basic.yaml
```

You can check the installation running
```
oc get smcp -n istio-system
----
NAME    READY   STATUS            PROFILES      VERSION   AGE
basic   9/9     ComponentsReady   ["default"]   2.0.8     2m9s
```

## Adding services to the mesh

### Openshift Service Mesh member roll

The *ServiceMeshMemberRoll* object list the projects that belong to the Control Plane. Any project that is not set in this object, is threated as external to the Service Mesh.

Create the SMMR
```
oc apply -f 2-/smmr.yaml
```

### Openshift Service Mesh member

Using this object, users who don't have privileges to add members to the *ServiceMeshMemberRoll* can join their namespaces to the Service Mesh. But, these users need the *mesh-user* role.

If you try to create the Service Mesh Member object, you will receive the following error:
```
oc apply -f 2-smcp/smm-2.yaml 
----
Error from server: error when creating "2-smcp/smm-2.yaml": admission webhook "smm.validation.maistra.io" denied the request: user '$user' does not have permission to use ServiceMeshControlPlane istio-system/basic
```

Grant user permissions to access the mesh by granting the *mesh-user* role:
```
oc policy add-role-to-user -n istio-system --role-namespace istio-system mesh-user $user
```

Now, create the SMM
```
oc apply -f 2-smcp/smm-2.yaml
----
servicemeshmember.maistra.io/default created
```

## Adding a new OSSM to the OpenShift cluster
Create the istio-system-pre namespace
```
oc new-project istio-system-pre
```

Installing a new OSSM in the same OpenShift cluster
```
oc apply -f 2-smcp/basic-pre.yaml
```

Create the SMMR
```
oc apply -f 2-smcp/smmr-pre.yaml