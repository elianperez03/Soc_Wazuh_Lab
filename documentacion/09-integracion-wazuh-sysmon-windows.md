# Integración de Wazuh y Sysmon en endpoints Windows

## Estado

> Completado

## Objetivo

Integrar los equipos Windows del laboratorio con el servidor Wazuh e instalar Sysmon para ampliar la visibilidad de seguridad sobre procesos, comandos, archivos y otras actividades ejecutadas en los endpoints.

Los equipos configurados fueron:

| Equipo | Sistema operativo | Dirección IP | Función |
|---|---|---:|---|
| `SRV-DC01` | Windows Server 2022 | `192.168.126.20` | Controlador de dominio |
| `WIN10-SOC` | Windows 10 Pro | `192.168.126.30` | Equipo cliente |
| Wazuh Server | Ubuntu Server | `192.168.126.10` | Manager, Indexer y Dashboard |

---

# 1. Estado inicial de Wazuh

Inicialmente, el panel principal de Wazuh indicaba que no existían agentes registrados.

Para comenzar a monitorear los endpoints fue necesario instalar el agente de Wazuh en Windows 10 y Windows Server 2022.

![Panel inicial de Wazuh sin agentes](../imagenes/134-panel-inicial-wazuh-sin-agentes.png)

---

# 2. Configuración del agente para Windows 10

Desde el panel de Wazuh se accedió a:

```text
Endpoints
→ Deploy new agent
```

Se seleccionó el paquete para Windows y se configuraron los siguientes datos:

```text
Sistema: Windows
Paquete: MSI 32/64 bits
Dirección del servidor: 192.168.126.10
Nombre del agente: WIN10-SOC
Grupo: default
```

La dirección `192.168.126.10` corresponde al servidor Wazuh del laboratorio.

![Configuración del agente Wazuh para Windows 10](../imagenes/135-configuracion-agente-wazuh-windows-10.png)

---

# 3. Comando de instalación del agente en Windows 10

Wazuh generó automáticamente el comando necesario para descargar e instalar el agente.

El comando fue ejecutado desde PowerShell con privilegios de administrador:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='192.168.126.10' WAZUH_AGENT_NAME='WIN10-SOC'
```

![Comando de instalación del agente Wazuh en Windows 10](../imagenes/136-comando-instalacion-agente-wazuh-windows-10.png)

---

# 4. Descarga e instalación del agente en Windows 10

PowerShell descargó el instalador desde los servidores oficiales de Wazuh y realizó la instalación silenciosa del agente.

Durante este proceso se asignaron automáticamente:

```text
Wazuh Manager: 192.168.126.10
Nombre del agente: WIN10-SOC
```

![Descarga e instalación del agente Wazuh en Windows 10](../imagenes/137-descarga-instalacion-agente-wazuh-windows-10.png)

---

# 5. Inicio del servicio Wazuh en Windows 10

Después de completar la instalación se inició el servicio del agente con:

```powershell
NET START Wazuh
```

PowerShell confirmó:

```text
The Wazuh service was started successfully.
```

![Inicio del servicio Wazuh en Windows 10](../imagenes/138-inicio-servicio-wazuh-windows-10.png)

---

# 6. Verificación del agente WIN10-SOC

Al regresar a la sección de endpoints del Dashboard, el equipo apareció registrado correctamente.

La información mostrada fue:

```text
ID: 001
Nombre: WIN10-SOC
Dirección IP: 192.168.126.30
Sistema operativo: Microsoft Windows 10 Pro
Versión del agente: 4.14.6
Estado: Active
```

![Agente WIN10-SOC activo en Wazuh](../imagenes/139-agente-win10-soc-activo-en-wazuh.png)

---

# 7. Configuración del agente para Windows Server

El mismo procedimiento fue realizado para el controlador de dominio.

Desde `Deploy new agent` se configuraron los siguientes valores:

```text
Sistema: Windows
Paquete: MSI 32/64 bits
Dirección del servidor: 192.168.126.10
Nombre del agente: SRV-DC01
Grupo: default
```

![Configuración del agente Wazuh para Windows Server](../imagenes/140-configuracion-agente-wazuh-windows-server.png)

---

# 8. Comando de instalación para Windows Server

Wazuh generó un nuevo comando utilizando el nombre exclusivo del servidor:

```powershell
Invoke-WebRequest -Uri https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.6-1.msi -OutFile $env:tmp\wazuh-agent; msiexec.exe /i $env:tmp\wazuh-agent /q WAZUH_MANAGER='192.168.126.10' WAZUH_AGENT_NAME='SRV-DC01'
```

El comando debía ejecutarse desde PowerShell como administrador.

![Comando de instalación del agente Wazuh en Windows Server](../imagenes/141-comando-instalacion-agente-wazuh-windows-server.png)

---

# 9. Descarga e instalación del agente en Windows Server

El instalador fue descargado y ejecutado en `SRV-DC01`.

El agente quedó configurado para comunicarse con el Wazuh Manager mediante la dirección:

```text
192.168.126.10
```

![Descarga e instalación del agente Wazuh en Windows Server](../imagenes/142-descarga-instalacion-agente-wazuh-windows-server.png)

---

# 10. Inicio del servicio Wazuh en Windows Server

Después de la instalación se inició el servicio:

```powershell
NET START Wazuh
```

El sistema confirmó que el servicio fue iniciado correctamente.

![Inicio del servicio Wazuh en Windows Server](../imagenes/143-inicio-servicio-wazuh-windows-server.png)

---

# 11. Verificación de los agentes registrados

En el Dashboard aparecieron los dos endpoints registrados:

```text
WIN10-SOC
IP: 192.168.126.30

