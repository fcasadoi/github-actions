# **Estrategias de sincronización de repositorios DevOps y GitLab**
Sincronizar repositorios entre **Azure DevOps** y **GitLab** puede hacerse de varias formas, dependiendo de si quieres una sincronización **unidireccional** (por ejemplo, solo desde DevOps hacia GitLab) o **bidireccional**. Aquí te explico las opciones más comunes:

## Opción 1: Sincronización unidireccional usando `mirroring`

### Desde GitLab (pull desde Azure DevOps)
1. Ve a tu proyecto en GitLab.
2. En **Settings > Repository > Mirroring repositories**, selecciona:
   - **Mirror direction**: *Pull*
   - **Git repository URL**: la URL del repositorio en Azure DevOps (usa HTTPS).
   - Añade las credenciales (usuario y token personal de Azure DevOps).
3. GitLab hará *pull* automáticamente desde Azure DevOps cada cierto tiempo.

> Ideal si quieres que GitLab refleje los cambios hechos en Azure DevOps.


## Opción 2: Sincronización unidireccional
### Desde Azure DevOps (push hacia GitLab)

1. En Azure DevOps, crea un **Service Connection** con GitLab (usando token personal).
2. En tu pipeline YAML, añade un paso para hacer `git push` hacia GitLab:

```yaml
- script: |
    git remote add gitlab https://oauth2:<GITLAB_TOKEN>@gitlab.com/usuario/proyecto.git
    git push gitlab main
  displayName: 'Push a GitLab'
```

> Útil si quieres que Azure DevOps envíe los cambios a GitLab como parte de una pipeline.


## Opción 3: Sincronización bidireccional

Más compleja y requiere:
- Configurar **webhooks** en ambos sistemas.
- Usar pipelines que hagan `git pull` y `git push` entre ambos repositorios.
- Resolver conflictos de forma manual o automatizada.

> Requiere control de versiones muy cuidadoso para evitar conflictos.

#### Recomendaciones de seguridad

- Usa **tokens personales** con permisos mínimos necesarios.
- Evita sincronizar ramas protegidas sin revisión.
- Configura notificaciones para detectar errores de sincronización.

# Webhook que sincronice repositorios entre Azure DevOps Server y GitLab
Para preparar un **webhook que sincronice repositorios entre Azure DevOps Server y GitLab**, lo más directo es configurar un webhook en Azure DevOps que, al detectar un evento (como un push), **envíe una solicitud HTTP a GitLab** para que este actualice su repositorio desde Azure DevOps.

## Webhook desde Azure DevOps Server hacia GitLab

### Objetivo
Cuando se haga un *push* en Azure DevOps, GitLab debe hacer un *pull* desde Azure DevOps para sincronizar su contenido.

### Configuración del webhook

**Método:** `POST`  
**URL:**  
```
https://gitlab.com/api/v4/projects/ID_DEL_PROYECTO/mirror/pull
```

