# Configuración de Ubuntu Server para Wazuh

## Estado

> En progreso

## Objetivo

Instalar y preparar una máquina virtual con Ubuntu Server para implementar los componentes principales de Wazuh dentro del laboratorio SOC.

En esta máquina se instalarán:

- Wazuh Manager.
- Wazuh Indexer.
- Wazuh Dashboard.

La máquina estará conectada a la red LAN administrada por OPNsense, de manera que su direccionamiento IP, acceso a Internet y tráfico de red pasen a través del firewall.

---

## Resumen de la máquina virtual

| Parámetro | Configuración |
|---|---|
| Hipervisor | VMware Workstation |
| Sistema operativo | Ubuntu Server |
| Tipo de instalación | Custom (Advanced) |
| Procesadores | 2 |
| Núcleos | 2 |
| Memoria RAM | 8 GB |
| Disco | 65 GB |
| Tipo de disco | SCSI |
| Adaptador de red | VMnet2 |
| Dirección IP inicial | DHCP |
| Dirección IP planificada | `192.168.126.10/24` |
| Puerta de enlace | `192.168.126.254` |
| DNS | `192.168.126.254` |
| Nombre del servidor | `wazuh-server` |

> La dirección IP fija será configurada después de comprobar que Ubuntu recibe una dirección correctamente mediante DHCP.

---

## 1. Creación de la máquina virtual

En VMware Workstation se seleccionó la opción:

```text
Create a New Virtual Machine
```

Para tener mayor control sobre el hardware virtual, se utilizó el tipo de configuración:

```text
Custom (Advanced)
```

![Creación de la máquina virtual de Ubuntu Server](../imagenes/99-creacion-maquina-virtual-ubuntu-wazuh.png)

---

## 2. Compatibilidad del hardware virtual

Se seleccionó una compatibilidad adecuada para la versión de VMware Workstation utilizada en el laboratorio.

```text
VMware Workstation 17.5 o posterior
```

![Selección de compatibilidad de hardware](../imagenes/100-compatibilidad-hardware-ubuntu-wazuh.png)

---

## 3. Selección de la imagen ISO

Se seleccionó la imagen ISO de Ubuntu Server que será utilizada para realizar la instalación.

La edición Server fue elegida porque no incluye un entorno gráfico completo y consume menos recursos que Ubuntu Desktop.

![Selección de la imagen ISO de Ubuntu Server](../imagenes/101-seleccion-iso-ubuntu-server-wazuh.png)

---

## 4. Selección del sistema operativo

La máquina virtual fue configurada con los siguientes valores:

```text
Guest operating system: Linux
Version: Ubuntu 64-bit
```

![Selección del sistema operativo Ubuntu de 64 bits](../imagenes/102-seleccion-sistema-operativo-ubuntu-64-bits.png)

---

## 5. Nombre y ubicación de la máquina virtual

Se asignó el siguiente nombre a la máquina virtual:

```text
Ubuntu-Wazuh
```

La máquina fue almacenada dentro de la carpeta correspondiente al laboratorio SOC.

![Nombre y ubicación de la máquina virtual](../imagenes/103-nombre-ubicacion-maquina-ubuntu-wazuh.png)

---

## 6. Configuración del procesador

La máquina virtual fue configurada con cuatro núcleos virtuales.

```text
Number of processors: 2
Number of cores per processor: 2
Total processor cores: 4
```

Esta cantidad permitirá ejecutar Wazuh Manager, Indexer y Dashboard en la misma máquina virtual.

![Configuración del procesador de Ubuntu Wazuh](../imagenes/104-configuracion-procesador-ubuntu-wazuh.png)

---

## 7. Configuración de la memoria RAM

Se asignaron:

```text
8 GB de memoria RAM
```

Esta cantidad representa el mínimo recomendado para una instalación pequeña de Wazuh con todos sus componentes en un solo servidor.

![Configuración de memoria RAM de Ubuntu Wazuh](../imagenes/105-configuracion-memoria-ram-ubuntu-wazuh.png)

---

## 8. Configuración del adaptador de red

La máquina virtual fue conectada a la red interna del laboratorio:

