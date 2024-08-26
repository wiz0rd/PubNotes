# Juniper Password Management, Hashing, and Configuration TTP

## Index
1. [[#Introduction]]
2. [[#Password Generation and Storage]]
   1. [[#Generating Password with KeePass]]
   2. [[#Saving Password in KeePass]]
3. [[#Hashing the Password]]
   1. [[#Setting Up the Hashing Script]]
   2. [[#Running the Hashing Script]]
4. [[#Configuring Hashed Password on Juniper Device]]
   1. [[#Accessing the Juniper Device]]
   2. [[#Applying the Hashed Password]]
5. [[#Best Practices]]
6. [[#Troubleshooting]]
7. [[#Links]]

## Introduction
This document outlines the complete process of generating a secure password, storing it safely, hashing it for use with Juniper devices, and applying the hashed password to a Juniper device configuration.

## Password Generation and Storage

### Generating Password with KeePass
1. Open KeePass Password Safe.
2. Create a new entry or edit an existing one for the Juniper device user.
3. Click on the password field, then click on the ellipsis (...) button next to it.
4. In the password generator:
   - Set an appropriate length (e.g., 20 characters).
   - Enable uppercase, lowercase, numbers, and special characters.
   - Ensure high entropy by using the built-in quality meter.
5. Click "OK" to generate the password.

### Saving Password in KeePass
1. In the KeePass entry, fill in other relevant fields:
   - Title: e.g., "Juniper Device Admin"
   - Username: The username for the Juniper device
   - URL: The management IP or hostname of the Juniper device (if applicable)
   - Notes: Add any relevant information, including the device model, location, etc.
2. Save the entry in KeePass.

## Hashing the Password

### Setting Up the Hashing Script
1. Create a new Python file named `juniper_hash.py` with the following content:

```python
import crypt
import os
import argparse

def hash_juniper_password(password):
    salt = os.urandom(6).hex()
    hashed = crypt.crypt(password, f'$5${salt}$')
    return hashed

if __name__ == "__main__":
    parser = argparse.ArgumentParser(description="Hash a password for Juniper devices.")
    parser.add_argument("password", help="The password to hash")
    args = parser.parse_args()

    hashed_password = hash_juniper_password(args.password)
    print(f"Hashed password: {hashed_password}")
```

2. Save this file in a secure location on your management workstation.

### Running the Hashing Script
1. Open a terminal or command prompt.
2. Navigate to the directory containing `juniper_hash.py`.
3. Run the script with the password as an argument:
   ```
   python juniper_hash.py "YourGeneratedPasswordHere"
   ```
4. The script will output the hashed password. Copy this output.
5. Update the KeePass entry:
   - Paste the hashed password into the 'Notes' field of the KeePass entry.
   - Label it clearly, e.g., "Hashed password for Juniper config:"

## Configuring Hashed Password on Juniper Device

### Accessing the Juniper Device
1. Establish a secure connection to your Juniper device (e.g., via SSH or console).
2. Enter configuration mode:
   ```
   configure
   ```

### Applying the Hashed Password
1. Use the following command to set the hashed password:
   ```
   set system login user <username> authentication encrypted-password "<hashed-password>"
   ```
   Replace `<username>` with the actual username and `<hashed-password>` with the output from the hashing script.
2. Commit the changes:
   ```
   commit check
   ```
   If no errors are reported, then:
   ```
   commit
   ```
3. Exit configuration mode:
   ```
   exit
   ```

## Best Practices
1. Use a unique, strong password for each device and user.
2. Regularly rotate passwords (e.g., every 90 days or according to your security policy).
3. Use multi-factor authentication when possible.
4. Keep the KeePass database secure and regularly backed up.
5. Ensure the Python script for hashing is stored securely and not accessible to unauthorized users.
6. Always double-check the hashed password input to avoid typos that could lead to lockouts.
7. Maintain an out-of-band access method (e.g., console) in case of authentication issues.

## Troubleshooting
- If the hashed password doesn't work, verify that you've copied the entire hash, including the `$5$` prefix and the salt.
- Ensure there are no extra spaces or line breaks in the hashed password when entering it on the Juniper device.
- If you're locked out, use your out-of-band access method and revert to the last known good configuration.

## Links
- [KeePass Password Safe](https://keepass.info/)
- [Juniper SRX Series User Authentication](https://www.juniper.net/documentation/us/en/software/junos/user-access/topics/topic-map/user-access-authentication-overview.html)
- [Juniper CLI User Guide](https://www.juniper.net/documentation/us/en/software/junos/junos-cli/index.html)
- [Python argparse module](https://docs.python.org/3/library/argparse.html)
- [Python crypt module](https://docs.python.org/3/library/crypt.html)
