Configuration WindowsWebServer
{
    Import-DscResource -ModuleName PSDesiredStateConfiguration

    Node localhost
    {
        # IIS Features from script1 and script2 (merged, deduplicated, sorted)
        $features = @(
            "Web-AppInit",
            "Web-App-Dev",
            "Web-ASP",
            "Web-Asp-Net45",
            "Web-Basic-Auth",
            "Web-Cert-Auth",
            "Web-CertProvider",
            "Web-Client-Auth",
            "Web-Common-Http",
            "Web-CGI",
            "Web-Custom-Logging",
            "Web-DAV-Publishing",
            "Web-Default-Doc",
            "Web-Dir-Browsing",
            "Web-Digest-Auth",
            "Web-Filtering",
            "Web-Health",
            "Web-Http-Errors",
            "Web-Http-Logging",
            "Web-Http-Redirect",
            "Web-Http-Tracing",
            "Web-Includes",
            "Web-IP-Security",
            "Web-ISAPI-Ext",
            "Web-ISAPI-Filter",
            "Web-Log-Libraries",
            "Web-Mgmt-Console",
            "Web-Mgmt-Service",
            "Web-Mgmt-Tools",
            "Web-Net-Ext45",
            "Web-ODBC-Logging",
            "Web-Performance",
            "Web-Request-Monitor",
            "Web-Stat-Compression",
            "Web-Scripting-Tools",
            "Web-Security",
            "Web-Server",
            "Web-Static-Content",
            "Web-Url-Auth",
            "Web-WebServer",
            "Web-WebSockets",
            "Web-Windows-Auth"
        )

        foreach ($feature in $features)
        {
            WindowsFeature $feature
            {
                Name = $feature
                Ensure = "Present"
            }
        }

        # Install URL Rewrite module
        Script InstallRewrite
        {
            GetScript = { @{ Result = "RewriteInstall" } }
            TestScript = { Test-Path "\\10.225.10.7\software\rewrite_amd64_en-US.msi" }
            SetScript = {
                Start-Process msiexec.exe -ArgumentList '/i "\\10.225.10.7\software\rewrite_amd64_en-US.msi" /quiet /norestart' -Wait
            }
            DependsOn = "[WindowsFeature]Web-Server"
        }

        # Windows Activation
        Script ActivateWindows
        {
            GetScript = { @{ Result = "WindowsActivation" } }
            TestScript = { $false }
            SetScript = {
                slmgr /ipk NTBV8-9K7Q8-V27C6-M2BTV-KHMXV
                slmgr.vbs /dlv
                slmgr.vbs /ato
            }
            DependsOn = "[WindowsFeature]Web-Server"
        }
    }
}

WindowsWebServer
# 啟用網路探索
Set-NetFirewallRule -DisplayGroup "Network Discovery" -Enabled True
# 啟用檔案與印表機共用
Set-NetFirewallRule -DisplayGroup "File and Printer Sharing" -Enabled True


# 登入帳密
$username = "azadmin"
$password = ConvertTo-SecureString "AzureDBA@2025" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ($username, $password)

# MSI 檔案來源與目的地
$sourceMsi = "\\10.225.10.7\software\rewrite_amd64_en-US.msi"
$destMsi = "C:\Temp\rewrite.msi"
