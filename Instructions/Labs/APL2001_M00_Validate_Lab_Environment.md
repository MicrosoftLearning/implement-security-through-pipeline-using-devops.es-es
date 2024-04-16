---
lab:
  title: Validación del entorno de laboratorio
  module: 'Module 0: Welcome'
---

# Validación del entorno de laboratorio

En la preparación para los laboratorios, es fundamental tener el entorno configurado correctamente. Esta página le guiará a través del proceso de configuración y garantiza que se cumplen todos los requisitos previos.

- Este laboratorio requiere **Microsoft Edge** o un [explorador compatible con Azure DevOps](https://learn.microsoft.com/azure/devops/server/compatibility?view=azure-devops#web-portal-supported-browsers).

- **Configurar una suscripción de Azure:** si aún no tiene una suscripción de Azure, cree una siguiendo las instrucciones de esta página o visite [https://azure.microsoft.com/free](https://azure.microsoft.com/free) para registrarse de forma gratuita.

- **Configurar una organización de Azure DevOps:**: si aún no tiene una organización Azure DevOps que pueda usar para este laboratorio, cree una siguiendo las instrucciones disponibles en [Creación de una organización o colección de proyectos](https://learn.microsoft.com/azure/devops/organizations/accounts/create-organization).
  
- Página de descarga de [Git para Windows](https://gitforwindows.org/). Esto se instalará como parte de los requisitos previos para este laboratorio.

- [Visual Studio Code](https://code.visualstudio.com/). Esto se instalará como parte de los requisitos previos para este laboratorio.

- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli). Instale la CLI de Azure en las máquinas del agente autohospedado.

## Instrucciones para crear una organización de Azure DevOps (solo tiene que hacerlo una vez)

> **Nota**: Comience en el paso 3 si ya tiene **cuenta de Microsoft** configurada y una suscripción activa de Azure vinculada a esa cuenta.

1. Use una sesión privada del explorador para obtener una nueva **cuenta de Microsoft (MSA) personal** en `https://account.microsoft.com`.

1. Con la misma sesión del explorador, regístrese para obtener una suscripción gratuita de Azure en `https://azure.microsoft.com/free`.

1. Abra un explorador y vaya a Azure Portal en `https://portal.azure.com` y busque **Azure DevOps** en la parte superior de la pantalla de Azure Portal. En la página que aparece, haga clic en **Organizaciones de Azure DevOps**.

1. Después haz clic en el enlace con la etiqueta **My Azure DevOps Organizations ** o ve directamente a `https://aex.dev.azure.com`.

1. En la página **Necesitamos más detalles**, seleccione **Continuar**.

1. En el cuadro desplegable de la izquierda, seleccione **Directorio predeterminado**, en lugar de **Cuenta de Microsoft**.

1. Si se le solicita (*“Necesitamos más detalles”),* proporcione su nombre, dirección de correo electrónico y ubicación, y haga clic en **Continuar**.

1. De nuevo en `https://aex.dev.azure.com` con **el directorio predeterminado** seleccionado, haz clic en el botón azul **Crear nueva organización.**

1. Acepta los *Términos de servicio* haciendo clic en **Continuar**.

1. Si aparece un mensaje (*“Casi listo”),* deje el nombre de la organización de Azure DevOps de forma predeterminada (debe ser un nombre único global) y elija una ubicación de hospedaje cercana a usted en la lista.

1. Una vez que se abra la organización recién creada en **Azure DevOps**, seleccione **Configuración de la organización** en la esquina inferior izquierda.

1. En la pantalla **Configuración de la organización**, seleccione **Facturación** (abrir esta pantalla tarda unos segundos).

1. Seleccione **Configurar facturación** y, en el lado derecho de la pantalla, seleccione la **suscripción de Azure** y, a continuación, seleccione **Guardar** para vincular la suscripción con la organización.

1. Una vez que la pantalla muestre el identificador de suscripción de Azure vinculado en la parte superior, cambie el número de **trabajos paralelos de pago** de **CI/CD hospedados de MS** de 0 a **1**. Después, haga clic en el botón **Guardar** de la parte inferior.

1. Puede **esperar al menos 3 horas antes de usar las funcionalidades de CI/CD** para que la nueva configuración se refleje en el back-end. De lo contrario, verá el mensaje *“No se ha comprado o concedido ningún paralelismo hospedado”.*

## Instrucciones para crear y configurar el proyecto de Azure DevOps (solo tiene que hacerlo una vez)

> **Nota**: Asegúrate de completar los pasos necesarios para crear la organización de Azure DevOps antes de continuar con estos pasos.

Para seguir todas las instrucciones del laboratorio, deberá configurar un nuevo proyecto de Azure DevOps, crear un repositorio basado en la aplicación [eShopOnWeb](https://github.com/MicrosoftLearning/eShopOnWeb) y crear una conexión de servicio a la suscripción de Azure.

### Creación del proyecto de equipo

En primer lugar, creará un proyecto **eShopOnWeb** de Azure DevOps que se usará en varios laboratorios.

1. Abra su explorador y navegue a su organización de Azure DevOps.

1. Seleccione la opción **Nuevo proyecto** y use la siguiente configuración:
   - Nombre: **eShopOnWeb**
   - visibilidad: **Privado**
   - Avanzado: Control de versiones: **Git**
   - Avanzado: Proceso de elemento de trabajo: **Scrum**

1. Seleccione **Crear**.

   ![Crear proyecto](media/create-project.png)

### Importación del repositorio de Git de eShopOnWeb

Ahora, importará eShopOnWeb en el repositorio de Git.

1. Abra su explorador y navegue a su organización de Azure DevOps.

1. Abra el proyecto **eShopOnWeb** creado anteriormente.

1. Seleccione **Repositorios > Archivos**, **Importar un repositorio** y, después, seleccione **Importar**.

1. En la ventana **Importar un repositorio de Git**, pegue la siguiente dirección URL `https://github.com/MicrosoftLearning/eShopOnWeb` y seleccione **Importar**:

   ![Importar repositorio](media/import-repo.png)

1. El repositorio se organiza de la siguiente manera:

   - La carpeta **.ado** contiene canalizaciones de YAML de Azure DevOps.
   - El contenedor de carpetas **.devcontainer** está configurado para realizar el desarrollo con contenedores (ya sea localmente en VS Code o GitHub Codespaces).
   - La carpeta **.azure** contiene infraestructura de la plantilla de ARM y Bicep como plantillas de código.
   - Definiciones de flujo de trabajo de GitHub del contenedor de carpetas **.github**.
   - La carpeta **src** contiene el sitio web de .NET 6 que se utiliza en los escenarios de laboratorio. 

1. Deje abierta la ventana del explorador web.  

### Creación de una entidad de servicio y una conexión de servicio para acceder a los recursos de Azure

A continuación, creará una entidad de servicio mediante la CLI de Azure, y una conexión de servicio en Azure DevOps que le permitirá implementar y acceder a los recursos de la suscripción de Azure.

1. Inicie un explorador web, vaya a Azure Portal en `https://portal.azure.com`, e inicie sesión con la cuenta de usuario que tenga el rol Propietario en la suscripción de Azure que va a usar en los laboratorios de este curso, así como el rol Administrador global en el inquilino de Microsoft Entra asociado a esta suscripción.

1. En Azure Portal, seleccione el botón **Cloud Shell** situado en la parte a la derecha del cuadro de búsqueda de la parte superior de la página.

1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   > [!NOTE]
   > Si es la primera vez que inicia **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, seleccione la suscripción que utiliza en este laboratorio y seleccione **Crear almacenamiento**.

1. En la solicitud de **Bash**, en el panel de **Cloud Shell**, ejecute los siguientes comandos para recuperar los valores del identificador de suscripción de Azure y los atributos de nombre de suscripción:

   ```bash
   subscriptionName=$(az account show --query name --output tsv)
   subscriptionId=$(az account show --query id --output tsv)
   echo $subscriptionName
   echo $subscriptionId
   ```

   > [!NOTE]
   > Copie ambos valores en un archivo de texto. Los necesitará en los laboratorios de este curso.

1. En la solicitud de **Bash**, en el panel de **Cloud Shell**, ejecute el siguiente comando para crear una entidad de servicio:

   ```bash
   az ad sp create-for-rbac --name sp-eshoponweb-azdo --role contributor --scopes /subscriptions/$subscriptionId
   ```

   > [!NOTE]
   > El comando generará una salida JSON. Copie los resultados en un archivo de texto. Lo necesitará en breve.

   > [!NOTE]
   > Registre el valor de, el nombre de la entidad de seguridad, su identificador y el identificador de inquilino incluidos en la salida JSON. Los necesitará en los laboratorios de este curso.

1. Vuelva a la ventana del explorador web que muestra el portal de Azure DevOps con el proyecto **eShopOnWeb** abierto y seleccione **Configuración del proyecto** en la esquina inferior izquierda del portal.

1. En Canalizaciones, seleccione **Conexiones de servicio** y, después, seleccione **Crear conexión de servicio**.

   ![Captura de pantalla del botón para crear la nueva conexión de servicio.](media/new-service-connection.png)

1. En la hoja **New service connection (Nueva conexión de servicio)**, seleccione **Azure Resource Manager** y, después, seleccione **Next (Siguiente)** (es posible que deba desplazarse hacia abajo).

1. Seleccione **Service Principal (Entidad de servicio) (manual)** y, después, seleccione **Next (Siguiente)**.

1. Rellene los campos vacíos con la información recopilada durante los pasos anteriores:

   - Identificador y nombre de la suscripción.
   - Id. de entidad de servicio (o clientId/AppId), clave de entidad de servicio (o contraseña) y TenantId.
   - En **Nombre de conexión de servicio**, escriba **azure subs**. Se hará referencia a este nombre en las canalizaciones de YAML para hacer referencia a la conexión de servicio con el fin de acceder a la suscripción de Azure.

   ![Captura de pantalla de la configuración de conexión de servicios de Azure](media/azure-service-connection.png)

1. No marque **Conceder permiso de acceso a todas las canalizaciones**. Seleccione **Verificar y guardar**.

   > [!NOTE]
   > No se recomienda la opción **Conceder permiso de acceso a todas las canalizaciones** para entornos de producción. Solo se usa en este laboratorio para simplificar la configuración de la canalización.

Ya ha completado los pasos previos necesarios para continuar con los laboratorios.