SRV-DC01
IP: 192.168.126.20
```

En la captura, `SRV-DC01` aparece activo y `WIN10-SOC` desconectado porque la máquina virtual de Windows 10 no se encontraba comunicándose con el servidor en ese momento.

El estado de un agente puede cambiar entre `active` y `disconnected` cuando una máquina virtual se apaga, se suspende o pierde temporalmente la conectividad.

![Agentes Windows registrados en Wazuh](../imagenes/144-agentes-wazuh-windows-registrados.png)

---

# 12. Creación del directorio para Sysmon en Windows Server

Para mejorar la recopilación de eventos se instaló Sysmon en `SRV-DC01`.

Primero se creó el directorio:

```powershell
New-Item -ItemType Directory -Path "C:\Tools\Sysmon" -Force
```

Luego se descargó Sysmon:

```powershell
Invoke-WebRequest `
    -Uri "https://download.sysinternals.com/files/Sysmon.zip" `
    -OutFile "$env:TEMP\Sysmon.zip"
```

![Descarga de Sysmon en Windows Server](../imagenes/145-descarga-sysmon-windows-server.png)

---

# 13. Extracción de los archivos de Sysmon

El archivo comprimido se extrajo dentro de `C:\Tools\Sysmon`:

```powershell
Expand-Archive `
    -Path "$env:TEMP\Sysmon.zip" `
    -DestinationPath "C:\Tools\Sysmon" `
    -Force
```

Después se accedió al directorio:

```powershell
Set-Location "C:\Tools\Sysmon"
```

![Extracción de los archivos de Sysmon en Windows Server](../imagenes/146-extraccion-archivos-sysmon-windows-server.png)

---

# 14. Descarga del archivo de configuración de Sysmon

Se descargó el archivo `sysmonconfig.xml`, utilizado para definir qué actividades debe registrar Sysmon:

```powershell
Invoke-WebRequest `
    -Uri "https://wazuh.com/resources/blog/emulation-of-attack-techniques-and-detection-with-wazuh/sysmonconfig.xml" `
    -OutFile ".\sysmonconfig.xml"
```

Para verificar su creación se ejecutó:

```powershell
Get-Item .\sysmonconfig.xml
```

También se comprobó que el directorio contuviera:

```text
Sysmon.exe
Sysmon64.exe
Sysmon64a.exe
Eula.txt
sysmonconfig.xml
```

![Descarga de la configuración de Sysmon](../imagenes/147-descarga-configuracion-sysmon.png)

---

# 15. Instalación de Sysmon en Windows Server

Sysmon fue instalado con el archivo de configuración descargado:

```powershell
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

La salida confirmó:

```text
Configuration file validated.
Sysmon64 installed.
SysmonDrv installed.
SysmonDrv started.
Sysmon64 started.
```

Esto confirmó que el archivo XML era válido y que el servicio de Sysmon había sido instalado correctamente.

![Instalación de Sysmon en Windows Server](../imagenes/148-instalacion-sysmon-windows-server.png)

---

# 16. Verificación del servicio y los eventos de Sysmon

Se verificó el servicio con:

```powershell
Get-Service Sysmon*
```

El estado obtenido fue:

```text
Status: Running
Name: Sysmon64
```

También se consultaron los eventos más recientes:

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-Sysmon/Operational" `
    -MaxEvents 5 |
