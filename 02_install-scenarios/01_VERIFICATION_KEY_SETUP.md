# Verification Key Setup

### Goal:
- User can setup a verification key to an ACM managed cluster.

### Prerequisite: 
- An ACM Hub Cluster and at least one ACM Managed Cluster attached to it.
- A pair of GPG Keys are setup as described in [doc](../prerequisite-setup/GPG_KEY_SETUP.md)


### Action Steps:
 [OC-HUB]
 1. Connect via oc to an ACM hub cluster
    
    [Command]
    ```
    oc login --token=xxxxxxxxxxxxxxxxx  --server=https://api.hub..openshiftv4test.com:6443
    ```
    [Result]
    ```
    Logged into "https://api.hub.openshiftv4test.com:6443" as "kube:admin" using the token provided.
    You have access to 65 projects, the list has been suppressed. You can list all projects with ' projects'
    Using project "default".
    ```
   
 2. Create a new namespace in the ACM hub cluster
 
    [Command]
    ```
    oc create ns integrity-shield-operator-system
    ```
    
    [Result]
    ```
    namespace/integrity-shield-operator-system created
    ```
    
    
 3. Deploy verification key to an ACM hub cluster so that it can propagate to a managed cluster(s).

    [Parameters]
      - secret name: `keyring-secret` 
      - namespace:  `integrity-shield-operator-system` 
      - pubring file:  /tmp/pubring.gpg
      - placement rule:   environment:dev (Note: Change this)
      
    [Command]  
    
    ```
    curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-shield/master/scripts/ACM/acm-verification-key-setup.sh | bash -s \
    --namespace integrity-shield-operator-system  \
    --secret keyring-secret  \
    --path /tmp/pubring.gpg \
    --label environment=dev  |  oc apply -f -
    ```
    
    [Result]
    ```
    secret/keyring-secret created
    channel.apps.open-cluster-management.io/keyring-secret-deployments created
    placementrule.apps.open-cluster-management.io/secret-placement created
    subscription.apps.open-cluster-management.io/keyring-secret created
    ```
    
### Expected Result:

 After a minute, the above steps will create  secret (that includes the verification key) in selected ACM managed cluster(s), so you can start confirming results below.
 
 [OC-HUB]
 1. Confirm secret is successfully created in ACM Hub Cluster
 
    [Command]
  
    ```
    oc get secret -n integrity-shield-operator-system keyring-secret
    ```
    
    [Result]
    ```
    NAME             TYPE     DATA   AGE
    keyring-secret   Opaque   1      26h
    ```
    
 [OC-MANAGED]  
 
 2. Switch connection to managed cluster and confirm secret is successfully propagated to  ACM Managed Cluster(s)
 
    [Command] 
    ```
    oc get secret -n integrity-shield-operator-system keyring-secret
    ```
    
    [Result]
    ```
    NAME             TYPE     DATA   AGE
    keyring-secret   Opaque   1      26h
    ``` 
   
