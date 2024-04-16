---
lab:
  title: 'Laboratorio: Administrar la identidad para proyectos y canalizaciones'
  module: 'Module 2: Manage identity for projects, pipelines, and agents'
---

# Laboratorio: Administrar la identidad para proyectos y canalizaciones

Las identidades administradas ofrecen un método seguro para controlar el acceso a los recursos de Azure. Azure controla estas identidades automáticamente, lo que le permite comprobar el acceso a los servicios compatibles con la autenticación de Azure AD. Esto significa que no es necesario insertar credenciales en el código, lo que mejora la seguridad. En Azure DevOps, las identidades administradas pueden autenticar los recursos de Azure dentro de los agentes autohospedados, lo que simplifica el control de acceso sin poner en peligro la seguridad.

En este laboratorio, creará una identidad administrada y la usará en canalizaciones de YAML de Azure DevOps que se ejecutan en agentes autohospedados para implementar recursos de Azure.

El laboratorio dura aproximadamente **45** minutos.

## Antes de comenzar

Necesitará una suscripción de Azure, una organización de Azure DevOps y la aplicación eShopOnWeb para seguir los laboratorios.

- Siga los pasos para [validar el entorno de laboratorio](APL2001_M00_Validate_Lab_Environment.md).
- Compruebe que tiene una cuenta de Microsoft o de Microsoft Entra con el rol Propietario o Colaborador en la suscripción a Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).
- Complete el laboratorio [Configurar agentes y grupos de agentes para canalizaciones seguras](APL2001_M03_L02_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md).

## Instrucciones

### Ejercicio 1: Importación y ejecución de canalizaciones de CI/CD.

En este ejercicio, importará y ejecutará la canalización de CI, configurará la conexión de servicio con la suscripción de Azure y, a continuación, importará y ejecutará la canalización de CD.

#### Tarea 1: importar y ejecutar la canalización de CI

Empecemos importando la canalización de CI denominada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra el proyecto **eShopOnWeb** .

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Seleccione el botón **Crear canalización**.

1. Selecciona **GIT de Azure Repos (YAML)**.

1. Selecciona el repositorio **eShopOnWeb**.

1. Selecciona el **archivo YAML de Azure Pipelines existente**.

1. Seleccione el archivo **/.ado/eshoponweb-ci.yml** y haga clic en **Continuar**.

1. Haga clic en el botón **Run (Ejecutar)** para ejecutar la canalización.

   > [!NOTE]
   > La canalización tomará un nombre basado en el nombre del proyecto. Cámbielo para identificar mejor la canalización.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**, seleccione la canalización creada recientemente, seleccione los puntos suspensivos y, después, seleccione la opción **Rename/move (Cambiar nombre/mover)**.

1. Asígnele el nombre **eshoponweb-ci** y seleccione **Guardar**.

#### Tarea 2: Importar y ejecutar la canalización de CD

