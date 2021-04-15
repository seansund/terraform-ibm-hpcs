# HPCS terraform module

Provisions an instance of Hyper Protect Crypto Service (HPCS) in the account.

## Example usage

```terraform-hcl
module "dev_infrastructure_hpcs" {
  source = "github.com/ibm-garage-cloud/terraform-ibm-key-hpcs?ref=v1.0.0"

  resource_group_name = module.dev_cluster.resource_group_name
  resource_location   = module.dev_cluster.region
  cluster_id          = module.dev_cluster.id
  namespaces          = []
  namespace_count     = 0
  name_prefix         = var.name_prefix
  tags                = []
  plan                = "standard"
  service_endpoints      = var.service_endpoints
  number_of_crypto_units = var.number_of_crypto_units
}
```

## Post-provisioning steps

After the HPCS instance has been provisioned it must be manually initialized before it can be used to encrypt services and volumes. The following provides a summary of the steps required to initialize the Hyper Protect instance. Please refer https://cloud.ibm.com/docs/hs-crypto?topic=hs-crypto-initialize-hsm for more detail.

![HPCS initialization](https://cloud.ibm.com/docs-content/v1/content/eaa381661ad91d7720185625ae7b5a047a6dc41c/hs-crypto//image/hsm_initialization_flow.svg)

### 1. Install the TKE CLI plug-in

Install the `tke` plug-in for the IBM Cloud cli:

```shell
ibmcloud plugin install tke
```

**Note:** If you have previously installed the plug-in you can update to the latest with the following:

```shell
ibmcloud plugin update tke
```

### 2. Set up the TKE environment

Set the `CLOUDTKEFILES` environment variable on your workstation to specify the directory where you want to save the master key part files and signature key files. The signature keys are used to sign TKE administrative commands.

```shell
export CLOUDTKEFILES=<local directory>
```

On the Linux® operating system or MacOS, that line can be added to the `~/.bash_profile` or `~/.zshrc` file if you want the setting to persist across terminal sessions.

### 3. Log in to IBM Cloud with the CLI

Log into the region and resource group where the HPCS instance has been provisioned. If you are authenticating with your IBM Cloud API Key, the command would look like:

```shell
ibmcloud login --apikey "${APIKEY}" -r "${REGION}" -g "${RESOURCE_GROUP}"
```

### 4. List the crypto units for the HPCS instance

Run the following command to list the crypto units that have been provisioned for the HPCS instance.

```shell
ibmcloud tke cryptounits
```

**Note:** If there are no crypto units available on the HPCS instance, follow the steps below in section X to add new crypto units.

### 5. Create the signature key

The following commands will add a signature key for the administrators of the HPCS instance. The first user added will be the overall admin of the instance. At least two administrators should be added to the instance.

#### 5.1 Add each user to the instance

1. Run the `sigkey-add` command - `ibmcloud tke sigkey-add`
2. Provide the name of the administrator when prompted
3. Provide a password for the administrator user

#### 5.2 Associate the signature key to the crypto units

1. Select the signature keys - `ibmcloud tke sigkey-sel`
2. Provide the numbers for the signature keys at the prompt. E.g. `1 2` to select the first and second signature keys.
3. Associate the signature key with the crypto unit - `ibmcloud tke cryptounit-admin-add`
4. Enter the number for the crypto unit and provide the password for the user

### 6. Set the Crypto Unit Threshold



# Reference

## List the service instances and crypto units

To display the service instances and crypto units in the target resource group under the current user account, use the following command:

```shell
ibmcloud tke cryptounits
```

Example output:
~~~~
API endpoint:     https://cloud.ibm.com
Region:           us-south
User:             john.smith@in.abc.com
Account:          Cloud-Native Squad (0f0adc9ace07413f96e2214f4bc0c40b)
Resource group:   appdev-cloud-native

SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   SELECTED   TYPE          LOCATION   
1                 false      OPERATIONAL   [us-south].[AZ2-CS5].[02].[06]   
2                 false      OPERATIONAL   [us-south].[AZ3-CS9].[01].[15]   
3                 false      RECOVERY      [us-south].[AZ3-CS9].[01].[14]   
4                 false      RECOVERY      [us-east].[AZ1-CS1].[01].[07]   
~~~~

**Note:** All operational crypto units in a service instance must be configured the same. Use `ibmcloud tke cryptounit-compare` to check how crypto units are configured.

## To add extra crypto units to the selected crypto unit list, use the following command:
### ibmcloud tke cryptounit-add

API endpoint:     https://cloud.ibm.com
Region:           us-south
User:             john.smith@in.abc.com
Account:          Cloud-Native Squad (0f0adc9ace07413f96e2214f4bc0c40b)
Resource group:   appdev-cloud-native

SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   SELECTED   TYPE          LOCATION   
1                 false      OPERATIONAL   [us-south].[AZ2-CS5].[02].[06]   
2                 false      OPERATIONAL   [us-south].[AZ3-CS9].[01].[15]   
3                 false      RECOVERY      [us-south].[AZ3-CS9].[01].[14]   
4                 false      RECOVERY      [us-east].[AZ1-CS1].[01].[07]   

Note: all operational crypto units in a service instance must be configured the same.
Use 'ibmcloud tke cryptounit-compare' to check how crypto units are configured.

Enter a list of CRYPTO UNIT NUM to add, separated by spaces:
> 1 2
OK

API endpoint:     https://cloud.ibm.com
Region:           us-south
User:             john.smith@in.abc.com
Account:          Cloud-Native Squad (0f0adc9ace07413f96e2214f4bc0c40b)
Resource group:   appdev-cloud-native

SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   SELECTED   TYPE          LOCATION   
1                 true       OPERATIONAL   [us-south].[AZ2-CS5].[02].[06]   
2                 true       OPERATIONAL   [us-south].[AZ3-CS9].[01].[15]   
3                 false      RECOVERY      [us-south].[AZ3-CS9].[01].[14]   
4                 false      RECOVERY      [us-east].[AZ1-CS1].[01].[07]   

Note: all operational crypto units in a service instance must be configured the same.
Use 'ibmcloud tke cryptounit-compare' to check how crypto units are configured.


## To add extra crypto units to the selected crypto unit list, use the following command:
### ibmcloud tke cryptounit-add

API endpoint:     https://cloud.ibm.com
Region:           us-south
User:             john.smith@in.abc.com
Account:          Cloud-Native Squad (0f0adc9ace07413f96e2214f4bc0c40b)
Resource group:   appdev-cloud-native

SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   SELECTED   TYPE          LOCATION   
1                 true       OPERATIONAL   [us-south].[AZ2-CS5].[02].[06]   
2                 true       OPERATIONAL   [us-south].[AZ3-CS9].[01].[15]   
3                 false      RECOVERY      [us-south].[AZ3-CS9].[01].[14]   
4                 false      RECOVERY      [us-east].[AZ1-CS1].[01].[07]   

Note: all operational crypto units in a service instance must be configured the same.
Use 'ibmcloud tke cryptounit-compare' to check how crypto units are configured.

Enter a list of CRYPTO UNIT NUM to add, separated by spaces:
> 3 4
OK

API endpoint:     https://cloud.ibm.com
Region:           us-south
User:             john.smith@in.abc.com
Account:          Cloud-Native Squad (0f0adc9ace07413f96e2214f4bc0c40b)
Resource group:   appdev-cloud-native

SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   SELECTED   TYPE          LOCATION   
1                 true       OPERATIONAL   [us-south].[AZ2-CS5].[02].[06]   
2                 true       OPERATIONAL   [us-south].[AZ3-CS9].[01].[15]   
3                 true       RECOVERY      [us-south].[AZ3-CS9].[01].[14]   
4                 true       RECOVERY      [us-east].[AZ1-CS1].[01].[07]   

Note: all operational crypto units in a service instance must be configured the same.
Use 'ibmcloud tke cryptounit-compare' to check how crypto units are configured.

## You can compare the configuration settings of the selected crypto units with the following command:
### ibmcloud tke cryptounit-compare


SIGNATURE THRESHOLDS
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   SIGNATURE THRESHOLD   REVOCATION THRESHOLD   
1                 0                     0   
2                 0                     0   

==> Crypto units with a signature threshold of 0 are in IMPRINT MODE. <==


CRYPTO UNIT ADMINISTRATORS
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   ADMIN NAME          SUBJECT KEY IDENTIFIER   
1                 No administrators      
2                 No administrators      


NEW MASTER KEY REGISTER
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   STATUS   VERIFICATION PATTERN   
1                 Empty    00000000000000000000000000000000   
                           00000000000000000000000000000000   
2                 Empty    00000000000000000000000000000000   
                           00000000000000000000000000000000   


CURRENT MASTER KEY REGISTER
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   STATUS   VERIFICATION PATTERN   
1                 Empty    00000000000000000000000000000000   
                           00000000000000000000000000000000   
2                 Empty    00000000000000000000000000000000   
                           00000000000000000000000000000000   


CONTROL POINTS
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   XCP_CPB_ALG_EC_25519   XCP_CPB_BTC   XCP_CPB_ECDSA_OTHER   
1                 Set                    Set           Set   
2                 Set                    Not Set       Not Set   

==> WARNING: Crypto units are configured differently. <==

## To display the existing signature keys on the workstation, use the following command:
### ibmcloud tke sigkeys

No files containing a signature key were found.

To create a file containing a signature key, use the 'ibmcloud tke sigkey-add' command.
## To create and save a new signature key on the workstation, use the following command:
### ibmcloud tke sigkey-add


Enter an administrator name to be associated with the signature key:
> john smith
Enter a password to protect the signature key:
> 
Re-enter the password to confirm:
> 
OK
A signature key was created.
The available signature keys on this workstation are:

KEYNUM   DESCRIPTION   SUBJECT KEY IDENTIFIER   
1        john smith    754adadbd731a5344c2e606b51993c...   

No KEYNUM are selected as current signature keys.


## To select the administrators to sign future commands, use the command:
### ibmcloud tke sigkey-sel

KEYNUM   DESCRIPTION   SUBJECT KEY IDENTIFIER   
1        john smith    754adadbd731a5344c2e606b51993c...   

No KEYNUM are selected as current signature keys.

Enter the KEYNUM values to select as current signature keys, separated by spaces:
> 1
Enter the password for KEYNUM 1:
> 
OK
KEYNUM 1 has been made the current signature key.
❯ ibmcloud tke cryptounit-admins


No crypto unit administrators for service instance 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
## To display the existing administrators for a crypto unit, use the following command:
### ibmcloud tke cryptounit-admins

No crypto unit administrators for service instance 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
## To add an administrator, use the following command:
### ibmcloud tke cryptounit-admin-add

KEYNUM   DESCRIPTION   SUBJECT KEY IDENTIFIER   
1        john smith    754adadbd731a5344c2e606b51993c...   

KEYNUM 1 is selected as the current signature key.

Enter the KEYNUM of the administrator signature key you wish to load:
> 1
Enter the password for the administrator signature key file:
> 
OK
The crypto unit administrator was added to the selected crypto units.
## Set quorum authentication thresholds,After you add one or more crypto unit administrators, exit imprint mode by using the command: 
### ibmcloud tke cryptounit-thrhld-set

Enter the new signature threshold value:
> 1

Enter the new revocation signature threshold value:
> 1
Enter the password for the signature key identified by:
john smith (754adadbd731a5344c2e606b51993c...)
> 
OK
New signature threshold values have been set in the selected crypto units.

SIGNATURE THRESHOLDS
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   SIGNATURE THRESHOLD   REVOCATION THRESHOLD   
1                 1                     1   
2                 1                     1   

==> Crypto units with a signature threshold of 0 are in IMPRINT MODE. <==
## To display the existing master key parts on the workstation, use the following command:
### ibmcloud tke mks

No files containing an EP11 master key part were found.
## To create and save a random master key part on the workstation, use the command:
### ibmcloud tke mk-add –random

Enter a description for the key part:
> First key
Enter a password to protect the key part:
> 
Re-enter the password to confirm:
> 
OK
A key part was created.
The available EP11 master key parts on this workstation are:

KEYNUM   DESCRIPTION   VERIFICATION PATTERN   
1        First key     34c73048e81a08b69243dae23e8943bb   
                       8874261d58bca67c4d308071ff2b6462
## To create and save a second random master key part on the workstation:
### ibmcloud tke mk-add –random

Enter a description for the key part:
> second key
Enter a password to protect the key part:
> 
Re-enter the password to confirm:
> 
OK
A key part was created.
The available EP11 master key parts on this workstation are:

KEYNUM   DESCRIPTION   VERIFICATION PATTERN   
1        First key     34c73048e81a08b69243dae23e8943bb   
                       8874261d58bca67c4d308071ff2b6462   
2        second key    ce7b7d2198de8f46195667e725b593fc   
                       cfbc10e9c211d9bf80b358c2513ac18c


## To load the new master key register, use the following command:
### ibmcloud tke cryptounit-mk-load


KEYNUM   DESCRIPTION   VERIFICATION PATTERN   
1        First key     34c73048e81a08b69243dae23e8943bb   
                       8874261d58bca67c4d308071ff2b6462   
2        second key    ce7b7d2198de8f46195667e725b593fc   
                       cfbc10e9c211d9bf80b358c2513ac18c   

Enter the KEYNUM values of the master key parts you wish to load.
2 or 3 values must be specified, separated by spaces.
> 1 2
Enter the password for the signature key identified by:
john smith (754adadbd731a5344c2e606b51993c...)
> 
Enter the password for key file 1
> 
Enter the password for key file 2
> 
OK
The new master key register has been loaded in the selected crypto units.

NEW MASTER KEY REGISTER
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   STATUS             VERIFICATION PATTERN   
1                 Full Uncommitted   7a6ba030d18a0bc1c80abd5931bff8e3   
                                     10154b88430968ee534ad01413dda886   
2                 Full Uncommitted   7a6ba030d18a0bc1c80abd5931bff8e3   
                                     10154b88430968ee534ad01413dda886  

## To commit the new master key register, use the following command:
### ibmcloud tke cryptounit-mk-commit

Enter the password for the signature key identified by:
john smith (754adadbd731a5344c2e606b51993c...)
> 
OK
The new master key register has been committed in the selected crypto units.

NEW MASTER KEY REGISTER
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   STATUS           VERIFICATION PATTERN   
1                 Full Committed   7a6ba030d18a0bc1c80abd5931bff8e3   
                                   10154b88430968ee534ad01413dda886   
2                 Full Committed   7a6ba030d18a0bc1c80abd5931bff8e3   
                                   10154b88430968ee534ad01413dda886

## Activate the master key by moving the master key to the current master key register with the following command:
### ibmcloud tke cryptounit-mk-setimm

Warning!  Any key storage associated with the targeted service instance must be prepared to accept the new master key value before running this command.  Otherwise, key storage may become unusable.
Do you want to continue?
Answer [y/N]:
> y
Enter the password for the signature key identified by:
john smith (754adadbd731a5344c2e606b51993c...)
> 
OK
Set immediate completed successfully in the selected crypto units.

NEW MASTER KEY REGISTER
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   STATUS   VERIFICATION PATTERN   
1                 Empty    00000000000000000000000000000000   
                           00000000000000000000000000000000   
2                 Empty    00000000000000000000000000000000   
                           00000000000000000000000000000000   


CURRENT MASTER KEY REGISTER
SERVICE INSTANCE: 4dd3b774-ddce-4b20-bb12-fd01d84fb92f
CRYPTO UNIT NUM   STATUS   VERIFICATION PATTERN   
1                 Valid    7a6ba030d18a0bc1c80abd5931bff8e3   
                           10154b88430968ee534ad01413dda886   
2                 Valid    7a6ba030d18a0bc1c80abd5931bff8e3   
                           10154b88430968ee534ad01413dda886










