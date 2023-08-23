# Создание виртуальной машины

https://winitpro.ru/index.php/2021/11/10/upravlenie-vm-hyper-v-powershell/

```
$VMName = "RS042 (RD Session host)"
```
```
$VMPath = "C:\ClusterStorage\Volume1"
```
```
$VMSwitchName = "Prod"
```
Создать диск
```
New-VHD -Path $VMPath\$VMName.vhdx -Fixed -SizeBytes 90Gb
```
Создать ВМ
```
$VM = @{ Name = $VMName
MemoryStartupBytes = 60Gb
Generation = 2
VHDPath = "$VMPath\$VMName\$VMName.vhdx"
BootDevice = "VHD"
Path = "$VMPath\$VMName"
SwitchName = $VMSwitchName
}
```
Создать ВМ
```
New-VM @VM
```
```
Set-VMProcessor $VMName -Count 12
```
Подключить ISO файл в виртуальное CD/DVD устройство:

```
Add-VMDvdDrive -VMName $VMName -Path C:\ClusterStorage\Volume1\distr\ru-ru_windows_server_2022_updated_nov_2022_x64_dvd_facdb267.iso
```
Разрешить автозапуск для виртуальной машину Hyper-V: 
```
Get-VM –VMname $VMName | Set-VM –AutomaticStartAction Start
```
## Управление ВМ
https://winitpro.ru/index.php/2021/11/10/upravlenie-vm-hyper-v-powershell/
Получить IP адреса гостевых ОС виртуальных машин:
```
Get-VM | Select -ExpandProperty NetworkAdapters | Select VMName, IPAddresses, Status
```
Подключиться к консоли определенной виртуальной машины:
```
vmconnect.exe localhost spb-app01
```
Для подключения PowerShell сессией напрямую к гостевым ОС виртуальных машин через шину vmbus можно использовать PowerShell Direct (доступен для гостевых ОС Windows Server 2016, Windows 10 и новее). Можно использовать командлеты Invoke-Command (для запуска скриптов) и Enter-PSSession (для входа в интерактивную PowerShell сессию):

```
Invoke-Command -VMName $VMName -ScriptBlock {Get-Process}
```

See Adapter Status (and Names to Use in Other Cmdlets)

```
Get-NetAdapter
```
Set an Adapter’s IP Address
```
Invoke-Command -VMName $VMName -ScriptBlock {New-NetIPAddress -InterfaceAlias Ethernet -IPAddress 10.77.3.47 -PrefixLength 24 -DefaultGateway 10.77.3.254}
```
Set DNS Server Addresses

```
Invoke-Command -VMName $VMName -ScriptBlock {Set-DNSClientServerAddresses -InterfaceAlias Ethernet -ServerAddresses 10.77.3.44, 10.77.1.44}
```
Вы можете включить RDP доступ в Windows с помощью пары PowerShell команд. Это гораздо быстрее:

Запустите консоль PowerShell.exe с правами администратора;
Включите RDP доступ в реестре с помощью командлета 
```
Invoke-Command -VMName $VMName -ScriptBlock {(Get-WmiObject Win32_TerminalServiceSetting -Namespace root\cimv2\TerminalServices).SetAllowTsConnections(1,1)}
```

Разрешите RDP подключения к компьютеру в Windows Defender Firewall. Для этого включите предустановленное правило (вроди и так уже включен предыдущей командой) : 
```
Enable-NetFirewallRule -DisplayGroup "Remote Desktop"
```
включить rdp в windows с помощью powershell

Если нужно добавить пользователя в группу в локальную группу RDP доступа, выполните: 
```
Add-LocalGroupMember -Group "Remote Desktop Users" -Member 'a.petrov'
```
Чтобы проверить, что на компьютере открыт RDP порт, воспользуйтесь командлетом Test-NetConnection:

Test-NetConnection -ComputerName deskcomp323 -CommonTCPPort rdp

Добавить локального пользователя
```
New-LocalUser -Name "rs042_admin"
```
Чтобы установить флаг «Срок действия пароля пользователя не истекает» («Password never expired»), выполните:
```
Set-LocalUser -Name rs042_admin –PasswordNeverExpires $True
```
Добавить пользователя в группу
```
Add-LocalGroupMember -Group 'Администраторы' -Member ('rs042_admin') -Verbose
```
Чтобы изменить пароль существующего пользователя, выполните команду:
```
$Password = Read-Host -AsSecureString
Set-LocalUser -Name Администратор -Password $UserPassword –Verbose
```

Отключить учетную запись
```
Disable-LocalUser -Name Администратор
```
