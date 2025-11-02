<h1>Device Inventory Tool</h1>
<p align="center">
<img src="https://www.shutterstock.com/image-vector/inventory-control-isometric-3d-flat-600nw-2125949336.jpg" alt="Computer Inventory Stock: Over 632 Royalty-Free Licensable Stock  Illustrations & Drawings | Shutterstock"/>
</p>  

This repository provides a great tool that is used to gather vital system information form Windows devices and outputs it into a text or CSV file. This is useful for asset management and tracking hardware/software configurations.  

***What the Script Features***  
Your PowerShell Script will gather and display:  
- **System Details** (Device name, OS version, architecture, uptime)  
- **Hardware Info** (CPU, RAM, disk space, GPU)  
- **Network Information** (IP, MAC address, adapter name)  
- Option to **export results** to `.csv` or `.txt`  
- **Progress Bar** while gathering data  

***How to Run It***  

1.) Save it as:   
`DeviceInventory.ps1`   
2.) Run **PowerShell** as an **Administrator**   
3.) Navigate to the folder and execute the file by right-clicking and selecting **"Run in PowerShell"**:  
```bash
.\DeviceInventory.ps1
```
4.) * Alternative: Just copy and paste the script in **PowerShell.** *   
5.) Choose your export format: `csv` or `txt`.   
6.) View the progress and system stats before/after the tool is used.

🎒<ins>***Device Inventory Tool Script***<ins>🎒

```bash
# Device Inventory Tool
Write-Host "Collecting system information..." -ForegroundColor Cyan
$progress = 0
$totalSteps = 8

function Update-ProgressBar {
    param([int]$step)
    $percent = [math]::Round(($step / $totalSteps) * 100)
    Write-Progress -Activity "Scanning System..." -Status "$percent% Complete" -PercentComplete $percent
}

Update-ProgressBar -step (++$progress)
$sysInfo = Get-ComputerInfo | Select-Object CsName, OsName, OsArchitecture, WindowsVersion, CsModel, CsManufacturer

Update-ProgressBar -step (++$progress)
$cpu = (Get-WmiObject Win32_Processor).Name

Update-ProgressBar -step (++$progress)
$ram = "{0:N0} MB" -f ((Get-WmiObject Win32_ComputerSystem).TotalPhysicalMemory / 1MB)

Update-ProgressBar -step (++$progress)
$gpu = (Get-WmiObject Win32_VideoController).Name

Update-ProgressBar -step (++$progress)
$disk = Get-PSDrive -PSProvider 'FileSystem' | Where-Object {$_.Name -eq 'C'}
$diskUsage = "{0:N0} GB free of {1:N0} GB" -f ($disk.Free/1GB), ($disk.Used+$disk.Free)/1GB

Update-ProgressBar -step (++$progress)
$net = Get-NetIPConfiguration | Where-Object {$_.IPv4Address -ne $null}
$ip = $net.IPv4Address.IPAddress
$mac = $net.NetAdapter.MacAddress

Update-ProgressBar -step (++$progress)
$uptime = (Get-CimInstance Win32_OperatingSystem).LastBootUpTime
$uptimeFormatted = (Get-Date) - $uptime
$up = "{0} days, {1} hours, {2} minutes" -f $uptimeFormatted.Days, $uptimeFormatted.Hours, $uptimeFormatted.Minutes

$report = [PSCustomObject]@{
    "Device Name"     = $sysInfo.CsName
    "OS Version"      = $sysInfo.OsName
    "Architecture"    = $sysInfo.OsArchitecture
    "CPU"             = $cpu
    "RAM"             = $ram
    "GPU"             = $gpu
    "Storage"         = $diskUsage
    "IP Address"      = $ip
    "MAC Address"     = $mac
    "Uptime"          = $up
}

Write-Progress -Activity "Scanning System..." -Completed
Write-Host "`nSystem Inventory Completed!" -ForegroundColor Green

# Export Options
$folder = "C:\DeviceReports"
if (!(Test-Path $folder)) { New-Item -Path $folder -ItemType Directory | Out-Null }
$date = Get-Date -Format "yyyy-MM-dd_HHmm"
$exportFile = "$folder\Device_Inventory_$date.csv"
$report | Export-Csv -Path $exportFile -NoTypeInformation
Write-Host "Report saved to: $exportFile" -ForegroundColor Yellow
```
