# Cracking Protected Files

The use of file encryption is often neglected in both `private` and `professional` contexts. Even today, emails containing job applications, account statements, or contracts are frequently sent without encryption—sometimes in violation of legal regulations. For example, within the European Union, the [General Data Protection Regulation (GDPR)](https://gdpr-info.eu/) requires that personal data be encrypted both in transit and at rest. Nevertheless, it remains standard practice to discuss `confidential` topics or transmit `sensitive` data via email, which may be intercepted by attackers positioned to exploit these communication channels.

As more companies enhance their IT security infrastructure through training programs and security awareness seminars, it is becoming increasingly common for employees to encrypt sensitive files. Nevertheless, encrypted files can still be cracked and accessed with the right combination of wordlists and tools. In many cases, `symmetric encryption` algorithms such as `AES-256` are used to securely store individual files or folders. In this method, the same key is used for both encryption and decryption. For transmitting files, `asymmetric encryption` is typically employed, which uses two distinct keys: the sender encrypts the file with the recipient's `public key`, and the recipient decrypts it using the corresponding `private ke`y.

Up until now, we've focused on cracking password hashes specifically. In the next two sections, we will shift our focus to techniques related to attacking password-protected files and archives.

## Hunting for Encrypted Files

Many different extensions correspond to encrypted files—a useful reference list can be found on [FileInfo](https://fileinfo.com/filetypes/encoded). As an example, consider this command we might use to locate commonly encrypted files on a Linux system:
```sh
for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

![[cpf_1.png]]

## Hunting for SSH keys

Certain files, such as SSH keys, do not have standard file extension. In cases like these, it may be possible to identify files by standard content such as header and footer values. For example, SSH private keys always begin with `-----BEGIN [...SNIP...] PRIVATE KEY-----`. We can use tools like `grep` to recursively search the file system for them during post-exploitation.

```sh
grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```

Some SSH keys are encrypted with a passphrase. With older PEM formats, it was possible to tell if an SSH key is encrypted based on the header, which contains the encryption method in use. Modern SSH keys, however, appear the same whether encrypted or not.
![[cpf_2.png]]

One way to tell whether an SSH key is encrypted or not, is to try reading the key with `ssh-keygen`
![[cpf_3.png]]

As shown below, attempting to read a password-protected SSH key will prompt the user for a passphrase:
![[cpf_4.png]]


### Q.1 Download the attached ZIP archive (cracking-protected-files.zip), and crack the file within. What is the password?
![[cpf_5.png]]

![[cpf_6.png]]

![[cpf_7.png]]

