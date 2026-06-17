# 📘 Fundamentos Teóricos: Tarjeta Electrónica ESP32

Este documento explica los principios de ingeniería electrónica aplicados en el diseño de esta tarjeta. El objetivo del proyecto es resolver un problema clásico en la automatización: **interconectar el mundo lógico (baja potencia y alta velocidad) con el mundo físico (cargas industriales y mayor voltaje)**.

A continuación, se detalla la teoría detrás de los tres pilares del diseño: Alimentación, Comunicación y Control de Potencia.

---

## ⚡ 1. Gestión de Potencia: ¿Por qué un Convertidor Buck?

En la industria, el estándar de alimentación para actuadores (relés, electroválvulas, tiras LED) suele ser de **12V o 24V**. Sin embargo, los microcontroladores modernos como el ESP32 funcionan estrictamente a **3.3V**.

Para reducir el voltaje de 12V a 3.3V, existen dos caminos principales:
1. **Regulador Lineal (LDO - ej. LM7833 o AMS1117):** Funciona "quemando" el voltaje sobrante en forma de calor. Si el ESP32 consume 0.5A (al transmitir por Wi-Fi), la potencia disipada sería: `(12V - 3.3V) * 0.5A = 4.35 Watts`. Esto derretiría el componente y arruinaría la placa.
2. **Convertidor Conmutado (Buck / Step-Down):** Esta es la **solución implementada en nuestra tarjeta**. En lugar de disipar calor, un regulador Buck actúa como un interruptor rapidísimo (conmuta a miles de hercios) que "corta" el voltaje y almacena la energía temporalmente en un **inductor** (bobina) y un **capacitor**. 

**Resultado teórico:** Pasamos de una eficiencia del ~27% (con un LDO) a una eficiencia superior al **90%**. El sistema se mantiene completamente frío incluso a máxima carga, garantizando la estabilidad del ESP32.

---

## 🔌 2. Comunicación: El Puente USB a UART y Auto-Reset

Los computadores modernos solo hablan por puertos **USB** (protocolo complejo y diferencial), pero el microcontrolador ESP32 se programa a través de un protocolo serie clásico llamado **UART** (pines TX, Transmisión, y RX, Recepción).

**¿Cómo los comunicamos?**
Usamos un circuito integrado "traductor" (el **CP2102**). Este chip se conecta al PC como un dispositivo USB estándar y convierte esos paquetes de datos en señales eléctricas TX y RX que el ESP32 puede entender.

**La Teoría del Auto-Reset (Auto-Flasheo):**
Para programar un ESP32, el chip necesita entrar en "Modo Descarga". Manualmente, esto requeriría presionar dos botones en la placa (EN y BOOT) en una secuencia específica.
Para automatizar esto, el puente USB-UART utiliza dos señales de control adicionales del puerto serie virtual (`DTR` y `RTS`). A través de un arreglo de dos **transistores NPN**, estas señales controlan automáticamente los pines `EN` (Reset) e `IO0` (Boot) del ESP32. 
* *Resultado:* Cuando le das a "Subir Código" en tu PC, la placa se reinicia sola, entra en modo programación, descarga el código y vuelve a arrancar sin intervención humana.

---

## ⚙️ 3. Etapa de Control: La Teoría del MOSFET

El microcontrolador ESP32 es el "cerebro", pero es muy débil físicamente. Sus pines (GPIO) solo pueden entregar unos **20 a 40 miliamperios (mA)** a 3.3V. Si intentas conectar un motor o una tira LED de 12V directamente, el chip se destruiría instantáneamente.

Para resolver esto, utilizamos **Transistores MOSFET de Canal N** (Interruptores de estado sólido) operando bajo los siguientes principios:

1. **Aislamiento de Lógica y Potencia:** El pin del ESP32 se conecta a la compuerta (*Gate*) del MOSFET. Al aplicarle 3.3V, se crea un campo eléctrico interno que "abre el grifo" y permite que la corriente fluya entre el Drenador (*Drain*) y el Surtidor (*Source*), donde está conectada nuestra carga de 12V. 
2. **Resistencia de Compuerta (Gate Resistor):** La compuerta de un MOSFET se comporta como un pequeño condensador. Al encenderse, exige un pico de corriente muy alto por una fracción de segundo. Colocamos una resistencia en serie (ej. 100Ω) para frenar este pico y proteger el pin del ESP32.
3. **Resistencia Pull-Down:** Si el ESP32 se está reiniciando, sus pines quedan "flotando" (sin un estado de voltaje definido). Esto podría causar que el MOSFET se encienda y apague erráticamente. Una resistencia conectada a tierra (GND) asegura que la compuerta se mantenga apagada hasta que el código del ESP32 diga lo contrario explícitamente.

---

## 📐 4. Teoría de Diseño de PCB (Hardware Layout)

Transformar el plano eléctrico (esquemático) en una placa física bidimensional requiere manejar conceptos de electromagnetismo:

* **Ancho de pista vs. Corriente:** Las pistas que llevan señales de datos (como TX/RX) son muy delgadas (10 mils) porque llevan corrientes minúsculas. Sin embargo, las pistas que alimentan los actuadores a 12V deben ser mucho más gruesas (35 mils o más) para evitar que la resistencia física del cobre genere calor y caídas de voltaje.
* **Planos de Tierra (GND Pours):** En lugar de usar pistas delgadas para conectar la masa/tierra de los componentes, rellenamos todo el espacio vacío de la placa con cobre conectado a tierra. Esto proporciona un camino de "retorno" de muy baja resistencia para la energía y actúa como un escudo que absorbe el ruido electromagnético emitido por la antena Wi-Fi del ESP32.
* **Ruteo Diferencial USB:** Las pistas de datos USB viajan a alta velocidad. Se rutean juntas, en paralelo y con la misma longitud. Cualquier ruido eléctrico del ambiente afectará a ambas por igual, y como el receptor lee la *diferencia* entre las dos (no el valor absoluto), el ruido se anula por completo.

---

### 📌 Resumen Teórico
El diseño no es una simple suma de componentes, sino una arquitectura pensada donde:
1. La **etapa Buck** asegura la viabilidad térmica.
2. El **puente UART** resuelve la accesibilidad del usuario.
3. Los **MOSFETs** actúan como la fuerza bruta controlada por la lógica.
4. El **Layout del PCB** garantiza que todas estas energías coexistan sin generar ruido o interferencias entre sí.
