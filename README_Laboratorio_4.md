# Laboratorio 4 — Configuración de Switch/Router y RS485

**Autores:** Daniel Felipe Pinilla Daza · Julian Sebastian Alvarado Monroy  
**Documento base:** `Laboratorio_4.pdf`  
**Asignatura/Contexto:** Redes y Comunicaciones Industriales

---

## Descripción general
Este README resume el **Laboratorio 4**, dividido en dos ejes:  
1) **Configuración básica de red** con *switch* y *router* (topología, addressing, ARP/MAC, interfaces y enrutamiento).  
2) **Pruebas con protocolo RS485**, cubriendo teoría de señalización diferencial, modos de operación, control de flujo (pines DE/RE), métricas de rendimiento y detección de errores.  
Como componente aplicado, se presenta un **dashboard web** (Flask + Socket.IO + Chart.js) para monitoreo en tiempo real (throughput, latencia, errores).

---

## 1. Introducción y objetivos
- Integrar PC, *switch*, *router* y **Raspberry Pi** en una topología funcional.  
- Comprender comandos clave de infraestructura (ARP, MAC, interfaces, rutas, reglas).  
- Analizar RS485 en **simplex/half‑duplex/full‑duplex**, control DE/RE y robustez ante ruido.  
- Medir **eficiencia, throughput, latencia y BER**; aplicar **checksum** y **CRC‑16 Modbus**.  
- Visualizar métricas en tiempo real vía dashboard.

---

## 2. Punto 1 — Configuración básica de Switch y Router
### 2.1 Marco teórico breve
- **Switch (Capa 2, OSI):** reenvío por **MAC**; segmentación de dominios de colisión.  
- **Router (Capa 3, OSI):** conmutación IP; decisiones de **routing** con tablas/protocolos.  
- **Raspberry Pi:** SBC versátil para *edge*, pruebas y servicios (Flask, sockets).

### 2.2 Topología implementada
- **PC** ↔ **Switch (Cisco 2960)** ↔ **Router (Cisco 1841)** ↔ **Raspberry Pi 4**.  
- Subredes de ejemplo: `192.168.1.0/24` (hacia switch) y `192.168.2.0/24` (hacia RPi).

### 2.3 Procedimiento (resumen)
- **Switch:** hostname, *enable secret*, credenciales de consola, **SVI VLAN 1** con IP y **gateway**, *write memory*.  
- **Router:** hostname, *enable secret*, IP en `G0/0` (LAN) y `G0/1` (RPi), *no shut*, *write memory*.  
- **Raspberry Pi:** IP estática en `eth0` (`/etc/dhcpcd.conf`), *DNS* y *router* por defecto.

### 2.4 Comandos clave del router/switch
- **ARP:** `show ip arp` → mapeo IP↔MAC.  
- **MAC table:** `show mac address-table` en switch.  
- **Interfaces:** `show ip interface brief` (estado admin/operativo).  
- **Routing:** `show ip route` y rutas estáticas `ip route <red> <mask> <next-hop>`.  
- **Conectividad:** *ping* entre nodos para RTT y reachability.

### 2.5 Resultados del punto 1
- Conectividad estable y **baja latencia (<5 ms)**. Segmentación y enrutamiento correctos.

---

## 3. Punto 2 — RS485: teoría, implementación y pruebas
### 3.1 Conceptos clave
- **RS485 (EIA‑485):** bus diferencial **multipunto** (A/B), alta inmunidad a EMI, hasta **1200 m**; **hasta 10 Mbps** en tramos cortos.  
- **Señalización diferencial:** “1” si **A > B**; “0” si **A < B** (típ. ±2–6 V de diferencial).

### 3.2 Modos de comunicación
- **Simplex:** unidireccional; bajo costo.  
- **Half‑duplex:** bidireccional alternado (el más común); requiere **DE/RE**.  
- **Full‑duplex:** bidireccional simultáneo (dos pares), menor latencia y mayor costo.

### 3.3 Control de flujo (DE/RE) y UART
- Gestión de **DE/RE** desde **GPIO** (ej. RPi, MAX485); *delays* cortos al conmutar TX↔RX.  
- Configuración UART: *baudrate*, *8N1*, *timeout*, colas de transmisión/recepción.

### 3.4 Métricas y eficiencia
- **Tasa bruta vs efectiva**, **eficiencia teórica 8N1 ≈ 80%** (10 bits/byte).  
- **Pruebas** a 9.6/19.2/38.4/115.2 kbps y distintas distancias (10–500 m):  
  - Eficiencia ≈ teórica; **BER** aumenta con **distancia/baudrate**; latencia crece con distancia.  
  - A **115200 bps** la BER se vuelve significativa si el cableado/terminación no es óptimo.

### 3.5 Detección de errores
- **Checksum simple** (suma módulo 256) vs **CRC‑16 Modbus** (polinomio 0xA001).  
- Ensayos con inyección de errores: CRC‑16 detecta prácticamente **100%** de los errores simulados.

### 3.6 Dashboard en tiempo real (Flask + Socket.IO + Chart.js)
- Backend en **RPi** (Python/Flask), WebSockets para *push* de métricas.  
- Frontend con **Chart.js** para **throughput** y **latencia**; tarjetas de **bytes enviados/recibidos** y **errores**.  
- Métricas agregadas por ventana y actualización suave (*no flicker*).

### 3.7 Pruebas de estrés y robustez
- **Carga 24 h:** uptime ~99.997%, throughput ~9.6 kB/s; 23 errores CRC detectados (BER ~2.77×10⁻⁶).  
- **EMI:** blindaje y terminación **120 Ω** reducen reflexiones/errores de forma notable.

### 3.8 Modbus RTU (extensión)
- Cliente **pymodbus**: lectura/escritura de registros *holding* (0x03/0x06), *timeout* y *retry*.

### 3.9 RS485 vs otros
| Característica | RS485 | RS232 | CAN | Ethernet |
|---|---:|---:|---:|---:|
| Distancia máx. | 1200 m | 15 m | 1000 m | 100 m (cobre) |
| Velocidad máx. | 10 Mbps | 115.2 kbps | 1 Mbps | 1+ Gbps |
| Nodos | ~32 (ext.) | 2 | ~110 | Ilimitado |
| Inmunidad ruido | Alta | Baja | Alta | Media |
| Topología | Bus | Punto‑a‑punto | Bus | Estrella |
| Costo | Bajo | Bajo | Medio | Medio‑Alto |

---

## 4. Conclusiones (síntesis)
- El montaje *switch–router–RPi* demuestra principios de **L2/L3** y verificación integral (ARP/MAC/route/ping).  
- **RS485** es **robusto y costo‑efectivo**; con **DE/RE** bien gestionado, terminación y blindaje adecuados, ofrece BER muy baja.  
- **CRC‑16** garantiza integridad; el **dashboard** aporta **observabilidad** y facilita diagnóstico.

---

## Repositorio sugerido
Si este laboratorio forma parte de una colección de tareas, se recomienda referenciar el repositorio:  
**GitHub:** https://github.com/JulianAlva24/TAREAS_COMUNICACIONES.git

