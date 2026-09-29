# Análisis de Vulnerabilidades con Nessus — Metasploitable 3

## 1. Introducción

Este proyecto documenta un análisis de vulnerabilidades realizado sobre una máquina virtual **Metasploitable 3**, utilizando **Tenable Nessus** como herramienta principal de evaluación.

El objetivo del laboratorio fue comprender el proceso de **detección, interpretación, clasificación, priorización y análisis de vulnerabilidades**, así como las diferentes formas mediante las cuales un scanner puede identificar un problema de seguridad.

El análisis se centró especialmente en comprender la información proporcionada por cada plugin y determinar qué nivel de evidencia existe detrás de cada resultado.

Para el análisis de los plugins se tienen en cuenta, cuando están disponibles:

* Plugin Output.
* CVSS.
* EPSS.
* VPR.
* Puerto.
* Servicio.
* Componente afectado.
* Evidencia proporcionada por Nessus.
* Método de detección utilizado por el plugin.
* Necesidad de validación manual.
* Tipo de validación realizada por Nessus.
* Resultado de la revisión manual, cuando corresponde.

> **Nota metodológica:** los resultados generados por Nessus no se interpretan automáticamente como vulnerabilidades explotadas o confirmadas manualmente. El nivel de certeza de cada hallazgo depende de la evidencia proporcionada por el plugin y del tipo de comprobación realizada.

---

# 2. Máquina evaluada

La evaluación se realizó sobre una máquina virtual **Metasploitable 3**, utilizada como objetivo de laboratorio para el análisis de vulnerabilidades.

| Parámetro                      | Valor                      |
| ------------------------------ | -------------------------- |
| Máquina                        | Metasploitable 3           |
| Sistema operativo identificado | Windows Server 2008 R2     |
| IP objetivo                    | `192.168.56.20`            |
| Scanner                        | Tenable Nessus             |
| Tipo de análisis               | Advanced Scan              |
| Origen del escaneo             | Máquina anfitriona Windows |
| Rango de puertos               | `1–65535`                  |

El objetivo fue evaluar la superficie de ataque del sistema, identificar servicios y componentes expuestos y analizar los resultados generados por Nessus.

---

# 3. Metodología del escaneo

Se utilizó un perfil **Advanced Scan** de Nessus, configurado específicamente para realizar una evaluación amplia del sistema.

La configuración incluyó:

* Descubrimiento del host.
* Escaneo completo de puertos TCP.
* Detección de servicios.
* Detección de SSL/TLS.
* Evaluaciones generales.
* Evaluación de aplicaciones web.
* Enumeración de Windows.
* Evaluación de malware.
* Uso de credenciales de Windows.
* Pruebas exhaustivas.
* Enumeración de plugins.
* Generación detallada de resultados.

El análisis se realizó desde la máquina anfitriona Windows.

---

# 4. Configuración del Advanced Scan

## 4.1 Discovery — Host Discovery

| Opción | Estado                         |
| ------ | ------------------------------ |
| Ping   | Desactivado                    |
| ARP    | Activado                       |
| TCP    | Activado                       |
| ICMP   | No utilizado para la detección |

La detección del objetivo se configuró utilizando mecanismos ARP y TCP, sin depender de respuestas ICMP.

---

## 4.2 Discovery — Port Scanning

| Opción                                                      | Configuración |
| ----------------------------------------------------------- | ------------- |
| Rango de puertos                                            | `1–65535`     |
| Considerar puertos no escaneados como cerrados              | Desactivado   |
| WMI / netstat                                               | Activado      |
| SSH / netstat                                               | Desactivado   |
| SNMP                                                        | Desactivado   |
| Ejecutar scanners de red solo si falla la enumeración local | Desactivado   |
| Verificar puertos TCP encontrados por enumeradores locales  | Activado      |
| TCP SYN                                                     | Activado      |
| Detección de firewall blando                                | Activada      |
| UDP                                                         | Desactivado   |

Se utilizó el rango completo de puertos TCP para ampliar la superficie de evaluación.

---

## 4.3 Discovery — Service Discovery

| Opción                                              | Configuración      |
| --------------------------------------------------- | ------------------ |
| Explorar todos los puertos para encontrar servicios | Activado           |
| Buscar SSL/TLS en todos los puertos TCP             | Activado           |
| DTLS                                                | Ninguno            |
| Detectar certificados próximos a expirar            | Activado — 60 días |
| Enumerar todos los cifrados SSL/TLS                 | Activado           |
| Comprobación CRL                                    | Desactivada        |

Esta configuración permitió realizar una detección amplia de servicios, independientemente del puerto utilizado.

---

# 5. Assessment

## 5.1 General

