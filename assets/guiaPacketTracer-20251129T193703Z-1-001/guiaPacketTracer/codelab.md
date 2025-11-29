summary: Guía de Configuración de Red
id: guia-configuracion-red
categories: guia
status: Published
authors: Sergio González Noria, Iván Paúl Alba, Manuel Pérez Romero, Javier Calvillo Acebedo
feedback link: https://github.com/IES-Rafael-Alberti/25-26-Ciberseguridad-Grupo-5

# Guía de Configuración de Red

## Introducción

Este documento describe la implementación de una red corporativa segmentada mediante VLANs, diseñada en Cisco Packet Tracer. La topología incluye tres routers interconectados en una arquitectura de red WAN, con dos switches de acceso que proporcionan conectividad a dos ubicaciones (A y B).

La red está segmentada en cuatro VLANs diferenciadas por tipo de usuario (empleados y clientes) y ubicación física, implementando medidas de seguridad como ACLs y port-security.

## Diseño y planificación de la segmentación (VLANs y VLSM)

### Esquema de direccionamiento VLSM

La red utiliza el espacio de direcciones 10.10.0.0/16 con la siguiente segmentación:

| VLAN | Nombre | Red | Máscara | Hosts disponibles | Ubicación |
|------|--------|-----|---------|-------------------|-----------|
| VLAN 10 | EMPLEADOS_A | 10.10.0.0/23 | 255.255.254.0 | 510 | Sede A |
| VLAN 20 | CLIENTES_A | 10.10.4.0/24 | 255.255.255.0 | 254 | Sede A |
| VLAN 11 | EMPLEADOS_B | 10.10.2.0/23 | 255.255.254.0 | 510 | Sede B |
| VLAN 21 | CLIENTES_B | 10.10.5.0/24 | 255.255.255.0 | 254 | Sede B |

### Redes WAN

Las conexiones entre routers utilizan subredes /30 para optimizar el uso de direcciones:

- **Router0 ↔ Router1**: 10.10.6.0/30
  - Router0: 10.10.6.1
  - Router1: 10.10.6.2
  
- **Router0 ↔ Router2**: 10.10.6.4/30
  - Router0: 10.10.6.5
  - Router2: 10.10.6.6

### Justificación del diseño

Hemos decidido separar la red en VLANs principalmente por seguridad. La idea es que los empleados y los clientes estén en redes diferentes para que los visitantes no puedan acceder a los recursos internos de la empresa. Básicamente, si alguien se conecta como cliente, no debería poder ver ni acceder a los equipos o servidores que usan los trabajadores.

También hemos separado las redes por ubicación (Sede A y Sede B) porque así es más fácil gestionar cada sitio de forma independiente. Si hay algún problema en una sede, no afecta a la otra, y además podemos aplicar reglas de seguridad diferentes si cada ubicación lo necesita. Esto también ayuda a que la red vaya más fluida, porque no hay tanto tráfico innecesario entre sedes.

**Por qué hemos usado estas redes:**

- **VLAN 10 y 11 (Empleados)**: Hemos usado una red /23 que da para 510 ordenadores, así tenemos margen de sobra si la empresa crece y contrata más gente
- **VLAN 20 y 21 (Clientes)**: Aquí con una red /24 que da para 254 dispositivos es suficiente, porque normalmente no hay tantos clientes conectados al mismo tiempo
- **Separación por ubicación**: Cada sede tiene sus propias VLANs, así si necesitamos cambiar algo en una sede no tocamos la otra
- **Segmentación de tráfico**: Al separar empleados de clientes evitamos que se vea tráfico entre ellos, mejorando tanto la seguridad como el rendimiento
- **Enlaces entre routers**: Para conectar los routers hemos usado redes /30 que solo dan 2 IPs útiles, justo lo que necesitamos, sin desperdiciar direcciones

## Configuración de los switches (VLANs, trunks, port-security)

### Switch0 (Sede A)

#### Creación de VLANs

```cisco
Switch0>enable
Switch0#configure terminal
Switch0(config)#vlan 10
Switch0(config-vlan)#name EMPLEADOS_A
Switch0(config-vlan)#exit
Switch0(config)#vlan 20
Switch0(config-vlan)#name CLIENTES_A
Switch0(config-vlan)#exit
```

