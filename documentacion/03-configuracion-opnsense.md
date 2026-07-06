# Configuración de OPNsense

## Estado

> Configuración base completada. Configuración de Suricata pendiente.

## Objetivo

Configurar OPNsense como firewall, router, servidor DHCP, servidor DNS y puerta de enlace principal de la red interna del laboratorio SOC.

Esta configuración permitirá que las máquinas virtuales conectadas a `VMnet2` reciban direccionamiento IP, resuelvan nombres de dominio y accedan a Internet por medio de OPNsense.

## Resumen de la configuración

| Parámetro | Configuración |
|---|---|
| Nombre del equipo | `opnsense` |
| Dominio | `soc.lab` |
| Interfaz WAN | DHCP mediante VMware NAT |
| Interfaz LAN | `192.168.126.254/24` |
| Red interna | `192.168.126.0/24` |
| Red virtual | `VMnet2` |
| Rango DHCP | `192.168.126.100–192.168.126.200` |
| DNS para los clientes | `192.168.126.254` |
| Unbound DNS | Puerto `53` |
| Dnsmasq | Puerto `53053` |
| NAT saliente | Automático |

---

## Configuración de la interfaz WAN

La interfaz WAN fue configurada mediante DHCP para recibir automáticamente una dirección IP desde la red NAT de VMware.

La opción **Block private networks** fue deshabilitada porque VMware asigna a la WAN una dirección privada perteneciente a la red `192.168.92.0/24`.

La opción **Block bogon networks** permaneció habilitada.

![Configuración de la interfaz WAN](../imagenes/44-configuracion-interfaz-wan-opnsense.png)

Los parámetros avanzados del cliente DHCP se mantuvieron con sus valores predeterminados.

```text
IPv4: DHCP
IPv6: None
MTU: Automático
MSS: Automático
Velocidad y dúplex: Default
```

![Configuración del cliente DHCP de la WAN](../imagenes/45-configuracion-dhcp-cliente-wan-opnsense.png)

---

## Verificación de la puerta de enlace

Se accedió a la configuración de las puertas de enlace desde:

```text
System → Gateways → Configuration
```

![Acceso a las puertas de enlace](../imagenes/46-acceso-puertas-enlace-opnsense.png)

OPNsense creó automáticamente la puerta de enlace `WAN_DHCP`.

| Parámetro | Valor |
|---|---|
| Nombre | `WAN_DHCP` |
| Interfaz | WAN |
| Familia | IPv4 |
| Puerta de enlace | `192.168.92.2` |
| Estado | Activa |

La dirección de la puerta de enlace es proporcionada por la red NAT de VMware.

![Verificación de la puerta de enlace WAN](../imagenes/47-verificacion-puerta-enlace-wan-opnsense.png)

---

## Configuración de Dnsmasq DNS y DHCP

Se accedió al servicio Dnsmasq desde:

```text
Services → Dnsmasq DNS & DHCP → General
```

![Acceso a Dnsmasq DNS y DHCP](../imagenes/48-acceso-dnsmasq-dns-dhcp-opnsense.png)

Dnsmasq fue habilitado en la interfaz LAN para administrar la asignación dinámica de direcciones IP y registrar los nombres de los clientes del laboratorio.

### Parámetros generales

| Parámetro | Valor |
|---|---|
| Servicio habilitado | Sí |
| Interfaz | LAN |
| Puerto de escucha | `53053` |
| DHCP FQDN | Habilitado |
| Dominio DHCP | `soc.lab` |
| Dominio local DHCP | Habilitado |
| DHCP autoritativo | Habilitado |
| Reglas automáticas de firewall DHCP | Habilitado |
| Router Advertisements | Deshabilitado |

Dnsmasq utiliza el puerto `53053` porque Unbound utiliza el puerto DNS estándar `53`.

![Configuración general de Dnsmasq](../imagenes/49-configuracion-general-dnsmasq-opnsense.png)

En la parte inferior se configuraron las opciones relacionadas con DHCP.

![Opciones DHCP de Dnsmasq](../imagenes/50-opciones-dhcp-dnsmasq-opnsense.png)

La configuración final dejó Dnsmasq como servidor DHCP autoritativo de la red `VMnet2`.

El servicio DHCP de VMware permanece deshabilitado para evitar conflictos.

![Configuración final de Dnsmasq](../imagenes/51-configuracion-final-dnsmasq-opnsense.png)

---

## Configuración del rango DHCP

Inicialmente, OPNsense tenía configurado el siguiente rango:

```text
192.168.126.41–192.168.126.245
```

![Rango DHCP inicial](../imagenes/52-rango-dhcp-inicial-lan-opnsense.png)

El rango fue modificado para reservar las direcciones inferiores para servidores y equipos con direccionamiento estático.

### Nuevo rango DHCP

| Parámetro | Valor |
|---|---|
| Interfaz | LAN |
| Dirección inicial | `192.168.126.100` |
| Dirección final | `192.168.126.200` |
| Máscara | Automática |
| Tiempo de concesión | `86400` segundos |
| Dominio | `soc.lab` |
| Descripción | Rango DHCP LAN del laboratorio |

