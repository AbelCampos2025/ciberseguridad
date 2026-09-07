# Libro maestro de ciberseguridad

Este documento consolida los conocimientos más importantes del curso. Los ejemplos describen un entorno de laboratorio y no contienen secretos.

## 1. Seguridad de la información

La seguridad de la información protege los datos y los sistemas que los procesan. Sus tres objetivos clásicos son:

- **Confidencialidad:** solo las personas y sistemas autorizados acceden a la información.
- **Integridad:** la información no se altera de forma no autorizada y conserva su exactitud.
- **Disponibilidad:** la información y los servicios están accesibles cuando se necesitan.

Otros principios importantes son autenticidad, trazabilidad y no repudio. La seguridad combina personas, procesos y tecnología; ninguna herramienta sustituye una gestión adecuada del riesgo.

Conceptos base:

- **Activo:** elemento de valor que debe protegerse.
- **Amenaza:** causa potencial de un incidente.
- **Vulnerabilidad:** debilidad que una amenaza puede aprovechar.
- **Riesgo:** combinación de probabilidad e impacto.
- **Control:** medida preventiva, detectiva o correctiva que reduce el riesgo.

## 2. Fundamentos de redes

### 2.1 IPv4 privada

Las direcciones IPv4 privadas se usan dentro de redes internas y no se enrutan directamente por Internet. Los rangos definidos para este propósito son:

- `10.0.0.0/8`
- `172.16.0.0/12`
- `192.168.0.0/16`

Para acceder a Internet, una red privada normalmente utiliza un router que realiza traducción de direcciones (NAT).

### 2.2 Máscara `/24`

El prefijo `/24` equivale a la máscara `255.255.255.0`: 24 bits identifican la red y 8 bits quedan para direcciones dentro de ella.

Para la red de laboratorio `192.168.1.0/24`:

- Dirección de red: `192.168.1.0`
- Hosts utilizables: `192.168.1.1` a `192.168.1.254`
- Broadcast: `192.168.1.255`
- Total: 256 direcciones, de las cuales 254 son asignables tradicionalmente a hosts

Dos equipos con direcciones dentro de esa red pueden comunicarse directamente en la capa local, sujeto a las reglas del sistema y del firewall.

### 2.3 Gateway

El gateway predeterminado del laboratorio es `192.168.1.1`. Un equipo envía al gateway el tráfico cuyo destino no pertenece a su propia subred. El router decide después por dónde reenviarlo.

Una configuración de gateway incorrecta puede permitir comunicación local y, al mismo tiempo, impedir el acceso a otras redes o a Internet.

### 2.4 DHCP

DHCP asigna automáticamente parámetros de red, normalmente:

- Dirección IP
- Máscara o prefijo
- Gateway predeterminado
- Servidores DNS
- Duración de la concesión

El intercambio inicial suele resumirse como DORA: Discover, Offer, Request y Acknowledge. Si DHCP falla, un equipo puede quedar sin configuración válida o autoasignarse una dirección local.

### 2.5 DNS

DNS traduce nombres legibles, como `example.com`, a direcciones IP y publica otros tipos de información. Tener conectividad IP no garantiza que DNS funcione: si se llega a una IP pero no a un nombre, conviene comprobar la resolución de nombres y los servidores DNS configurados.

### 2.6 Dirección MAC

Una dirección MAC identifica una interfaz en la capa de enlace dentro del segmento local. Los equipos usan ARP en IPv4 —y Neighbor Discovery en IPv6— para relacionar direcciones de red con direcciones de enlace.

La MAC no sustituye a la IP: la primera permite la entrega local de tramas y la segunda el direccionamiento y enrutamiento entre redes. Además, una MAC puede cambiarse o falsificarse, por lo que no debe considerarse una prueba fuerte de identidad.

### 2.7 IPv6

IPv6 utiliza direcciones de 128 bits y representación hexadecimal. Tipos relevantes:

