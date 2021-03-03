## Goal:
Cluster scope resource can be protected with signature

## Prerequisite: 
1. Integrity shield is ready in a `managed cluster`
2. you can connect via oc to a `managed cluster`
> Prerequisites are already done by `1. Prepare the test environement` and `2. Complete Install Scenarios`

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

### 2. prepare sample resource (ClusterRoleBinding)    
[Command]
```
cat << EOF > /tmp/test-crb.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: sample-crb
subjects:
- kind: User
  name: dave
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: secret-reader
  apiGroup: rbac.authorization.k8s.io
EOF
```


### 3. setup RSP  
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
    - kind: ClusterRoleBinding
      name: sample-crb
EOF
```
[Result]
```
resourcesigningprofile.apis.integrityshield.io/sample-rsp created
```
    

### 4. deploy the resource without signature  
[Command]  
```
oc create -f /tmp/test-crb.yaml
```
[Result]  
The request is blocked by Integrity Shield.
```
Error from server: error when creating "/tmp/test-crb.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"ClusterRoleBinding","name":"sample-crb","namespace":"","operation":"CREATE","request.uid":"647daec5-3d66-4b75-9a73-44aebed7e1a0","scope":"Cluster","userName":"kubernetes-admin"})
```


### 5. generate signature  
[Command] 
```
curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-enforcer/master/scripts/gpg-annotation-sign.sh | bash -s \
signer@enterprise.com \
/tmp/test-crb.yaml 
```
[Command]

[Result]  
Check if the signatrue is attached.  
[Command]
```
cat /tmp/test-crb.yaml | grep integrityshield.io |  wc -l
```
[Result]
```
2
```


### 6. deploy resource  
[Command]
```
oc create -f /tmp/test-crb.yaml
```
[Result]
```
clusterrolebinding.rbac.authorization.k8s.io/sample-crb created
```


## Expected result:  
ClusterRoleBinding is deployed.  

[Command]
```
oc get clusterrolebinding | grep sample-crb
```
[Result]
```
sample-crb   ClusterRole/secret-reader   5m3s
```