| Opción                   | Estado                  |
| ------------------------ | ----------------------- |
| Pruebas exhaustivas      | Activadas               |
| Inventario criptográfico | Desactivado             |
| Otras opciones           | Valores predeterminados |

Las pruebas exhaustivas permitieron ampliar la cantidad de comprobaciones ejecutadas durante la evaluación.

---

## 5.2 Brute Force

| Opción                                                         | Estado      |
| -------------------------------------------------------------- | ----------- |
| Utilizar únicamente credenciales proporcionadas por el usuario | Desactivado |
| Oracle                                                         | Desactivado |

---

## 5.3 Web Applications

| Opción                        | Estado    |
| ----------------------------- | --------- |
| Escanear aplicaciones web     | Activado  |
| Servidores web embebidos      | Activado  |
| XSS                           | Activado  |
| SQL Injection                 | Activado  |
| Otras pruebas web disponibles | Activadas |

La evaluación de aplicaciones web permitió realizar comprobaciones relacionadas con diferentes categorías de vulnerabilidades web.

---

## 5.4 Windows

| Opción                       | Estado      |
| ---------------------------- | ----------- |
| Dominio SMB                  | Desactivado |
| Enumeración del registro SAM | Activada    |
| Enumeración mediante WMI     | Activada    |
| ADSI                         | Desactivado |
| Fuerza bruta de RID          | Activada    |
| Rango RID                    | `1000–1200` |

---

## 5.5 Malware

| Opción                                  | Estado      |
| --------------------------------------- | ----------- |
| Escaneo de malware                      | Activado    |
| Escaneo/listado del sistema de archivos | Desactivado |

---

# 6. Configuración del informe

| Opción                                     | Configuración               |
| ------------------------------------------ | --------------------------- |
| Verbosidad                                 | Toda la información posible |
| Mostrar parches reemplazados               | Activado                    |
| Ocultar resultados de plugins dependientes | Activado                    |
| Permitir editar resultados                 | Desactivado                 |
| Mostrar hosts inaccesibles                 | Activado                    |
| Nombre DNS                                 | Desactivado                 |
| Unicode                                    | Desactivado                 |

La configuración buscó conservar la mayor cantidad posible de información útil generada durante el escaneo.

---

# 7. Opciones avanzadas

| Opción                                    | Estado                  |
| ----------------------------------------- | ----------------------- |
| Safe Checks                               | Desactivado             |
| Detener escaneo de hosts que no responden | Activado                |
| Enumerar plugins lanzados                 | Activado                |
| Rendimiento                               | Valores predeterminados |
| Debug                                     | Valores predeterminados |
| Compliance                                | Valores predeterminados |

---

# 8. Credenciales de Windows

Se configuraron credenciales de Windows para permitir a Nessus realizar comprobaciones locales adicionales.

| Parámetro                                | Valor                                                 |
| ---------------------------------------- | ----------------------------------------------------- |
| Método                                   | Password                                              |
| Usuario                                  | `vagrant`                                             |
| Contraseña                               | `vagrant`                                             |
| Dominio                                  | Vacío                                                 |
| Nunca enviar credenciales en texto claro | Activado                                              |
| Iniciar Remote Registry                  | Activado                                              |
| Habilitar Administrative Shares          | Activado                                              |
| NTLMv1                                   | Deshabilitado únicamente si el inicio de sesión falla |

El uso de credenciales permite obtener información adicional del sistema que puede no estar disponible mediante un análisis exclusivamente remoto.

---

# 9. Familias de plugins activadas

Las siguientes familias de plugins fueron habilitadas:

* Windows
* Windows: Boletines de Microsoft
* Windows: Gestión de usuarios
* Servidores web
* Abusos de CGI
* Abusos de CGI: XSS
* Bases de datos
* Puertas traseras
* Ataques de fuerza bruta
* Obtener una concha de forma remota
* FTP
* SMTP
* SNMP
* RPC
* DNS
* Cortafuegos
* Compartición de archivos entre pares
* Detección de servicio
* General
* Misceláneos
* Denegación de Servicio
* Cuentas Unix predeterminadas

La familia **Cuentas Unix predeterminadas** fue incluida de manera opcional debido a la presencia de servicios como OpenSSH en el entorno evaluado.

---

# 10. Resultados globales

El escaneo produjo **1.421 resultados** distribuidos de la siguiente manera:

| Severidad     |  Cantidad |
| ------------- | --------: |
| Crítica       |       177 |
| Alta          |       481 |
| Media         |       179 |
| Baja          |        33 |
| Informacional |       551 |
| **Total**     | **1.421** |

Estos valores corresponden a la clasificación realizada por Nessus.

Los resultados informacionales no deben interpretarse automáticamente como vulnerabilidades, ya que pueden representar información obtenida durante la enumeración o características detectadas en el objetivo.

