# Создание виртуальной машины

```
$VMName = "RS042 (RD Session host)"
```
```
$VMPath = "C:\ClusterStorage\Volume1"
```
```
$VMSwitchName = "Prod"
```
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
Подключить ISO файл в виртуальное CD/DVD устройство:

```
Set-VMDvdDrive -VMName $VMName -Path C:\ClusterStorage\Volume1\distr\ru-ru_windows_server_2022_updated_nov_2022_x64_dvd_facdb267.iso
```
Разрешить автозапуск для виртуальной машину Hyper-V: 
```
Get-VM –VMname $VMName | Set-VM –AutomaticStartAction Start
```
