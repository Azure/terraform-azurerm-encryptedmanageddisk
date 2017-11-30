# terraform-azurerm-encryptedmanageddisk #
[![Build Status](https://travis-ci.org/Azure/terraform-azurerm-manageddisk.svg?branch=master)](https://travis-ci.org/Azure/terraform-azurerm-manageddisk)


Deploys 1 Encrypted Managed Disk
===================================

This Terraform module deploys an encrypted managed disk ([overview here](https://azure.microsoft.com/en-us/services/managed-disks/)) in Azure according to one of four available scenarios. Which scenario applies depends on the parameters that you provide. Details are [here](https://github.com/Azure/terraform-azurerm-manageddisk). The only differences are:  
1. you must provide 3 additional parameters regarding your key vault setup
2. specify "Azure/encryptedmanageddisk/azurerm" as the source for the module  

## Example of the scripting for encrypted managed disk:

Empty Disk
----------

```hcl
resource "azurerm_resource_group" "diskRg" {
  name     = "managedDiskRg"
  location = "west us"
}

data "azurerm_client_config" "current" {}

resource "azurerm_key_vault" "test" {
  name                = "acctestkv"
  location            = "${azurerm_resource_group.diskRg.location}"
  resource_group_name = "${azurerm_resource_group.diskRg.name}"
  tenant_id           = "${data.azurerm_client_config.current.tenant_id}"

  sku {
    name = "premium"
  }

  access_policy {
    tenant_id = "${data.azurerm_client_config.current.tenant_id}"
    object_id = "${data.azurerm_client_config.current.service_principal_object_id}"

    key_permissions = [
      "create",
      "delete",
      "get",
    ]

    secret_permissions = [
      "delete",
      "get",
      "set",
    ]
  }

  enabled_for_disk_encryption = true

  tags {
    environment = "Production"
  }
}

resource "azurerm_key_vault_secret" "test" {
  name      = "secret"
  value     = "szechuan"
  vault_uri = "${azurerm_key_vault.test.vault_uri}"
}

resource "azurerm_key_vault_key" "test" {
  name      = "key"
  vault_uri = "${azurerm_key_vault.test.vault_uri}"
  key_type  = "EC"
  key_size  = 2048

  key_opts = [
    "sign",
    "verify",
  ]
}

module "emptyDisk" {
    source               = "Azure/encryptedmanageddisk/azurerm"
    resource_group_name  = "${azurerm_resource_group.diskRg.name}"
    disk_size_gb         = 1

    keyVaultID           = "${azurerm_key_vault.test.id}"
    secretURL            = "${azurerm_key_vault_secret.test.id}"
    keyURL               = "${azurerm_key_vault_key.test.id}"
}

output "empty_disk_id" {
  description = "The id of the newly created managed disk"  
  value = "${module.emptyDisk.managed_disk_id}"
}
```


Authors
=======
Originally created by [Greg Oliver](http://github.com/sebastus)

License
=======

[MIT](LICENSE)

# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., label, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