---

# 11. Método de análisis de los resultados

El análisis utilizado para revisar los resultados de Nessus se basa en estudiar primero la información que proporciona cada plugin.

El proceso utilizado es:

```text
Resultado del scanner
        │
        ▼
Identificación del plugin
        │
        ▼
Revisión del Plugin Output
        │
        ├── CVSS
        ├── EPSS
        ├── VPR
        ├── Puerto
        ├── Servicio
        ├── Componente afectado
        └── Evidencia
        │
        ▼
Identificación del método de detección
        │
        ├── Evidencia técnica
        ├── Detección por versión
        ├── Configuración
        ├── Comprobación activa
        └── Output insuficiente
        │
        ▼
¿Requiere validación manual?
        │
        ▼
Validación / interpretación
        │
        ▼
Conclusión del hallazgo
```

Este procedimiento permite evitar interpretar todos los resultados de Nessus de la misma manera.

---

# 12. Plugin Output

El **Plugin Output** es uno de los elementos principales utilizados durante el análisis.

Se revisa si el plugin proporciona información como:

* versión instalada;
* versión corregida;
* archivo afectado;
* puerto;
* servicio;
* configuración;
* respuesta del servicio;
* información obtenida mediante enumeración;
* evidencia específica del sistema;
* resultado de una comprobación realizada por Nessus.

La cantidad y calidad de esta información determina el nivel de análisis que puede realizarse sobre el resultado.

---

# 13. CVSS

El **CVSS (Common Vulnerability Scoring System)** se utiliza para representar la severidad técnica de una vulnerabilidad.

Durante el análisis se registra el valor proporcionado por Nessus, pero no se utiliza como único criterio para determinar la prioridad de un hallazgo.

Un CVSS elevado indica una determinada severidad técnica, pero no demuestra por sí mismo que:

* exista un exploit público;
* el sistema esté siendo explotado;
* la vulnerabilidad sea fácilmente explotable en cualquier entorno;
* o que Nessus haya realizado una explotación.

---

# 14. EPSS

El **EPSS (Exploit Prediction Scoring System)** se utiliza como una métrica complementaria relacionada con la probabilidad estimada de explotación de una vulnerabilidad.

Se mantiene separado del CVSS:

```text
CVSS → Severidad técnica

EPSS → Probabilidad estimada de explotación
```

Por lo tanto, ambas métricas proporcionan información diferente.

---

# 15. VPR

El **VPR (Vulnerability Priority Rating)** es otra métrica utilizada por Tenable para ayudar en la priorización de vulnerabilidades.

Durante el análisis se registra junto con CVSS y EPSS, pero no se considera equivalente a ninguno de ellos.

La interpretación conjunta permite disponer de una visión más amplia del resultado.

---

# 16. Puerto y servicio

Para cada resultado se revisa el puerto y servicio asociados.

Sin embargo, el puerto mostrado por Nessus no se interpreta automáticamente como el vector exacto de explotación.

El análisis considera conjuntamente:

* puerto;
* protocolo;
* servicio;
* componente;
* descripción del plugin;
* CVE;
* evidencia;
* versión;
* método de detección.

Esto permite evitar conclusiones incorrectas basadas únicamente en el número de puerto.

---

# 17. ¿Cómo realizó Nessus la detección?

Una parte fundamental del análisis consiste en determinar **qué hizo realmente Nessus para generar el resultado**.

Se pueden encontrar diferentes situaciones.

## 17.1 Evidencia técnica

Nessus realiza una comprobación y obtiene información directamente del objetivo.

Por ejemplo:

```text
Objetivo
   ↓
Comprobación del plugin
   ↓
Obtención de evidencia
   ↓
Resultado
```

La evidencia puede corresponder a versiones, configuraciones, respuestas de servicios u otros datos.

---

## 17.2 Detección basada en versión

En determinados casos Nessus identifica una versión vulnerable de un componente.

El proceso puede representarse como:

```text
Versión detectada
       ↓
Comparación con versiones afectadas
       ↓
Asociación con vulnerabilidad conocida
       ↓
Resultado del plugin
```

En este escenario, Nessus puede determinar que un componente es vulnerable sin necesidad de explotar la vulnerabilidad.

---

## 17.3 Comprobación activa

Algunos plugins pueden realizar comprobaciones directamente sobre un servicio o componente para determinar si una determinada condición está presente.

En estos casos debe revisarse cuidadosamente el Plugin Output para determinar qué comprobación realizó realmente el plugin.

---

## 17.4 Plugin Output insuficiente

Puede haber resultados donde el Plugin Output no proporcione información suficiente para confirmar completamente la condición.

