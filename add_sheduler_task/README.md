# Создать задание в планировщике с запуском от сервисной учетной записи
## Создать управляему учетную запись MSA в Active Directory 
Создать учетку 
```
New-ADServiceAccount -Name msaRS037Tasks –RestrictToSingleComputer
```
Привязать учетку к компьютеру
```
$Identity = Get-ADComputer -identity RS037
Add-ADComputerServiceAccount -Identity $identity -ServiceAccount msaRS037Tasks
```
Проверить учетку можно
```
Get-ADServiceAccount msaRS037Tasks
```
## Привязать сервисную учетную запись к компьютеру
при необходимсости выпольнить
```
Add-WindowsFeature RSAT-AD-PowerShell
```
Установить учетную запись MSA/gMSA на сервере: 
```
Install-ADServiceAccount -Identity msaRS037Tasks
```
Проверить, сервисная учетная запись установлена корректно: 
```
Test-ADServiceAccount msaRS037Tasks
```
Если команда вернет True – все настроено правильно. 

## Назанчить права сервисной учетной записи на "Вход в качестве пакетного задния"
Для того чтобы управлять привилегиями и правами, нужно использовать ММС-оснастку «Локальная политика безопасности» (secpol.msc). 
Локальная политикиа - Назаначение прав пользователям - Вход в качестве пакетного задания 

## Создать задание планировщика
Вывести список сервисов
```
Get-Service|where -Property Name -Like "1*"|ft -Property Name
```
Зарегистрировать задание
```
$trigger = New-ScheduledTaskTrigger -At 03:55 -Daily
$principal = New-ScheduledTaskPrincipal -UserID rostgroup\msaRS037Tasks$ -LogonType Password
$serviceName = "1C Server 1541"
$action = New-ScheduledTaskAction -Execute powershell.exe  -Argument "-NoProfile -Command ""&{Stop-Service '$serviceName';Start-Sleep -Seconds 60;Start-Service '$serviceName'}"""
Register-ScheduledTask "1C Server 1541" -Action $action -Trigger $trigger -Principal $principal
```
Изменить задание
```
Set-ScheduledTask "1C agent ERPWORK" -Action $action -Trigger $trigger -Principal $principal
```

rttps://winitpro.ru/index.php/2014/03/28/group-managed-service-accounts-v-windows-server-2012/
