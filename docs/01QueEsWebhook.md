# Webhook de Azure DevOps

Un **webhook de Azure DevOps** es una forma de integrar Azure DevOps con otros sistemas o servicios externos mediante el envío automático de mensajes HTTP (normalmente en formato JSON) cuando ocurre un evento específico dentro de Azure DevOps.

### ¿Para qué se usa un webhook?
Los webhooks permiten **automatizar flujos de trabajo** y **notificar a otros sistemas** cuando suceden cosas como:
- Se crea o actualiza un *work item* (elemento de trabajo).
- Se realiza un *push* a un repositorio.
- Se completa una *build* o *release*.
- Se crea un *pull request*.

### ¿Cómo funciona?
1. **Configuración**: En Azure DevOps, se configura un webhook especificando:
   - El tipo de evento que lo activará.
   - La URL del servicio externo que recibirá la notificación.
2. **Activación**: Cuando ocurre el evento, Azure DevOps envía una solicitud HTTP POST a la URL configurada.
3. **Procesamiento**: El servicio externo recibe los datos y puede realizar acciones como registrar el evento, iniciar un proceso, enviar una alerta, etc.

### Ejemplo de uso
Supongamos que tienes una aplicación que necesita actualizarse cada vez que se hace un *push* en el repositorio. Puedes configurar un webhook para que, al detectar ese evento, llame a tu API y esta inicie el proceso de despliegue.