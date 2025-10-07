{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "defaultValue": "windowsVM",
      "metadata": {
        "description": "虛擬機名稱"
      }
    },
    "resourceGroup": {
      "type": "string",
      "defaultValue": "test_A",
      "metadata": {
        "description": "資源所在資源組"
      }
    },
    "newHostName": {
      "type": "string",
      "defaultValue": "changebytm",
      "metadata": {
        "description": "設定的新主機名"
      }
    },
    "vmSize": {
      "type": "string",
      "defaultValue": "Standard_D2s_v3",
      "metadata": {
        "description": "虛擬機規格"
      }
    },
    "adminUsername": {
      "type": "string",
      "defaultValue": "auoMichel",
      "metadata": {
        "description": "管理帳號名稱"
      }
    },
    "adminPassword": {
      "type": "string",
      "defaultValue": "AuoMichael",
      "metadata": {
        "description": "管理帳號密碼"
      }
    }
  },
  "variables": {
    "nicName": "[concat(parameters('vmName'), '-nic')]",
    "vnetName": "myVnet",
    "subnetName": "default",
    "ipName": "[concat(parameters('vmName'), '-pip')]",
    "addressPrefix": "10.0.0.0/16",
    "subnetPrefix": "10.0.0.0/24"
  },
  "resources": [
    // 虛擬網路
    {
      "type": "Microsoft.Network/virtualNetworks",
      "apiVersion": "2021-02-01",
      "name": "[variables('vnetName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "addressSpace": {
          "addressPrefixes": [ "[variables('addressPrefix')]" ]
        },
        "subnets": [
          {
            "name": "[variables('subnetName')]",
            "properties": {
              "addressPrefix": "[variables('subnetPrefix')]"
            }
          }
        ]
      }
    },
    // 公網IP
    {
      "type": "Microsoft.Network/publicIPAddresses",
      "apiVersion": "2021-02-01",
      "name": "[variables('ipName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "publicIPAllocationMethod": "Dynamic"
      }
    },
    // NIC
    {
      "type": "Microsoft.Network/networkInterfaces",
      "apiVersion": "2021-02-01",
      "name": "[variables('nicName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/publicIPAddresses', variables('ipName'))]",
        "[resourceId('Microsoft.Network/virtualNetworks', variables('vnetName'))]"
      ],
      "properties": {
        "ipConfigurations": [
          {
            "name": "ipconfig1",
            "properties": {
              "subnet": {
                "id": "[resourceId('Microsoft.Network/virtualNetworks/subnets', variables('vnetName'), variables('subnetName'))]"
              },
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses', variables('ipName'))]"
              }
            }
          }
        ]
      }
    },
    // Windows虛擬機
    {
      "type": "Microsoft.Compute/virtualMachines",
      "apiVersion": "2022-08-01",
      "name": "[parameters('vmName')]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
      ],
      "properties": {
        "hardwareProfile": {
          "vmSize": "[parameters('vmSize')]"
        },
        "storageProfile": {
          "imageReference": {
            "publisher": "MicrosoftWindowsServer",
            "offer": "WindowsServer",
            "sku": "2022-Datacenter",
            "version": "latest"
          },
          "osDisk": {
            "createOption": "FromImage"
          }
        },
        "osProfile": {
          "computerName": "[parameters('newHostName')]",
          "adminUsername": "[parameters('adminUsername')]",
          "adminPassword": "[parameters('adminPassword')]"
        },
        "networkProfile": {
          "networkInterfaces": [
            {
              "id": "[resourceId('Microsoft.Network/networkInterfaces', variables('nicName'))]"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "vmPublicIp": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.Network/publicIPAddresses', variables('ipName'))).ipAddress]"
    },
    "status": {
      "type": "string",
      "value": "Windows虛擬機已建立完成。"
    }
  }
}
