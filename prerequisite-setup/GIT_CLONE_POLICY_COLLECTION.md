# Clone Policy Collection Git repositoy.

### Goal:
- Use can retrieve the source of ACM policies by  GitHub repository.

### Prerequisite: 
- Git setup in signing host
 
### Action Steps:

1. Fork [policy-collection](https://github.com/open-cluster-management/policy-collection) Git repository to your organization Git repostitory.

2. Git clone your forked policy collection repository.

   Replace `<YOUR-ORG-NAME>` for your organization name.
   
   [Command]
   ```
   git clone https://github.com/<YOUR-ORG-NAME>/policy-collection.git
   cd policy-collection
   ls .
   ```
   
### Expected Result:

   The following directories must exist.
   
   [Result]
   ```
   LICENSE  OWNERS  README.md  SECURITY.md  community  deploy  docs  stable
   ```


   