Select-Object TimeCreated, Id, ProviderName
```

La salida mostró eventos generados por:

```text
Microsoft-Windows-Sysmon
```

![Verificación del servicio y eventos de Sysmon](../imagenes/149-verificacion-servicio-eventos-sysmon.png)

---

# 17. Configuración del agente Wazuh para recopilar Sysmon

Antes de modificar el archivo de configuración del agente se creó una copia de seguridad:

```powershell
Copy-Item `
    "C:\Program Files (x86)\ossec-agent\ossec.conf" `
    "C:\Program Files (x86)\ossec-agent\ossec.conf.backup" `
    -Force
```

Luego se abrió el archivo con el Bloc de notas:

```powershell
notepad "C:\Program Files (x86)\ossec-agent\ossec.conf"
```

![Edición de la configuración del agente Wazuh](../imagenes/150-edicion-configuracion-agente-wazuh.png)

---

# 18. Recolección del canal de eventos de Sysmon

Dentro del archivo `ossec.conf`, antes de la etiqueta final `</ossec_config>`, se agregó:

```xml
<!-- Recopilación de eventos de Sysmon -->
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

Esta configuración indica al agente de Wazuh que debe leer y enviar los eventos almacenados en el canal operacional de Sysmon.

![Configuración de la recolección de eventos de Sysmon](../imagenes/151-configuracion-recoleccion-eventos-sysmon.png)

---

# 19. Reinicio del agente Wazuh

Después de guardar los cambios se reinició el servicio:

```powershell
Restart-Service WazuhSvc
```

Su estado fue comprobado con:

```powershell
Get-Service WazuhSvc
```

El resultado confirmó:

```text
Status: Running
Name: WazuhSvc
DisplayName: Wazuh
```

![Reinicio del servicio del agente Wazuh](../imagenes/152-reinicio-servicio-agente-wazuh.png)

---

# 20. Generación de eventos de prueba en Windows Server

Para comprobar la integración se ejecutaron varias acciones:

```powershell
Start-Process notepad.exe
cmd.exe /c whoami
powershell.exe -NoProfile -Command "Get-Date"
```

Estas acciones generaron eventos relacionados con:

- Inicio de procesos.
- Uso de PowerShell.
- Ejecución de CMD.
- Actividades de descubrimiento.
- Creación de procesos secundarios.

Los eventos también fueron verificados localmente:

```powershell
Get-WinEvent `
    -FilterHashtable @{
        LogName = "Microsoft-Windows-Sysmon/Operational"
        Id = 1
    } `
    -MaxEvents 5 |
Select-Object TimeCreated, Id, ProviderName
```

![Generación y verificación de eventos de Sysmon](../imagenes/153-generacion-verificacion-eventos-sysmon.png)

---

# 21. Eventos de SRV-DC01 en Threat Hunting

Desde el Dashboard se accedió a:

```text
Threat Hunting
→ SRV-DC01
```

Wazuh mostró `59` eventos asociados al servidor.

Entre las detecciones se observaron:

- Ejecución de PowerShell.
- Ejecución sospechosa de CMD.
- Actividades de descubrimiento.
- Creación de archivos de scripts.
- Carga del módulo `taskschd.dll`.
- Eliminación de archivos o directorios.
- Archivos ejecutables creados en directorios utilizados frecuentemente por malware.

También se generó una alerta de nivel `15`, correspondiente a una detección de severidad crítica.

Estas alertas fueron provocadas dentro del entorno controlado del laboratorio y permitieron verificar el funcionamiento de Sysmon y Wazuh.

![Eventos de Sysmon de SRV-DC01 en Threat Hunting](../imagenes/154-eventos-sysmon-srv-dc01-en-threat-hunting.png)

---

# 22. Instalación de Sysmon en Windows 10

El mismo procedimiento fue realizado en `WIN10-SOC`.

Primero se creó el directorio y se descargó Sysmon:

```powershell
New-Item -ItemType Directory `
    -Path "C:\Tools\Sysmon" `
    -Force

Invoke-WebRequest `
    -Uri "https://download.sysinternals.com/files/Sysmon.zip" `
    -OutFile "$env:TEMP\Sysmon.zip"
```

![Descarga de Sysmon en Windows 10](../imagenes/155-descarga-sysmon-windows-10.png)

---

# 23. Extracción y verificación de Sysmon en Windows 10

Se extrajeron los archivos:

```powershell
Expand-Archive `
    -Path "$env:TEMP\Sysmon.zip" `
    -DestinationPath "C:\Tools\Sysmon" `
    -Force
```

