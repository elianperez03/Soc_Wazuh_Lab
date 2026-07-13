# Configuración de Kali Linux

## Estado

> Completado

## Objetivo

Configurar Kali Linux como máquina de pruebas de seguridad dentro del laboratorio SOC.

Kali será utilizado para generar tráfico, realizar pruebas de conectividad y ejecutar futuras simulaciones controladas contra los equipos monitoreados por Wazuh.

La máquina debe enviar todo su tráfico a través de OPNsense para que Suricata pueda inspeccionarlo.

---

## Información de red del laboratorio

| Componente | Dirección IP |
|---|---:|
| Red LAN | `192.168.126.0/24` |
| OPNsense / Gateway | `192.168.126.254` |
| Wazuh Server | `192.168.126.10` |
| Windows Server | `192.168.126.20` |
| Windows 10 | `192.168.126.30` |
| DHCP de OPNsense | `192.168.126.100-192.168.126.200` |

---

# 1. Configuración de la máquina virtual

La máquina virtual de Kali Linux se configuró dentro de VMware Workstation.

La máquina puede utilizar los siguientes recursos:

```text
Procesadores: 2
Memoria RAM: 4 GB
Disco virtual: 40 GB o superior
Adaptador de red: VMnet2 (Host-only)
```

El adaptador de red debe estar conectado únicamente a `VMnet2`.

Esto permite que Kali forme parte de la red LAN protegida por OPNsense.

> No se debe utilizar un adaptador NAT adicional, porque el tráfico podría salir directamente a Internet sin pasar por OPNsense y Suricata.


---

# 2. Configuración del adaptador de red

En VMware se accedió a:

```text
VM
→ Settings
→ Network Adapter
```

Se seleccionó:

```text
Custom: Specific virtual network
VMnet2 (Host-only)
```

También se verificó que las opciones siguientes estuvieran activadas:

```text
Connected
Connect at power on
```


---

# 3. Inicio de Kali Linux

Después de configurar el adaptador de red se inició la máquina virtual.

Kali Linux quedó conectado a la misma red interna que los demás componentes del laboratorio:

```text
Kali Linux
     |
   VMnet2
     |
OPNsense / Suricata
     |
Red del laboratorio SOC
```

Toda la comunicación de Kali hacia los demás equipos y hacia Internet debe pasar por OPNsense.

---

# 4. Verificación de la interfaz de red

Para identificar la interfaz de red y la dirección IP asignada se ejecutó:

```bash
ip addr
```

También puede utilizarse:

```bash
ip a
```

La interfaz puede aparecer con un nombre similar a:

```text
eth0
ens33
```

Kali debe recibir automáticamente una dirección dentro del rango DHCP configurado en OPNsense:

```text
192.168.126.100-192.168.126.200
```

---

# 5. Verificación de la ruta predeterminada

Para consultar la tabla de enrutamiento se ejecutó:

```bash
ip route
```

La salida debe mostrar una ruta predeterminada similar a:

```text
default via 192.168.126.254
```

Esto confirma que OPNsense está funcionando como puerta de enlace para Kali Linux.

También debe aparecer la red local:

```text
192.168.126.0/24
```


---

# 6. Verificación del servidor DNS

La configuración DNS puede consultarse con:

```bash
cat /etc/resolv.conf
```

También puede verificarse mediante NetworkManager:

```bash
nmcli device show
```

Para consultar específicamente los servidores DNS:

```bash
nmcli device show | grep IP4.DNS
```

El DNS debe permitir que Kali resuelva nombres de dominio y acceda a Internet.

---

# 7. Prueba de conectividad con OPNsense

Se comprobó la comunicación con la puerta de enlace:

```bash
ping -c 4 192.168.126.254
```

Una respuesta correcta confirma que Kali puede comunicarse con la interfaz LAN de OPNsense.


---

# 8. Prueba de conectividad con Wazuh

Se verificó la comunicación con el servidor Wazuh:

```bash
ping -c 4 192.168.126.10
```

También puede comprobarse el acceso al Dashboard mediante:

```bash
curl -k -I https://192.168.126.10
```

La opción `-k` permite realizar la prueba aunque el Dashboard utilice un certificado autofirmado.

---

# 9. Prueba de conectividad con Windows Server

Se verificó la comunicación con el controlador de dominio:

```bash
ping -c 4 192.168.126.20
```

El equipo corresponde a:

```text
Nombre: SRV-DC01
IP: 192.168.126.20
Dominio: soc.lab
```

Esta comunicación permitirá realizar pruebas controladas contra el servidor y verificar las alertas generadas en Wazuh.

---

# 10. Prueba de conectividad con Windows 10

Se verificó la comunicación con el endpoint Windows 10:

```bash
ping -c 4 192.168.126.30
```

