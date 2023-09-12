# Экспорт импорт пользователей AD  
## Выгрузка пользователей  
В корне диск C: создать папку `Scripts` и создать в ней файл `Export-ADUsers.ps1`  
```
<#
    .SYNOPSIS
    Export-ADUsers.ps1

    .DESCRIPTION
    Export Active Directory users to CSV file.

    .LINK
    alitajran.com/export-ad-users-to-csv-powershell

    .NOTES
    Written by: ALI TAJRAN
    Website:    alitajran.com
    LinkedIn:   linkedin.com/in/alitajran

    .CHANGELOG
    V1.00, 05/24/2021 - Initial version
    V1.10, 04/01/2023 - Added progress bar, user created date, and OU info
    V1.20, 05/19/2023 - Added function for OU path extraction
#>

# Split path
$Path = Split-Path -Parent "C:\scripts\*.*"

# Create variable for the date stamp in log file
$LogDate = Get-Date -f yyyyMMddhhmm

# Define CSV and log file location variables
# They have to be on the same location as the script
$Csvfile = $Path + "\AllADUsers_$LogDate.csv"

# Import Active Directory module
Import-Module ActiveDirectory

# Function to extract OU from DistinguishedName
function Get-OUFromDistinguishedName {
    param(
        [string]$DistinguishedName
    )

    $ouf = ($DistinguishedName -split ',', 2)[1]
    if (-not ($ouf.StartsWith('OU') -or $ouf.StartsWith('CN'))) {
        $ou = ($ouf -split ',', 2)[1]
    }
    else {
        $ou = $ouf
    }
    return $ou
}

# Set distinguishedName as searchbase, you can use one OU or multiple OUs
# Or use the root domain like DC=exoip,DC=local
$DNs = @(
    "OU=Нужный ОУ,DC=rost,DC=local"#,
#    "OU=IT,OU=Users,OU=Company,DC=exoip,DC=local",
#    "OU=Finance,OU=Users,OU=Company,DC=exoip,DC=local"
)

# Create empty array
$AllADUsers = @()

# Loop through every DN
foreach ($DN in $DNs) {
    $Users = Get-ADUser -SearchBase $DN -Filter * -Properties *

    # Add users to array
    $AllADUsers += $Users

    # Display progress bar
    $progressCount = 0
    for ($i = 0; $i -lt $AllADUsers.Count; $i++) {

        Write-Progress `
            -Id 0 `
            -Activity "Retrieving User " `
            -Status "$progressCount of $($AllADUsers.Count)" `
            -PercentComplete (($progressCount / $AllADUsers.Count) * 100)

        $progressCount++
    }
}

# Create list
$AllADUsers | Sort-Object Name | Select-Object `
@{Label = "First name"; Expression = { $_.GivenName } },
@{Label = "Last name"; Expression = { $_.Surname } },
@{Label = "Display name"; Expression = { $_.DisplayName } },
@{Label = "User logon name"; Expression = { $_.SamAccountName } },
@{Label = "User principal name"; Expression = { $_.UserPrincipalName } },
@{Label = "Street"; Expression = { $_.StreetAddress } },
@{Label = "City"; Expression = { $_.City } },
@{Label = "State/province"; Expression = { $_.State } },
@{Label = "Zip/Postal Code"; Expression = { $_.PostalCode } },
@{Label = "Country/region"; Expression = { $_.Country } },
@{Label = "Job Title"; Expression = { $_.Title } },
@{Label = "Department"; Expression = { $_.Department } },
@{Label = "Company"; Expression = { $_.Company } },
@{Label = "Manager"; Expression = { (Get-AdUser $_.Manager -Properties DisplayName).DisplayName } },
@{Label = "OU"; Expression = { Get-OUFromDistinguishedName $_.DistinguishedName } },
@{Label = "Description"; Expression = { $_.Description } },
@{Label = "Office"; Expression = { $_.Office } },
@{Label = "Telephone number"; Expression = { $_.telephoneNumber } },
@{Label = "Other Telephone"; Expression = { $_.otherTelephone -join ";"} },
@{Label = "E-mail"; Expression = { $_.Mail } },
@{Label = "Mobile"; Expression = { $_.mobile } },
@{Label = "Pager"; Expression = { $_.pager } },
@{Label = "Notes"; Expression = { $_.info } },
@{Label = "Account status"; Expression = { if (($_.Enabled -eq 'TRUE') ) { 'Enabled' } Else { 'Disabled' } } },
@{Label = "User created date"; Expression = { $_.WhenCreated } },
@{Label = "Last logon date"; Expression = { $_.lastlogondate } } |

