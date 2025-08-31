Ejemplo de pipeline básico en GitLab que puede ser **disparado por un trigger token desde Azure DevOps Server mediante webhook**, y que además realiza un **merge automático de una rama en `main`**.

---

### 📄 `.gitlab-ci.yml con rama dinámica`

```yaml
stages:
  - merge
  - build
  - test

variables:
  TARGET_BRANCH: "main"
  GITLAB_TOKEN: $GITLAB_TOKEN  # Token de acceso personal, definido en GitLab CI/CD

merge_branch:
  stage: merge
  script:
    - echo "Realizando merge de la rama '$SOURCE_BRANCH' en '$TARGET_BRANCH'..."
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

### 🛠️ Requisitos para que funcione correctamente

1. **Token de acceso personal (`GITLAB_TOKEN`)**:
   - Crea un token en GitLab con permisos de `api` y configúralo como variable protegida en el proyecto (`Settings > CI/CD > Variables`).

2. **Rama `feature` existente**:
   - Asegúrate de que la rama que se va a fusionar (`feature`) exista en el repositorio.

3. **Webhook en Azure DevOps Server**:
   - Configura el webhook para hacer un `POST` a:
     ```
     https://gitlab.com/api/v4/projects/ID_DEL_PROYECTO/trigger/pipeline
     ```
   - Con el cuerpo:
     ```json
    {
        "token": "TOKEN_DEL_TRIGGER",
        "ref": "main",
        "variables": {
            "SOURCE_BRANCH": "feature-xyz"
        }
    }
     ```

---
✅ Requisitos
- El token (GITLAB_TOKEN) debe tener permisos para crear merge requests.
- La rama SOURCE_BRANCH debe existir en el repositorio.
- El trigger debe estar configurado para aceptar variables.