#### Configuración de puertos de acceso

**Puertos para VLAN 10 (Empleados) - FastEthernet 0/1-10:**
```cisco
Switch0(config)#interface range fastEthernet 0/1-10
Switch0(config-if-range)#switchport mode access
Switch0(config-if-range)#switchport access vlan 10
Switch0(config-if-range)#exit
```

**Puertos para VLAN 20 (Clientes) - FastEthernet 0/11-20:**
```cisco
Switch0(config)#interface range fastEthernet 0/11-20
Switch0(config-if-range)#switchport mode access
Switch0(config-if-range)#switchport access vlan 20
Switch0(config-if-range)#exit
```

![alt text](./vlanbrief.png)

#### Configuración de puerto trunk

```cisco
Switch0(config)#interface gigabitEthernet 0/1
Switch0(config-if)#switchport mode trunk
Switch0(config-if)#exit
```

![alt text](./trunk.png)

#### Port-security

Hemos configurado port-security en todos los puertos de acceso para permitir solo 1 MAC address por puerto, con modo de violación shutdown:

```cisco
Switch0(config)#interface range fastEthernet 0/1-20
Switch0(config-if-range)#switchport port-security
Switch0(config-if-range)#switchport port-security maximum 1
Switch0(config-if-range)#exit
```

![alt text](./port-security.png)

**Verificación de VLANs:**
```cisco
Switch0#show vlan brief
```

![alt text](./vlanbrief.png)

**Verificación de Port-security:**
```cisco
Switch0#show port-security
```

![alt text](./port-security.png)

### Switch1 (Sede B)

#### Creación de VLANs

```cisco
Switch1>enable
Switch1#configure terminal
Switch1(config)#vlan 11
Switch1(config-vlan)#name EMPLEADOS_B
Switch1(config-vlan)#exit
Switch1(config)#vlan 21
Switch1(config-vlan)#name CLIENTES_B
Switch1(config-vlan)#exit
```

#### Configuración de puertos de acceso

**Puertos para VLAN 11 (Empleados) - FastEthernet 0/1-10:**
```cisco
Switch1(config)#interface range fastEthernet 0/1-10
Switch1(config-if-range)#switchport mode access
Switch1(config-if-range)#switchport access vlan 11
Switch1(config-if-range)#exit
```

**Puertos para VLAN 21 (Clientes) - FastEthernet 0/11-20:**
```cisco
Switch1(config)#interface range fastEthernet 0/11-20
Switch1(config-if-range)#switchport mode access
Switch1(config-if-range)#switchport access vlan 21
Switch1(config-if-range)#exit
```

![alt text](./vlanbrief2.png)

#### Configuración de puerto trunk

```cisco
Switch1(config)#interface gigabitEthernet 0/1
Switch1(config-if)#switchport mode trunk
Switch1(config-if)#exit
```

![alt text](./trunk2.png)

#### Port-security

Igual que en Switch0, hemos configurado port-security en todos los puertos de acceso:

```cisco
Switch1(config)#interface range fastEthernet 0/1-20
Switch1(config-if-range)#switchport port-security
Switch1(config-if-range)#switchport port-security maximum 1
Switch1(config-if-range)#exit
```

![alt text](./port-security2.png)

## Configuración de los routers (router-on-a-stick, ACLs)

### Router0 (Intermediario)

Este router actúa como intermediario entre las dos sedes, enrutando el tráfico entre Router1 y Router2.

#### Configuración de interfaces WAN

```cisco
Router0>enable
Router0#configure terminal

Router0(config)#interface gigabitEthernet 0/0/0
Router0(config-if)#ip address 10.10.6.1 255.255.255.252
Router0(config-if)#no shutdown
Router0(config-if)#exit

Router0(config)#interface gigabitEthernet 0/0/1
Router0(config-if)#ip address 10.10.6.5 255.255.255.252
Router0(config-if)#no shutdown
Router0(config-if)#exit
```

#### Enrutamiento estático

```cisco
Router0(config)#ip route 10.10.0.0 255.255.254.0 10.10.6.2
Router0(config)#ip route 10.10.2.0 255.255.254.0 10.10.6.6
```