Luego se descargó la configuración:

```powershell
Invoke-WebRequest `
    -Uri "https://wazuh.com/resources/blog/emulation-of-attack-techniques-and-detection-with-wazuh/sysmonconfig.xml" `
    -OutFile "C:\Tools\Sysmon\sysmonconfig.xml"
```

Los archivos fueron verificados con:

```powershell
Set-Location "C:\Tools\Sysmon"
Get-ChildItem
```

![Extracción y verificación de Sysmon en Windows 10](../imagenes/156-extraccion-y-verificacion-archivos-sysmon-windows-10.png)

---

# 24. Instalación y configuración de Sysmon en Windows 10

Sysmon fue instalado con:

```powershell
.\Sysmon64.exe -accepteula -i .\sysmonconfig.xml
```

Se comprobó el servicio:

```powershell
Get-Service Sysmon*
```

También se verificaron los eventos:

```powershell
Get-WinEvent `
    -LogName "Microsoft-Windows-Sysmon/Operational" `
    -MaxEvents 5 |
Select-Object TimeCreated, Id, ProviderName
```

Finalmente se agregó al archivo `ossec.conf`:

```xml
<!-- Recopilación de eventos de Sysmon -->
<localfile>
  <location>Microsoft-Windows-Sysmon/Operational</location>
  <log_format>eventchannel</log_format>
</localfile>
```

![Instalación y configuración de Sysmon en Windows 10](../imagenes/157-instalacion-y-configuracion-sysmon-windows-10.png)

---

# 25. Reinicio de Wazuh y generación de eventos en Windows 10

Después de guardar la configuración se reinició el agente:

```powershell
Restart-Service WazuhSvc
Start-Sleep -Seconds 5
Get-Service WazuhSvc
```

Para generar eventos de prueba se ejecutó:

```powershell
Start-Process notepad.exe
cmd.exe /c whoami
powershell.exe -NoProfile -Command "Get-Date"
```

El servicio permaneció en estado `Running`.

![Reinicio de Wazuh y generación de eventos en Windows 10](../imagenes/158-reinicio-wazuh-y-generacion-eventos-windows-10.png)

---

# 26. Eventos de WIN10-SOC en Threat Hunting

Desde el Dashboard se accedió a:

```text
Threat Hunting
→ WIN10-SOC
```

Wazuh mostró `37` eventos asociados al equipo Windows 10.

Entre las detecciones aparecieron:

- Procesos de PowerShell.
- Ejecución de CMD.
- Carga del módulo `taskschd.dll`.
- Archivos ejecutables creados en carpetas utilizadas frecuentemente por malware.
- Alertas de nivel `15`.
- Actividades asociadas con comandos ejecutados durante las pruebas.

La recepción de estos eventos confirmó que Sysmon y el agente Wazuh estaban funcionando correctamente en `WIN10-SOC`.

![Eventos de Sysmon de WIN10-SOC en Threat Hunting](../imagenes/159-eventos-sysmon-win10-soc-en-threat-hunting.png)

---

# Resultado final

Los agentes de Wazuh quedaron instalados y registrados correctamente en los dos endpoints Windows del laboratorio:

```text
WIN10-SOC
IP: 192.168.126.30

SRV-DC01
IP: 192.168.126.20
```

También se completó la instalación de Sysmon y la recopilación del canal:

```text
Microsoft-Windows-Sysmon/Operational
```

La integración permite:

- Supervisar procesos ejecutados en los endpoints.
- Detectar el uso de PowerShell y CMD.
- Registrar creación y eliminación de archivos.
- Observar actividades de descubrimiento.
- Detectar comportamientos potencialmente sospechosos.
- Enviar los eventos de Sysmon al Wazuh Manager.
- Analizar alertas desde Threat Hunting.
- Clasificar los eventos mediante reglas y niveles de severidad.
- Utilizar los endpoints para futuras simulaciones de ataques.

La visualización de eventos y alertas en Threat Hunting confirmó el funcionamiento completo de la integración entre:

```text
Sysmon
→ Agente Wazuh
→ Wazuh Manager
→ Wazuh Indexer
→ Wazuh Dashboard
```

Los equipos quedaron preparados para continuar con pruebas de seguridad, simulación de amenazas y análisis de eventos desde el entorno SOC.
