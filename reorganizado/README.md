# Curso de Kubernetes - Material Reorganizado

Este directorio contiene el material del curso de Kubernetes reorganizado en una estructura lógica de directorios y archivos.

## Estructura del Curso

```
reorganizado/
└── seccion-1-entendiendo-contenedores/
    ├── modulo-1-por-que-contenedores/
    ├── modulo-2-docker/
    ├── modulo-3-laboratorio-docker/
    └── modulo-4-containerd/
```

## Sección 1: Entendiendo los Contenedores

Proporciona una comprensión sólida de qué son los contenedores, por qué son útiles y la relación entre Docker y containerd, sentando las bases para Kubernetes.

### [Módulo 1: ¿Por qué Contenedores?](./seccion-1-entendiendo-contenedores/modulo-1-por-que-contenedores/)

Introduce el problema que resuelven los contenedores y compara diferentes enfoques de virtualización.

**Contenido:**
- 1.1 Beneficios de los contenedores y comparación con bare metal y VMs
- 1.2 Conceptos fundamentales (Imágenes, Contenedores, Capas, CoW)

### [Módulo 2: Docker - La Plataforma Pionera](./seccion-1-entendiendo-contenedores/modulo-2-docker/)

Cubre la historia, arquitectura y uso práctico de Docker.

**Contenido:**
- 2.1 Historia y orígenes de Docker
- 2.2 Arquitectura de Docker (Daemon, CLI, Images, Containers)
- 2.3 Demostración práctica con Docker
- 2.4 Docker Compose - Orquestación local
- 2.5 Almacenamiento en Docker (Volumes, Bind Mounts)
- 2.6 Redes en Docker

### [Módulo 3: Laboratorio con Docker](./seccion-1-entendiendo-contenedores/modulo-3-laboratorio-docker/)

Ejercicios prácticos hands-on para aplicar los conceptos aprendidos.

**Contenido:**
- Ejercicios básicos de Docker
- Ejercicios de Dockerfile (próximamente)
- Ejercicios de volúmenes (próximamente)
- Ejercicios de redes (próximamente)
- Ejercicios de Docker Compose (próximamente)
- Proyecto final (próximamente)

### [Módulo 4: Containerd y la Evolución del Runtime](./seccion-1-entendiendo-contenedores/modulo-4-containerd/)

Explora la estandarización OCI, containerd y su papel en Kubernetes.

**Contenido:**
- 4.1 La necesidad de estandarización - OCI
- 4.2 Introducción a containerd
- 4.3 Containerd como motor por defecto de Kubernetes
- 4.4 Laboratorio con containerd

## Cómo Navegar el Curso

1. **Comienza por la Sección 1:** Lee el [README principal](./seccion-1-entendiendo-contenedores/README.md) de la sección
2. **Sigue los módulos en orden:** Cada módulo tiene su propio README con la estructura de contenidos
3. **Haz los ejercicios:** Los laboratorios son fundamentales para el aprendizaje
4. **Consulta las demos:** Cada módulo incluye demostraciones prácticas

## Formato de los Archivos

- **README.md:** Índice y resumen del módulo/sección
- **Archivos numerados (.md):** Contenido teórico de cada tema
- **ejercicios-*.md:** Ejercicios prácticos con instrucciones paso a paso

## Próximos Pasos

Después de completar la Sección 1, estarás preparado para:
- Entender los fundamentos de la orquestación de contenedores
- Comenzar con Kubernetes
- Comprender la arquitectura de clusters
- Trabajar con pods, servicios y deployments

## Requisitos Técnicos

Para seguir el curso necesitarás:
- **Para Docker:** Docker Desktop o Docker Engine instalado
- **Para Containerd:** Acceso a una máquina Linux
- **Sistema operativo:** Windows, macOS o Linux
- **Recursos mínimos:** 4GB RAM, 20GB espacio en disco

## Recursos Adicionales

- [Documentación oficial de Docker](https://docs.docker.com/)
- [Documentación oficial de containerd](https://containerd.io/)
- [Docker Hub](https://hub.docker.com/)
- [OCI Specifications](https://opencontainers.org/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

## Estructura de Carpetas Detallada

```
reorganizado/
└── seccion-1-entendiendo-contenedores/
    ├── README.md
    │
    ├── modulo-1-por-que-contenedores/
    │   ├── README.md
    │   ├── 1.1-beneficios-contenedores.md
    │   └── 1.2-conceptos-fundamentales.md
    │
    ├── modulo-2-docker/
    │   ├── README.md
    │   ├── 2.1-historia-docker.md
    │   ├── 2.2-arquitectura-docker.md
    │   ├── 2.3-demo-practica-docker.md
    │   ├── 2.4-docker-compose.md
    │   ├── 2.5-almacenamiento-docker.md
    │   └── 2.6-redes-docker.md
    │
    ├── modulo-3-laboratorio-docker/
    │   ├── README.md
    │   └── ejercicios-basicos.md
    │
    └── modulo-4-containerd/
        ├── README.md
        ├── 4.1-estandarizacion-oci.md
        ├── 4.2-introduccion-containerd.md
        ├── 4.3-containerd-kubernetes.md
        └── 4.4-laboratorio-containerd.md
```

## Contribuciones

Este material está en constante evolución. Si encuentras errores o tienes sugerencias, por favor repórtalos.

## Licencia

Material educativo para el curso de Kubernetes.

---

**¡Bienvenido al curso! Comienza tu viaje en el mundo de los contenedores y Kubernetes.**