**Verificación de interfaces:**
```cisco
Router0#show ip interface brief
```

![alt text](router01.png)

**Verificación de rutas:**
```cisco
Router0#show ip route
```

![alt text](router02.png)

### Router1 (Sede A) - Router-on-a-stick

Este router gestiona las VLANs de la Sede A mediante subinterfaces y se conecta al Router0.

#### Configuración WAN (hacia Router0)

```cisco
Router1>enable
Router1#configure terminal

Router1(config)#interface gigabitEthernet 0/0/0
Router1(config-if)#ip address 10.10.6.2 255.255.255.252
Router1(config-if)#no shutdown
Router1(config-if)#exit
```

#### Configuración de subinterfaces (router-on-a-stick)

```cisco
Router1(config)#interface gigabitEthernet 0/0/1
Router1(config-if)#no shutdown
Router1(config-if)#exit

Router1(config)#interface gigabitEthernet 0/0/1.10
Router1(config-subif)#encapsulation dot1Q 10
Router1(config-subif)#ip address 10.10.0.1 255.255.254.0
Router1(config-subif)#exit

Router1(config)#interface gigabitEthernet 0/0/1.20
Router1(config-subif)#encapsulation dot1Q 20
Router1(config-subif)#ip address 10.10.4.1 255.255.255.0
Router1(config-subif)#exit
```

#### Enrutamiento estático

```cisco
Router1(config)#ip route 10.10.2.0 255.255.254.0 10.10.6.1
```

**Verificación de interfaces:**
```cisco
Router1#show ip interface brief
```

![alt text](router11.png)

**Verificación de rutas:**
```cisco
Router1#show ip route
```

![alt text](router12.png)

#### Configuración de ACLs

**ACL 101 para VLAN 20 (Clientes_A) - Bloquea acceso a redes de empleados:**
```cisco
Router1(config)#access-list 101 deny ip 10.10.4.0 0.0.0.255 10.10.0.0 0.0.1.255
Router1(config)#access-list 101 deny ip 10.10.4.0 0.0.0.255 10.10.2.0 0.0.1.255
Router1(config)#access-list 101 permit ip any any
Router1(config)#access-list 101 deny ip 10.10.4.0 0.0.0.255 10.10.5.0 0.0.0.255
Router1(config)#interface gigabitEthernet 0/0/1.20
Router1(config-subif)#ip access-group 101 in
Router1(config-subif)#exit
```

**ACL 102 para VLAN 10 (Empleados_A) - Bloquea acceso a redes de clientes:**
```cisco
Router1(config)#access-list 102 deny ip 10.10.0.0 0.0.1.255 10.10.4.0 0.0.0.255
Router1(config)#access-list 102 deny ip 10.10.0.0 0.0.1.255 10.10.5.0 0.0.0.255
Router1(config)#access-list 102 permit ip any any
Router1(config)#interface gigabitEthernet 0/0/1.10
Router1(config-subif)#ip access-group 102 in
Router1(config-subif)#exit
```

**Verificación de ACLs:**
```cisco
Router1#show access-lists
```

![alt text](router13.png)

### Router2 (Sede B) - Router-on-a-stick

Este router gestiona las VLANs de la Sede B mediante subinterfaces y se conecta al Router0.

#### Configuración WAN (hacia Router0)

```cisco
Router2>enable
Router2#configure terminal

Router2(config)#interface gigabitEthernet 0/0/0
Router2(config-if)#ip address 10.10.6.6 255.255.255.252
Router2(config-if)#no shutdown
Router2(config-if)#exit
```

#### Configuración de subinterfaces (router-on-a-stick)

```cisco
Router2(config)#interface gigabitEthernet 0/0/1
Router2(config-if)#no shutdown
Router2(config-if)#exit

Router2(config)#interface gigabitEthernet 0/0/1.11
Router2(config-subif)#encapsulation dot1Q 11
Router2(config-subif)#ip address 10.10.2.1 255.255.254.0
Router2(config-subif)#exit

Router2(config)#interface gigabitEthernet 0/0/1.21
Router2(config-subif)#encapsulation dot1Q 21
Router2(config-subif)#ip address 10.10.5.1 255.255.255.0
Router2(config-subif)#exit
```

