# Создание виртуальной машины

```
$VMName = "RS042 (RD Session host)"
```
```
$VMPath = "C:\ClusterStorage\Volume1"
```
$VMSwitchName = "Prod"
```
$VM = @{ Name = $VMName
MemoryStartupBytes = 60Gb
Generation = 2
NewVHDPath = "$VMPath\$VMName\$VMName.vhdx"
NewVHDSizeBytes = 90Gb
BootDevice = "VHD"
Path = "$VMPath\$VMName"
SwitchName = $VMSwitchName
}
```
```
Set-VMProcessor $VMName -Count 12
```
