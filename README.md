{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "michaeltest",
      "metadata": {
        "description": "要修改hostname的虚拟机名称"
      }
    },
    "resourceGroup": {
      "type": "string",
      "defaultValue": "test_A",
      "metadata": {
        "description": "虚拟机所在的资源组"
      }
    },
    "newHostName": {
      "type": "string",
      "defaultValue": "changesuccess",
      "metadata": {
        "description": "要设置的新主机名"
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2022-08-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "osProfile": {
          "computerName": "[parameters('newHostName')]"
        }
      }
    }
  ],
  "outputs": {
    "status": {
      "type": "string",
      "value": "虚拟机 hostname 已更新。请确保虚拟机在重启后生效。"
    }
  }
}