```text
Custom: Specific virtual network
VMnet2
```

También se habilitaron las opciones:

```text
Connected
Connect at power on
```

No se agregó un adaptador NAT adicional, ya que Ubuntu debe utilizar OPNsense como puerta de enlace.

![Configuración del adaptador VMnet2](../imagenes/106-configuracion-adaptador-vmnet2-ubuntu-wazuh.png)

---

## 9. Configuración del controlador de almacenamiento

Se utilizó el controlador recomendado por VMware:

```text
LSI Logic SAS
```

![Configuración del controlador de almacenamiento](../imagenes/107-controlador-almacenamiento-ubuntu-wazuh.png)

---

## 10. Creación del disco virtual

Se creó un nuevo disco virtual de tipo:

```text
SCSI
```

Con una capacidad de:

```text
65 GB
```

El disco será utilizado para almacenar:

- El sistema operativo.
- Los componentes de Wazuh.
- Los índices generados por Wazuh Indexer.
- Los registros de seguridad.
- Los archivos de configuración.

![Creación del disco virtual de Ubuntu Wazuh](../imagenes/108-creacion-disco-virtual-ubuntu-wazuh.png)

---

## 11. Resumen de la máquina virtual

Antes de finalizar, se verificó que la configuración de la máquina virtual fuera correcta.

```text
CPU: 4 núcleos
RAM: 8 GB
Disco: 65 GB
Red: VMnet2
Sistema: Ubuntu Server 64-bit
```

![Resumen de la máquina virtual Ubuntu Wazuh](../imagenes/109-resumen-maquina-virtual-ubuntu-wazuh.png)

---

## 12. Inicio de la instalación

Se inició la máquina virtual con la imagen ISO de Ubuntu Server conectada.

En el menú de arranque se seleccionó:

```text
Try or Install Ubuntu Server
```

![Inicio de la instalación de Ubuntu Server](../imagenes/110-inicio-instalacion-ubuntu-server.png)

---

## 13. Selección del idioma

Se seleccionó el idioma que será utilizado durante la instalación.

```text
English
```

Se mantuvo el idioma inglés porque facilita el seguimiento de la documentación técnica y la identificación de mensajes de error.

![Selección del idioma de Ubuntu Server](../imagenes/111-seleccion-idioma-ubuntu-server.png)

---

## 14. Tipo de instalación

Se seleccionó la instalación estándar de Ubuntu Server.

```text
Ubuntu Server
```

No se utilizó la versión minimizada, ya que se instalarán diferentes dependencias y componentes para Wazuh.

![Selección del tipo de instalación de Ubuntu Server](../imagenes/113-tipo-instalacion-ubuntu-server.png)

---

## 16. Configuración de red mediante DHCP

Durante la instalación, Ubuntu detectó el adaptador de red conectado a VMnet2.

OPNsense asignó automáticamente una dirección dentro del rango:

```text
192.168.126.100 - 192.168.126.200
```

La puerta de enlace asignada corresponde a:

```text
192.168.126.254
```

![Dirección IP asignada por OPNsense a Ubuntu](../imagenes/114-direccion-ip-dhcp-ubuntu-wazuh.png)

---

## 17. Configuración del proxy

El laboratorio no utiliza un servidor proxy.

Por esta razón, el campo fue dejado vacío.

![Configuración del proxy de Ubuntu Server](../imagenes/115-configuracion-proxy-ubuntu-server.png)

---

## 18. Configuración del repositorio de Ubuntu

Se mantuvo el repositorio recomendado automáticamente por Ubuntu Server.

El instalador comprobó que el servidor pudiera acceder correctamente al repositorio mediante OPNsense.

![Configuración del repositorio de Ubuntu](../imagenes/116-configuracion-repositorio-ubuntu-server.png)

---

## 19. Configuración del almacenamiento

Se seleccionó la opción para utilizar el disco virtual completo.

```text
Use an entire disk
```

El disco seleccionado fue el disco virtual de 80 GB creado exclusivamente para Ubuntu Server y Wazuh.

![Selección del disco de instalación](../imagenes/117-seleccion-disco-instalacion-ubuntu-wazuh.png)

---

## 20. Confirmación de las particiones

