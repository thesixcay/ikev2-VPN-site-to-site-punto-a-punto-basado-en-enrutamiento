# VPN Site-to-Site IKEv2 Route-Based (VTI)

## Descripción

Este laboratorio implementa una **VPN Site-to-Site basada en enrutamiento (Route-Based VPN)** utilizando **IKEv2**, **IPsec** y **Virtual Tunnel Interfaces (VTI)** entre dos oficinas conectadas a través de un Router ISP.

A diferencia de una VPN basada en políticas, esta implementación utiliza una **interfaz de túnel virtual (Tunnel0)** protegida por IPsec, permitiendo enrutar el tráfico mediante rutas estáticas o protocolos de enrutamiento dinámico.

---

# Objetivos

- Configurar una VPN Site-to-Site utilizando IKEv2.
- Implementar IPsec mediante Virtual Tunnel Interfaces (VTI).
- Configurar una VPN basada en enrutamiento (Route-Based).
- Proteger el tráfico entre dos redes LAN remotas.
- Verificar el funcionamiento del túnel IPsec.

---

# Topología

```text
                     Router ISP
                23.6.1.9/30
                23.6.1.13/30
                   /        \
                  /          \
                 /            \
          23.6.1.10      23.6.1.14
             Peer1          Peer2
         23.6.1.1/29    23.6.1.17/29
             |               |
          Switch          Switch
             |               |
          VPC P1          VPC P2
        23.6.1.2       23.6.1.18

               Tunnel0 (VTI)
      23.6.1.25/30 <-------> 23.6.1.26/30
```

---

# Direccionamiento IP

| Dispositivo | Interfaz | Dirección IP |
|-------------|-----------|--------------|
| Peer1 | Ethernet0/0 | 23.6.1.1/29 |
| Peer1 | Ethernet0/1 | 23.6.1.10/30 |
| Peer1 | Tunnel0 | 23.6.1.25/30 |
| ISP | Ethernet0/0 | 23.6.1.9/30 |
| ISP | Ethernet0/1 | 23.6.1.13/30 |
| Peer2 | Ethernet0/0 | 23.6.1.14/30 |
| Peer2 | Ethernet0/1 | 23.6.1.17/29 |
| Peer2 | Tunnel0 | 23.6.1.26/30 |
| VPC P1 | NIC | 23.6.1.2/29 |
| VPC P2 | NIC | 23.6.1.18/29 |

---

# Equipos utilizados

- 2 Routers Cisco IOS
- 1 Router ISP
- 2 Switches Cisco IOS Layer 2
- 2 VPCS
- GNS3 o PNETLab

---

# Tecnologías utilizadas

- Cisco IOS
- IKEv2
- IPsec
- Virtual Tunnel Interface (VTI)
- Route-Based VPN
- AES-256
- SHA-256
- Diffie-Hellman Grupo 14
- IPsec Profile
- Rutas estáticas

---

# Configuración realizada

## Limpieza de configuración anterior

Antes de configurar la VPN basada en enrutamiento, se eliminó la configuración de la VPN basada en políticas:

- Crypto Map.
- Proposal IKEv2.
- Policy IKEv2.

Esto evita conflictos entre ambas implementaciones.

---

## Router ISP

El Router ISP únicamente proporciona conectividad entre ambos peers.

Interfaces configuradas:

- Ethernet0/0 → 23.6.1.9/30
- Ethernet0/1 → 23.6.1.13/30

No participa en la negociación de la VPN.

---

## Peer1

Se configuró:

- Interfaces LAN y WAN.
- Ruta por defecto hacia el ISP.
- Proposal IKEv2.
- Policy IKEv2.
- Keyring.
- Perfil IKEv2.
- Transform Set.
- Perfil IPsec.
- Interfaz Tunnel0 (VTI).
- Ruta estática hacia la LAN remota.

### Red LAN

```text
23.6.1.0/29
```

### Dirección del túnel

```text
23.6.1.25/30
```

---

## Peer2

Se configuró:

- Interfaces LAN y WAN.
- Ruta por defecto hacia el ISP.
- Proposal IKEv2.
- Policy IKEv2.
- Keyring.
- Perfil IKEv2.
- Transform Set.
- Perfil IPsec.
- Interfaz Tunnel0 (VTI).
- Ruta estática hacia la LAN remota.

### Red LAN

```text
23.6.1.16/29
```

### Dirección del túnel

```text
23.6.1.26/30
```

---

# Fase 1 - IKEv2

La negociación IKEv2 utiliza los siguientes parámetros:

| Parámetro | Valor |
|-----------|-------|
| Versión | IKEv2 |
| Cifrado | AES-CBC-256 |
| Integridad | SHA-256 |
| Grupo Diffie-Hellman | 14 |
| Autenticación | Pre-Shared Key |

---

# Keyring

Cada router utiliza un Keyring para almacenar la información del peer remoto.

La autenticación utiliza la siguiente clave:

```text
cisco123
```

---

# Perfil IKEv2

El perfil IKEv2 asocia:

