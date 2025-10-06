# Wiki Técnica — Sistema Automático de Clasificación de Cajas

### Proyecto académico — Automatización de procesos industriales

**Empresa simulada:** *Quimpaquet S.A.*  
**Controlador:** Arduino Uno con OpenPLC  
**Integrantes:**
- **Sebastián Rodríguez** — Cableado, documentación y redacción de la Wiki.  
- **Juan Andrés Valderrama** — Simulación en CODESYS y desarrollo del maquetado físico.  

---

## 1. Descripción general del proyecto
El sistema propuesto consiste en un **proceso automatizado de clasificación de cajas** basado en la medición de su altura mediante tres sensores de proximidad infrarrojos.  
El objetivo principal es **automatizar la etapa de clasificación** dentro del proceso de empaquetamiento de *Quimpaquet S.A.*, eliminando la intervención manual y reduciendo errores humanos en la identificación del tamaño de las cajas.  

El sistema fue diseñado como **maqueta educativa**, representando el funcionamiento real de una estación de clasificación en una planta industrial.  

---

## 2. Contexto de aplicación
La empresa **Quimpaquet S.A.**, dedicada al empaquetamiento de productos químicos industriales, busca optimizar su línea de despacho automatizando la clasificación de cajas de distintos tamaños (pequeña, mediana, grande).  
Actualmente, la tarea es manual, lo que ocasiona cuellos de botella y errores en el control de inventario.  

El proyecto académico tiene como propósito **simular la automatización** de este proceso empleando sensores, servomotores y un microcontrolador programado en **Ladder** mediante OpenPLC.  

---

## 3. Objetivos específicos
- Diseñar y construir un **sistema de detección y clasificación automática** de cajas mediante sensores infrarrojos.  
- Implementar la **lógica de control en lenguaje Ladder** bajo la norma IEC 61131-3.  
- Validar el sistema a nivel de **simulación y prototipo físico**.  
- Documentar las etapas de diseño, desarrollo, implementación y validación en formato Wiki.  

---

## 4. Componentes y materiales

| Elemento                 | Descripción / Especificación                                                           |
| ------------------------ | -------------------------------------------------------------------------------------- |
| **Controlador**          | Arduino Uno corriendo OpenPLC                                                          |
| **Sensores**             | 3 × Módulos de proximidad infrarrojos con salida digital ON/OFF                        |
| **Actuadores**           | 3 × Servomotores (4.8–6 VDC, 0.12 s/60°, torque 1.8–2.2 kg·cm, piñonería plástica)     |
| **Motores de banda**     | 2 × Motores DC de 12 V para tracción                                                   |
| **Fuente de energía**    | Powerbank 5 V                                                                          |
| **Entradas adicionales** | SWitch START y botón RESET                                                             |
| **Salidas adicionales**  | LED de error (E)                                                                       |
| **Estructura mecánica**  | Banda transportadora de tela, rodillos de plástico y base de carton paja               |

---

## 5. Esquema general del proceso
1. La **banda transportadora** lleva la caja hasta la zona de medición.  
2. La **banda se detiene durante 3 segundos** para permitir la lectura estable de los tres sensores de altura.  
3. Según la combinación de sensores activos, se determina el tamaño de la caja.  
4. El **servomotor correspondiente** (A, B o C) se acciona para desviar la caja hacia su destino.  
5. En caso de una lectura incoherente, se enciende el **LED de error (E)**.  
6. La banda se reactiva para recibir la siguiente caja.  

---

## 6. Restricciones de diseño
- Complejidad en la fabricación de la **banda transportadora** (equilibrio entre elasticidad y rigidez del material).  
- El **motor inicial** presentaba alta velocidad y baja fuerza; se reemplazó por **dos motores de 12 V**. 

---

## 7. Estándares aplicados
| Norma           | Aplicación                                                    |
| --------------- | ------------------------------------------------------------- |
| **IEC 61131-3** | Lenguaje de programación Ladder.                              |
| **IEC 60204-1** | Principios de seguridad eléctrica y cableado.                 |
| **ISO 12100**   | Evaluación de riesgos en diseño de maquinaria.                |

---

## 8. Diagrama eléctrico (texto descriptivo)

**Entradas (Arduino Uno):**
- D2 → Sensor S1 (nivel inferior)  
- D3 → Sensor S2 (nivel medio)  
- D4 → Sensor S3 (nivel superior)  
- D5 → Botón START  
- D6 → Botón RESET  

**Salidas (Arduino Uno):**
- D9 → Servo A  
- D10 → Servo B  
- D11 → Servo C  
- D7 → LED de Error (E)  
- D8 → Motor banda (PWM vía driver H-bridge 12 V)  