Se revisó el esquema de particiones generado por el instalador.

Después de comprobar que el disco correcto estaba seleccionado, se confirmó la escritura de los cambios.

![Confirmación de las particiones de Ubuntu Server](../imagenes/118-confirmacion-particiones-ubuntu-server.png)

---

## 21. Creación del usuario administrador

Se configuraron los datos principales del servidor.

Ejemplo:

```text
Your name: ubuntu
Your server's name: ubuntu
Username: ubuntu
Password: contraseña segura
```

> La contraseña utilizada no fue almacenada en este repositorio.

![Creación del usuario administrador de Ubuntu](../imagenes/119-creacion-usuario-administrador-ubuntu.png)

---

## 22. Configuración de Ubuntu Pro

La suscripción de Ubuntu Pro no fue activada durante la instalación.

Esta opción podrá configurarse posteriormente si fuera necesaria.

![Configuración de Ubuntu Pro](../imagenes/120-configuracion-ubuntu-pro.png)

---

## 23. Instalación de OpenSSH Server

Se habilitó la opción:

```text
Install OpenSSH server
```

Esto permitirá administrar Ubuntu Server remotamente desde otra máquina del laboratorio.

![Instalación de OpenSSH Server](../imagenes/121-instalacion-openssh-server-ubuntu.png)

---

## 24. Selección de paquetes adicionales

No se seleccionaron paquetes adicionales durante esta etapa.

Los componentes necesarios serán instalados posteriormente mediante el administrador de paquetes de Ubuntu.

![Selección de paquetes adicionales](../imagenes/122-paquetes-adicionales-ubuntu-server.png)

---

## 25. Proceso de instalación

Ubuntu Server inició la copia e instalación de los archivos en el disco virtual.

Durante este proceso también fueron instalados los componentes base del sistema.

![Proceso de instalación de Ubuntu Server](../imagenes/123-proceso-instalacion-ubuntu-server.png)

---

## 26. Reinicio del sistema

Al finalizar la instalación, se seleccionó:

```text
Reboot Now
```

Antes de iniciar nuevamente, se desconectó la imagen ISO para evitar que la máquina regresara al instalador.

![Reinicio después de instalar Ubuntu Server](../imagenes/124-reinicio-ubuntu-server-instalado.png)

---

## 27. Primer inicio de sesión

Después del reinicio se inició sesión con el usuario creado durante la instalación.

![Primer inicio de sesión en Ubuntu Server](../imagenes/125-primer-inicio-sesion-ubuntu-server.png)

---

## 28. Verificación del direccionamiento IP

Se ejecutó:

```bash
ip a
```

El comando permitió comprobar la dirección asignada por OPNsense.

También se ejecutó:

```bash
ip route
```

La ruta predeterminada debe apuntar hacia:

```text
192.168.126.254
```

![Verificación de la dirección IP de Ubuntu](../imagenes/126-verificacion-direccion-ip-ubuntu-wazuh.png)

---

## 29. Pruebas de conectividad

Se realizaron las siguientes pruebas:

```bash
ping -c 4 192.168.126.254
ping -c 4 1.1.1.1
ping -c 4 google.com
```

Estas pruebas permiten confirmar:

- Comunicación con OPNsense.
- Acceso a Internet.
- Funcionamiento de NAT.
- Resolución DNS.
- Funcionamiento del adaptador VMnet2.

![Pruebas de conectividad de Ubuntu Server](../imagenes/127-pruebas-conectividad-ubuntu-wazuh.png)

---

## 30. Actualización del sistema operativo

Después de comprobar la conectividad, se actualizaron los repositorios:

```bash
sudo apt update
```

Posteriormente, se instalaron las actualizaciones disponibles:

```bash
sudo apt upgrade -y
```

Finalmente, se eliminaron paquetes innecesarios:

```bash
sudo apt autoremove -y
```

![Actualización de Ubuntu Server](../imagenes/128-actualizacion-ubuntu-server.png)

---

## 31. Configuración de la zona horaria

Se verificó la zona horaria con:

```bash
timedatectl
```

La zona horaria del laboratorio debe configurarse como:

```text
America/Santo_Domingo
```

