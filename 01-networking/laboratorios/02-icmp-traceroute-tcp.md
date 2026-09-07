# Laboratorio 02 — ICMP, traceroute y conectividad TCP

## Objetivo

Validar de forma separada la red local, el enrutamiento IPv4, la resolución DNS, la conectividad IPv6 y el acceso a un servicio TCP.

## Entorno sanitizado

- Equipo Windows conectado mediante Wi-Fi.
- LAN de laboratorio: `192.168.1.0/24`.
- Gateway: `192.168.1.1`.
- Las direcciones IPv6 del equipo y los saltos públicos del proveedor se omitieron.

## Comandos

```powershell
ping 192.168.1.1
ping 1.1.1.1
ping example.com
Test-NetConnection example.com -Port 443
tracert -d 1.1.1.1
```

## Resultados

| Capa o función | Resultado sanitizado | Interpretación |
|---|---|---|
| LAN/gateway | 4/4 respuestas, 0 % de pérdida, 2 ms | Enlace local operativo |
| Internet por IPv4 | 4/4 respuestas, 0 % de pérdida, media de 48 ms | Ruta IPv4 externa operativa |
| DNS e ICMP | El nombre resolvió a IPv6 y respondió | DNS e IPv6 operativos |
| TCP 443 | `TcpTestSucceeded: True` | Conexión TCP HTTPS alcanzable |
| Trazado IPv4 | Destino alcanzado en 12 saltos | Ruta completa observada |

## Hallazgos

1. El primer salto del trazado fue el gateway `192.168.1.1`.
2. El segundo salto pertenecía a `100.64.0.0/10`, lo que sugiere uso de CGNAT por parte del proveedor.
3. Otro salto usaba direccionamiento privado del proveedor; las direcciones posteriores se sanitizaron.
4. Una prueba individual de un salto no respondió, pero los saltos posteriores sí. Esto no representó una pérdida total de conectividad.
5. Windows prefirió un registro `AAAA` para `example.com` y utilizó una dirección IPv6 global temporal como origen.
6. El puerto TCP `443` aceptó la conexión.

## Matriz de diagnóstico

```text
Gateway responde
  → La comunicación local básica funciona.

IP externa responde
  → Existe enrutamiento hacia Internet sin depender de DNS.

Nombre resuelve
  → DNS funciona.

Puerto TCP 443 responde
  → El transporte TCP alcanza el servicio HTTPS.
```

## Notas de seguridad

- Un `ping` fallido no demuestra que el destino esté caído; ICMP puede estar filtrado.
- `tracert` revela información de topología y debe sanitizarse antes de publicarse.
- Una conexión TCP exitosa no valida por sí sola el certificado TLS ni la respuesta de la aplicación.
- La dirección IPv6 global del cliente puede identificar su conexión y no debe publicarse sin necesidad.