El equipo corresponde a:

```text
Nombre: WIN10-SOC
IP: 192.168.126.30
Dominio: soc.lab
```

El endpoint está configurado con el agente Wazuh y Sysmon, por lo que las actividades realizadas durante futuras pruebas podrán ser enviadas al Wazuh Manager.

---

# 11. Prueba de acceso a Internet

Para comprobar que OPNsense permite la salida a Internet se ejecutó:

```bash
ping -c 4 1.1.1.1
```

La respuesta confirmó que Kali podía acceder a Internet utilizando a OPNsense como puerta de enlace.


Para comprobar también la resolución DNS se puede ejecutar:

```bash
ping -c 4 google.com
```

La diferencia entre ambas pruebas es:

```text
ping 1.1.1.1
Verifica conectividad con Internet.

ping google.com
Verifica conectividad y resolución DNS.
```

---

# 12. Actualización de Kali Linux

Después de verificar la conectividad se actualizaron los repositorios:

```bash
sudo apt update
```

Para instalar las actualizaciones disponibles:

```bash
sudo apt full-upgrade -y
```

Finalmente se eliminaron paquetes innecesarios:

```bash
sudo apt autoremove -y
```


---

# 13. Verificación de herramientas básicas

Se verificó la disponibilidad de herramientas que serán utilizadas durante las pruebas del laboratorio:

```bash
nmap --version
```

```bash
curl --version
```

```bash
python3 --version
```

```bash
whoami
```

Kali Linux ya incluye diferentes herramientas para reconocimiento, análisis de red y pruebas de seguridad.

Todas las pruebas deben ejecutarse únicamente contra los equipos autorizados dentro del laboratorio.

---

# 14. Identificación de equipos activos

Para verificar los equipos disponibles dentro de la red se puede ejecutar un descubrimiento básico:

```bash
sudo nmap -sn 192.168.126.0/24
```

Este comando realiza una búsqueda de hosts activos sin ejecutar un análisis completo de puertos.

Entre los equipos esperados se encuentran:

```text
192.168.126.10     Wazuh Server
192.168.126.20     SRV-DC01
192.168.126.30     WIN10-SOC
192.168.126.254    OPNsense
```


---

# 15. Validación del tráfico mediante Suricata

Kali Linux fue utilizado para generar tráfico desde la red LAN.

Debido a que su puerta de enlace es:

```text
192.168.126.254
```

el tráfico pasa por OPNsense antes de llegar a Internet o a otras redes.

Suricata puede inspeccionar este tráfico utilizando:

- Reglas ET Open.
- Indicadores de compromiso de abuse.ch.
- Detección de escaneos.
- Detección de actividad sospechosa.
- Alertas de malware.
- Bloqueo de tráfico en modo IPS.
- Reglas personalizadas por dirección IP.

Durante las pruebas se comprobó la conectividad con:

```bash
ping -c 4 1.1.1.1
```

Posteriormente, las reglas de Suricata permitieron verificar el registro y bloqueo de tráfico dentro del entorno controlado.

---

# Función de Kali Linux en el laboratorio

Kali Linux funcionará como máquina de pruebas y simulación.

Desde este equipo se podrán realizar actividades como:

- Pruebas de conectividad.
- Descubrimiento de hosts.
- Análisis de puertos.
- Generación de tráfico para Suricata.
- Simulaciones controladas de reconocimiento.
- Pruebas contra Windows Server.
- Pruebas contra Windows 10.
- Validación de alertas de Wazuh.
- Validación de reglas de Sysmon.
- Análisis de eventos desde Threat Hunting.

Las pruebas deben mantenerse dentro de la red:

```text
192.168.126.0/24
```

---

# Resultado final

Kali Linux quedó configurado correctamente dentro de la red interna del laboratorio SOC.

La máquina quedó conectada mediante:

```text
VMnet2 (Host-only)
```

La configuración permite que Kali:

- Reciba una dirección IP desde OPNsense.
- Utilice `192.168.126.254` como puerta de enlace.
- Se comunique con el servidor Wazuh.
- Se comunique con Windows Server.
- Se comunique con Windows 10.
- Acceda a Internet a través de OPNsense.
- Genere tráfico que pueda ser inspeccionado por Suricata.
- Sea utilizado para futuras pruebas de seguridad.
- Genere actividad que pueda ser detectada por Sysmon y Wazuh.

La ruta general del tráfico quedó configurada de la siguiente manera:

```text
Kali Linux
     |
     v
VMnet2 - Red LAN
     |
     v
OPNsense + Suricata IPS
     |
     v
Internet / Equipos del laboratorio
```

Con esta configuración, Kali Linux quedó preparado para continuar con la fase de simulación de ataques y validación de alertas del laboratorio SOC.