**Alimentación:**
- Powerbank 5 V (Arduino + servos)  

---

## 9. Definición de variables

| Nombre    | Tipo | Atributo   | Descripción                       |
| --------- | ---- | ---------- | --------------------------------- |
| S1        | BOOL | Entrada    | Sensor inferior (detección base)  |
| S2        | BOOL | Entrada    | Sensor medio                      |
| S3        | BOOL | Entrada    | Sensor superior                   |
| START     | BOOL | Entrada    | Botón de inicio                   |
| STOP      | BOOL | Entrada    | Botón de parar                    |
| A         | BOOL | Salida     | Servo desviador de cajas pequeñas |
| B         | BOOL | Salida     | Servo desviador de cajas medianas |
| C         | BOOL | Salida     | Servo desviador de cajas grandes  |
| E         | BOOL | Salida     | LED de error                      |
| MOTOR     | INT  | Salida PWM | Motor de banda                    |
| T_MEASURE | TIME | Interna    | Temporizador de medición (3 s)    |
| REL_READY | BOOL | Interna    | Estado listo del sistema          |

---

## 10. Tabla de verdad de clasificación

| s1 | s2 | s3 | A | B | C | E | Descripción                      |
| -- | -- | -- | - | - | - | - | -------------------------------- |
| 0  | 0  | 0  | 0 | 0 | 0 | 0 | Sin caja                         |
| 0  | 0  | 1  | 0 | 0 | 0 | 1 | Error (detección alta sin base)  |
| 0  | 1  | 0  | 0 | 0 | 0 | 1 | Error (detección media sin base) |
| 0  | 1  | 1  | 0 | 0 | 0 | 1 | Error (detección sin base)       |
| 1  | 0  | 0  | 1 | 0 | 0 | 0 | Caja pequeña                     |
| 1  | 0  | 1  | 0 | 1 | 0 | 0 | Caja mediana                     |
| 1  | 1  | 0  | 0 | 0 | 0 | 1 | Error (base y tapa no conectadas)|
| 1  | 1  | 1  | 0 | 0 | 1 | 0 | Caja grande                      |

---

## 11. Diagrama de actividades
**Inicio → Esperar START → Activar banda → Caja detectada por S1 → Detener banda → Esperar 3 s → Evaluar sensores →**  

➡ Si lectura válida → Activar servo correspondiente (A/B/C)  
➡ Si lectura inválida → Encender LED de error (E)  

**→ Reactivar banda → Continuar con siguiente caja → Repetir ciclo.**  

*(RESET desactiva todos los servos, apaga LED y reinicia el sistema a estado inicial.)*  

---

## 12. Lógica Ladder

# Codesys
<img width="1159" height="647" alt="image" src="https://github.com/user-attachments/assets/2db45996-f003-4883-8e9f-4297c3d6e001" />

# Openplc
<img width="1612" height="458" alt="image" src="https://github.com/user-attachments/assets/b0fba230-1452-4845-9f47-265c1e32dad1" />
<img width="1580" height="197" alt="image" src="https://github.com/user-attachments/assets/2560e43e-3380-46c8-a459-d82588c17683" />
<img width="1641" height="566" alt="image" src="https://github.com/user-attachments/assets/9cf19273-cdb7-482d-8f6b-c334dfa97ba8" />
<img width="1588" height="521" alt="image" src="https://github.com/user-attachments/assets/1b7e44e3-acda-48f3-b914-91c2d851fc05" />
<img width="1583" height="443" alt="image" src="https://github.com/user-attachments/assets/70842661-0e83-458a-ad0e-ca3284421e76" />
<img width="1590" height="274" alt="image" src="https://github.com/user-attachments/assets/19ffcd2e-d4bc-4e2b-b0fb-38f1c3288655" />
<img width="1573" height="222" alt="image" src="https://github.com/user-attachments/assets/67cbaa61-d369-4d47-8d1f-7cf4a2c6eed9" />
<img width="914" height="692" alt="image" src="https://github.com/user-attachments/assets/c6690453-a046-4932-b7c1-69e5e91cfe50" />



---

## 13. Validación y resultados
El sistema fue validado en **CODESYS** y en prototipo físico.  
La clasificación funcionó según lo esperado. Hubo dificultades con la banda, solucionadas con ajuste de material y motores.  
El botón RESET y el LED de error respondieron adecuadamente.  

---

## 14. Conclusiones
- Sensores IR permiten una clasificación confiable en entorno controlado.  
- Arduino + OpenPLC es viable como plataforma educativa.  
- Futuras mejoras: control de velocidad, conteo de cajas, integración con HMI.  
- El ciclo de vida seguido refleja buenas prácticas de ingeniería de automatización.  
---

