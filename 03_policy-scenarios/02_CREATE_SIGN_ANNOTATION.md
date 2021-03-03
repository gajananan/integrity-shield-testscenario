# Create signature annotatoion

### Goal:
- User can attach a signature to policy file

### Prerequisite: 
- Policy collection is already clone locally in signing host (already done in [prerequisite-setup](../prerequisite-setup/GIT_CLONE_POLICY_COLLECTION.md)
- Setup GPG keys (already done in [doc](../prerequisite-setup/GPG_KEY_SETUP.md)
 
### Action Steps:
1. Go to the directory of your cloned policy collection Git repository in the signing host
   
   [Command]
   ```
   cd <SIGING HOST DIR>/policy-collection
   ```
   
2. Sign the policy file `community/SC-System-and-Communications-Protection/policy-ocp4-certs.yaml`
   
   [Command]
   ```
   curl -s  https://raw.githubusercontent.com/open-cluster-management/integrity-shield/master/scripts/gpg-annotation-sign.sh | bash -s \
   signer@enterprise.com \
   community/SC-System-and-Communications-Protection/policy-ocp4-certs.yaml
   ```
   
### Expected Result:

   Confirm signature annotations are attached to the policy file.
   
   [Command]
   ```
   cat community/SC-System-and-Communications-Protection/policy-ocp4-certs.yaml | grep 'integrityshield.io/' | wc -l
   ```
   
   [Result]
   ```
    3
   ```
