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
## Проверка получения данных агентом  
В файл `C:\Program Files\Zabbix Agent\zabbix_agentd.d\zabbix_agentd_rds.conf` добавить строку (разрешаем локальное подключение)  
```
Server=10.77.1.41,127.0.0.1
```
Перезапустить агента zabbix:  
```
Get-Service "Zabbix Agent"| Restart-Service -force
```
Проверить получение данных агентом:  
```
zabbix_get -s 127.0.0.1 -p 10050 -k ActiveRDSSessions
```
Вывод:  
```
C:\Windows\system32>zabbix_get -s 127.0.0.1 -p 10050 -k ActiveRDSSessions
1
```

## Подключить шаблон в zabbix  
К узлу сети нужно подключить шаблон `RDS`  
