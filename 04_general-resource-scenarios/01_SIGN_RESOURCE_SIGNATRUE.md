## Goal:

Define RSP and protect a resource with signature(resource signature) in newly created namespace

## Prerequisite: 
1. Integrity shield is ready in a `managed cluster`
2. you can connect via oc to a `managed cluster`

## Action steps:

[OC-MANAGED]
### 1. Create a namespace in the cluster  
[Command]
```
  oc create ns secure-ns 
```
[Result]  
If successful, the result will be:
```
 namespace/secure-ns created
```
On failure, 
```
Error from server (AlreadyExists): namespaces "secure-ns" already exists
```
If `secure-ns` already exists, refresh the namespace or create another namespace for this test.   
[Command]  
```
oc delete ns secure-ns
oc create ns secure-ns
```


### 2. Define which reource(s) should be [protected](https://github.com/IBM/integrity-enforcer/blob/master/docs/README_QUICK.md#define-which-reources-should-be-protected)

[Command]
```
cat <<EOF | oc apply -n secure-ns -f -
apiVersion: apis.integrityshield.io/v1alpha1
kind: ResourceSigningProfile
metadata:
  name: sample-rsp
spec:
  protectRules:
  - match:
    - kind: ConfigMap
    - kind: Deployment
    - kind: Service
EOF
```
[Result]
```
resourcesigningprofile.apis.integrityshield.io/sample-rsp created
```

### 3. Prepare a sample resource  

[Command]
```
cat << EOF > /tmp/test-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm
data:
  key1: val1
  key2: val2
  key4: val4
EOF
```

[OC-MANAGED]

### 4. Create a resource without signature

[Command]
```
oc create -f /tmp/test-cm.yaml -n secure-ns
```
[Result]  
The resource cannot be created when it does not have a signatrue.
```
Error from server: error when creating "/tmp/test-cm.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"ConfigMap","name":"test-cm","namespace":"secure-ns","operation":"CREATE","request.uid":"3d0420fc-f53f-4de8-affc-fa67daace142","scope":"Namespaced","userName":"kubernetes-admin"})
```

[OC-MANAGED]  
### 5. Create a resource with [signature](https://github.com/IBM/integrity-enforcer/blob/master/docs/README_QUICK.md#create-a-resource-with-signature)

a. Generate a signature for the sample resource  

[Command]
```
curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-enforcer/master/scripts/gpg-rs-sign.sh | bash -s \
		  signer@enterprise.com \
		  /tmp/test-cm.yaml \
		  /tmp/test-cm-rs.yaml
```

[Result]
```
$ cat /tmp/test-cm-rs.yaml
apiVersion: apis.integrityshield.io/v1alpha1
kind: ResourceSignature
metadata:
  annotations:
    integrityshield.io/messageScope: spec
    integrityshield.io/signature: LS0tLS1CRUdJTiBQR1AgU0lHTkFUVVJFLS0tLS0KCmlRRkZCQUFCQ0FB...
  name: rsig-configmap-test-cm
  labels:
    integrityshield.io/sigobject-apiversion: v1
    integrityshield.io/sigobject-kind: ConfigMap
    integrityshield.io/sigtime: "1614143715"
spec:
  data:
    - message: H4sIAOPgNWAAA0ssyAxLLSrOzM...
      signature: LS0tLS1CRUdJTiBQR1AgU0lHTkFUVVJFLS0tLS0KCmlRRkZCQUFCQ0...
      type: resource
```
b. Create the signature in the cluster  

[Command]
```
oc create -f /tmp/test-cm-rs.yaml -n secure-ns
```
[Result]
```
resourcesignature.apis.integrityshield.io/rsig-configmap-test-cm created
```

c. Create the sample resource in the cluster

[Command]
```
oc create -f /tmp/test-cm.yaml -n secure-ns
```
[Result]
```
configmap/test-cm created
```


### 6. Prepare a changed sample resource  

[Command]
```
cat << EOF > /tmp/test-cm2.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm
data:
  key1: val1
  key2: val2
EOF
```

[OC-MANAGED]  
### 7. Try to update sample resource and confirm the request is blocked  

[Command]
```
oc replace -f /tmp/test-cm2.yaml -n secure-ns
```
[Result]
```
Error from server: error when replacing "/tmp/test-cm2.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but failed to verify signature; The message for this signature in ResourceSignature is not identical with the requested object. diff: {"items":[{"key":"data.key4","values":{"after":null,"before":"val4"}}]} (Request: {"kind":"ConfigMap","name":"test-cm","namespace":"secure-ns","operation":"UPDATE","request.uid":"61360b1b-64e9-4cd5-ac2d-62d6223d5288","scope":"Namespaced","userName":"IAM#rurikudo@ibm.com"})
```


## Expected result

### 1. A ResourceSignature is created in the cluster.

[Command]
```
oc get resourcesignatures.apis.integrityshield.io -n secure-ns
```

[Result]
```
NAME                     AGE
rsig-configmap-test-cm   10s
```
### 2. A ConfigMap is created in the cluster.

[Command]
```
oc get cm -n secure-ns
```
[Result]
```
NAME      DATA   AGE
test-cm   3      9s
```

### 3. Confirm data of configmap was not changed from the content in `/tmp/test-cm.yaml`

[Command]
```
oc get cm -n secure-ns test-cm -o json | jq .data
```

[Result] 
```
{
  "key1": "val1",
  "key2": "val2",
  "key4": "val4"
}
```
