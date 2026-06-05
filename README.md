# Agronica - Sistema de Riego Inteligente

## Descripción

Agronica es un sistema de riego inteligente basado en ESP32 que monitorea la humedad del suelo y la intensidad de luz para controlar automáticamente una electroválvula y un dosificador electrónico. Incluye un dashboard web en tiempo real accesible vía WiFi, permitiendo control manual y visualización de datos.

El proyecto está desarrollado con ESP-IDF (Espressif IoT Development Framework) y utiliza sensores ADC para lecturas ambientales, actuadores GPIO para control, y un servidor HTTP integrado para la interfaz web.

## Características

- **Monitoreo en tiempo real**: Sensores de humedad del suelo e intensidad de luz.
- **Control automático**: Lógica para activar válvula y dosificador electrónico basada en umbrales.
- **Dashboard web**: Interfaz moderna y responsiva con gráficos en vivo.
- **Control manual**: Botones en el dashboard para activar/desactivar actuadores.
- **Conectividad WiFi**: Configuración STA para acceso a red local.
- **Arquitectura modular**: Código separado en headers y sources por funcionalidad.

## Arquitectura del Proyecto

El código está organizado en dos carpetas principales dentro de `main/`:

- **`headers/`**: Archivos de cabecera (.h) con definiciones, constantes y prototipos.
  - `agronica.h`: Hardware, sensores y actuadores.
  - `dashboard.h`: Servidor web y handlers HTTP.
  - `wifi_manager.h`: Gestión de conexión WiFi.

- **`sources/`**: Archivos fuente (.c) con implementaciones.
  - `main.c`: Punto de entrada y loop principal.
  - `agronica.c`: Inicialización hardware y lecturas ADC.
  - `dashboard.c`: Servidor HTTP y página web.
  - `wifi_manager.c`: Conexión y gestión WiFi.

Esta separación facilita el mantenimiento y escalabilidad del código.

## Prueba de Concepto (POC)

La versión actual es una **prueba de concepto** diseñada para validar la funcionalidad básica sin riesgos de corto circuito o daños a componentes reales. En lugar de conectar actuadores de alto voltaje directamente, se utilizan LEDs para simular el comportamiento.

### ¿Qué se prueba en la POC?

- **Lectura de sensores**: Verificación de que los ADC capturan correctamente humedad y luz.
- **Lógica de control**: Simulación de activación/desactivación basada en lecturas.
- **Dashboard web**: Funcionalidad completa de la interfaz y actualizaciones en tiempo real.
- **Conectividad**: Establecimiento de red WiFi y acceso al dashboard.
- **Estabilidad**: Monitoreo continuo sin fallos durante periodos extendidos.

### Conexiones para POC

Para la prueba de concepto, conecta LEDs en lugar de los actuadores reales:

- **GPIO 25 (Válvula)**: LED rojo con resistor de 220Ω a GND.
- **GPIO 26 (Dosificador electrónico)**: LED azul con resistor de 220Ω a GND.
- **GPIO 34 (Humedad)**: Potenciómetro o sensor simulado (0-3.3V).
- **GPIO 35 (Luz)**: Fotoresistor o potenciómetro (0-3.3V).

**Diagrama simplificado:**
```
ESP32          LEDs
GPIO 25 ----[220Ω]----|>|---- GND
GPIO 26 ----[220Ω]----|>|---- GND

ESP32          Sensores
GPIO 34 (ADC6) ---- Potenciómetro ---- GND/3.3V
GPIO 35 (ADC7) ---- Fotoresistor ---- GND/3.3V
```

### Comportamiento en POC

- Los LEDs se encienden/apagan según el estado de los actuadores.
- El dashboard muestra "ON/OFF" para válvula y dosificador electrónico.
- Las lecturas ADC se muestran en el dashboard y logs.
- No hay riesgo de corto circuito o activación accidental de equipos reales.

## Implementación Real (Con Electroválvula)

Cuando pases a la implementación real:

### Conexiones Reales

- **GPIO 25 (Válvula)**: Conecta a un relé de 5V/12V que controle la electroválvula.
- **GPIO 26 (Dosificador electrónico)**: Conecta a un relé para el dosificador electrónico.
- **Sensores**: Conecta sensores reales de humedad (capacitivos) y luz (LDR).

**Importante**: Los GPIOs del ESP32 son 3.3V. Para actuadores de 12V/220V, usa relés con optoacopladores para aislamiento.

**Diagrama básico:**
```
ESP32          Relé          Actuador
GPIO 25 ---- IN (Relé) ---- NO/NC ---- Electroválvula
GND/3.3V ---- GND (Relé) ---- COM ---- Alimentación

ESP32          Sensor
GPIO 34 ---- Sensor Humedad ---- GND/3.3V
GPIO 35 ---- Sensor Luz ---- GND/3.3V
```

### Funcionamiento con Electroválvula

1. **Activación**: Cuando el ESP32 pone GPIO 25 en HIGH, el relé se cierra y energiza la válvula.
2. **Desactivación**: GPIO en LOW abre el relé y corta la alimentación.
3. **Flujo de agua**: La electroválvula debe permanecer siempre conectada a una fuente de agua; su función es solo permitir o bloquear el paso.
4. **Seguridad**: Implementa límites de tiempo para evitar sobre-riego.
5. **Monitoreo**: El dashboard refleja el estado real de la válvula y del dosificador.

### Consideraciones de Seguridad

- **Aislamiento**: Usa relés con optoacopladores para proteger el ESP32.
- **Alimentación**: Separa la alimentación del ESP32 (5V) de los actuadores (12V/220V).
- **Corriente**: Verifica que los relés puedan manejar la corriente de tus actuadores.
- **Protecciones**: Agrega fusibles y diodos de protección.

## Instalación y Configuración

### Prerrequisitos

- ESP-IDF v6.0+
- ESP32 DevKit
- Cables, LEDs/resistores para POC, o componentes reales para implementación final

### Configuración WiFi

Edita `main/headers/wifi_manager.h`:

```c
#define SSID_WIFI "TU_RED_WIFI"
#define PASS_WIFI "TU_CONTRASEÑA"
```

### Compilación y Flasheo

```bash
cd /ruta/a/Agronica
idf.py set-target esp32
idf.py menuconfig  # Configura si es necesario
idf.py build
idf.py flash
idf.py monitor
```

### Acceso al Dashboard

Una vez conectado a WiFi, accede a `http://[IP_DEL_ESP32]` en tu navegador.

## API del Dashboard

- `GET /`: Página principal con dashboard.
- `GET /data`: JSON con lecturas de sensores y estado de actuadores.
- `GET /actuador?d=valvula&s=1`: Activar válvula.
- `GET /actuador?d=dosificador&s=0`: Desactivar dosificador.

## Logs y Depuración

- Usa `idf.py monitor` para ver logs en tiempo real.
- Los logs incluyen lecturas ADC y estados de actuadores.
- Para debugging, conecta sensores simulados (potenciómetros) para variar lecturas.

## Próximos Pasos

- Implementar lógica automática basada en umbrales.
- Agregar notificaciones (email/SMS) para alertas.
- Integrar con bases de datos para historial.
- Optimizar consumo de energía para funcionamiento solar.

## Licencia

Este proyecto es de código abierto. Úsalo bajo tu propio riesgo.

## Soporte

Para preguntas o issues, revisa los logs del ESP32 y verifica conexiones hardware.
* For a feature request or bug report, create a [GitHub issue](https://github.com/espressif/esp-idf/issues)

We will get back to you as soon as possible.