#### Enrutamiento estático

```cisco
Router2(config)#ip route 10.10.0.0 255.255.254.0 10.10.6.5
```

**Verificación de interfaces:**
```cisco
Router2#show ip interface brief
```

![alt text](router21.png)

**Verificación de rutas:**
```cisco
Router2#show ip route
```

![alt text](router22.png)

#### Configuración de ACLs

**ACL 101 para VLAN 21 (Clientes_B) - Bloquea acceso a redes de empleados:**
```cisco
Router2(config)#access-list 101 deny ip 10.10.5.0 0.0.0.255 10.10.2.0 0.0.1.255
Router2(config)#access-list 101 deny ip 10.10.5.0 0.0.0.255 10.10.0.0 0.0.1.255
Router2(config)#access-list 101 permit ip any any
Router2(config)#access-list 101 deny ip 10.10.5.0 0.0.0.255 10.10.4.0 0.0.0.255
Router2(config)#interface gigabitEthernet 0/0/1.21
Router2(config-subif)#ip access-group 101 in
Router2(config-subif)#exit
```

**ACL 102 para VLAN 11 (Empleados_B) - Bloquea acceso a redes de clientes:**
```cisco
Router2(config)#access-list 102 deny ip 10.10.2.0 0.0.1.255 10.10.5.0 0.0.0.255
Router2(config)#access-list 102 deny ip 10.10.2.0 0.0.1.255 10.10.4.0 0.0.0.255
Router2(config)#access-list 102 permit ip any any
Router2(config)#interface gigabitEthernet 0/0/1.11
Router2(config-subif)#ip access-group 102 in
Router2(config-subif)#exit
```

**Verificación de ACLs:**
```cisco
Router2#show access-lists
```

![alt text](router23.png)

### Explicación de las ACLs

Las ACLs implementadas siguen una política de seguridad clara:

**Para las VLANs de Clientes (20 y 21):**
- Bloquean el acceso a todas las redes de empleados (VLAN 10 y 11)
- Bloquean el acceso entre sí (clientes de Sede A no acceden a clientes de Sede B y viceversa)
- Permiten el resto del tráfico (acceso a Internet/otros recursos)

**Para las VLANs de Empleados (10 y 11):**
- Bloquean el acceso a las redes de clientes (VLAN 20 y 21)
- Permiten el acceso entre empleados de diferentes sedes
- Permiten el resto del tráfico

Esta configuración asegura que los clientes estén completamente aislados de los recursos corporativos y entre sí, mientras que los empleados pueden comunicarse entre sedes pero no acceder a las redes de clientes.

## Configuración de los hosts

### Sede A - VLAN 10 (Empleados)

**PC-PT PC0:**
- IP: 10.10.0.2
- Máscara: 255.255.254.0
- Gateway: 10.10.0.1

![alt text](pc1.png)

**PC-PT PC1:**
- IP: 10.10.1.254
- Máscara: 255.255.254.0
- Gateway: 10.10.0.1

![alt text](pc2.png)

### Sede A - VLAN 20 (Clientes)

**PC-PT PC2:**
- IP: 10.10.4.2
- Máscara: 255.255.255.0
- Gateway: 10.10.4.1

![alt text](pc3.png)

**PC-PT PC3:**
- IP: 10.10.4.254
- Máscara: 255.255.255.0
- Gateway: 10.10.4.1

![alt text](pc4.png)

### Sede B - VLAN 11 (Empleados)

**PC-PT PC4:**
- IP: 10.10.2.2
- Máscara: 255.255.254.0
- Gateway: 10.10.2.1

![alt text](pc5.png)

**PC-PT PC5:**
- IP: 10.10.3.254
- Máscara: 255.255.254.0
- Gateway: 10.10.2.1

![alt text](pc6.png)

### Sede B - VLAN 21 (Clientes)

**PC-PT PC6:**
- IP: 10.10.5.2
- Máscara: 255.255.255.0
- Gateway: 10.10.5.1

![alt text](pc7.png)

