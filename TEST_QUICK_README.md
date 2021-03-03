# Test IShield protection for RHACM Policies and any application resources

### Goal:
- User can verify if IShield enabled in an ACM Managed Cluster can successfully protect the integrity of RHACM Policies in [policy-collection](https://github.com/open-cluster-management/policy-collection)
- User can verify if IShield enabled in an OpenShift Cluster can successfully protect the integrity of specified application resources.

### How to perform test: 

- Execute the commands shown under `[Command]` in the following test scenarios.

- Confirm the results shown under `[Result]` of executing a command in the following test scenarios.
 
### Action Steps:


1. Prepare the test environement as describe in [doc](./01_tester-setup/PREPARE_TEST_ENV.md)

2. Complete Install Scenarios
   - Verification key setup as in [doc](./02_install-scenarios/01_VERIFICATION_KEY_SETUP.md)
   - Enable IShield protection in an ACM Managed Cluster as in [doc](./02_install-scenarios/02_ENABLE_ISHIELD.md)
   
3. Verify Policy Scenarios
   - Update Policy without a signature as in [doc](./03_policy-scenarios/01_CHANGE_POLICY_NO_SIGN.md)
   - Create signature annotation in a policy file as in [doc](./03_policy-scenarios/02_CREATE_SIGN_ANNOTATION.md)
   - Update Policy with a valid signature as in [doc](./03_policy-scenarios/03_CHANGE_POLICY_WITH_SIGN.md)
   - Update Policy with an invalid signature as in [doc](./03_policy-scenarios/04_CHANGE_POLICY_INVALID_SIGNER.md)
   - Create stable policies with valid signatures as in [doc](./03_policy-scenarios/05_CREATE_STABLE_POLICY_WITH_SIGN.md)
   
4. Verify General Resources
   - Install general resources with signature(resource signature) [doc](./04_general-resource-scenarios/01_SIGN_RESOURCE_SIGNATRUE.md)
   - Install general resources with signature(annotation) [doc](./04_general-resource-scenarios/02_SIGN_ANNOTATION.md)
   - Install cluster scope resource [doc](./04_general-resource-scenarios/03_RSP_CLUSTER_SCOPE.md)
   - Install custom resource [doc](./04_general-resource-scenarios/04_RSP_CRD.md)
   - Various RSP patterns [doc](./04_general-resource-scenarios/05_RSP_VARIOUS_PATTERNS.md)
   - Multiple signers and multiple keys [doc](./04_general-resource-scenarios/06_MULTI_SIGNERS_MULTI_KEYS.md)
   - Check IntegrityShieldEvent [doc](./04_general-resource-scenarios/07_EVENTS.md)

5. Complete Uninstall Scenarios
   - Disable IShield protection in an ACM Managed Cluster as in [doc](./05_uninstall-scenarios/01_DISABLE_ISHIELD.md)