## 15. Actas
### ACTA 1
**Fecha:** 29/09/2025  
**Hora:** 3:00 – 4:00  
**Lugar:** Salón de automatización  

**Objetivo:**  
Empezar a diseñar cómo solucionar el problema planteado y, una vez definido el diseño, elaborar la lista de materiales para su compra.  

**Temas tratados:**  
- Diseño del proyecto  
- Lista de materiales  
- Compra de materiales  

**Acuerdos y compromisos:**  
- **Actividad:** Comprar materiales  
- **Responsable:** Sebastián   
- **Fecha de cumplimiento:** 30/09/2025  

**Próximos pasos:**  
- Diseño de simulación en CODESYS  
- Programación del código en OpenPLC  
- Cableado del proyecto  

---

### ACTA 2
**Fecha:** 01/10/2025  
**Hora:** 9:00 – 1:30  
**Lugar:** Salón de automatización y casa de Juan  

**Objetivo:**  
Trasladar la idea del diseño planteado a una simulación funcional en CODESYS; después de confirmar su funcionamiento, implementar el código en OpenPLC.  

**Temas tratados:**  
- Lógica de programación  

**Acuerdos y compromisos:**  
- **Actividad:** Comenzar la Wiki  
  - **Responsable:** Sebastián  
  - **Fecha de cumplimiento:** 04/10/2025  
- **Actividad:** Hacer actas  
  - **Responsable:** Sebastián  
  - **Fecha de cumplimiento:** 05/10/2025  

**Próximos pasos:**  
- Cableado del proyecto  

---

### ACTA 3
**Fecha:** 03/10/2025  
**Hora:** 11:00 – 4:30  
**Lugar:** Casa de Sebastián  

**Objetivo:**  
Finalizar el proyecto cableando la solución propuesta.  

**Temas tratados:**  
- Maqueta y conexiones
- 
---

## 16. Diseño modular de la maqueta

La maqueta del sistema fue construida de manera **modular**, dividiendo cada sección en bloques independientes:

- **Módulo de sensado:** incluye los tres sensores infrarrojos de proximidad.  
- **Módulo de actuadores:** contiene los servomotores y el LED indicador de error.  
- **Módulo de transporte:** integra la banda transportadora con sus motores de tracción.  
- **Módulo de control:** corresponde al Arduino Uno con sus conexiones a entradas y salidas.  

Este enfoque **facilitó el cableado**, ya que permitió organizar el sistema por funciones, simplificar las pruebas de cada bloque por separado y reducir la posibilidad de errores durante la integración.

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/ff0f5a1d-39cc-4cf8-8563-8d02275b3fc8" />

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/b5cba8e4-eac7-4fdb-8750-36e78ec88d92" />

<img width="1600" height="900" alt="image" src="https://github.com/user-attachments/assets/94a5ae1d-4927-41ad-972f-294c3a26682f" />

<img width="900" height="1600" alt="image" src="https://github.com/user-attachments/assets/7c944a56-c6ba-4bfb-80d7-1dab78a1217d" />

<img width="720" height="1280" alt="image" src="https://github.com/user-attachments/assets/21174d2b-d187-4d8f-bbb5-3939f608cff7" />



## 17. Uso de inteligencia artificial
Se utilizó **ChatGPT (GPT-5)** únicamente para corrección de redacción y formato.  
El contenido técnico fue desarrollado por el equipo.  
Referencias técnicas: IEC 61131-3, IEC 60204-1, ISO 12100.  

---

## 18. Referencias
[1] I. Patait, S. Bhosale, S. Shinde, S. Shinde, P. Gokhale, M. Kokare, “Automated Height Based Box Sorting System Using PLC,” International Engineering Journal For Research & Development, vol. 5, no. 5, p. 6, Jun. 2020. 
iejrd.com

[2] A. H. Embong, L. Asbollah, S. B. Abdul Hamid, “Empowering Industrial Automation Labs with IoT: A Case Study on Real-Time Monitoring and Control of Induction Motors using Siemens PLC and Node-RED,” Journal of Mechanical Engineering and Sciences, 2024.

[3] MY CREATIVE ENGINEERING, "Arduino Obstacle Detector Using IR Sensor & Buzzer | Simple DIY Project," YouTube, 25-Jun-2025. [Online]. Available: https://www.youtube.com/watch?v=RU3AELndZbk. [Accessed: 05-Oct-2025].

[4] Muy Fácil De Hacer, "Proyectos | Cinta Transportadora Casera (muy fácil de hacer)," YouTube, 12-Mar-2016. [Online]. Available: https://www.youtube.com/watch?v=7UsmJgHU6wk. [Accessed: 05-Oct-2025].