**PC-PT PC7:**
- IP: 10.10.5.254
- Máscara: 255.255.255.0
- Gateway: 10.10.5.1

![alt text](pc8.png)

## Medidas de seguridad aplicadas

En nuestra red hemos implementado varias capas de seguridad para proteger tanto los datos como los dispositivos. La idea es que no solo haya una barrera, sino varias, para que si alguien consigue pasar una, todavía tenga que enfrentarse a otras.

### 1. Segmentación mediante VLANs

La primera medida de seguridad es la separación en VLANs. Básicamente hemos dividido la red en 4 segmentos diferentes:

- **Separación empleados-clientes**: Los empleados (VLAN 10 y 11) están completamente separados de los clientes (VLAN 20 y 21). Esto evita que un visitante pueda ver o acceder a equipos internos de la empresa
- **Separación por sedes**: Cada ubicación tiene sus propias VLANs, así si hay algún problema en una sede, la otra no se ve afectada
- **Reducción de tráfico innecesario**: Al dividir en VLANs, cada una tiene su propio dominio de broadcast, lo que significa menos tráfico molesto en la red

Esta separación es la base de todo el sistema de seguridad, porque si alguien se conecta a una VLAN, automáticamente ya está limitado a lo que puede ver y hacer.

### 2. Port-security en los switches

Hemos activado port-security en todos los puertos de los switches (FastEthernet 0/1-20) para evitar que alguien conecte dispositivos no autorizados:

- **Máximo 1 dirección MAC por puerto**: Cada puerto solo permite un dispositivo conectado
- **Modo de violación shutdown**: Si alguien intenta conectar más dispositivos o cambiar el equipo, el puerto se apaga automáticamente
- **Protección contra ataques**: Esto nos protege de ataques como MAC flooding o MAC spoofing, donde alguien intenta saturar el switch con direcciones MAC falsas

Esta medida es muy efectiva porque si un atacante intenta conectarse a un puerto donde ya hay un PC, el puerto se desactiva y nos damos cuenta inmediatamente de que algo raro está pasando.

### 3. Access Control Lists (ACLs)

Las ACLs son listas de reglas que controlan qué tráfico puede pasar y cuál no. Las hemos configurado en los routers 1 y 2 para controlar el tráfico entre VLANs:

**ACL 101 - Para VLANs de clientes (VLAN 20 y 21):**
- Bloquean el acceso a las redes de empleados (VLAN 10 y 11)
- Bloquean el acceso entre clientes de diferentes sedes
- Permiten el resto del tráfico

**ACL 102 - Para VLANs de empleados (VLAN 10 y 11):**
- Bloquean el acceso a las redes de clientes (VLAN 20 y 21)
- Permiten la comunicación entre empleados de diferentes sedes
- Permiten el resto del tráfico

**¿Por qué estas reglas?**
La idea es que los clientes solo puedan navegar por internet o usar servicios específicos, pero nunca acceder a los recursos internos de la empresa. Los empleados también tienen restricciones para que no puedan acceder a las redes de clientes, evitando posibles problemas de seguridad.

### 4. Router-on-a-stick

Esta técnica nos permite controlar todo el tráfico entre VLANs desde un solo punto (el router). Las ventajas de seguridad son:

- **Punto de control centralizado**: Todo el tráfico entre VLANs pasa por el router, donde aplicamos las ACLs
- **Más fácil de gestionar**: En lugar de tener que configurar seguridad en varios sitios, lo hacemos todo en el router
- **Visibilidad total**: Podemos ver y controlar todo el tráfico que pasa entre las diferentes VLANs

Básicamente, si las VLANs son como habitaciones separadas, el router es la única puerta que comunica esas habitaciones, y ahí es donde controlamos quién puede pasar y quién no.

### 5. Enrutamiento estático

Hemos configurado las rutas entre los routers de forma manual (estática) en lugar de usar protocolos dinámicos:

- **Rutas específicas**: Cada router solo conoce las rutas exactas que necesita
- **Sin protocolos dinámicos vulnerables**: Evitamos usar protocolos como RIP u OSPF que podrían ser explotados por atacantes
- **Mayor control**: Sabemos exactamente por dónde pasa cada paquete

