# Instalación y configuración de OPNsense

## Estado

> En progreso

## Objetivo

Instalar OPNsense como firewall principal del laboratorio SOC y utilizarlo como
puerta de enlace para las máquinas virtuales de la red interna.

Posteriormente se instalará Suricata para proporcionar funciones de IDS/IPS.

## Funciones de OPNsense

- Firewall del laboratorio.
- Enrutamiento entre la red WAN y la red LAN.
- Acceso a Internet para las máquinas virtuales.
- Servicio DHCP, si se decide utilizarlo.
- Implementación de Suricata como IDS/IPS.
- Monitoreo del tráfico de red.

## Requisitos de la máquina virtual

| Recurso | Configuración |
|---|---|
| Sistema | OPNsense |
| Procesadores | 2 |
| Memoria RAM | 4 GB |
| Disco | 20 GB |
| Adaptador 1 | NAT o Bridge — WAN |
| Adaptador 2 | Red interna — LAN |

## Interfaces de red

| Interfaz | Tipo de red | Función |
|---|---|---|
| WAN | NAT | Acceso a Internet |
| LAN | Red interna | Red protegida del laboratorio |

## Creación de la máquina virtual

1. Se creó una nueva máquina virtual.
2. Se seleccionó la imagen ISO de OPNsense.
3. Se asignaron 2 procesadores.
4. Se asignaron 4 GB de memoria RAM.
5. Se creó un disco virtual de 20 GB.
6. Se agregaron dos adaptadores de red.

## Configuración de los adaptadores

### Adaptador WAN

Configurado en modo NAT para proporcionar acceso a Internet.

### Adaptador LAN

Configurado en una red interna exclusiva para las máquinas virtuales del laboratorio.

Nombre de la red interna:

```text
SOC-LAN
```

## Instalación

La instalación será documentada paso por paso a medida que se configure la máquina virtual.

## Direccionamiento IP

| Interfaz | Dirección IP |
|---|---|
| WAN | DHCP |
| LAN | Pendiente |

## Evidencias

A continuación se muestran las evidencias recopiladas durante la creación e instalación de la máquina virtual OPNsense.

### Configuración de la red VMnet2

VMnet2 fue configurada como una red Host-only para funcionar como la red LAN interna del laboratorio. El servicio DHCP de VMware fue deshabilitado porque OPNsense administrará la asignación de direcciones IP.

![Configuración de VMnet2](../imagenes/01-configuracion-vmnet2.png)

### Compatibilidad de hardware

Se utilizó la compatibilidad de hardware de VMware Workstation 17.5 o posterior.

![Compatibilidad de hardware](../imagenes/02-compatibilidad-hardware-opnsense.png)

### Selección de la imagen ISO

Se seleccionó la imagen DVD ISO de OPNsense para iniciar la instalación.

![Selección de la ISO de OPNsense](../imagenes/03-seleccion-iso-opnsense.png)

### Configuración del procesador

La máquina virtual fue configurada con un procesador virtual y dos núcleos.

![Configuración del procesador](../imagenes/04-configuracion-procesador-opnsense.png)

### Configuración del adaptador WAN

El primer adaptador utilizado para la conexión WAN fue configurado en modo NAT.

![Adaptador WAN en modo NAT](../imagenes/05-adaptador-wan-nat.png)

### Adaptadores de red

Se agregaron dos adaptadores de red a la máquina virtual.

![Adaptadores de OPNsense](../imagenes/06-adaptadores-opnsense.png)

### Configuración final de los adaptadores

El adaptador LAN fue conectado a VMnet2 y el adaptador WAN fue conectado a la red NAT de VMware.

![Adaptadores configurados](../imagenes/07-adaptadores-configurados-opnsense.png)

### Arranque en modo Live

OPNsense inició desde la imagen ISO y detectó las interfaces LAN y WAN.

![Arranque Live de OPNsense](../imagenes/08-arranque-live-opnsense.png)

### Selección del teclado

Durante la instalación se mantuvo el mapa de teclado predeterminado.

![Selección del teclado](../imagenes/09-seleccion-teclado-opnsense.png)

### Instalación mediante ZFS

Se seleccionó ZFS como sistema de archivos para la instalación.

![Instalación ZFS](../imagenes/10-instalacion-zfs-opnsense.png)

### Tipo de disco

Se seleccionó la configuración correspondiente a un solo disco virtual.

![Tipo de disco](../imagenes/11-tipo-de-disco-opnsense.png)

### Selección del disco virtual

