# Setup GPG Key

### Goal:
- User can setup GPG key in signing host

### Prerequisite: 
- GPG installed in signing host

 
### Action Steps:
1. Create signing and verification key pairs

   Run the following command to generate a GNUPG key and use default option for `kind of key`, `keysize`, and `how long key is valid` with your name `Signer` email address e.g. `signer@enterprise.com`).
   
   
   [Command]
   ```
   gpg --full-generate-key
   ```
   
   Run the following command to check if gpg key generated:
   
   [Command]
   ```
   gpg -k signer@enterprise.com
   ```
   
   [Result]
   ```
   pub   rsa3072 2021-01-18 [SC]
   0BC55659CFC149AF6AEE908EA7EA2BA4AC506AFC
   uid           [ unknown] Signer <signer@enterprise.com>
   sub   rsa3072 2021-01-18 [E]
   ```
   

2. Export the verification key a file

   [Command]
   ```
   gpg --export signer@enterprise.com > /tmp/pubring.gpg
   ```

### Expected Result:

   [Command]
   ```
   ls -al /tmp/pubring.gpg
   ```
   
   [Result]
   ```
   -rw-rw-r-- 1 user user 1776 Feb 16 01:23 /tmp/pubring.gpg
   ```
