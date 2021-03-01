# Disable IShield in an ACM Managed Cluster

### Goal:
- User can disable IShield protection to an ACM managed cluster.

### Prerequisite: 
- Policy collection is already cloned locally in signing host as described in [doc](../prerequisite-setup/GIT_CLONE_POLICY_COLLECTION.md)
- IShield protection is enabled as described in [doc](../install-scenarios/DEPLOY_ISHIELD.md)
 
### Action Steps:
1. Go to the directory of your cloned policy collection Git repository in the signing host.

   [Command]
   ```
   cd <SIGING HOST DIR>/policy-collection
   ```
2. Edit `community/CM-Configuration-Management/policy-integrity-shield.yaml`
   - In lines 26, 41, 60, 83,  change from `musthave` to `mustnothave`
   
3. Sign `community/CM-Configuration-Management/policy-integrity-shield.yaml` policy with `signer@enterprise.com`
 
    ```
    curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-shield/master/scripts/gpg-annotation-sign.sh | bash -s \
        signer@enterprise.com \
        community/CM-Configuration-Management/policy-integrity-shield.yaml
    ```
    
4. Check if two annotations started with "integrityshield.io" are attached to community/CM-Configuration-Management/policy-integrity-shield.yaml
 
    ```
    cat community/CM-Configuration-Management/policy-integrity-shield.yaml | grep 'integrityshield.io/' | wc -l
    7
    ```        
    
5. Commit your changes in `policy-integrity-shield.yaml` to your cloned policy-collection git repository.

   [Command]
   ```
   git add community/CM-Configuration-Management/policy-integrity-shield.yaml
   git commit -m 'policy-integrity-shield.yaml with signature'
   git push origin master
   ```   
   [Result]
   
   <ScreenShot>       
 
[OC-HUB]   

6. Switch to your console connecting to ACM Hub Cluster, run the following command to remove verification key from ACM HUB/Managed Clusters
   
   [Command]
   ```
   curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-shield/master/scripts/ACM/acm-verification-key-setup.sh | bash -s \
          integrity-shield-operator-system  \
          keyring-secret  \
          /tmp/pubring.gpg \
          environment:dev  |  kubectl delete -f -
   ```

  
### Expected Result:

Above changes in Git repository will be synced by ACM Hub Cluster to update the changes in policy.  
After a minute, continue to check the expected results.
    
[WebConsle-HUB]

1. Connect to ACM Hub Cluster WebConsole and go to polices page.
2. Search for `policy-integrity-shield`  in Find Policies.  
3. Click  `policy-integrity-shield`  policy. 
4. Check if  `policy-integrity-shield`  is in compliant state (Cluster violation -> green) as show below.

![Policy Compliant After Disabling IShield](../images/policy-compliant-after-delete.PNG)

5. Click status tab in policy-integrity-shield policy page and confirm the compliant as below:

![Policy Compliant Status After Disabling IShield](../images/policy-compliant-status-after-delete.PNG)

[OC-MANAGED]

6. Check if IShield components are removed in in the ACM Managed Cluster via OC commands.
   
   [Command]
   ```
   oc get pod -n integrity-shield-operator-system
   ```
   
   [Result]
   ```
   No resources found in integrity-shield-operator-system namespace.
   ```