![Edición del rango DHCP](../imagenes/53-edicion-rango-dhcp-lan-opnsense.png)

La distribución del direccionamiento quedó organizada de la siguiente manera:

```text
192.168.126.1–99       Servidores y direcciones estáticas
192.168.126.100–200    Equipos mediante DHCP
192.168.126.201–253    Direcciones reservadas para uso futuro
192.168.126.254        OPNsense y puerta de enlace
```

![Rango DHCP configurado](../imagenes/54-rango-dhcp-configurado-lan-opnsense.png)

---

## Configuración de Unbound DNS

Se accedió a Unbound desde:

```text
Services → Unbound DNS → General
```

![Acceso a Unbound DNS](../imagenes/55-acceso-unbound-dns-opnsense.png)

Unbound fue configurado como el servidor DNS principal de las máquinas del laboratorio.

| Parámetro | Valor |
|---|---|
| Unbound habilitado | Sí |
| Puerto de escucha | `53` |
| Interfaces de red | Todas |
| DNS64 | Deshabilitado |
| AAAA-only | Deshabilitado |
| Local Zone Type | `transparent` |

![Configuración general de Unbound DNS](../imagenes/56-configuracion-general-unbound-dns-opnsense.png)

---

## Reenvío de consultas DNS internas

Para permitir que Unbound consulte en Dnsmasq los nombres registrados mediante DHCP, se configuró:

```text
Services → Unbound DNS → Query Forwarding
```

![Acceso a Query Forwarding](../imagenes/57-acceso-query-forwarding-unbound-opnsense.png)

Inicialmente existía una entrada para el dominio predeterminado `internal`.

![Configuración inicial de Query Forwarding](../imagenes/58-query-forwarding-inicial-unbound-opnsense.png)

La entrada fue modificada para utilizar el dominio del laboratorio:

```text
Dominio: soc.lab
Servidor: 127.0.0.1
Puerto: 53053
```

Esta regla permite resolver nombres internos como:

```text
kali.soc.lab
windows10.soc.lab
windows-server.soc.lab
```

![Reenvío del dominio soc.lab](../imagenes/59-configuracion-dominio-soc-lab-unbound-opnsense.png)

También se configuró una zona de resolución inversa:

```text
Dominio: 126.168.192.in-addr.arpa
Servidor: 127.0.0.1
Puerto: 53053
```

Los octetos se escriben en orden inverso porque las consultas DNS inversas para la red `192.168.126.0/24` utilizan el dominio especial `in-addr.arpa`.

![Configuración de resolución inversa](../imagenes/60-configuracion-resolucion-inversa-unbound-opnsense.png)

La configuración final quedó compuesta por dos entradas:

| Dominio | Servidor | Puerto | Función |
|---|---|---|---|
| `soc.lab` | `127.0.0.1` | `53053` | Resolución de nombre a IP |
| `126.168.192.in-addr.arpa` | `127.0.0.1` | `53053` | Resolución de IP a nombre |

![Query Forwarding configurado](../imagenes/61-query-forwarding-configurado-unbound-opnsense.png)

El flujo DNS funciona de la siguiente manera:

```text
Endpoint
   ↓ Puerto 53
Unbound DNS
   ├── Dominios de Internet → resolución normal
   └── Dominio soc.lab → Dnsmasq en 127.0.0.1:53053
```

---

## Configuración de las reglas del firewall

Se accedió a las reglas de la interfaz LAN desde:

```text
Firewall → Rules → LAN
```

![Acceso a las reglas LAN](../imagenes/62-acceso-reglas-firewall-lan-opnsense.png)

OPNsense creó una regla IPv4 que permite el tráfico desde la red LAN hacia cualquier destino.

| Parámetro | Valor |
|---|---|
| Acción | Pass |
| Protocolo | IPv4 |
| Origen | LAN net |
| Destino | Any |
| Descripción | Default allow LAN to any rule |

![Reglas del firewall LAN](../imagenes/63-reglas-firewall-lan-opnsense.png)

Debido a que el laboratorio utilizará inicialmente IPv4, la regla que permitía tráfico IPv6 desde la LAN fue deshabilitada.

La regla IPv4 permaneció activa.

![Regla IPv6 deshabilitada](../imagenes/64-regla-ipv6-lan-deshabilitada-opnsense.png)

---

## Configuración del NAT saliente

Se accedió al NAT saliente desde:

```text
Firewall → NAT → Outbound
```

![Acceso al NAT saliente](../imagenes/65-acceso-nat-saliente-opnsense.png)

Se mantuvo seleccionado el modo:

```text
Automatic outbound NAT rule generation
```

OPNsense generó automáticamente las reglas necesarias para traducir las direcciones privadas de la LAN utilizando la dirección de la interfaz WAN.

```text
Origen: LAN networks
Interfaz de salida: WAN
Dirección NAT: WAN address
```

