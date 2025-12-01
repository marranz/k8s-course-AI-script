# Propuesta de Refinamiento: Historia de Docker (Pre-2015)

Revisando la primera parte del documento, sugiero los siguientes cambios para darle más profundidad técnica y precisión histórica:

## 1. Profundizar en "Cómo" se aísla (Namespaces & Cgroups)
El texto actual menciona "aislar procesos" de forma genérica.
*   **Cambio**: Mencionar explícitamente **Linux Namespaces** (para aislamiento de vista: PID, Mount, Network) y **Cgroups** (para aislamiento de recursos: CPU, RAM).
*   **Por qué**: Es la base técnica real. Docker no es magia, es una interfaz amigable para estas features del kernel.

## 2. La relación con LXC (Linux Containers)
El texto menciona LXC como una "solución anterior".
*   **Cambio**: Aclarar que **Docker originalmente usaba LXC** como su "driver" de ejecución.
*   **Hito Técnico**: Explicar que luego Docker creó **libcontainer** (ahora parte de `runc`) para dejar de depender de LXC y hablar directamente con el Kernel. Esto fue clave para su estabilidad y portabilidad.

## 3. El verdadero valor: "The Shipping Container"
El texto dice que "emergió como una nueva plataforma".
*   **Cambio**: Enfatizar la metáfora del **Contenedor de Transporte**. Antes de Docker, mover una app de Dev a Prod era un infierno de dependencias. Docker estandarizó el *empaquetado* (la imagen), no solo la ejecución.
*   **Frase sugerida**: "Docker resolvió el problema de 'en mi máquina funciona' empaquetando la aplicación y sus dependencias en un artefacto inmutable".

## 4. Pequeña corrección en DotCloud
*   **Cambio**: Mencionar que Solomon Hykes (el fundador) presentó Docker en una charla relámpago en PyCon 2013 (aunque el proyecto empezó en 2008 internamente, ese fue el momento público clave).

---

### Ejemplo de cómo quedaría la sección "De dotCloud a Docker Inc."

> **Antes:**
> "Técnicamente se podía hacer lo mismo, pero en este contexto emergió Docker..."
>
> **Propuesta:**
> "Aunque tecnologías como LXC ya permitían usar **Namespaces** y **Cgroups** del kernel de Linux para aislar procesos, eran complejas de configurar.
>
> Docker revolucionó esto al:
> 1.  **Democratizar el uso**: Creó una CLI sencilla que cualquier desarrollador podía usar.
> 2.  **Estandarizar el envío**: Introdujo el concepto de **Imagen de Docker**, permitiendo empaquetar la app con todas sus librerías. Si funcionaba en tu portátil, funcionaría en el servidor.
> 3.  **Independencia**: Inicialmente Docker usaba LXC, pero luego desarrolló su propia tecnología (`libcontainer`) para interactuar directamente con el Kernel, haciéndolo más robusto."