# Export report to CSV file
Export-Csv -Encoding UTF8 -Path $Csvfile -NoTypeInformation #-Delimiter ";"
```
поправить на нужные OU  
```
$DNs = @(
    "OU=Нужный OU,DC=rost,DC=local"#,
#    "OU=IT,OU=Users,OU=Company,DC=exoip,DC=local",
#    "OU=Finance,OU=Users,OU=Company,DC=exoip,DC=local"
)
```
В полученном файле поправить  
`,OU=Нужный OU,DC=rost,DC=local`  
на  
`,OU=Нужный OU,DC=rostgroup,DC=ru`  
https://www.alitajran.com/export-ad-users-to-csv-powershell/  

## Подготовка структуры организации
Создаем нужные OU  
```
New-ADOrganizationalUnit -Name "Нужный OU" -Path “OU=ОУ2,OU=ОУ1,DC=corp,DC=rostgroup,DC=ru” -PassThru
```

https://winitpro.ru/index.php/2021/11/15/sozdat-strukturu-ou-v-active-directory-powershell/
## Загрузка пользователей  
В корне диск C: создать папку `Scripts` и создать в ней файл `Import-ADUsers.ps1`  
```
<#
    .SYNOPSIS
    Import-ADUsers.ps1

    .DESCRIPTION
    Import Active Directory users from CSV file.

    .LINK
    alitajran.com/import-ad-users-from-csv-powershell

    .NOTES
    Written by: ALI TAJRAN
    Website:    alitajran.com
    LinkedIn:   linkedin.com/in/alitajran

    .CHANGELOG
    V1.00, 04/24/2023 - Initial version
#>

# Define the CSV file location and import the data
$Csvfile = "C:\scripts\AllADUsers_202309121002.csv"
$Users = Import-Csv $Csvfile

# Import the Active Directory module
Import-Module ActiveDirectory

# Loop through each user
foreach ($User in $Users) {
    $GivenName = $User.'First name'
    $Surname = $User.'Last name'
    $DisplayName = $User.'Display name'
    $SamAccountName = $User.'User logon name'
    $UserPrincipalName = $User.'User principal name'
    $StreetAddress = $User.'Street'
    $City = $User.'City'
    $State = $User.'State/province'
    $PostalCode = $User.'Zip/Postal Code'
    $Country = $User.'Country/region'
    $JobTitle = $User.'Job Title'
    $Department = $User.'Department'
    $Company = $User.'Company'
    $ManagerDisplayName = $User.'Manager'
    $Manager = if ($ManagerDisplayName) {
        Get-ADUser -Filter "DisplayName -eq '$ManagerDisplayName'" -Properties DisplayName |
        Select-Object -ExpandProperty DistinguishedName
    }
    $OU = $User.'OU'
    $Description = $User.'Description'
    $Office = $User.'Office'
    $TelephoneNumber = $User.'Telephone number'
    $Email = $User.'E-mail'
    $Mobile = $User.'Mobile'
    $Notes = $User.'Notes'
    $AccountStatus = $User.'Account status'

    # Check if the user already exists in AD
    $UserExists = Get-ADUser -Filter { SamAccountName -eq $SamAccountName } -ErrorAction SilentlyContinue

    if ($UserExists) {
        Write-Warning "User '$SamAccountName' already exists in Active Directory."
        continue
    }

    # Create new user parameters
    $NewUserParams = @{
        Name                  = "$GivenName $Surname"
        GivenName             = $GivenName
        Surname               = $Surname
        DisplayName           = $DisplayName
        SamAccountName        = $SamAccountName
        UserPrincipalName     = $UserPrincipalName
        StreetAddress         = $StreetAddress
        City                  = $City
        State                 = $State
        PostalCode            = $PostalCode
        Country               = $Country
        Title                 = $JobTitle
        Department            = $Department
        Company               = $Company
        Manager               = $Manager
        Path                  = $OU
        Description           = $Description
        Office                = $Office
        OfficePhone           = $TelephoneNumber
        EmailAddress          = $Email
        MobilePhone           = $Mobile
        AccountPassword       = (ConvertTo-SecureString "Password" -AsPlainText -Force) # Заменить на подходящий пароль
        Enabled               = if ($AccountStatus -eq "Enabled") { $true } else { $false }
        ChangePasswordAtLogon = $true # Set the "User must change password at next logon" flag
    }

    # Add the info attribute to OtherAttributes only if Notes field contains a value
    if (![string]::IsNullOrEmpty($Notes)) {
        $NewUserParams.OtherAttributes = @{info = $Notes }
    }

    try {
        # Create the new AD user
        New-ADUser @NewUserParams
        Write-Host "User $SamAccountName created successfully." -ForegroundColor Cyan
    }
    catch {
        # Failed to create the new AD user
        Write-Warning "Failed to create user $SamAccountName. $_"
    }
}
```
https://www.alitajran.com/import-ad-users-from-csv-powershell/
