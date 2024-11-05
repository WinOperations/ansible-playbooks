# Paso 0: Solicita ejecución con privilegios elevados
if (-not ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrador")) {
    Write-Output "Este script requiere privilegios elevados. Ejecuta como administrador."
    exit
}

# Paso 1: Inicia la máquina virtual
$vmName = "07 - LAB - BADAJOZ"
Start-VM -Name $vmName -ErrorAction Stop
Write-Output "Máquina virtual '$vmName' iniciada."

# Paso 2: Realiza un shrink de la base de datos
$connectionString = "Server=localhost;Database=wgdb_000;Integrated Security=True;"
Invoke-Sqlcmd -Query "DBCC SHRINKDATABASE (wgdb_000, 0);" -ConnectionString $connectionString
Write-Output "Shrink de la base de datos 'wgdb_000' completado."

# Paso 3: Espera a que concluya el shrink (ya se realiza al completar el comando anterior)

# Paso 4: Realiza la actualización de Windows
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
Set-ExecutionPolicy RemoteSigned -Scope Process -Force
Set-ExecutionPolicy Unrestricted -Scope Process -Force

# Instala el módulo de PSWindowsUpdate si no está ya instalado
if (-not (Get-Module -ListAvailable -Name PSWindowsUpdate)) {
    Install-Module PSWindowsUpdate -Force
}
Import-Module PSWindowsUpdate
Add-WUServiceManager -MicrosoftUpdate
Install-WindowsUpdate -MicrosoftUpdate -AcceptAll -AutoReboot:$false
Write-Output "Actualización de Windows completada."

# Paso 5: Asegúrate de que no se reinicie automáticamente
Write-Output "Reboot is required. Do it Now? N"

# Paso 6: Limpieza del sistema
Write-Output "Iniciando limpieza del sistema..."

# Limpia archivos de la carpeta Temp del sistema y de los usuarios
Remove-Item -Path "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item -Path "$env:SystemRoot\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue

# Limpia la papelera de reciclaje
$Shell = New-Object -ComObject Shell.Application
$Shell.NameSpace(0x0a).Items() | ForEach-Object { Remove-Item $_.Path -Recurse -Force -ErrorAction SilentlyContinue }

# Limpia archivos de actualización de Windows que ya no son necesarios
Start-Process -FilePath "cleanmgr.exe" -ArgumentList "/verylowdisk" -NoNewWindow -Wait

# Limpia archivos de registro de eventos
wevtutil el | ForEach-Object { wevtutil cl $_ }

# Limpia archivos de prefetch
Remove-Item -Path "$env:SystemRoot\Prefetch\*" -Force -ErrorAction SilentlyContinue

Write-Output "Limpieza del sistema completada."

# Paso 8: Apaga la máquina
Stop-VM -Name $vmName -Force
Write-Output "Máquina virtual '$vmName' apagada."
