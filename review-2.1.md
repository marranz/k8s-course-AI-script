# Revisión: 2.1 Historia y Orígenes de Docker

El contenido actual es correcto históricamente, pero para un curso de **Kubernetes en 2025**, se queda un poco corto en explicar "qué pasó después" y por qué Docker no es lo mismo que Kubernetes.

## Sugerencias de Cambios y Adiciones

### 1. Añadir la Estandarización (OCI) - **CRÍTICO**
Es fundamental mencionar que en **2015** se creó la **Open Container Initiative (OCI)**.
*   **Por qué**: Para explicar que Docker "donó" su formato de imagen y runtime (`runc`). Esto es la base para entender por qué hoy podemos usar imágenes de Docker en Kubernetes sin necesidad de tener Docker instalado (usando containerd o CRI-O).

### 2. La separación de capas (Docker vs containerd vs runc)
Explicar brevemente que Docker se dividió en componentes más pequeños.
*   **Docker** (la CLI, la UX).
*   **containerd** (el runtime que gestiona el ciclo de vida - usado por K8s).
*   **runc** (el runtime de bajo nivel que habla con el Kernel).
*   *Valor*: Ayuda a desmitificar la "muerte de Docker en Kubernetes".

### 3. El contexto de la "Guerra de Orquestadores"
Mencionar brevemente que Docker intentó competir con **Docker Swarm**, pero Kubernetes se convirtió en el estándar de facto para la orquestación.

### 4. Actualizar la línea de tiempo
El texto termina en 2012. Sugiero añadir hitos clave:
*   **2013**: Lanzamiento público de Docker.
*   **2015**: Fundación de la CNCF y OCI (Kubernetes 1.0).
*   **2016/2017**: Separación de `containerd`.
*   **2020**: Kubernetes anuncia la deprecación de Dockershim (el fin de Docker como runtime directo de K8s).

## Propuesta de Estructura Actualizada
Mantener lo actual como "Los Orígenes" y añadir una sección nueva:
**"La Evolución hacia el Estándar (2015-Presente)"**
*   Explicar OCI.
*   Explicar la diferencia entre Container Engine (Docker) y Container Runtime (containerd/CRI-O).