> [!NOTE]
> En esta tarea, importará y ejecutará la canalización de CD denominada [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. En el panel **Canalizaciones** del proyecto **eShopOnWeb**, seleccione el botón **Nueva canalización**.

1. Selecciona **GIT de Azure Repos (YAML)**.

1. Selecciona el repositorio **eShopOnWeb**.

1. Seleccione **Archivo YAML de Azure Pipelines existente**.

1. Seleccione el archivo **/.ado/eshoponweb-cd-webapp-code.yml** y, después, seleccione **Continuar**.

1. En la definición de canalización de YAML, establezca la sección variables de la siguiente manera:

   ```yaml
   variables:
     resource-group: 'AZ400-EWebShop-NAME'
     location: 'southcentralus'
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs'
     webappname: 'az400-webapp-NAME'
   ```

1. En la sección de variables, reemplace los marcadores de posición del comando anterior por los valores siguientes:

   - **AZ400-EWebShop-NAME** con el nombre que prefiera, por ejemplo, **rg-eshoponweb**.
   - **location** con el nombre de la región de Azure que desea implementar los recursos, por ejemplo, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** por el id. de la suscripción a Azure.
   - **az400-webapp-NAME** con un nombre único global de la aplicación web que se va a implementar, por ejemplo, la cadena **eshoponweb-lab-id-** seguida de un número aleatorio de seis dígitos. 

1. Seleccione **Guardar y ejecutar** y elija confirmar directamente en la rama principal.

1. Seleccione **Save and Run (Guardar y ejecutar)** de nuevo.

1. Abra la canalización. Si ve el mensaje “This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp (Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con Implementar en WebApp)”, seleccione **View (Ver)**, **Permit (Permitir)** y **Permit (Permitir)** de nuevo. Esto es necesario para permitir que la canalización cree el recurso de Azure App Service.

   ![Captura de pantalla del permiso de acceso desde la canalización de YAML.](media/pipeline-deploy-permit-resource.png)

1. La implementación puede tardar unos minutos en completarse y esperar a que se ejecute la canalización. La definición de CD consta de las siguientes tareas:

   - **AzureResourceManagerTemplateDeployment**: Implementa la aplicación web de Azure App Service mediante una plantilla de Bicep.
   - **AzureRmWebAppDeployment**: Publica el sitio web en la aplicación web de Azure App Service.

   > [!NOTE]
   > En caso de que se produzca un error en la implementación, vaya a la página de ejecución de la canalización y seleccione **Volver a ejecutar trabajos con errores** para invocar otra ejecución de canalización.

   > [!NOTE]
   > La canalización tomará un nombre en función del nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**, seleccione la canalización creada recientemente, seleccione los puntos suspensivos y, después, seleccione la opción **Rename/move (Cambiar nombre/mover)**.

1. Asígnele el nombre **eshoponweb-cd-webapp-code** y, después, seleccione **Guardar**.

### Ejercicio 2: Configurar la identidad administrada en canalizaciones de Azure

En este ejercicio, usará una identidad administrada para configurar una nueva conexión de servicio e incorporarla a las canalizaciones de CI/CD.

#### Tarea 1: Configurar un agente autohospedado para usar la identidad administrada y actualizar la canalización de CI

1. En el explorador, abra Azure Portal desde `https://portal.azure.com`.

1. En Azure Portal, vaya a la página que muestra la máquina virtual de Azure **eshoponweb-vm** que ha implementado en este laboratorio

1. En la página de la máquina virtual de Azure **eshoponweb-vm**, en la barra de herramientas, seleccione **Iniciar** para iniciarla.

1. En la página de la máquina virtual de Azure **eshoponweb-vm**, en el lado izquierdo, en la sección **Seguridad**, seleccione **Identidad**.

1. En la página **eshoponweb-vm /| Identidad**, compruebe que **Estado** está **Activado** y seleccione **Asignaciones de roles de Azure**.

1. Seleccione el botón **Agregar asignación de roles** y realice las siguientes acciones:

   | Configuración | Acción |
   | -- | -- |
   | Lista desplegable de **Ámbito** | Seleccione**Suscripción**. |
   | Lista desplegable de **Suscripción** | Seleccione su suscripción a Azure. |
   | Lista desplegable de **Roles** | Seleccione el rol **Colaborador**. |

   > [!NOTE]
   > El ámbito de la suscripción es necesario para dar cabida a las implementaciones en los laboratorios posteriores.

1. Seleccione el botón **Guardar**.

    ![Captura de pantalla del panel Agregar asignación de roles.](media/add-role-assignment.png)

#### Tarea 2: Crear una conexión de servicio basada en identidad administrada

1. Cambie al explorador web que muestra el proyecto **eShopOnWeb** en el portal de Azure DevOps en `https://dev.azure.com`.

1. En el proyecto **eShopOnWeb**, vaya a **Configuración del proyecto > Conexiones de servicio**.

1. Haga clic en el botón **Nueva conexión de servicio** y seleccione **Azure Resource Manager**.

1. En la sección **Método de autenticación**, seleccione **Identidad administrada**.

1. Establezca el nivel de ámbito en **Suscripción** y proporcione la información recopilada durante la fase de validación del entorno de laboratorio, incluido el identificador de suscripción, el nombre de la suscripción y el identificador de inquilino. 

1. En **Nombre de conexión de servicio**, escriba **azure subs managed**. Se hará referencia a este nombre en las canalizaciones YAML al acceder a la suscripción de Azure.

1. Seleccione **Guardar**.

#### Tarea 3: Actualizar la canalización de CD

1. Cambie a la ventana del explorador que muestra el proyecto **eShopOnWeb** en el portal de Azure DevOps.

1. En la página del proyecto **eShopOnWeb**, vaya a **Canalizaciones > Canalizaciones**.

1. Seleccione la canalización **eshoponweb-cd-webapp-code** y seleccione **Editar**.

1. En la sección de variables, actualice la variable **serviceConnection** para usar el nombre de la conexión de servicio que creó en la tarea anterior, **azure subs managed**.

   ```yaml
     azureserviceconnection: 'azure subs managed'
   ```

1. En la subsección **trabajos** de la sección **fases**, actualice el valor de la propiedad **grupo** para hacer referencia al grupo de agentes autohospedados que creó en el laboratorio anterior, **eShopOnWebSelfPool**, para que tenga el siguiente formato:

   ```yaml
     jobs:
     - job: Deploy
       pool: eShopOnWebSelfPool
       steps:
       #download artifacts
       - download: eshoponweb-ci
   ```

1. Seleccione **Guardar** y elija confirmar directamente en la rama principal.

1. Seleccione **Guardar** otra vez.

1. Seleccione **Ejecutar** la canalización y, a continuación, haga clic en **Ejecutar** de nuevo.

1. Abra la canalización. Si ve el mensaje “This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp (Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con Implementar en WebApp)”, seleccione **View (Ver)**, **Permit (Permitir)** y **Permit (Permitir)** de nuevo. Esto es necesario para permitir que la canalización cree el recurso de Azure App Service.

1. La implementación puede tardar unos minutos en completarse y esperar a que se ejecute la canalización.

1. Debería ver en los registros de canalización que la canalización usa la identidad administrada.

   ![Captura de pantalla de los registros de canalización mediante la identidad administrada.](media/pipeline-logs-managed-identity.png)

   > [!NOTE]
   > Una vez finalizada la canalización, puede usar Azure Portal para comprobar el estado de los recursos de la aplicación web de App Service.

### Ejercicio 3: Limpiar recursos de Azure y Azure DevOps

En este ejercicio, realizará la limpieza posterior al laboratorio de los recursos de Azure y Azure DevOps creados en este laboratorio.

#### Tarea 1: Detener y desasignar la máquina virtual de Azure

1. Vaya a la página que muestra el grupo de recursos **rg-eshoponweb** y seleccione **Eliminar grupo de recursos** para eliminar todos sus recursos.

   > [!IMPORTANT]
   > No elimine el grupo de recursos **rg-eshoponweb-agentpool** que contiene los recursos del agente autohospedado. Lo necesitará en el próximo laboratorio.

1. En Azure Portal, vaya a la página que muestra la máquina virtual de Azure **eshoponweb-vm** que ha implementado en este laboratorio

1. En la página de la máquina virtual de Azure **eshoponweb-vm**, en la barra de herramientas, seleccione **Detener** para desasignarla.

#### Tarea 2: Eliminación de canalizaciones de Azure DevOps

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra el proyecto **eShopOnWeb** .

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Vaya a **Canalizaciones > Canalizaciones** y elimine las canalizaciones existentes.

   > [!IMPORTANT]
   > No elimine la conexión de servicio y el grupo de agentes que creó en este laboratorio. Los necesitará en el siguiente.

#### Tarea 3: Volver a crear el repositorio de Azure DevOps

1. En el portal de Azure DevOps, en el proyecto **eShopOnWeb**, seleccione **Configuración del proyecto** en la esquina inferior izquierda.

1. En el menú vertical del lado izquierdo **Configuración del proyecto**, en la sección **Repositorios**, seleccione **Repositorios**.

1. En el panel **Todos los repositorios**, mantenga el puntero sobre el extremo derecho de la entrada del repositorio **eShopOnWeb** hasta que aparezca el icono de puntos suspensivos **Más opciones**; selecciónelo y, en el menú **Más opciones**, seleccione **Cambiar nombre**.  

1. En la ventana **Cambiar nombre del repositorio eShopOnWeb**, en el cuadro de texto **Nombre del repositorio**, escriba **eShopOnWeb_old** y seleccione **Cambiar nombre**.

1. De nuevo en el panel **Todos los repositorios**, seleccione **+ Crear**.

1. En el panel **Crear un repositorio**, en el cuadro de texto **Nombre del repositorio**, escriba **eShopOnWeb**, desactive la casilla **Agregar un archivo LÉAME** y seleccione **Crear**.

1. En el panel **Todos los repositorios**, mantenga el puntero sobre el extremo derecho de la entrada del repositorio **eShopOnWeb_old** hasta que aparezca el icono de puntos suspensivos **Más opciones**; selecciónelo y, en el menú **Más opciones**, seleccione **Eliminar**.  

1. En la ventana **Eliminar repositorio eShopOnWeb_old**, escriba **eShopOnWeb_old** y seleccione **Eliminar**.

1. En el menú de navegación izquierdo del portal de Azure DevOps, seleccione **Repositorios**.

1. En el panel **eShopOnWeb está vacío. Agregue código**, seleccione **Importar un repositorio**.

1. En la ventana **Importar un repositorio de Git**, pegue la siguiente dirección URL `https://github.com/MicrosoftLearning/eShopOnWeb` y seleccione **Importar**:

## Revisar

En este laboratorio, ha aprendido a usar una identidad administrada asignada a agentes autohospedados en canalizaciones YAML de Azure DevOps.