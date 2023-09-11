# Сбор данных о сессиях RDS в zabbix 
## Скрипты 
Создать папку `C:\Program Files\Zabbix Agent\Scripts`
Создать скрипт `GetActiveRDPSessionCount.ps1`
```
$RDSsessions= qwinsta |ForEach-Object {$_ -replace "\s{2,25}",","} | ConvertFrom-Csv
$RDSActiveSessions=@($RDSsessions| where -Property СТАТУС -Like 'Актив*').count
Write-Host $RDSActiveSessions
```
## Конфиг zabbix
В папке `C:\Program Files\Zabbix Agent\zabbix_agentd.d`
Созать файл `zabbix_agentd_rds.conf`:
```
UserParameter=ActiveRDSSessions,powershell -NoProfile -ExecutionPolicy bypass -File "C:\Program Files\Zabbix Agent\Scripts\GetActiveRDPSessionCount.ps1"
```
Перезапустить агента zabbix:
```
Get-Service "Zabbix Agent"| Restart-Service -force
```
Проверить статус сервиса
```
Get-Service "Zabbix Agent"
```
вывод
```
Status   Name               DisplayName
------   ----               -----------
Running  Zabbix Agent       Zabbix Agent
```
## Подключить шаблон в zabbix
К узлу сети нужно подключить шаблон `RDS`