El Router0 tiene rutas hacia las redes de ambas sedes (10.10.0.0/23 y 10.10.2.0/23), mientras que Router1 y Router2 tienen rutas hacia la otra sede pasando siempre por Router0.

### 6. Diseño de red WAN con subredes /30

Para conectar los routers entre sí hemos usado redes /30, que son las más pequeñas posibles (solo 2 IPs útiles):

- **Red 10.10.6.0/30**: Conecta Router0 con Router1
- **Red 10.10.6.4/30**: Conecta Router0 con Router2

Esto no solo optimiza el uso de direcciones IP, sino que también reduce la superficie de ataque, porque solo hay dos dispositivos por enlace y nada más puede conectarse ahí.

### 7. Separación física de sedes

Aunque sea obvio, el hecho de tener las sedes físicamente separadas y conectadas solo a través del Router0 también es una medida de seguridad:

- Si hay un problema en una sede (ataque, virus, etc.), está contenido en esa sede
- El Router0 actúa como punto de filtrado entre sedes
- Cada sede puede tener sus propias políticas de seguridad adicionales

### Resumen de la estrategia de seguridad

Nuestra estrategia sigue el principio de "defensa en profundidad": múltiples capas de seguridad que se complementan entre sí. Si un atacante consigue conectarse a la red (lo cual ya es difícil por el port-security), aún así está limitado por su VLAN. Si intenta saltar a otra VLAN, las ACLs del router lo bloquean. Y si intenta atacar otra sede, tiene que pasar por el Router0 con todas sus reglas.

Todo esto hace que la red sea mucho más segura que si solo tuviéramos una red plana donde todos pueden ver a todos.

## Gemelo digital en Packet Tracer

### ¿Qué hemos montado?

Todo el proyecto está en el archivo `Proyecto2.pkt`. Básicamente es una simulación completa de la red que funcionaría en la vida real, pero hecha en Packet Tracer para poder probarla sin necesidad de tener todo el hardware físico.

### Los equipos que hemos usado

**Routers:**
- 3 routers ISR4331 (Router0, Router1 y Router2)
- Router0 hace de intermediario entre las dos sedes
- Router1 y Router2 gestionan las VLANs de cada sede con router-on-a-stick

**Switches:**
- 2 switches Cisco 2960-24TT
- Switch0 en la Sede A con las VLANs 10 y 20
- Switch1 en la Sede B con las VLANs 11 y 21

**Ordenadores:**
- 8 PCs en total (2 por cada VLAN)
- PC0 y PC1 son empleados de la Sede A
- PC2 y PC3 son clientes de la Sede A
- PC4 y PC5 son empleados de la Sede B
- PC6 y PC7 son clientes de la Sede B

### Lo que hemos configurado

**VLANs:**
- VLAN 10 (EMPLEADOS_A): Puertos Fa0/1-10 del Switch0
- VLAN 20 (CLIENTES_A): Puertos Fa0/11-20 del Switch0
- VLAN 11 (EMPLEADOS_B): Puertos Fa0/1-10 del Switch1
- VLAN 21 (CLIENTES_B): Puertos Fa0/11-20 del Switch1

**Enlaces trunk:**
- Los puertos GigabitEthernet 0/1 de ambos switches están en modo trunk para llevar todas las VLANs hasta los routers

**Subinterfaces en los routers:**
- Router1 tiene las subinterfaces 0/0/1.10 y 0/0/1.20 para las VLANs de la Sede A
- Router2 tiene las subinterfaces 0/0/1.11 y 0/0/1.21 para las VLANs de la Sede B

**Rutas estáticas:**
- Router0 sabe cómo llegar a las redes de las dos sedes
- Router1 sabe cómo llegar a las redes de la Sede B pasando por Router0
- Router2 sabe cómo llegar a las redes de la Sede A pasando por Router0

**ACLs de seguridad:**
- ACL 101 en cada router para bloquear a los clientes el acceso a las redes de empleados
- ACL 102 en cada router para bloquear a los empleados el acceso a las redes de clientes

**Port-security:**
- Todos los puertos de acceso (Fa0/1-20) solo permiten 1 MAC por puerto
- Si intentas conectar otro dispositivo, el puerto se apaga automáticamente

