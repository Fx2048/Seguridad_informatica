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
