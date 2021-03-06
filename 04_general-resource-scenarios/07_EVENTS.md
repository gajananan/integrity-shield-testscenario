## Goal:
When the request is blocked, user can check the Events

## Prerequisite: 
1. Integrity shield is ready in a `managed cluster`
2. you can connect via oc to a `managed cluster`

## Action steps:
[OC-MANAGED]The following oc commands should be run on <font color="IndianRed"> Managed Cluster</font>

### 1. create a namespace  

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


### 2. Define which reource(s) should be protected  
[Command]  Type the following command
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

### 3. prepare a sample resource   
[Command] 
```
cat << EOF > /tmp/test-cm-event-test.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-cm-event-test
data:
  key1: val1
  key2: val2
  key4: val4
EOF
```

### 4. Create a respirce without signatrue  
[Command] 
```
oc create -f /tmp/test-cm-event-test.yaml -n secure-ns
```
[Result]  
The resource cannot be created when it does not have a signatrue.
```
Error from server: error when creating "/tmp/test-cm.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"ConfigMap","name":"test-cm","namespace":"secure-ns","operation":"CREATE","request.uid":"3d0420fc-f53f-4de8-affc-fa67daace142","scope":"Namespaced","userName":"kubernetes-admin"})
```

## Expected result:
Check that IntegrityShieldEvent is generated.  
[Command]
```
oc get EVENT -n secure-ns | grep configmap/test-cm-event-test | head -n 1 | grep 'Result: deny' | wc -l
```
[Result]
```
1
```