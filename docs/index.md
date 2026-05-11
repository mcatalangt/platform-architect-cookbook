#Platform Architect's Cookbook

Bienvenido a **Stack Kitchen.**  Recetas, arquitecturas e implementaciones probadas en batalla, desde los cimientos de Kubernetes hasta Inteligencia Artificial.

Diseñado para ingenieros que buscan agilidad, Stack Kitchen permite desplegar desde entornos de pruebas (Sandbox) hasta arquitecturas de **Alta Disponibilidad** (HA) en cuestión de minutos. 

Todo el código sigue las mejores prácticas de **SRE**, **DevOps**, **Ingeniería de Datos** y **ML**, garantizando infraestructuras seguras y resilientes.

Muchas de las buenas practicas y herramientas son parte del Cloud Native Landscape


##Cloud Native Landscape

Es el "mapa" oficial mantenido por la **CNCF (Cloud Native Computing Foundation)** que organiza el inmenso ecosistema de herramientas necesarias para construir, desplegar y gestionar aplicaciones modernas en la nube.

En resumen, es la guía definitiva para pasar de servidores tradicionales a arquitecturas de microservicios, contenedores y automatización.

Para **The Platform Architect's Cookbook**, el landscape se divide en capas clave:

###1. Provisioning (infraestructura Core)

**Qué hace:** Crea y endurece la base sobre la que corren las apps.

**Herramientas clave:** Terraform (Automatización), Ansible, Keycloak (Seguridad/Identidad).

###2. Runtime (Entorno de ejecución de contenedores)

**Qué hace:** Gestiona el almacenamiento, la red y la ejecución de contenedores.

**Herramientas clave:** Kubernetes (Orquestación), Docker, Containerd, CRI-O.

###3. Orchestration & Management (Conexión entre servicios)

**Qué hace:** Service Mesh, descubrimiento de servicios y coordinación.

**Herramientas clave:** Istio, CoreDNS, etcd, Envoy.

###4. App Definition & Development (IC/DC)

**Qué hace:** CI/CD, bases de datos y gestión de imágenes.

**Herramientas clave:** ArgoCD, Helm, GitLab CI, Harbor.

###5. Observability & Analysis

**Qué hace:** Monitoreo, Logging y Tracing para saber qué pasa en tiempo real.

**Herramientas clave:** Prometheus, Grafana, ELK Stack (Elasticsearch, Logstash, Kibana), Jaeger.


[https://landscape.cncf.io/](https://landscape.cncf.io/)
![Cloud Native Landscape](assets/landscape.png){ align=center width="100%" }