> Esta URL funciona si tienes configurado [**repository mirroring**](/docs/02PasoAPaso.md/#opción-1-sincronización-unidireccional-usando-mirroring) en GitLab con dirección *pull* desde Azure DevOps.

**Headers personalizados:**
```json
{
  "PRIVATE-TOKEN": "TOKEN_PERSONAL_GITLAB"
}
```

**Body (opcional):**
```json
{}
```

## Requisitos previos

1. **En GitLab**:
   - Ve a **Settings > Repository > Mirroring repositories**.
   - Añade la URL del repositorio de Azure DevOps (HTTPS).
   - Usa credenciales válidas (usuario + token personal de Azure DevOps).
   - Selecciona **Mirror direction: Pull**.

2. **En Azure DevOps Server**:
   - Ve a **Project Settings > Service Hooks**.
   - Crea un nuevo webhook que se active con el evento deseado (por ejemplo, *Code pushed*).
   - Configura la URL y los headers como se indica arriba.

### Seguridad

- Usa tokens personales con permisos mínimos necesarios.
- Protege las URLs y tokens en variables seguras.
- Verifica que el repositorio en GitLab tenga permisos para acceder al de Azure DevOps.

---

# **Configurar un webhook desde Azure DevOps Server que dispare un pipeline en GitLab**

La idea es que cuando ocurra un evento en Azure DevOps (como un push o un merge), se envíe una solicitud HTTP a GitLab para iniciar un pipeline.

## **Flujo general**

1. [**Configurar el repositorio local**](/docs/02PasoAPaso.md/#1-configurar-el-repositorio-local) → push rama local en Devops y GitLab.
2. **Azure DevOps Server** → Detecta un evento (ej. push).
3. **Webhook** → Envía una solicitud HTTP POST a GitLab.
4. **GitLab** → Recibe la solicitud y dispara un pipeline usando su API.

## **Paso a paso**

### 1. **Configurar el repositorio local**
en el directorio **.\\.git\\** del repositorio, en el fichero **config**, añadir:
```
[remote "origin"]
	url = https://tfsapp.tracasa.es:8088/tfs/ISSICollection/SIC_DataHub/_git/project_name
	fetch = +refs/heads/*:refs/remotes/origin/*
	pushurl = https://tfsapp.tracasa.es:8088/tfs/ISSICollection/SIC_DataHub/_git/project_name
	pushurl = https://gesfuentes.admon-cfnavarra.es/repos/GoberNa/project_name.git



[user]
	name = D429662
	email = fcasado@itracasa.es

```
Con esto conseguimos, que al pushear la rama a Devops la rama también esté pusheada a GitLab.
### 2. **Crea pipeline con rama dinámica en GitLab**
#### 2.1. **Crea el archivo `.gitlab-ci.yml`**
En la raíz de tu repositorio, crea un archivo llamado:

```
.gitlab-ci.yml
```

#### 2.2. **Define los stages y jobs**
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
#### 2.3. **Hacer commit del archivo**
Una vez creado el archivo, haz commit y push al repositorio:

```bash
git add .gitlab-ci.yml
git commit -m "Añadir pipeline básica"
git push origin main
```

GitLab detectará automáticamente el archivo y ejecutará la pipeline según lo definido.

### 3. **Requisitos para que funcione correctamente**

1. **Token de acceso personal (`GITLAB_TOKEN`)**:
   - Crea un token en GitLab con permisos de `api` y configúralo como variable protegida en el proyecto (`Settings > CI/CD > Variables`).

2. **Rama `\"$SOURCE_BRANCH\"` existente**:
   - Asegúrate de que la rama que se va a fusionar (`\"$SOURCE_BRANCH\"`) exista en el repositorio de GitLab.

3. **Webhook en Azure DevOps Server**:
    1. Selecciona la **organización** y luego el **proyecto** donde quieres configurar el webhook.
    2. Ve a **Project Settings > Service Hooks**.
    3. Haz clic en **+ Crear suscripción**.
    4. Selecciona **Web Hooks** como servicio.
    5. En la pantalla de **Desencadenador**, elige el evento que quieres usar (por ejemplo, *Code pushed* o *Pull request merged*).
    6. En la pantalla de **Acción**, configura:
       - **URL de destino**: la URL del endpoint en GitLab que puede iniciar un pipeline -> https://gitlab.com/api/v4/projects/ID_DEL_PROYECTO/trigger/pipeline
       - **Método HTTP**: normalmente `POST`.
       - **Encabezados personalizados**: si GitLab requiere autenticación, puedes incluir un token.
       - **Formato del cuerpo del mensaje**
       - **Cuerpo del mensaje**: puedes personalizar el JSON que se enviará:
     ```
        {
           "token": "TOKEN_DEL_TRIGGER",
           "ref": "main",
           "variables": {
               "SOURCE_BRANCH": "feature-xyz"
            }
        }
     ```

    7. Haz clic en **Probar** para verificar la conexión.
    8. Finaliza la configuración.

> Asegúrate de que la URL de GitLab sea pública y accesible desde Azure DevOps Server.


#### **Ejemplo de configuración del webhook en Azure DevOps**
- **URL**: `https://gitlab.com/api/v4/projects/123456/trigger/pipeline`
- **Headers personalizados**:
  ```json
  {
    "Content-Type": "application/json"
  }
  ```
- **Body**:
  ```json
  {
    "token": "abc123xyz",
    "ref": "main"
  }
  ```

- **Seguridad**
    - Usa tokens de trigger seguros en GitLab.
    - Puedes restringir el uso del token a ciertos eventos o ramas.

4. **API de GitLab para disparar el pipeline**

    GitLab no tiene un endpoint genérico para recibir webhooks externos que disparen pipelines directamente, pero puedes usar la **API de GitLab** para iniciar un pipeline:

    #### Opción 1: Crear un endpoint personalizado en GitLab
    Por ejemplo, con GitLab Pages o un servidor intermedio:
    - Este endpoint recibe el webhook de Azure DevOps.
    - Luego hace una llamada a la API de GitLab para iniciar el pipeline.

    #### Opción 2: Llamar directamente a la API de GitLab desde Azure DevOps

    Usa esta URL como destino del webhook:
    
    ```bash
    POST https://gitlab.com/api/v4/projects/<ID_DEL_PROYECTO>/    trigger/pipeline
    ```
    
    Con los siguientes parámetros en el cuerpo:
    
    ```json
    {
      "token": "TOKEN_DE_DISPARO",
      "ref": "main"
    }
    ```

    > Puedes obtener el `token` desde **Settings > CI/CD > Pipeline triggers** en GitLab.


### 4. **Validar y probar**

- Realiza un push o merge en Azure DevOps.
- Verifica que el webhook se haya enviado correctamente.
- Confirma que el pipeline en GitLab se haya iniciado.

