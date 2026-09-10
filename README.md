# Seguridad_informatica

Sí. Pero aquí hay una distinción importante: **si tu Windows 11 en ISO es el sistema operativo de tu laptop**, no necesitas instalarlo otra vez. En el esquema que hicimos:

```text
TU LAPTOP
Windows 11
   │
   ▼
VirtualBox
   │
   ▼
Windows 10 21H2 ← ESTA será nuestra máquina virtual forense
```

Por tanto, **el siguiente paso no es instalar otro Windows 11**, sino conseguir/preparar el **Windows 10 21H2** que será nuestra VM de laboratorio.

### Paso 1 — No instales todavía las herramientas DFIR

Primero vamos a construir correctamente la máquina virtual.

Necesitamos:

* VirtualBox ✅
* Windows 10 21H2 ISO ⬅️ falta esto
* espacio suficiente en disco
* idealmente **8 GB o más de RAM en tu laptop** para trabajar cómodamente

La arquitectura final será:

```text
┌─────────────────────────────────────────┐
│          TU LAPTOP — WINDOWS 11         │
│                                         │
│  ┌───────────────────────────────────┐  │
│  │          VirtualBox               │  │
│  │                                   │  │
│  │  ┌─────────────────────────────┐  │  │
│  │  │       Windows 10 21H2       │  │  │
│  │  │                             │  │  │
│  │  │  Máquina de laboratorio     │  │  │
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### Paso 2 — Conseguir Windows 10 21H2

Para reproducir **exactamente la práctica que tienes**, lo ideal es trabajar con Windows 10 21H2.

No te recomiendo que descargues cualquier ISO de Windows 10 al azar, porque después podemos terminar con una versión diferente y los artefactos, herramientas o resultados pueden variar.

Si quieres, puedo guiarte **paso por paso para conseguir una ISO adecuada de Windows 10 21H2 y verificar que sea la correcta**.

Después hacemos:

**ISO → crear VM → configurar RAM/CPU/disco → instalar Windows → snapshot limpio → herramientas DFIR → preparar evidencia.**

Y algo muy importante: **no vamos a tocar todavía la evidencia `Win10_PC001.vmdk`**. Primero dejamos nuestra máquina de análisis limpia y reproducible.



Perfecto. Entonces ya estás en el punto correcto: **VM limpia + Windows instalado**. Ahora no conviene instalar cosas al azar; vamos a convertir esa VM en tu **estación de análisis DFIR reproducible**.

Yo lo haría en este orden:

### 1. Primero: congela la VM limpia

Antes de instalar herramientas:

* Apaga Windows.
* En VirtualBox crea un snapshot:
  **`WIN10-BASELINE-CLEAN`**
* Documenta:

  * Windows 10 versión/build.
  * RAM asignada.
  * CPU asignadas.
  * versión de VirtualBox.
  * nombre de la VM.

Esto te permite volver al estado limpio si una herramienta rompe algo.

---

### 2. Instala las herramientas DFIR

Para vuestro proyecto, no necesitas instalar 30 herramientas. El conjunto base puede ser:

| Área         | Herramienta            | Para qué                         |
| ------------ | ---------------------- | -------------------------------- |
| Imagen/disco | **FTK Imager**         | Examinar/adquirir imágenes       |
| Registro     | **Registry Explorer**  | Analizar Registry                |
| Prefetch     | **PECmd**              | Analizar `.pf`                   |
| Event Logs   | **EvtxECmd**           | Analizar `.evtx`                 |
| Memoria      | **Volatility 3**       | Analizar RAM                     |
| Metadata     | **ExifTool**           | Analizar metadatos               |
| Hashes       | **7-Zip / PowerShell** | SHA-256, MD5, etc.               |
| Timeline     | **Timeline Explorer**  | Visualizar resultados temporales |

Y opcionalmente:

* KAPE, si quieren automatizar la extracción de artefactos.
* Autopsy, si quieren una segunda plataforma para contraste.
* Wireshark **solo si finalmente incorporan captura/análisis de red**.

Para el proyecto que planteamos, **FTK + Registry Explorer + PECmd + EvtxECmd + Volatility + ExifTool** ya cubren bastante.

---

### 3. Después viene lo MÁS importante: preparar la evidencia

Aquí quiero que tengan mucho cuidado.

Si tienen:

```text
Win10_PC001.vmdk
```

**NO lo arranquen como una máquina virtual normal.**

Porque al arrancarlo Windows podría modificar:

* timestamps,
* logs,
* archivos,
* Registry,
* Prefetch,
* otros artefactos.

Y entonces ya no estarían analizando la evidencia original.

La estructura debería ser aproximadamente:

```text
DFIR-PROJECT/
│
├── evidence/
│   ├── original/
│   │   └── Win10_PC001.vmdk
│   │
│   └── working/
│       └── Win10_PC001_WORKING.vmdk
│
├── hashes/
│   └── evidence_hashes.txt
│
├── exports/
│   ├── registry/
│   ├── eventlogs/
│   ├── prefetch/
│   ├── memory/
│   └── metadata/
│
├── timeline/
│
└── reports/
```

La idea es:

**original → preservar → hash → trabajar sobre copia**

No:

**original → abrir/modificar → analizar → intentar recordar qué cambió**

---

### 4. Calculamos el hash de la evidencia

Por ejemplo, en PowerShell:

```powershell
Get-FileHash "Win10_PC001.vmdk" -Algorithm SHA256
```

Guardas el resultado.

Algo como:

```text
Evidence: Win10_PC001.vmdk
Algorithm: SHA-256
Hash: XXXXXXXX...
Date: 2026-09-10
```

Ese hash será parte de vuestra **cadena de custodia / verificación de integridad**.

Y más adelante pueden volver a calcularlo:

```powershell
Get-FileHash "Win10_PC001.vmdk" -Algorithm SHA256
```

y comprobar:

```text
HASH INICIAL = HASH FINAL
```

Eso es mucho más defendible académicamente que simplemente decir *"analizamos la imagen"*.

---

### 5. Para los 6 integrantes: estandaricen el entorno

Esto es especialmente importante porque trabajan desde **seis laptops diferentes**.

Todos deberían tener:

```text
Windows 10 21H2
VirtualBox [misma versión]
Volatility 3 [misma versión]
FTK Imager [misma versión]
Registry Explorer [misma versión]
PECmd [misma versión]
EvtxECmd [misma versión]
ExifTool [misma versión]
```

Y documentar las versiones.

Así, si tú obtienes:

```text
resultado X
```

y otro integrante obtiene:

```text
resultado X
```

pueden argumentar que el procedimiento es **reproducible**.

Eso encaja directamente con vuestro proyecto.

---

## El orden que yo seguiría AHORA

No intentaría hacer todo hoy de golpe.

**FASE A — Estación DFIR**

```text
☑ VM Windows 10 instalada
☐ Snapshot WIN10-BASELINE-CLEAN
☐ Actualizar/configurar Windows si corresponde
☐ Instalar FTK Imager
☐ Instalar Registry Explorer
☐ Instalar PECmd
☐ Instalar EvtxECmd
☐ Instalar Volatility 3
☐ Instalar ExifTool
☐ Instalar Timeline Explorer
☐ Registrar versiones
☐ Snapshot DFIR-TOOLS-READY
```

**FASE B — Evidencia**

```text
☐ Obtener Win10_PC001.vmdk
☐ Preservar copia original
☐ Calcular SHA-256
☐ Crear copia de trabajo
☐ Verificar hash
☐ Registrar evidencia
☐ NO arrancar la imagen original
```

**FASE C — Primera investigación**

```text
☐ Identificar sistema
☐ Identificar usuario(s)
☐ Analizar Registry
☐ Analizar Event Logs
☐ Analizar Prefetch
☐ Analizar MFT/NTFS
☐ Analizar memoria
☐ Extraer IOCs
☐ Construir timeline
☐ Correlacionar evidencias
```

Y **recién después** empezamos a resolver los 27 desafíos como parte de los escenarios de investigación.

De hecho, yo convertiría cada CTF en algo del estilo:

> **Pregunta → evidencia necesaria → herramienta → procedimiento → hallazgo → interpretación → corroboración**

Eso hará que vuestro informe parezca un **proyecto DFIR serio** y no un documento de respuestas de CTF.

### Siguiente paso

Si quieres, hacemos **instalación herramienta por herramienta**, empezando por **FTK Imager**, y te digo exactamente qué descargar, dónde instalarlo y qué configuración usar para que después la metodología sea reproducible para los otros 5 integrantes.