- **Global unicast (`2000::/3`):** direccionable globalmente; su alcance es comparable al de una IPv4 pública, aunque el firewall sigue siendo esencial.
- **ULA (`fc00::/7`, normalmente `fd00::/8`):** uso interno; no está pensada para Internet público.
- **Link-local (`fe80::/10`):** existe en el enlace local y no se enruta. Suele requerir indicar la interfaz o zona al utilizarla.

IPv6 no usa broadcast; se apoya en multicast y Neighbor Discovery. Un equipo puede tener simultáneamente varias direcciones IPv6 con alcances distintos.

### 2.8 WSL e Hyper-V

En el entorno observado, `172.24.192.1/20` corresponde a una interfaz virtual asociada a WSL/Hyper-V y actúa como puerta de enlace para una red virtual.

El prefijo `/20` equivale a `255.255.240.0`. Sus bloques avanzan de 16 en el tercer octeto, por lo que:

- Red: `172.24.192.0/20`
- Rango del bloque: `172.24.192.0` a `172.24.207.255`
- Broadcast IPv4: `172.24.207.255`
- Hosts utilizables tradicionales: `172.24.192.1` a `172.24.207.254`

Esta red virtual es distinta de la LAN física `192.168.1.0/24`. Windows puede enrutar y aplicar NAT entre el entorno virtual y otras redes.

### 2.9 Subnetting

Subnetting divide un bloque de direcciones en redes menores. El prefijo indica los bits de red; los restantes identifican hosts.

Procedimiento básico:

1. Convertir el prefijo a máscara.
2. Identificar el octeto donde termina la porción de red.
3. Calcular el tamaño del bloque: `256 - valor del octeto de máscara`.
4. Localizar la dirección de red y la siguiente frontera de subred.
5. Obtener broadcast y rango de hosts.

En IPv4, para una subred tradicional, el número de direcciones es `2^(32 - prefijo)` y el de hosts utilizables suele ser dos menos. Existen excepciones, como enlaces `/31` y rutas de host `/32`.

## 3. Diagnóstico básico de red

Diagnosticar por capas evita conclusiones precipitadas:

1. **Estado local:** comprobar que la interfaz esté activa y tenga IP, prefijo, gateway y DNS esperados.
2. **Pila TCP/IP:** probar loopback (`127.0.0.1` o `::1`).
3. **Red local:** probar la IP propia y luego el gateway.
4. **Enrutamiento:** revisar la tabla de rutas y probar un destino externo por IP.
5. **DNS:** resolver un nombre y comparar el resultado con la prueba por IP.
6. **Ruta:** observar los saltos cuando sea necesario.
7. **Servicio:** comprobar puerto, protocolo, firewall y aplicación destino.

Comandos útiles en Windows/PowerShell:

```powershell
ipconfig /all
Get-NetIPConfiguration
Get-NetRoute
ping 127.0.0.1
ping 192.168.1.1
Resolve-DnsName example.com
Test-NetConnection example.com -Port 443
tracert example.com
arp -a
```

Interpretación rápida:

- Gateway inaccesible: revisar enlace, Wi-Fi/Ethernet, dirección, máscara, VLAN o firewall local.
- IP externa accesible pero nombre no: revisar DNS.
- Nombre resuelve pero el servicio falla: revisar ruta, puerto, firewall, proxy y servicio remoto.
- WSL funciona de forma distinta a Windows: comparar interfaces, rutas, DNS, NAT y reglas de firewall de ambos entornos.

### 3.1 ARP y entrega local

La caché ARP relaciona direcciones IPv4 con direcciones MAC dentro de un enlace local. Las asociaciones aprendidas aparecen como dinámicas; broadcast y multicast suelen aparecer como entradas estáticas definidas por el protocolo.

