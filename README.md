# Lab 3 Estructura de un backend

Código base del curso para leer, ejecutar y modificar a mano, sin IA.
Hardware de referencia: NodeMCU ESP8266, DHT11 y LED RGB de cátodo común.
Este proyecto es una propuesta de material base para acompañar la guía.

## Backend en Windows
Desde backend, en una terminal Command Prompt de VS Code:
```
py -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements.txt
.venv\Scripts\python.exe -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```
Swagger: http://127.0.0.1:8000/docs
En macOS/Linux: python3 -m venv .venv y usar .venv/bin/python.
Los cinco endpoints base están en app/routes.py. La lista guarda como máximo
100 lecturas en memoria y se pierde al reiniciar. Ejecutar un solo proceso.
POST /lecturas retorna HTTP 201 con {"lectura": {...}, "led": "apagado"}.
PUT /led acepta rojo, verde, azul o apagado. No hay autenticación:
utilizar únicamente en la red de laboratorio autorizada.

## Firmware
Abrir firmware con PlatformIO IDE en VS Code. Copiar include/config.example.h
a include/config.h y completar WiFi, URL con IPv4 del PC e identificador.
Build, Upload y Monitor a 115200 baudios. El ESP envía cada 5 segundos y aplica
el color devuelto por POST /lecturas; PUT /led se refleja en el próximo envío.
Conexiones y pruebas: ver guía. Las dependencias se descargan la primera vez.
No publicar config.h, contraseñas ni tokens.

## Reto
Implementar GET /lecturas/promedio?n=3 a mano; no se incluye la solución.
Devolver cantidad, temperatura_promedio y humedad_promedio (dos decimales).
Validar n de 1 a 100; sin lecturas responder 404; si hay menos de n, usar todas.

## Antes de la sesión
El docente debe probar el firmware en el hardware real y la red del aula,
precargar las herramientas y validar las dependencias en los equipos.
La guía destina 105 minutos a trabajar con el entorno ya preparado. 

## Actividad


### 1. Modificación de la Ruta de Salud (Parte 5)
* **Archivo modificado:** `backend/app/routes.py`
* **Descripción:** Se añadió el campo `"laboratorio": 3` dentro del diccionario que retorna la función del endpoint `/salud`. Al realizar la petición GET, la respuesta JSON muestra de forma correcta el estado actual junto con el número de práctica correspondiente[cite: 5].

### 2. Implementación del Reto: Promedio de Lecturas (`GET /lecturas/promedio`)
* **Lógica implementada:** 
  * Se creó el endpoint `GET /promedio` que acepta un parámetro de consulta `n` (con valor por defecto de 3)[cite: 5].
  * En el servicio (`services.py`), se programó la validación para verificar si existen registros en la memoria (`store.lecturas`). Si la lista está vacía, se dispara una excepción HTTP 404 con un mensaje explicativo[cite: 5].
  * Se utilizan las últimas `n` lecturas mediante un corte de lista (`store.lecturas[-n:]`), calculando de forma independiente los promedios de temperatura y humedad, y redondeando ambos resultados a 2 decimales[cite: 5].
  * Se implementaron validaciones de rango con `Query` para asegurar que `n` se encuentre estrictamente entre 1 y 100, devolviendo un error HTTP 422 en caso de incumplimiento.