 # Cómo generar un **webhook dinámico desde Azure DevOps Server**
 
 Aquí tienes el ejemplo completo de cómo generar un **webhook desde Azure DevOps Server** que dispare un pipeline en GitLab y **pase dinámicamente la variable `SOURCE_BRANCH`** para que el pipeline haga un *merge* automático:

---

### 🔗 **Webhook en Azure DevOps Server**

**Método:** `POST`  
**URL:**  
```
https://gitlab.com/api/v4/projects/ID_DEL_PROYECTO/trigger/pipeline
```

**Headers personalizados:**
```json
{
  "Content-Type": "application/json"
}
```

**Body del webhook:**
```json
{
  "token": "TOKEN_DEL_TRIGGER",
  "ref": "main",
  "variables": {
    "SOURCE_BRANCH": "feature-xyz"
  }
}
```

> 🔐 Reemplaza `ID_DEL_PROYECTO` por el ID numérico de tu proyecto en GitLab, y `TOKEN_DEL_TRIGGER` por el token generado en **Settings > CI/CD > Pipeline triggers**.

---

### 📄 `.gitlab-ci.yml` correspondiente

Este archivo debe estar en el repositorio GitLab y permite que el pipeline use la variable `SOURCE_BRANCH`:

```yaml
stages:
  - merge
  - build
  - test

variables:
  TARGET_BRANCH: "main"
  GITLAB_TOKEN: $GITLAB_TOKEN  # Definido como variable protegida en GitLab

merge_branch:
  stage: merge
  script:
    - echo "Realizando merge de '$SOURCE_BRANCH' en '$TARGET_BRANCH'..."
    - curl --request POST "https://gitlab.com/api/v4/projects/$CI_PROJECT_ID/merge_requests" \
        --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
        --header "Content-Type: application/json" \
        --data "{
          \"source_branch\": \"$SOURCE_BRANCH\",
          \"target_branch\": \"$TARGET_BRANCH\",
          \"title\": \"Merge automático desde Azure DevOps\",
          \"remove_source_branch\": true
        }"
  only:
    - triggers

build_job:
  stage: build
  script:
    - echo "Compilando el proyecto..."

test_job:
  stage: test
  script:
    - echo "Ejecutando tests..."
```

---

¿Quieres que te ayude a obtener el ID del proyecto en GitLab o generar el token de trigger? También puedo ayudarte a probar el webhook con una herramienta como Webhook.site o simularlo con `curl`. ¿Cómo te gustaría continuar?