- Identidad local.
- Identidad remota.
- Método de autenticación.
- Keyring.
- Asociación con el perfil IPsec.

---

# Fase 2 - IPsec

La protección del túnel utiliza el siguiente Transform Set:

| Parámetro | Valor |
|-----------|-------|
| ESP Encryption | AES-256 |
| ESP Authentication | SHA-256 HMAC |
| Modo | Tunnel |

Posteriormente se crea un **IPsec Profile**, el cual se aplica directamente sobre la interfaz Tunnel0.

---

# Virtual Tunnel Interface (VTI)

Cada router crea una interfaz virtual protegida mediante IPsec.

## Peer1

```text
Tunnel0
23.6.1.25/30
```

## Peer2

```text
Tunnel0
23.6.1.26/30
```

Toda la comunicación entre ambas oficinas viaja a través de esta interfaz.

---

# Enrutamiento

La VPN utiliza rutas estáticas para enviar el tráfico remoto por Tunnel0.

## Peer1

```bash
ip route 23.6.1.16 255.255.255.248 Tunnel0
```

## Peer2

```bash
ip route 23.6.1.0 255.255.255.248 Tunnel0
```

Todo el tráfico destinado a la red remota será enviado automáticamente por el túnel IPsec.

---

# Switches

Los switches funcionan únicamente en Capa 2.

No requieren configuración especial para la VPN.

Solo deben mantener sus interfaces activas.

---

# Configuración de las VPCS

## VPC P1

```bash
ip 23.6.1.2 255.255.255.248 23.6.1.1
save
```

---

## VPC P2

```bash
ip 23.6.1.18 255.255.255.248 23.6.1.17
save
```

---

# Funcionamiento

1. Un equipo de la LAN P1 envía tráfico hacia la LAN P2.
2. El router consulta su tabla de enrutamiento.
3. La ruta estática envía el tráfico por Tunnel0.
4. IKEv2 negocia automáticamente la sesión segura.
5. IPsec cifra el tráfico.
6. Los paquetes atraviesan la red pública.
7. Peer2 descifra el tráfico.
8. Los datos llegan a la LAN remota de forma segura.

---

# Comandos de verificación

## Verificar asociaciones IKEv2

```bash
show crypto ikev2 sa
```

---

## Verificar estadísticas IPsec

```bash
show crypto ipsec sa
```

Comprobar que aumenten los contadores de:

- encaps
- decaps
- encrypt
- decrypt

---

## Verificar la interfaz Tunnel

```bash
show interface tunnel0
```

Debe mostrar:

```text
Tunnel0 is up
Line protocol is up
```

---

## Verificar rutas

```bash
show ip route
```

Las rutas hacia la red remota deben utilizar **Tunnel0**.

---

## Verificar sesiones IPsec

```bash
show crypto session
```

---

## Verificar interfaces

```bash
show ip interface brief
```

---

# Pruebas de conectividad

Desde VPC P1:

```bash
ping 23.6.1.18
```

Desde VPC P2:

```bash
ping 23.6.1.2
```

Si la VPN está correctamente configurada:

- El túnel permanecerá activo.
- El tráfico viajará cifrado.
- Las estadísticas IPsec aumentarán durante las pruebas.

---

# Solución de problemas

## El túnel no levanta

Verificar:

- Direcciones IP WAN.
- Proposal IKEv2.
- Policy IKEv2.
- Keyring.
- Perfil IKEv2.
- Perfil IPsec.
- Tunnel Source.
- Tunnel Destination.
- Ruta por defecto hacia el ISP.
- Estado de las interfaces físicas.

---

## No existe comunicación

Revisar:

```bash
show crypto ikev2 sa
show crypto ipsec sa
show crypto session
show interface tunnel0
show ip route
```

También verificar que las rutas estáticas apunten correctamente a **Tunnel0**.

---

# Ventajas de una VPN basada en enrutamiento

- Configuración más sencilla que una VPN basada en políticas.
- No requiere ACL para definir tráfico interesante.
- Compatible con protocolos de enrutamiento dinámico como OSPF, EIGRP o BGP.
- Mayor escalabilidad para redes empresariales.
- Administración más simple mediante interfaces de túnel.

---

# Seguridad implementada

- Autenticación mediante Pre-Shared Key.
- Negociación segura utilizando IKEv2.
- Cifrado AES-256.
- Integridad mediante SHA-256.
- Intercambio de claves mediante Diffie-Hellman Grupo 14.
- Protección del tráfico mediante IPsec sobre una Virtual Tunnel Interface (VTI).

---

# Resultado esperado

Al finalizar este laboratorio:

- La VPN Site-to-Site basada en enrutamiento debe establecerse correctamente.
- La interfaz **Tunnel0** debe encontrarse activa en ambos routers.
- Las asociaciones IKEv2 e IPsec deben permanecer establecidas.
- Las redes **23.6.1.0/29** y **23.6.1.16/29** deben comunicarse de forma segura.
- Todo el tráfico entre ambas oficinas debe viajar cifrado a través del túnel VTI.

---

