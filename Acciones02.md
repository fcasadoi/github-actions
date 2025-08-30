Para **configurar un webhook desde Azure DevOps Server que dispare un pipeline en GitLab**, puedes seguir estos pasos. La idea es que cuando ocurra un evento en Azure DevOps (como un push o un merge), se envíe una notificación HTTP a GitLab, que luego puede iniciar un pipeline.

---

### 🧩 Paso 1: Crear el webhook en Azure DevOps Server

1. Ve a **Project Settings > Service Hooks**.
2. Haz clic en **+ Crear suscripción**.
3. Selecciona **Web Hooks** como servicio.
4. En la pantalla de **Desencadenador**, elige el evento que quieres usar (por ejemplo, *Code pushed* o *Pull request merged*).
5. En la pantalla de **Acción**, configura:
   - **URL de destino**: esta será la URL del endpoint en GitLab que puede iniciar un pipeline.
   - **Método HTTP**: normalmente `POST`.
   - **Encabezados personalizados**: si GitLab requiere autenticación, puedes incluir un token.
   - **Cuerpo del mensaje**: puedes personalizar el JSON que se enviará.

6. Haz clic en **Probar** para verificar la conexión.
7. Finaliza la configuración.

> ⚠️ Asegúrate de que la URL de GitLab sea pública y accesible desde Azure DevOps Server.

---

### 🔗 Paso 2: Configurar el endpoint en GitLab para recibir el webhook

GitLab no tiene un endpoint genérico para recibir webhooks externos que disparen pipelines directamente, pero puedes usar la **API de GitLab** para iniciar un pipeline:

#### Opción 1: Crear un endpoint personalizado en GitLab (por ejemplo, con GitLab Pages o un servidor intermedio)
- Este endpoint recibe el webhook de Azure DevOps.
- Luego hace una llamada a la API de GitLab para iniciar el pipeline.

#### Opción 2: Llamar directamente a la API de GitLab desde Azure DevOps

Usa esta URL como destino del webhook:

```
POST https://gitlab.com/api/v4/projects/<ID_DEL_PROYECTO>/trigger/pipeline
```

Con los siguientes parámetros en el cuerpo:

```json
{
  "token": "TOKEN_DE_DISPARO",
  "ref": "main"
}
```

> Puedes obtener el `token` desde **Settings > CI/CD > Pipeline triggers** en GitLab.

---

### 🧪 Paso 3: Validar y probar

- Realiza un push o merge en Azure DevOps.
- Verifica que el webhook se haya enviado correctamente.
- Confirma que el pipeline en GitLab se haya iniciado.

---

¿Quieres que te ayude a generar el JSON del webhook o el script para configurar el webhook en Azure DevOps Server? También puedo ayudarte a crear el token de disparo en GitLab si lo necesitas.