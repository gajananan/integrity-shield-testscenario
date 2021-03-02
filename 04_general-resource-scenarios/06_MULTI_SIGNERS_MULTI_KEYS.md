## Goal:
Resources can be signed by multiple signers and multiple signing keys are available

## Prerequisite: 
1. Connect via oc to an ACM Hub/Managed cluster
2. Integrity shield is ready in an `ACM managed cluster`
3. Setup a another signer.   
   a. Setup a new GPG key with email `signer2@enterprise.com` and export the verification key to `/tmp/signer2_pubring.gpg` as described in [doc](../prerequisite-setup/GPG_KEY_SETUP.md) 
   
    [OC-HUB]  
      b. Create another secret `keyring-secret-signer2` in the namespace `integrity-shield-operator-system` from file `tmp/signer2_pubring.gpg`
      
      [Command]
      ```
        oc create secret generic --save-config keyring-secret-signer2  -n integrity-shield-operator-system --from-file=/tmp/signer2_pubring.gpg
      ```
        
4. SignerConfig is configured as follows.
    ```
      signerConfig:
        policies:
        - namespaces:
          - "*"
          signers:
          - "SampleSigner"
          - "SampleSigner2"
        - scope: "Cluster"
          signers:
          - "SampleSigner"      
        signers:
        - name: "SampleSigner"
          keyConfig: sample-signer-keyconfig
          subjects:
          - email: "*"
        - name: "SampleSigner2"
          keyConfig: sample-signer2-keyconfig
          subjects:
          - email: "signer2@enterprise.com"
      keyConfig:
      - name: sample-signer-keyconfig
        secretName: keyring-secret
      - name: sample-signer2-keyconfig
        secretName: keyring-secret-signer2
        fileName: signer2_pubring.gpg
    ```

    [OC-MANAGED]

5. Check if multiple verification keys are created

    [Command]
    ```
    oc get secret -n integrity-shield-operator-system | grep keyring
    ```
    ```
    keyring-secret           Opaque                                1         18s
    keyring-secret-signer2   Opaque                                1         2m20s
    ```



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

### 2. define and install RSP into secure-ns  
  [Command]
  
  If `secure-ns` already exists, refresh the namespace or create another namespace for this test.

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

### 3. prepare sample configmap and deployment  
  a. configmap  
  [Command]
  ```
  cat << EOF > /tmp/signer1-cm.yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: signer1-cm
  data:
    comment1: val1
    comment2: val2
  EOF
  ```

  b. deployment  
  [Command]
  ```
  cat << EOF > /tmp/signer2-deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: signer2-deployment
    labels:
      app: nginx
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
  EOF
  ```
    
[OC-MANAGED]

### 4. deploy resources without signature  
  a. configmap  
  
  [Command] 
  ```
  oc create -f /tmp/signer1-cm.yaml -n secure-ns
  ```
  [Result]
  ```
  Error from server: error when creating "/tmp/signer1-cm.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"ConfigMap","name":"signer1-cm","namespace":"secure-ns","operation":"CREATE","request.uid":"1449a473-5046-49ce-9b1e-b27c2a39569c","scope":"Namespaced","userName":"kubernetes-admin"})
  ```
  b. deployment  
  
  [Command] 
  ```
  oc create -f /tmp/signer2-deployment.yaml -n secure-ns
  ```
  [Result]
  ```
  Error from server: error when creating "/tmp/signer2-deployment.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"Deployment","name":"signer2-deployment","namespace":"secure-ns","operation":"CREATE","request.uid":"38265a0e-ee9e-4f7a-860d-f3c8a538d1dd","scope":"Namespaced","userName":"kubernetes-admin"})
  ```

### 5. generate a signature with each key  
a. configmap with signer1  
[Command]
```
curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-enforcer/master/scripts/gpg-annotation-sign.sh | bash -s \
signer@enterprise.com \
/tmp/signer1-cm.yaml
```

  
[Result]  
Check if the signatrue is attached.  
[Command]
```
cat /tmp/signer1-cm.yaml | grep integrityshield.io |  wc -l
```
[Result]
```
2
```

  b. deployment with signer2  
  
  [Command]
  ```
  curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-enforcer/master/scripts/gpg-annotation-sign.sh | bash -s \
  signer2@enterprise.com \
  /tmp/signer2-deployment.yaml
  ```
[Result]  
Check if the signatrue is attached.  
[Command]
```
cat /tmp/signer2-deployment.yaml | grep integrityshield.io |  wc -l
```
[Result]
```
2
```
    
[OC-MANAGED]    

### 6. deploy resources with signature  
  a. configmap  
  [Command] 
  ```
  oc create -f /tmp/signer1-cm.yaml -n secure-ns
  ```
  [Result]
  ```
  configmap/signer1-cm created
  ```
  b. deployment  
  [Command] 
  ```
  oc create -f /tmp/signer2-deployment.yaml -n secure-ns
  ```
  [Result]
  ```
  deployment.apps/signer2-deployment created
  ```

[OC-MANAGED]

## Expected result:
Configmap and Deployment are deployed.  
  a. Configmap  
   [Command]
   ```
    oc get cm -n secure-ns
   ```
    
   [Result]
   ```
   NAME         DATA      AGE
   signer1-cm   2         17m
   ```
  b. Deployment  
   [Command]
   ```
   oc get deployment -n secure-ns
   ```
   [Result]
   ```
   NAME                 READY     UP-TO-DATE   AVAILABLE   AGE
   signer2-deployment   1/1       1            1           18m
   ```
