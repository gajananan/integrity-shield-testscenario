### Goal:
When the request is blocked, user can check the Events

### Prerequisite: 
1. Integrity shield is ready in a `managed cluster`
2. you can connect via oc to a `managed cluster`

### Action steps:
1. create a namespace  
 
[Action by oc command on ACM Managed Cluster]
    ```
    oc create ns secure-ns 
    ```
    [Result]
    ```
    namespace/secure-ns created
    ```
If `secure-ns` already exists, refresh the namespace or create another namespace for this test.

2. Define which reource(s) should be protected  
[Action by oc command on ACM Managed Cluster] Type the following command
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

3. prepare a sample resource   
[Action by console command]
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

4. Create a respirce without signatrue  
[Action by oc command on ACM Managed Cluster]
    ```
    oc create -f /tmp/test-cm.yaml -n secure-ns
    ```
    [Result]  
    The resource cannot be created when it does not have a signatrue.
    ```
    Error from server: error when creating "/tmp/test-cm.yaml": admission webhook "ac-server.integrity-shield-operator-system.svc" denied the request: Signature verification is required for this request, but no signature is found. Please attach a valid signature. (Request: {"kind":"ConfigMap","name":"test-cm","namespace":"secure-ns","operation":"CREATE","request.uid":"3d0420fc-f53f-4de8-affc-fa67daace142","scope":"Namespaced","userName":"kubernetes-admin"})
    ```

### Expected result:
IntegrityShieldEvent is generated.  
[Action by oc command on ACM Managed Cluster]
```
oc get EVENT -n secure-ns
```
[Result]
```
LAST SEEN   TYPE              REASON         OBJECT              MESSAGE
4s          IntegrityShield   no-signature   configmap/test-cm   [IntegrityShieldEvent] Result: deny, Reason: "Signature verification is required for this request, but no signature is found. Please attach a valid signature." (RSP `namespace: secure-ns, name: sample-rsp`), Request: {"kind":"ConfigMap","name":"test-cm","namespace":"secure-ns","operation":"CREATE","request.uid":"61f9c54e-dad9-4c0b-8bd8-81723a63aec3","scope":"Namespaced","userName":"kubernetes-admin"}
```