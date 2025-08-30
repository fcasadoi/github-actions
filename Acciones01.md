Para configurar un **webhook entre Azure DevOps Server y GitLab**, de forma que eventos en GitLab (como un push o merge request) disparen un pipeline en Azure DevOps, puedes seguir estos pasos:

---

### 🔧 **Paso 1: Crear Webhook en Azure DevOps Server**

1. Ve a **Project Settings > Service Hooks** en tu proyecto de Azure DevOps.
2. Haz clic en **+ Crear suscripción**.
3. Selecciona **Web Hooks** como servicio.
4. Elige el evento que quieres que dispare el webhook (por ejemplo, "Code pushed").
5. En la pantalla de acción:
   - Introduce la **URL del webhook** que GitLab usará para enviar eventos.
   - Configura los detalles del recurso, mensajes y nivel de detalle del JSON.
6. Haz clic en **Test** para verificar la conexión.
7. Finaliza la configuración [1](https://learn.microsoft.com/en-us/azure/devops/service-hooks/services/webhooks?view=azure-devops).

---

### 🔗 **Paso 2: Crear Webhook en GitLab**

1. Ve a tu proyecto en GitLab.
2. En el menú lateral, selecciona **Settings > Webhooks**.
3. En el campo **URL**, introduce la URL del webhook de Azure DevOps:
   ```
   https://dev.azure.com/<ORGANIZATION>/_apis/public/distributedtask/webhooks/<WEBHOOK_NAME>?api-version=6.0-preview
   ```
4. Selecciona los eventos que deseas que disparen el webhook (por ejemplo, "Push events").
5. Opcionalmente, añade un **token secreto** para validar la autenticidad del webhook.
6. Haz clic en **Add webhook** y luego en **Test** para verificar que se envía correctamente [2](https://docs.gitlab.com/user/project/integrations/webhooks/).

---

### 🧪 **Paso 3: Configurar el Pipeline en Azure DevOps**

En tu archivo `azure-pipelines.yml`, debes declarar el webhook como recurso:

```yaml
resources:
  webhooks:
    - webhook: ADOWebHook
      connection: ADOWebHookSvcCnn

trigger: none

pr:
  branches:
    include:
      - main

jobs:
- job: RunFromWebhook
  steps:
    - script: echo "Triggered by GitLab webhook"
```

Este pipeline se activará solo cuando reciba el evento desde GitLab [3](https://stackoverflow.com/questions/77879532/how-to-trigger-azuredevops-pipeline-from-gitlab-through-gitlab-webhook).

---

### 🛠️ Alternativa: Usar la API REST de Azure DevOps

Si no puedes usar YAML o necesitas más control, puedes hacer un **POST** desde GitLab a la API de Azure DevOps:

```http
POST https://dev.azure.com/{organization}/{project}/_apis/pipelines/{pipelineId}/runs?api-version=7.1-preview.1
```

Este método requiere autenticación con un **Personal Access Token (PAT)** de Azure DevOps [3](https://stackoverflow.com/questions/77879532/how-to-trigger-azuredevops-pipeline-from-gitlab-through-gitlab-webhook).

---

¿Quieres que te ayude a generar el webhook URL exacto para tu organización y proyecto? ¿O prefieres que prepare el YAML completo para tu pipeline?