![NAT saliente automático](../imagenes/66-nat-saliente-automatico-opnsense.png)

No fue necesario crear reglas NAT manuales.

---

## Pruebas de conectividad desde OPNsense

Se utilizaron las herramientas de diagnóstico desde:

```text
Interfaces → Diagnostics → Ping
```

La primera prueba fue realizada contra la puerta de enlace de VMware:

```text
192.168.92.2
```

En el primer intento se introdujo `WAN` en el campo **Source address**.

![Configuración inicial de la prueba de ping](../imagenes/67-configuracion-prueba-ping-puerta-enlace-opnsense.png)

El campo **Source address** requiere una dirección IP específica o debe permanecer vacío.

Se dejó vacío para que OPNsense seleccionara automáticamente la dirección de origen.

La prueba final obtuvo el siguiente resultado:

```text
Paquetes enviados: 1
Paquetes recibidos: 1
Pérdida: 0 %
Latencia: 0.484 ms
```

![Resultado del ping a la puerta de enlace](../imagenes/68-resultado-ping-puerta-enlace-opnsense.png)

También se realizaron pruebas satisfactorias contra:

```text
1.1.1.1
google.com
```

Esto confirmó:

- Comunicación con la puerta de enlace de VMware.
- Acceso a Internet.
- Funcionamiento de la resolución DNS.

---

## Preparación de las interfaces para Suricata

Antes de habilitar Suricata en modo IPS, se desactivaron las funciones de hardware offloading.

Se configuraron las siguientes opciones:

```text
Disable hardware checksum offload
Disable hardware TCP segmentation offload
Disable hardware large receive offload
Disable VLAN Hardware Filtering
```

Estas funciones pueden interferir con la inspección del tráfico cuando Suricata trabaja en modo IPS.

![Hardware offloading deshabilitado](../imagenes/69-hardware-offloading-deshabilitado-opnsense.png)

---

## Verificación desde Kali Linux

Kali Linux fue conectado a la red virtual `VMnet2`.

El equipo recibió automáticamente la siguiente dirección mediante DHCP:

```text
Interfaz: eth0
Dirección IP: 192.168.126.188/24
Broadcast: 192.168.126.255
Método: DHCP
```

![Dirección IP asignada a Kali por DHCP](../imagenes/70-kali-direccion-ip-asignada-por-dhcp.png)

La dirección recibida se encuentra dentro del rango configurado:

```text
192.168.126.100–192.168.126.200
```

La tabla de rutas de Kali confirmó que OPNsense funciona como puerta de enlace:

```text
default via 192.168.126.254 dev eth0
```

También se verificó la comunicación con:

```text
192.168.126.254
1.1.1.1
google.com
```

Las pruebas confirmaron el funcionamiento de:

- DHCP.
- Puerta de enlace de la LAN.
- NAT saliente.
- Acceso a Internet.
- Resolución DNS.

---

## Problemas encontrados

### Error en el campo Source address

Durante la primera prueba de ping se introdujo el nombre `WAN` en el campo **Source address**.

Ese campo no acepta el nombre de la interfaz. Debe contener una dirección IP configurada en OPNsense o permanecer vacío.

### Solución aplicada

El campo fue dejado vacío para permitir que OPNsense seleccionara automáticamente la dirección de origen.

Después del cambio, la prueba se completó sin pérdida de paquetes.

---

## Verificaciones

- [x] La interfaz WAN recibe una dirección mediante DHCP.
- [x] La puerta de enlace `WAN_DHCP` está activa.
- [x] La interfaz LAN utiliza `192.168.126.254/24`.
- [x] Dnsmasq está habilitado en la LAN.
- [x] El rango DHCP utiliza `192.168.126.100–200`.
- [x] Unbound escucha en el puerto `53`.
- [x] Dnsmasq escucha en el puerto `53053`.
- [x] El dominio interno es `soc.lab`.
- [x] La resolución inversa está configurada.
- [x] La regla IPv4 de la LAN está activa.
- [x] La regla IPv6 de la LAN está deshabilitada.
- [x] El NAT saliente está en modo automático.
- [x] OPNsense tiene acceso a Internet.
- [x] La resolución DNS funciona.
- [x] Kali recibe una dirección mediante DHCP.
- [x] Kali utiliza OPNsense como puerta de enlace.
- [x] El hardware offloading fue deshabilitado.
- [ ] Suricata está configurado en modo IDS.
- [ ] Suricata está configurado en modo IPS.
- [ ] Los eventos de Suricata se envían a Wazuh.
- [ ] Se exportó una copia de seguridad de OPNsense.

## Resultado

La configuración base de OPNsense se completó correctamente.

OPNsense funciona actualmente como:

- Firewall de la red interna.
- Router entre la LAN y la WAN.
- Puerta de enlace para los endpoints.
- Servidor DHCP mediante Dnsmasq.
- Servidor DNS mediante Unbound.
- Sistema NAT para proporcionar acceso a Internet.

La red se encuentra preparada para continuar con la instalación y configuración de Suricata.
