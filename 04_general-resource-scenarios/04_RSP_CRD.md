## Goal:
Custom resource can be protected with signature

## Prerequisite: 
1. Integrity shield is ready in a `managed cluster`
2. you can connect via oc to a `managed cluster`

## Action steps:

[OC-MANAGED]
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


### 2. Prepare sample Customer resources  
a. CustomResourceDefinition  
[Command]  
```
cat <<EOF > /tmp/test-crd.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  # name must match the spec fields below, and be in the form: <plural>.<group>
  name: crontabs.stable.example.com
spec:
  # group name to use for REST API: /apis/<group>/<version>
  group: stable.example.com
  # list of versions supported by this CustomResourceDefinition
  versions:
    - name: v1
      # Each version can be enabled/disabled by Served flag.
      served: true
      # One and only one version must be marked as the storage version.
      storage: true
      schema:
        openAPIV3Schema:
          type: object
          properties:
            spec:
              type: object
              properties:
                cronSpec:
                  type: string
                image:
                  type: string
                replicas:
                  type: integer
  # either Namespaced or Cluster
  scope: Namespaced
  names:
    # plural name to be used in the URL: /apis/<group>/<version>/<plural>
    plural: crontabs
    # singular name to be used as an alias on the CLI and for display
    singular: crontab
    # kind is normally the CamelCased singular type. Your resource manifests use this.
    kind: CronTab
    # shortNames allow shorter string to match your resource on the CLI
    shortNames:
    - ct
EOF
```

b. CustomResource  
[Command]

```
cat << EOF > /tmp/test-cr.yaml
apiVersion: "stable.example.com/v1"
kind: CronTab
metadata:
  name: my-new-cron-object
spec:
  cronSpec: "* * * * */5"
  image: my-awesome-cron-image
EOF
```

[OC-MANAGED]  

### 3. Setup RSP  
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
    - kind: CronTab
    - kind: CustomResourceDefinition
        name: crontabs.stable.example.com 
    - kind: Service
EOF
```

[Result]

```
resourcesigningprofile.apis.integrityshield.io/sample-rsp created
```

[OC-MANAGED]    

### 4. Deploy CustomResourceDefinition without signature  

[Command]

```
oc create -f /tmp/test-crd.yaml
```

[Result]

```
Error from server: error when creating "/tmp/test-crd.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"CustomResourceDefinition","name":"crontabs.stable.example.com","namespace":"","operation":"CREATE","request.uid":"8536c2db-4431-4f26-ada5-2fdde6fe7166","scope":"Cluster","userName":"kubernetes-admin"})
```
### 5. Generate signature for CustomResourceDefinition
[Command]
```
curl -s  https://raw.githubusercontent.com/IBM/integrity-enforcer/master/scripts/gpg-annotation-sign.sh | bash -s \
signer@enterprise.com \
/tmp/test-crd.yaml
```
[Result]  
Check if the signatrue is attached.  
[Command]
```
cat /tmp/test-crd.yaml | grep integrityshield.io |  wc -l
```
[Result]
```
2
```
### 6. Deploy CustomResourceDefinition with signature
[Command]
```
oc create -f /tmp/test-crd.yaml
```

[Result]
```
customresourcedefinition.apiextensions.k8s.io/crontabs.stable.example.com created
```
### 7. Deploy CustomResource without signature    

[Command]
```
oc create -f /tmp/test-cr.yaml -n secure-ns
```
[Result]
```
Error from server: error when creating "/tmp/test-cr.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"CronTab","name":"my-new-cron-object","namespace":"secure-ns","operation":"CREATE","request.uid":"75b3c6e9-0cf6-47e9-a03a-b68f90527a92","scope":"Namespaced","userName":"kubernetes-admin"})
```

### 8. Generate signature for CustomResource
[Command]   
```
curl -s  https://raw.githubusercontent.com/IBM/integrity-enforcer/master/scripts/gpg-annotation-sign.sh | bash -s \
signer@enterprise.com \
/tmp/test-cr.yaml
```
[Result]  
Check if the signatrue is attached.  
[Command]
```
cat /tmp/test-cr.yaml | grep integrityshield.io |  wc -l
```
[Result]
```
2
```
### 9. Deploy CustomResource with signature  

[Command]
```
oc create -f /tmp/test-cr.yaml -n secure-ns
```
[Result]
```
crontab.stable.example.com/my-new-cron-object created
```

[OC-MANAGED]   
## Expected result: 
CustomResourceDefinition and CustomResource are deployed.  
a.  CustomResourceDefinition  
[Command]
```
oc get crd | grep crontabs.stable.example.com
```
[Result]
```
crontabs.stable.example.com                       2021-02-25T04:44:18Z
```
b. CustomResource  
[Command]
```
oc get crontab -n secure-ns
```
[Result]
```
NAME                 AGE
my-new-cron-object   26s
```