El instalador detectó el disco virtual `da0`, creado exclusivamente para OPNsense.

![Selección del disco](../imagenes/12-seleccion-disco-opnsense.png)

### Progreso de la instalación

El instalador clonó el sistema y preparó el disco virtual de destino.

![Progreso de la instalación](../imagenes/13-progreso-instalacion-opnsense.png)

### Configuración de la contraseña

Al finalizar la instalación se configuró una contraseña para la cuenta `root`. La contraseña no fue almacenada en este repositorio.

![Configuración de la contraseña root](../imagenes/14-cambio-de-password-opnsense.png)

### Reinicio del sistema

Después de completar la instalación se seleccionó la opción para reiniciar OPNsense.

![Reinicio de OPNsense](../imagenes/15-reinicio-opnsense.png)

### Desconexión de la imagen ISO

Se desconectó la unidad CD/DVD y se deshabilitó la opción `Connect at power on` para evitar que la máquina iniciara nuevamente desde el instalador.

![Desconexión de la imagen ISO](../imagenes/16-desconectar-iso-opnsense.png)

### Primer inicio desde el disco

OPNsense inició correctamente desde el disco virtual y mostró las interfaces LAN y WAN en la consola.

![Consola de OPNsense instalada](../imagenes/17-consola-opnsense-instalada.png)


## Configuración de la interfaz LAN

### Menú principal de OPNsense

![Menú principal de la consola de OPNsense](../imagenes/18-opnsense-menu-principal-consola.png)

### Selección de la interfaz LAN

![Selección de la interfaz LAN](../imagenes/19-opnsense-seleccionar-interfaz-lan.png)

### Configuración de IPv4 en la LAN

![Configuración IPv4 de la interfaz LAN](../imagenes/20-opnsense-configurar-ipv4-lan.png)

### Asignación de la dirección IP

![Asignación de la dirección IP LAN](../imagenes/21-opnsense-asignar-direccion-ip-lan.png)

### Configuración de la máscara de red

![Configuración de la máscara LAN](../imagenes/22-opnsense-configurar-mascara-lan.png)

### Configuración de IPv6

![Configuración IPv6 de la LAN](../imagenes/23-opnsense-configurar-ipv6-lan.png)

## Configuración del servidor DHCP

### Habilitación del servidor DHCP

![Habilitación del servidor DHCP](../imagenes/24-opnsense-habilitar-servidor-dhcp.png)

### Configuración del rango DHCP

![Configuración del rango DHCP](../imagenes/25-opnsense-configurar-rango-dhcp.png)

## Acceso a la interfaz web

### Configuración de HTTPS y certificado

![Configuración HTTPS de OPNsense](../imagenes/26-opnsense-configurar-https-certificado.png)

### Configuración LAN completada

![Configuración LAN completada](../imagenes/27-opnsense-configuracion-lan-completada.png)

### Inicio de sesión web

![Inicio de sesión en OPNsense](../imagenes/28-opnsense-inicio-sesion-web.png)

### Asistente de configuración inicial

![Asistente de configuración inicial](../imagenes/29-opnsense-asistente-configuracion-inicial.png)

## Configuración general

### Acceso a los ajustes generales

![Acceso a los ajustes generales](../imagenes/30-opnsense-acceder-ajustes-generales.png)

### Configuración de servidores DNS

![Configuración de servidores DNS](../imagenes/31-opnsense-configurar-servidores-dns.png)

### Configuración general del sistema

![Configuración general de OPNsense](../imagenes/32-opnsense-configuracion-general-sistema.png)

### Aplicación de los cambios

![Cambios generales aplicados](../imagenes/33-opnsense-cambios-generales-aplicados.png)

## Preparación del IDS/IPS

### Acceso a detección de intrusiones

![Acceso a detección de intrusiones](../imagenes/34-opnsense-acceder-deteccion-intrusiones.png)

## Verificaciones

- [ ] OPNsense inicia correctamente.
- [ ] La interfaz WAN recibe una dirección IP.
- [ ] La interfaz LAN tiene una dirección IP estática.
- [ ] Los equipos de la red LAN pueden comunicarse con OPNsense.
- [ ] Los equipos de la red LAN tienen acceso a Internet.
- [ ] Es posible acceder al panel web de OPNsense.

## Problemas encontrados

En esta sección se documentarán los errores presentados durante la instalación.

## Soluciones aplicadas

En esta sección se registrarán las soluciones utilizadas.

## Resultado

Pendiente hasta completar la instalación y las pruebas.
