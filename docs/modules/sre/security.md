# Security

## 1. Autenticación con Workload Identity Federation (WIF) en Google Cloud {: #workload-identity }

### ¿Qué es y para qué sirve?

**Workload Identity Federation (WIF)** es un mecanismo de gestión de identidades que permite a cargas de trabajo externas (como GitHub Actions, GitLab CI, AWS, Azure o servidores On-Premise) autenticarse y acceder a los recursos de Google Cloud de forma segura.

En lugar de requerir que almacenes contraseñas estáticas o archivos JSON en sistemas de terceros, WIF establece una relación de confianza basada en el protocolo **OIDC (OpenID Connect)** o SAML. Cuando tu pipeline de CI/CD necesita hacer un despliegue, solicita un token de corta duración a Google Cloud demostrando quién es mediante tokens criptográficos.

### Workload Identity Federation vs. Service Account Keys (Archivos JSON)

El uso de llaves estáticas (`.json`) es considerado un antipatrón en la seguridad de infraestructura moderna debido a los altos riesgos de exfiltración.

| Característica | Llaves de Service Account (JSON) | Workload Identity Federation (WIF) |
| :--- | :--- | :--- |
| **Ciclo de Vida** | Permanente (hasta que se revoquen o expiren manualmente). | Temporal (Tokens de corta duración, típicamente 1 hora). |
| **Riesgo de Fuga** | Crítico. Si alguien roba el JSON, tiene acceso total desde cualquier lugar. | Prácticamente nulo. No hay secretos físicos que robar. |
| **Sobrecarga Operativa** | Alta. Requiere políticas de rotación de llaves, almacenamiento seguro y auditoría constante. | Cero. Google y el proveedor (ej. GitHub) manejan la negociación del token automáticamente. |
| **Granularidad de Acceso** | Baja. Quien tenga la llave tiene todos los permisos asignados a la cuenta. | Alta (ABAC). Puedes restringir el acceso para que solo un repositorio, rama o *tag* específico pueda asumir la identidad. |

---

### Guía de Configuración: GitHub Actions a Google Cloud

A continuación, se detallan los pasos para crear un túnel de confianza OIDC entre GitHub y un proyecto de Google Cloud utilizando `gcloud`.

#### Requisitos Previos
* Tener instalada y autenticada la CLI de Google Cloud (`gcloud`).
* Permisos de administrador (`roles/iam.workloadIdentityPoolAdmin` y `roles/iam.serviceAccountAdmin`) en el proyecto destino.

#### 1. Obtener el Número del Proyecto
Guarda el número de tu proyecto (no el ID alfanumérico) en una variable de entorno para facilitar los siguientes comandos.

```bash export PROJECT_ID="tu-id-de-proyecto"
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")
```


## 2. Checkov {: #checkov }

### ¿Qué es y para qué sirve?

Checkov es una herramienta de código abierto para la detección de vulnerabilidades de seguridad en código de infraestructura.