### Pruebas que hemos hecho

Hemos probado la red haciendo pings entre diferentes PCs para comprobar que las ACLs funcionan bien:

**Conexiones que SÍ funcionan:**
- ✅ **PC0 → PC1**: Dos empleados de la misma sede se pueden comunicar sin problemas
- ✅ **PC0 → PC4**: Los empleados de diferentes sedes pueden hablarse entre sí (pasa por Router0)
- ✅ **PC2 → PC3**: Los clientes de la misma sede también pueden comunicarse
- ✅ **Cualquier PC → Router0**: Todos los dispositivos pueden acceder al router intermediario

![alt text](comprobacion1.png)

**Conexiones que NO funcionan (bloqueadas por las ACLs):**
- ❌ **PC2 → PC0**: Un cliente no puede hacer ping a un empleado de su misma sede
- ❌ **PC2 → PC4**: Un cliente de la Sede A no puede acceder a empleados de la Sede B
- ❌ **PC6 → PC4**: Un cliente de la Sede B no puede acceder a empleados de la Sede B
- ❌ **PC0 → PC2**: Un empleado no puede acceder a la red de clientes (también bloqueado)
- ❌ **PC2 → PC6**: Los clientes de diferentes sedes no pueden comunicarse entre sí

Estas pruebas confirman que las ACLs están funcionando correctamente y que tenemos la seguridad que queríamos.

![alt text](comprobacion2.png)

### ¿Por qué es útil tener esto en Packet Tracer?

La verdad es que hacer esto en Packet Tracer antes de montarlo con equipos reales tiene muchas ventajas:

1. **Podemos probar sin romper nada**: Si nos equivocamos en alguna configuración, simplemente lo reseteamos y ya está, sin haber tocado ningún equipo físico
2. **Es más barato**: No necesitamos comprar 3 routers, 2 switches y 8 ordenadores para hacer pruebas
3. **Aprendemos mejor**: Podemos experimentar, ver qué pasa si cambiamos cosas, y entender cómo funciona todo sin presión
4. **Documentación visual**: Es más fácil explicarle a alguien cómo funciona la red si puede ver el esquema en Packet Tracer
5. **Detectar errores antes**: Si algo no funciona en la simulación, sabemos que tampoco funcionará en la realidad, así que lo arreglamos aquí primero

Básicamente, este gemelo digital nos sirve como banco de pruebas para validar que todo está bien antes de implementarlo de verdad.

## Conclusiones

### Lo que hemos conseguido

Después de todo el trabajo, hemos conseguido montar una red que cumple con todos los objetivos de seguridad que nos planteamos al principio. La red está dividida en 4 VLANs diferentes (dos para empleados y dos para clientes, una de cada en cada sede), y gracias a las ACLs los clientes no pueden acceder a las redes de empleados ni tampoco pueden comunicarse entre sedes.

**Los puntos clave que hemos logrado:**

1. **Separación efectiva de empleados y clientes**: Hemos conseguido que los visitantes estén completamente aislados de los recursos internos de la empresa. Un cliente puede conectarse a la red para usar internet, pero no puede ver ni acceder a ningún ordenador o servidor de empleados.

2. **Comunicación entre sedes**: Los empleados de ambas sedes pueden trabajar juntos sin problemas, compartir archivos y comunicarse, todo pasando por el Router0 que hace de intermediario.

3. **Seguridad en varias capas**: No nos hemos conformado con solo poner VLANs. Tenemos VLANs + ACLs + port-security, así que hay múltiples barreras que un atacante tendría que superar.

4. **Red escalable**: Si la empresa crece, podemos añadir más PCs fácilmente porque hemos dejado espacio de sobra en las redes (510 hosts para empleados y 254 para clientes en cada sede).

5. **Todo documentado y probado**: Tenemos el proyecto en Packet Tracer con todas las configuraciones documentadas, así que si alguien necesita replicarlo o hacer cambios, sabe exactamente qué hacer.

### Lo bueno del diseño

**Por qué nos gusta esta configuración:**

