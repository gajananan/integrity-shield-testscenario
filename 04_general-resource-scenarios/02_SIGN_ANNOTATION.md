## Goal:

Define RSP and protect a resource with signature(annotation) in newly created namespace

## Prerequisite: 
1. Integrity shield is ready in a `managed cluster`
2. you can connect via oc to a `managed cluster`

## Action steps:

[OC-MANAGED]
### 1. Create a namespace  

 [Command]
  ```
  oc create ns secure-ns 
  ```
  [Result]
- succeeded 
```
 namespace/secure-ns created
```
- failed
```
Error from server (AlreadyExists): namespaces "secure-ns" already exists
```
If `secure-ns` already exists, refresh the namespace or create another namespace for this test.   
[Command]  
```
oc delete ns secure-ns
oc create ns secure-ns
```

[OC-MANAGED]  
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

### 3. Prepare a sample resource. 

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
### 4. Create a resource without signatrue  
[Command]
  ```
  oc create -f /tmp/test-cm.yaml -n secure-ns
  ```
[Result]  
The resource cannot be created when it does not have a signatrue.
```
Error from server: error when creating "/tmp/test-cm.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"ConfigMap","name":"test-cm","namespace":"secure-ns","operation":"CREATE","request.uid":"3d0420fc-f53f-4de8-affc-fa67daace142","scope":"Namespaced","userName":"kubernetes-admin"})
```

### 5. Create a resource with signature   
a. Generate a signature annotation for the sample resource  

[Command]
```
curl -s  curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-enforcer/master/scripts/gpg-annotation-sign.sh | bash -s \
signer@enterprise.com \
/tmp/test-cm.yaml

```
[Command]
```
cat /tmp/test-cm.yaml | grep integrityshield.io |  wc -l
```
[Result]
```
2
```

[OC-MANAGED]  
b. Install the sample resource  
[Command]
```
oc create -f /tmp/test-cm.yaml -n secure-ns
```
[Result]
```
configmap/test-cm created
```


### 6. Updating sample resource without signature is not accepted.
a. Prepare changed sample resource  
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
Error from server: error when replacing "/tmp/test-cm2.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"ConfigMap","name":"test-cm","namespace":"secure-ns","operation":"UPDATE","request.uid":"31573fbd-85e6-4eba-a4f6-9b1891c55a44","scope":"Namespaced","userName":"IAM#rurikudo@ibm.com"})
```

## Expected result

### 1. A ConfigMap is created in the cluster.

[Command]
```
oc get cm -n secure-ns
```
[Result]
```
NAME      DATA   AGE
test-cm   3      9s
```

### 2. Confirm data of configmap was not changed from the content in `/tmp/test-cm.yaml`

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

