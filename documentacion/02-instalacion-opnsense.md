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

Las capturas serán almacenadas en:

```text
imagenes/opnsense/
```

Ejemplos:

```text
imagenes/opnsense/01-creacion-maquina.png
imagenes/opnsense/02-adaptadores-red.png
imagenes/opnsense/03-instalacion.png
imagenes/opnsense/04-configuracion-lan.png
imagenes/opnsense/05-panel-web.png
```

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