- **Cada cosa en su sitio**: Las VLANs mantienen todo organizado. Sabes que del puerto 1 al 10 son empleados y del 11 al 20 son clientes. Simple y claro.

- **Fácil de mantener**: Si hay un problema en la Sede A, solo tenemos que revisar el Switch0 y el Router1. No hay que tocar nada de la Sede B. Cada cosa está separada.

- **Router-on-a-stick funciona genial**: Solo necesitamos un cable entre el switch y el router para gestionar todas las VLANs. No hace falta tener un cable por cada VLAN.

- **Port-security nos protege**: Si alguien intenta conectar un dispositivo raro o hacer algo sospechoso, el puerto se apaga solo. Es como tener un guardia de seguridad en cada puerto.

- **Control total del tráfico**: Como todo pasa por los routers, podemos ver exactamente qué está pasando y aplicar las reglas de seguridad que necesitemos.

### Cosas que se podrían mejorar

Aunque la red funciona bien, hay algunas cosas que reconocemos que podrían ser mejores:

**Limitaciones que tiene ahora:**

- **Sin redundancia**: Si se cae Router0, las dos sedes se quedan desconectadas entre sí. Deberíamos tener un enlace de respaldo.

- **IPs estáticas**: Hemos configurado todas las IPs a mano. Si tuviéramos que conectar 50 PCs nuevos, sería un rollo tener que configurar cada uno. Un servidor DHCP nos vendría bien.

- **Enrutamiento estático**: Si cambia algo en la red, tenemos que reconfigurar las rutas manualmente en todos los routers. Con protocolos dinámicos como OSPF esto sería automático.

- **Un solo router por sede**: Si se estropea Router1, toda la Sede A se queda sin acceso a la otra sede. Tener dos routers sería más seguro.

**Cosas que añadiríamos si tuviéramos más tiempo:**

1. **Servidor DHCP**: Para que los PCs obtengan la IP automáticamente. Así es mucho más cómodo cuando conectas equipos nuevos.

2. **Más routers y enlaces de backup**: Para que si falla uno, la red siga funcionando. Esto se llama redundancia y es súper importante en redes profesionales.

3. **Monitorización**: Algún sistema que nos avise si hay problemas en la red, como tráfico raro o puertos que se caen.

4. **Autenticación de usuarios**: Algo como 802.1X para que antes de conectarse a la red tengas que poner usuario y contraseña. Ahora mismo solo comprobamos la MAC del dispositivo.

5. **Priorización de tráfico (QoS)**: Para que si hay llamadas por internet o videoconferencias, ese tráfico vaya antes que el resto.

### Para qué serviría esto en la vida real

Este diseño que hemos hecho sería perfecto para:

- **Empresas pequeñas o medianas con varias oficinas**: Que necesitan conectar dos o tres sedes y mantener seguridad entre empleados y visitantes.

- **Hoteles o centros comerciales**: Donde hay WiFi para clientes pero los empleados necesitan su propia red privada para trabajar.

- **Colegios o institutos**: Donde los alumnos tienen acceso limitado a internet pero los profesores necesitan acceder a sistemas internos.

- **Oficinas con zonas de coworking**: Donde trabaja gente de fuera que necesita internet pero no puede acceder a los servidores de la empresa.

Básicamente, cualquier sitio donde haya que separar claramente quién puede acceder a qué recursos.

### Reflexión final

Lo que más hemos aprendido con este proyecto es que la seguridad no es solo poner una contraseña WiFi. Hay que pensar en capas: VLANs para separar, ACLs para filtrar, port-security para controlar los dispositivos... Y todo tiene que estar bien configurado y probado.

También hemos visto lo útil que es Packet Tracer para hacer pruebas. Poder simular toda la red, hacer pings, probar que las ACLs bloquean lo que deben bloquear, y ver cómo funciona todo sin necesidad de tener el hardware físico es muy práctico.

En resumen, hemos montado una red segura, funcional y escalable que separa correctamente los diferentes tipos de usuarios. Las ACLs funcionan, el port-security protege, las VLANs organizan, y todo está documentado para que se pueda replicar o modificar fácilmente. Estamos contentos con el resultado y creemos que cumple con los objetivos de seguridad que nos planteamos al inicio del proyecto.