Para cambiarla se utiliza:

```bash
sudo timedatectl set-timezone America/Santo_Domingo
```

![Configuración de la zona horaria](../imagenes/129-configuracion-zona-horaria-ubuntu.png)

---

## 32. Configuración del nombre del servidor

Se comprobó el nombre del servidor con:

```bash
hostnamectl
```

El nombre utilizado será:

```text
wazuh-server
```

En caso de ser necesario, puede modificarse con:

```bash
sudo hostnamectl set-hostname wazuh-server
```

![Configuración del hostname del servidor](../imagenes/130-configuracion-hostname-wazuh-server.png)

---

## 33. Configuración de la dirección IP estática

Después de comprobar que DHCP funciona correctamente, se configurará una dirección IP fija para el servidor Wazuh.

La configuración planificada es:

```text
Dirección IP: 192.168.126.10/24
Puerta de enlace: 192.168.126.254
DNS: 192.168.126.254
```

Esta dirección está fuera del rango DHCP configurado en OPNsense:

```text
192.168.126.100 - 192.168.126.200
```

Esto evita conflictos con las direcciones asignadas automáticamente a otros equipos.

La configuración será realizada mediante Netplan.

![Configuración de la dirección IP estática](../imagenes/131-configuracion-ip-estatica-ubuntu-wazuh.png)

---

## 34. Verificación final del servidor

Después de configurar la dirección IP estática, se realizarán nuevamente las siguientes pruebas:

```bash
ip a
ip route
ping -c 4 192.168.126.254
ping -c 4 1.1.1.1
ping -c 4 google.com
```

También se comprobará el acceso SSH desde otra máquina del laboratorio:

```bash
ssh wazuhadmin@192.168.126.10
```

![Verificación final del servidor Ubuntu](../imagenes/132-verificacion-final-ubuntu-wazuh.png)

---

## Problemas encontrados

### Ubuntu no recibe una dirección IP

Se debe comprobar que:

- El adaptador de la máquina esté conectado a VMnet2.
- La opción **Connected** esté marcada.
- La opción **Connect at power on** esté marcada.
- El servidor DHCP de OPNsense esté activo.
- El servicio DHCP de VMware esté deshabilitado para VMnet2.

### Ubuntu no tiene acceso a Internet

Se debe comprobar:

```bash
ip route
```

La puerta de enlace debe ser:

```text
192.168.126.254
```

También debe comprobarse que la regla de firewall de la LAN permita la salida hacia Internet.

### Ubuntu tiene acceso por IP, pero no resuelve dominios

Se debe comprobar:

```bash
resolvectl status
```

El servidor DNS debe ser:

```text
192.168.126.254
```

---

## Verificaciones finales

- [ ] La máquina virtual fue creada con configuración personalizada.
- [ ] Se asignaron cuatro núcleos.
- [ ] Se asignaron 8 GB de RAM.
- [ ] Se creó un disco virtual de 80 GB.
- [ ] El adaptador está conectado a VMnet2.
- [ ] Ubuntu Server fue instalado correctamente.
- [ ] OpenSSH Server fue habilitado.
- [ ] Ubuntu recibió una dirección mediante DHCP.
- [ ] La puerta de enlace corresponde a OPNsense.
- [ ] El servidor tiene acceso a Internet.
- [ ] La resolución DNS funciona.
- [ ] El sistema operativo fue actualizado.
- [ ] La zona horaria fue configurada.
- [ ] El hostname fue configurado como `wazuh-server`.
- [ ] La dirección IP estática fue configurada.
- [ ] El acceso mediante SSH funciona.
- [ ] El servidor está preparado para instalar Wazuh.

---

## Resultado final

Ubuntu Server quedó instalado y conectado correctamente a la red interna del laboratorio SOC.

La máquina utiliza OPNsense como:

- Servidor DHCP.
- Puerta de enlace.
- Servidor DNS.
- Firewall.
- Sistema de prevención de intrusiones.

El servidor quedó preparado para la instalación de:

```text
Wazuh Manager
Wazuh Indexer
Wazuh Dashboard
```

La siguiente etapa será instalar y configurar los componentes de Wazuh.
