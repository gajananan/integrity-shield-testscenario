## Goal:
User can define various rules in RSP, and these rules work

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


[OC-MANAGED]    

### 2. Define and install a ResourceSigningProfile into `secure-ns` namespace

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
  ignoreRules:
  - match:
    - kind: ConfigMap
      name: ignored-cm
  ignoreAttrs:
  - attrs:
    - data.comment1
    match:
    - name: protected-cm
      kind: ConfigMap
EOF
```
[Result]
```
resourcesigningprofile.apis.integrityshield.io/sample-rsp created
```
### 3. Prepare sample configmaps  
    
[Command]  
a. protected configmap
```
cat << EOF > /tmp/protected-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: protected-cm
data:
  comment1: val1
  comment2: val2
EOF
```

b. ignored configmap
```
cat << EOF > /tmp/ignored-cm.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: ignored-cm
data:
  comment1: val1
  comment2: val2
EOF
```

[OC-MANAGED]

### 4. Try to deploy configmaps without signature  
a. deploy protected ConfigMap without signature  
    [Command]
```
oc create -f /tmp/protected-cm.yaml -n secure-ns
```

[Result]  
The protected configmap cannot be created because it does not have a signature.
```
Error from server: error when creating "/tmp/protected-cm.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"ConfigMap","name":"protected-cm","namespace":"secure-ns","operation":"CREATE","request.uid":"38d4faee-b7ed-4e53-8c65-b9d4d5dd2dce","scope":"Namespaced","userName":"kubernetes-admin"})
```    
b. deploy ignored ConfigMap without signature  
[Command]
```
oc create -f /tmp/ignored-cm.yaml -n secure-ns
```
[Result]  
The ignored configmap can be created without signature because it is defined in `ignoreRules`.
```
configmap/ignored-cm created
```

### 5. generate a signature for protected comfigmap  
[Command]
```
curl -s  https://raw.githubusercontent.com/IBM/integrity-enforcer/master/scripts/gpg-annotation-sign.sh | bash -s \
signer@enterprise.com \
/tmp/protected-cm.yaml
```
[OC-MANAGED]    

### 6. deploy protected ConfigMap with signature  
[Command]
```
oc create -f /tmp/protected-cm.yaml -n secure-ns
```
[Result]
```
configmap/protected-cm created
```

### 7. edit protected configmap without signature  
a. change the whitelisted part  
[Command]
```
oc edit cm protected-cm -n secure-ns
```
then change the value for `data.comment2` as show below and save it.

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  comment1: val1
  comment2: changed
kind: ConfigMap
metadata:
  annotations:
    integrityshield.io/lastVerifiedTimestamp: 2021-02-25T02:22:15Z
    
```

[Result]  

```
error: configmaps "protected-cm" could not be patched: admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but failed to verify signature; The message for this signature in annotation is not identical with the requested object. diff: {"items":[{"key":"data.comment2","values":{"after":"changed","before":"val2"}}]} (Request: {"kind":"ConfigMap","name":"protected-cm","namespace":"secure-ns","operation":"UPDATE","request.uid":"e93368cc-096e-424e-a933-90b04e6897f0","scope":"Namespaced","userName":"kubernetes-admin"})
You can run `oc replace -f /var/folders/92/2jrp2pzs7cn0tzn49s78wjyr0000gn/T/oc-edit-c3orm.yaml` to try this update again.
```

b. change the whitelisted part  
[Command]
```
oc edit cm protected-cm -n secure-ns
```
then change the value for `data.comment1` as shown below and save it.

```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
data:
  comment1: changed
  comment2: val2
kind: ConfigMap
metadata:
  annotations:
    integrityshield.io/lastVerifiedTimestamp: 2021-02-25T02:22:15Z
```

[Result]
```
configmap/protected-cm edited
```