En estos casos:

* se registra el resultado;
* se analiza la información disponible;
* se determina si requiere validación manual;
* y no se presenta automáticamente como una vulnerabilidad confirmada.

---

# 18. Validación manual

La validación manual se utiliza cuando la información proporcionada por Nessus no es suficiente para determinar completamente la condición del sistema.

El criterio utilizado es:

```text
Nessus reporta
      ↓
Revisión del Plugin Output
      ↓
¿La evidencia es suficiente?
     / \
   Sí   No
   │     │
   ▼     ▼
Analizar  Validar manualmente
```

La validación manual puede utilizar herramientas adicionales o comprobaciones específicas sobre el objetivo.

La finalidad no es repetir automáticamente todo el escaneo, sino determinar si el resultado reportado por Nessus está respaldado por evidencia suficiente.

---

# 19. Diferencia entre detección y explotación

Un punto fundamental del análisis consiste en distinguir:

**Nessus detectó o reportó un hallazgo**

de:

**La vulnerabilidad fue explotada.**

El hecho de que un plugin muestre:

* un CVSS elevado;
* un EPSS elevado;
* un VPR elevado;
* o disponibilidad de exploit;

no significa automáticamente que Nessus haya ejecutado un exploit contra el objetivo.

En numerosos casos, el scanner determina el estado mediante:

* versiones;
* archivos;
* configuraciones;
* respuestas de servicios;
* enumeración;
* o comprobaciones específicas.

---

# 20. Triage de vulnerabilidades

Debido al volumen de resultados obtenidos, el análisis se realizó mediante un proceso de **triage**.

El objetivo del triage es determinar:

1. Qué reportó Nessus.
2. Qué evidencia proporciona.
3. Qué componente está involucrado.
4. Qué severidad tiene.
5. Qué probabilidad de explotación presenta.
6. Qué prioridad puede tener.
7. Si necesita validación manual.
8. Qué información adicional se necesita para confirmar el resultado.

El triage permite trabajar de manera estructurada cuando un scanner genera cientos o miles de resultados.

---

# 21. Limitaciones del análisis

El escaneo produjo **1.421 resultados**, por lo que no todos fueron sometidos a una validación manual individual.

El propósito del laboratorio fue aprender el proceso de:

* detección;
* interpretación;
* clasificación;
* priorización;
* análisis de evidencia;
* y validación.

Por esta razón, los resultados se diferencian entre:

**Hallazgos reportados por Nessus**

y

**Hallazgos que requieren o recibieron validación manual.**

No se debe interpretar el número total de resultados como el número de vulnerabilidades que fueron explotadas durante el laboratorio.

---

# 22. Conclusiones

El análisis realizado permitió estudiar el funcionamiento de un proceso de evaluación de vulnerabilidades utilizando Nessus sobre un entorno controlado.

El escaneo produjo:

* **177 resultados críticos**
* **481 resultados altos**
* **179 resultados medios**
* **33 resultados bajos**
* **551 resultados informacionales**

para un total de **1.421 resultados**.

El principal objetivo del laboratorio fue aprender a interpretar los resultados de un scanner profesional y no limitarse a observar la clasificación de severidad.

La metodología utilizada considera diferentes elementos:

```text
Plugin Output
     +
CVSS
     +
EPSS
     +
VPR
     +
Puerto
     +
Servicio
     +
Evidencia
     +
Método de detección
     +
Validación manual
```

Este enfoque permite diferenciar entre un resultado que está respaldado por evidencia técnica, un resultado detectado principalmente mediante una versión vulnerable y un resultado que requiere una comprobación adicional.

También permite evitar errores comunes como asumir que un puerto representa necesariamente el vector de ataque, interpretar CVSS como probabilidad de explotación o considerar que un resultado de Nessus significa automáticamente que se ejecutó un exploit.

El laboratorio permitió comprender que una herramienta de vulnerability scanning es principalmente una fuente de **hallazgos y evidencia técnica**. La interpretación profesional de esos resultados requiere análisis, priorización y, cuando sea necesario, validación manual.

---

# 23. Tecnologías y conceptos utilizados

* Tenable Nessus
* Metasploitable 3
* Windows Server 2008 R2
* Advanced Scan
* Vulnerability Scanning
* Vulnerability Assessment
* Vulnerability Triage
* Plugin Output
* CVSS
* EPSS
* VPR
* Validación manual
* Discovery
* Port Scanning
* Service Discovery
* SSL/TLS
* Web Application Scanning
* Windows Enumeration
* WMI
* SMB
* TCP
* ARP
* XSS
* SQL Injection
* Brute Force
* Malware Detection
* Gestión de vulnerabilidades
* Detección basada en versiones
* Análisis de evidencia
