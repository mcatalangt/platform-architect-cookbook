# Security

## 1. Autenticación con Workload Identity Federation (WIF) en Google Cloud {: #workload-identity }

### ¿Qué es y para qué sirve?

**Workload Identity Federation (WIF)** es un mecanismo de gestión de identidades que permite a cargas de trabajo externas (como GitHub Actions, GitLab CI, AWS, Azure o servidores On-Premise) autenticarse y acceder a los recursos de Google Cloud de forma segura.

En lugar de requerir que almacenes contraseñas estáticas o archivos JSON en sistemas de terceros, WIF establece una relación de confianza basada en el protocolo **OIDC (OpenID Connect)** o SAML. Cuando tu pipeline de CI/CD necesita hacer un despliegue, solicita un token de corta duración a Google Cloud demostrando quién es mediante tokens criptográficos.

### Workload Identity Federation vs. Service Account Keys (Archivos JSON)

El uso de llaves estáticas (`.json`) es considerado un antipatrón en la seguridad de infraestructura moderna debido a los altos riesgos de exfiltración.

<div style="font-size: 0.60rem;">

| Característica | Llaves de Service Account (JSON) | Workload Identity Federation (WIF) |
| :--- | :--- | :--- |
| **Ciclo de Vida** | Permanente (hasta que se revoquen o expiren manualmente). | Temporal (Tokens de corta duración, típicamente 1 hora). |
| **Riesgo de Fuga** | Crítico. Si alguien roba el JSON, tiene acceso total desde cualquier lugar. | Prácticamente nulo. No hay secretos físicos que robar. |
| **Sobrecarga Operativa** | Alta. Requiere políticas de rotación de llaves, almacenamiento seguro y auditoría constante. | Cero. Google y el proveedor (ej. GitHub) manejan la negociación del token automáticamente. |
| **Granularidad de Acceso** | Baja. Quien tenga la llave tiene todos los permisos asignados a la cuenta. | Alta (ABAC). Puedes restringir el acceso para que solo un repositorio, rama o *tag* específico pueda asumir la identidad. |

</div>
---

### Guía de Configuración: GitHub Actions a Google Cloud

A continuación, se detallan los pasos para crear un túnel de confianza OIDC entre GitHub y un proyecto de Google Cloud utilizando `gcloud`.

#### Requisitos Previos
* Tener instalada y autenticada la CLI de Google Cloud (`gcloud`).
* Permisos de administrador (`roles/iam.workloadIdentityPoolAdmin` y `roles/iam.serviceAccountAdmin`) en el proyecto destino.

#### 1. Obtener el Número del Proyecto
Guarda el número de tu proyecto (no el ID alfanumérico) en una variable de entorno para facilitar los siguientes comandos.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash 
export PROJECT_ID="tu-id-de-proyecto"
export PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format="value(projectNumber)")
```
</div>

#### 2. Crear el Workload Identity Pool
El "Pool" es el contenedor lógico que agrupará las identidades externas.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash 
gcloud iam workload-identity-pools create "github-actions-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --display-name="GitHub Actions Pool"
```
</div>

#### 3. Crear el Proveedor OIDC
El "Provider" establece la conexión y define cómo se mapean los atributos del token de GitHub hacia Google Cloud.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash
gcloud iam workload-identity-pools providers create-oidc "github-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-actions-pool" \
  --display-name="GitHub OIDC Provider" \
  --attribute-mapping="google.subject=assertion.sub,attribute.actor=assertion.actor,attribute.repository=assertion.repository" \
  --issuer-uri="https://token.actions.githubusercontent.com"
```
</div>

#### 4. Vincular la Service Account con el Repositorio
Este paso autoriza a un repositorio específico de GitHub a generar tokens a nombre de tu Service Account de GCP.

!!! note
    Reemplaza `tu-organizacion/tu-repositorio` con los valores exactos.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash
export SERVICE_ACCOUNT="tu-service-account@${PROJECT_ID}.iam.gserviceaccount.com"
export REPO="tu-organizacion/tu-repositorio"

gcloud iam service-accounts add-iam-policy-binding "${SERVICE_ACCOUNT}" \
  --project="${PROJECT_ID}" \
  --role="roles/iam.workloadIdentityUser" \
  --member="principalSet://iam.googleapis.com/projects/${PROJECT_NUMBER}/locations/global/workloadIdentityPools/github-actions-pool/attribute.repository/${REPO}"
```
</div>


#### 5. Obtener el Identificador del Proveedor
Extrae la ruta completa del proveedor generado, la cual necesitarás en tu pipeline de GitHub.

<div style="font-size: 0.60rem; line-height: 1.2;">
```bash
gcloud iam workload-identity-pools providers describe "github-provider" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-actions-pool" \
  --format="value(name)"
```
</div>

!!! success "Obtención del identificador del proveedor"
    (Copia la salida de este comando, se verá como: `projects/123456789/locations/global/workloadIdentityPools/...`)






## 2. Checkov {: #checkov }

### ¿Qué es y para qué sirve?

Checkov es una herramienta de código abierto para la detección de vulnerabilidades de seguridad en código de infraestructura.