Al comunicarse con un servidor remoto, la trama Ethernet lleva como destino la MAC del gateway, mientras que el paquete IP conserva como destino la IP del servidor. La MAC cambia en cada segmento; la IP identifica el destino lógico de extremo a extremo, salvo mecanismos como NAT.

ARP no autentica al propietario de una dirección. Esta falta de autenticación permite ataques como ARP spoofing, que solo deben estudiarse en laboratorios aislados y autorizados.

### 3.2 Selección de rutas

Windows selecciona primero la ruta que coincida con el prefijo más específico. La métrica sirve principalmente para decidir entre rutas de especificidad comparable.

Ejemplos del laboratorio:

- `192.168.1.218` coincide con `192.168.1.0/24`: entrega directa por Wi-Fi.
- `172.24.198.76` coincide con `172.24.192.0/20`: entrega directa por Hyper-V.
- `8.8.8.8` no coincide con una red conectada: utiliza `0.0.0.0/0` y el gateway `192.168.1.1`.
- El bloque `172.24.192.0/20` termina en `172.24.207.255`; `172.24.208.1` ya queda fuera.

“En vínculo” significa que el destino está conectado directamente. Windows intenta obtener la MAC del destino mediante ARP y no utiliza un gateway intermedio.

### 3.3 Interpretación de DNS

Una respuesta DNS puede incluir registros `A` para IPv4 y `AAAA` para IPv6. “Respuesta no autoritativa” significa que respondió un resolver recursivo o su caché, no que la consulta haya fallado. Si `nslookup` muestra el servidor como `Unknown`, normalmente falta una resolución inversa para el servidor DNS; esto tampoco implica un fallo de resolución.

### 3.4 ICMP, TTL y traceroute

`ping` usa ICMP Echo Request y Echo Reply para comprobar alcance y medir el tiempo de ida y vuelta. Una respuesta confirma conectividad ICMP, pero la ausencia de respuesta no demuestra por sí sola que el equipo o servicio esté caído: ICMP puede estar filtrado.

El campo TTL evita que un paquete circule indefinidamente. Cada router lo reduce en uno. `tracert` aprovecha este comportamiento enviando pruebas con TTL creciente y observando los mensajes ICMP Time Exceeded. La ruta obtenida es aproximada: puede haber filtrado, balanceo, rutas asimétricas o routers que reenvían tráfico sin responder.

El bloque `100.64.0.0/10` es espacio compartido usado habitualmente para Carrier-Grade NAT (CGNAT). Su presencia inmediatamente después del router doméstico sugiere que el proveedor realiza una traducción IPv4 adicional. Esto puede dificultar conexiones entrantes y la publicación directa de servicios IPv4.

### 3.5 Diagnóstico por capas

Una comprobación ordenada permite aislar fallos:

| Prueba | Qué valida | Limitación |
|---|---|---|
| `ping <gateway>` | Enlace y red local hasta el router | Puede ser bloqueado por firewall |
| `ping <IP externa>` | Enrutamiento hacia Internet sin depender de DNS | No valida nombres ni servicios TCP |
| `ping <nombre>` | Resolución DNS y alcance ICMP | El servidor puede bloquear ICMP |
| `Test-NetConnection <host> -Port 443` | Resolución, ruta y establecimiento TCP al puerto | No valida por completo TLS ni la aplicación HTTP |

Cuando un nombre tiene registros IPv4 e IPv6, el sistema puede preferir IPv6 si dispone de conectividad funcional. Ver una dirección IPv6 en `ping` o `Test-NetConnection` demuestra qué familia se seleccionó para esa prueba concreta.

## 4. Higiene para laboratorios

- Usar entornos aislados y objetivos autorizados.
- Registrar fecha, objetivo, topología, herramientas y resultados.
- Nunca guardar credenciales, tokens, cookies o claves privadas en Git.
- Sanitizar capturas y registros antes de publicarlos.
- Distinguir observaciones, hipótesis, pruebas y conclusiones.
- Anotar cómo revertir cambios y restaurar el entorno.
