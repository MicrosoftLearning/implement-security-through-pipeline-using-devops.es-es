---
lab:
  title: "Integrar Azure\_Key\_Vault con Azure Pipelines"
  module: 'Module 6: Configure secure access to Azure Repos from pipelines'
---

# Integrar Azure Key Vault con Azure Pipelines

Azure Key Vault proporciona un almacenamiento y una administración seguros de los datos confidenciales, como claves, contraseñas y certificados. Azure Key Vault incluye compatibilidad con módulos de seguridad de hardware y una gama de algoritmos de cifrado y longitudes de clave. El uso de Azure Key Vault puede minimizar la posibilidad de revelar datos confidenciales desde el código fuente, un error habitual que cometen los desarrolladores. El acceso a Azure Key Vault requiere una autenticación y autorización adecuadas, y admite permisos específicos para su contenido.

Estos ejercicios duran aproximadamente **40** minutos.

## Antes de comenzar

Necesitará una suscripción de Azure, una organización de Azure DevOps y la aplicación eShopOnWeb para seguir los laboratorios.

- Siga los pasos para [validar el entorno de laboratorio](APL2001_M00_Validate_Lab_Environment.md).

## Instrucciones

En este laboratorio, verá cómo puede integrar Azure Key Vault con una canalización de Azure Pipelines mediante los pasos siguientes:

- Crear un almacén de Azure Key Vault para almacenar una contraseña ACR como secreto.
- Crear una entidad de servicio de Azure para acceder a los secretos de Azure Key Vault.
- Configurar los permisos para permitir que la entidad de servicio lea el secreto.
- Configurar la canalización para recuperar la contraseña de Azure Key Vault y pasarla a las tareas posteriores.

### Ejercicio 1: Configuración de la canalización de CI para crear el contenedor eShopOnWeb.

Configure la canalización de YAML de CI para:

- Crear un Azure Container Registry para conservar las imágenes de contenedor
- Usar Docker Compose para compilar e enviar las imágenes de contenedor **eshoppublicapi** y **eshopwebmvc**. Solo se implementará el contenedor **eshopwebmvc**.

#### Tarea 1: Creación de una entidad de servicio (si ya la ha completado, puede omitirla)

En esta tarea, creará una entidad de servicio mediante la CLI de Azure, que permitirá a Azure DevOps hacer lo siguiente:

- Implementar recursos en una suscripción de Azure
- Tener acceso de lectura en los secretos de Key Vault creados más adelante.

Necesitará una entidad de servicio para implementar recursos de Azure desde Azure Pipelines. Dado que recuperará secretos en una canalización, debe conceder permiso al servicio al crear la Azure Key Vault.

Azure Pipelines crea automáticamente una entidad de servicio cuando se conecta a una suscripción de Azure desde una definición de canalización o cuando se crea una nueva conexión de servicio desde la página de configuración del proyecto (opción automática). También puede crear manualmente la entidad de servicio desde el portal o usar la CLI de Azure y reutilizarla en proyectos.

1. Inicie un explorador web, vaya a Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de usuario con el rol Propietario en la suscripción de Azure que va a usar en este laboratorio y que tenga el rol Administrador global en el inquilino de Azure AD asociado con esta suscripción.

1. En Azure Portal, haga clic en el icono de **Cloud Shell**, situado a la derecha del cuadro de texto de búsqueda de la parte superior de la página.

1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   > [!NOTE]
   > Si es la primera vez que inicia **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, seleccione la suscripción que utiliza en este laboratorio y seleccione **Crear almacenamiento**.

1. En la solicitud de **Bash**, en el panel de **Cloud Shell**, ejecute los siguientes comandos para recuperar los valores del identificador de suscripción de Azure y los atributos de nombre de suscripción:

    ```bash
    az account show --query id --output tsv
    az account show --query name --output tsv
    ```

    > [!NOTE]
    > Copie ambos valores en un archivo de texto. Los necesitará más adelante en este laboratorio.

1. En la solicitud de **Bash**, en el panel de **Cloud Shell**, ejecute el siguiente comando para crear una entidad de servicio (reemplace **myServicePrincipalName** por cualquier cadena única de caracteres que conste de letras y dígitos) y **mySubscriptionID** por su subscriptionId de Azure:

    ```bash
    az ad sp create-for-rbac --name myServicePrincipalName \
                         --role contributor \
                         --scopes /subscriptions/mySubscriptionID
    ```

    > [!NOTE]
    > El comando generará una salida JSON. Copie los resultados en un archivo de texto. Lo necesitará más adelante en este laboratorio.

1. Después, vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Navegue al proyecto **eShopOnWeb** de Azure DevOps. Seleccione **Project Settings (Configuración del proyecto) > Service Connections (Conexiones de servicio) en Pipelines (Canalizaciones)** y haga clic en **New Service Connection (Nueva conexión de servicio)**

    ![Captura de pantalla del botón para crear la nueva conexión de servicio.](media/new-service-connection.png)

1. En la hoja **New service connection (Nueva conexión de servicio)**, seleccione **Azure Resource Manager** y, después, seleccione **Next (Siguiente)** (es posible que deba desplazarse hacia abajo).

1. Seleccione **Service Principal (Entidad de servicio) (manual)** y, después, seleccione **Next (Siguiente)**.

1. Rellene los campos vacíos con la información recopilada durante los pasos anteriores:
    - Identificador y nombre de la suscripción.
    - Id. de entidad de servicio (o clientId/AppId), clave de entidad de servicio (o contraseña) y TenantId.
    - En **Nombre de conexión de servicio**, escriba **azure subs**. Se hará referencia a este nombre en canalizaciones de YAML cuando necesite una conexión de servicio de Azure DevOps para comunicarse con la suscripción de Azure.

        ![Captura de pantalla de la configuración de conexión de servicios de Azure](media/azure-service-connection.png)

1. No marque **Conceder permiso de acceso a todas las canalizaciones**. Seleccione **Verificar y guardar**.

    > [!NOTE]
    > No se recomienda la opción **Conceder permiso de acceso a todas las canalizaciones** para entornos de producción. Solo se usa en este laboratorio para simplificar la configuración de la canalización.

#### Tarea 2: Configuración y ejecución de una canalización de CI

En esta tarea, importará una definición de canalización de YAML de CI existente, la modificará y ejecutará. Creará una nueva instancia de Azure Container Registry (ACR) y compilará o publicará las imágenes de contenedor eShopOnWeb.

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Navegue al proyecto **eShopOnWeb** de Azure DevOps. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)** y haga clic en **Create Pipeline (Crear canalización)** o **New Pipeline (Nueva canalización)**.

1. En la ventana **Where is your code? (¿Dónde está el código?)**, seleccione **Git de Azure Repos (YAML)** y seleccione el repositorio **eShopOnWeb**.

1. En la sección **Configure (Configurar)**, seleccione **Existing Azure Pipelines YAML file (Archivo YAML de Azure Pipelines existente)**. Proporcione la siguiente ruta de acceso **/.ado/eshoponweb-ci-dockercompose.yml** y seleccione **Continue (Continuar)**.

    ![Captura de pantalla de la selección de canalización del archivo YAML.](media/select-ci-container-compose.png)

1. En la definición de canalización de YAML, en la sección variables, personalice el nombre del grupo de recursos reemplazando **AZ400-EWebShop-NAME** por el nombre de su preferencia, por ejemplo, **rg-eshoponweb**, reemplace **YOUR-SUBSCRIPTION-ID** por su propio identificador de suscripción de Azure y elija la ubicación de su preferencia más cercana a la ubicación, por ejemplo **southcentralus**.

1. (Opcional) Puede usar el agente autohospedado creado en el laboratorio anterior actualizando el nombre del grupo establecido actualmente en el agente hospedado por Microsoft en el nombre del grupo de agentes que creó, **eShopOnWebSelfPool**.

    En lugar de:

    ```YAML
      - job: Build
        pool:
          vmImage: ubuntu-latest
    
    ```

    Uso:

    ```YAML
      - job: Build
        pool: eShopOnWebSelfPool
    
    ```

    > [!NOTE]
    > Para ejecutar la canalización con el agente autohospedado, deberá tener el agente en ejecución y todos los requisitos previos instalados, por ejemplo, Visual Studio para compilar la solución. Si no tiene instalados los requisitos previos, puede usar el agente hospedado por Microsoft.

1. Seleccione **Save and Run (Guardar y ejecutar)**, y elija confirmar directamente en la rama principal o crear una nueva rama.

1. Seleccione **Save and Run (Guardar y ejecutar)** de nuevo.

    > [!NOTE]
    > Si decide crear una rama, deberá crear una solicitud de incorporación de cambios para combinar los cambios en la rama principal.

1. Abra la canalización. Si ve el mensaje “This pipeline needs permission to access a resource before this run can continue to Create ACR for images (Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con Crear ACR para imágenes)”, seleccione **View (Ver)**, **Permit (Permitir)** y **Permit (Permitir)** de nuevo. Esto es necesario para permitir que la canalización cree el recurso de Azure Container Registry (ACR).

    ![Captura de pantalla del permiso de acceso desde la canalización de YAML.](media/pipeline-permit-resource.png)

1. La compilación puede tardar unos minutos en completarse, espere a que se ejecute la canalización. La definición de compilación consta de las siguientes tareas:
      - **AzureResourceManagerTemplateDeployment** usa **bicep** para implementar una instancia de Azure Container Registry.
      - La tarea de **PowerShell** toma la salida de bicep (servidor de inicio de sesión acr) y crea una variable de canalización.
      - La tarea **DockerCompose** compila e inserta las imágenes de contenedor para eShopOnWeb en Azure Container Registry.

1. La canalización tomará un nombre basado en el nombre del proyecto. Cambie su nombre para identificar mejor la canalización.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)** en la canalización creada recientemente, pase el mouse sobre la canalización ejecutada y seleccione los puntos suspensivos y la opción **Rename/move (Cambiar nombre/mover)**.

1. Asígnele el nombre **eshoponweb-ci-dockercompose** y seleccione **Save (Guardar)**.

1. Una vez finalizada la ejecución, en Azure Portal, abra el grupo de recursos definido anteriormente y seleccione la entrada que representa azure Container Registry (ACR) implementada por la canalización.

    > [!NOTE]
    > Para ver repositorios en el Registro, debe asignar un rol que proporcione dicho acceso. Usará para este propósito el rol AcrPull.

1. En la página Container registry (registro de contenedor), seleccione **Access control (Control de acceso) (IAM)**, seleccione **+ Add (+ Agregar)** y, en la lista desplegable, seleccione **Add role assignment (Agregar asignación de rol)**.

1. En la pestaña **Role (Rol)** de la página **Add role assignment (Agregar asignación de roles)**, seleccione **AcrPull** y, a continuación, seleccione **Next (Siguiente)**.

1. En la pestaña **Members (Miembros)**, haga clic en **+Select members (+ Seleccionar miembros)**, seleccione su cuenta de usuario, haga clic en **Select (Seleccionar)** y, a continuación, seleccione **Next (Siguiente)**.

1. Seleccione **Review + assign (Revisar y asignar)** y, una vez completada correctamente la asignación, actualice la página del explorador.

1. De nuevo en la página Container registry (Registro de contenedor), en la barra de menús vertical de la izquierda, en la sección **Services (Servicios)**, seleccione **Repositories (Repositorios)**.

1. Compruebe que el registro contiene las imágenes **eshoppublicapi** y **eshopwebmvc**. Solo usará **eshopwebmvc** en la fase de implementación.

    ![Captura de pantalla de las imágenes de contenedor de ACR de Azure Portal.](media/azure-container-registry.png)

1. Seleccione **Claves de acceso**, habilite la casilla **Administración usuario** y copie el valor de **contraseña**, que se usará en la siguiente tarea, ya que lo mantendremos como secreto en Azure Key Vault.

    ![Captura de pantalla de la contraseña de ACR desde la configuración Claves de acceso.](media/acr-password.png)

#### Tarea 3: Creación de una instancia de Azure Key Vault

En esta tarea, creará un Azure Key Vault mediante Azure Portal.

En este escenario de laboratorio, tendremos una instancia de Azure Container Instance (ACI) que extraiga y ejecute una imagen de contenedor almacenada en Azure Container Registry (ACR). Tenemos previsto almacenar la contraseña de ACR como un secreto en el almacén de claves.

1. En Azure Portal, en el cuadro de texto **Buscar recursos, servicios y documentos**, escriba **Almacén de claves** y presione la tecla **Entrar**.

1. En la hoja **Almacén de claves**, seleccione **Crear > Almacén de claves**.

1. En la pestaña **Datos básicos** de la hoja **Crear un almacén de claves**, especifique la siguiente configuración y seleccione **Siguiente**:

    | Configuración | Valor |
    | --- | --- |
    | Suscripción | nombre de la suscripción de Azure que usa en este laboratorio |
    | Grupo de recursos | el nombre del grupo de recursos **rg-eshoponweb** |
    | Nombre del almacén de claves | cualquier nombre válido único, como **ewebshop-kv-NAME** (reemplace NAME) |
    | Region | una región de Azure cercana a la ubicación del entorno de laboratorio |
    | Plan de tarifa | **Estándar** |
    | Días durante los cuales se conservarán los almacenes eliminados | **7** |
    | Protección de purgas | **Deshabilitar la protección de purgas** |

1. En la pestaña **Configuración de acceso** de la hoja **Crear almacén de claves**, en la sección **Modelo de permisos**, seleccione **Directiva de acceso del almacén**. 

1. En la sección **Directivas de acceso**, seleccione **+ Crear** para configurar una nueva directiva.

    > **Nota**: Debe proteger el acceso a los almacenes de claves permitiendo el acceso unicamente a aplicaciones y usuarios autorizados. Para acceder a los datos del almacén, deberá proporcionar permisos de lectura (obtener/enumerar) a la entidad de servicio creada anteriormente que usará para la autenticación en la canalización.

    - En la hoja **Permiso**, marque los permisos **Obtener** y **enumerar** en de **Permiso secreto**. Seleccione **Siguiente**.
    - en la hoja **Principal**, busque la **entidad de servicio** creada anteriormente, ya sea mediante el identificador o el nombre especificados. Seleccione **Siguiente** y luego **Siguiente** de nuevo.
    - En la hoja **Revisar y crear**, seleccione **Crear**.

1. De nuevo en la hoja **Crear un almacén de claves**, seleccione **Revisar y crear > Crear**.

    > [!NOTE]
    > Espere a que se aprovisione el almacén de claves de Azure. Debería tardar menos de un minuto.

1. En el hoja **Se completó la implementación**, haga clic en **Ir al recurso**.

1. En la hoja Azure Key Vault, en el menú vertical del lado izquierdo de la hoja, en la sección **Objetos**, seleccione **Secretos**.

1. En la hoja **Secretos**, seleccione **Generar/importar**.

1. En la hoja **Crear un secreto**, especifique las de configuración siguientes y seleccione **Crear** (deje las demás con los valores predeterminados):

    | Configuración | Valor |
    | --- | --- |
    | Opciones de carga | **Manual** |
    | Nombre | **acr-secret** |
    | Valor | Contraseña de acceso de ACR copiada en la tarea anterior |

#### Tarea 3: Creación de un grupo de variables conectado a Azure Key Vault

En esta tarea, creará un grupo de variables en Azure DevOps que recuperará el secreto de contraseña de ACR de Key Vault mediante la Conexión de servicio (entidad de servicio).

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Navegue al proyecto **eShopOnWeb** de Azure DevOps.

1. En el panel de navegación vertical del portal de Azure DevOps, seleccione **Pipelines (Canalizaciones) > Library (Biblioteca)**. Seleccione **+ Variable Group (+ Grupo de variables)**.

1. En la hoja **New variable group (Nuevo grupo de variables)**, configure las opciones siguientes:

    | Configuración | Valor |
    | --- | --- |
    | Variable Group Name (Nombre del grupo de variables) | **eshopweb-vg** |
    | Link secrets from Azure KV (Vinculación de secretos desde Azure KV...) | **enable** |
    | Suscripción de Azure | **Conexión de servicio de Azure disponible > Azure subs** |
    | Nombre del almacén de claves | Nombre del almacén de claves|

1. En **Variables**, seleccione **+Add (+ Agregar)** y seleccione el secreto **acr-secret**. Seleccione **Aceptar**.

1. Seleccione **Guardar**.

    ![Captura de pantalla de la creación del grupo de variables.](media/vg-create.png)

#### Tarea 4: Configuración de la canalización de CD para implementar el contenedor en Azure Container Instance (ACI)

En esta tarea, importará una canalización de CD, la personalizará y la ejecutará para implementar la imagen de contenedor creada antes en una instancia de Azure Container Instance.

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Navegue al proyecto **eShopOnWeb** de Azure DevOps. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)** y seleccione **New Pipeline (Nueva canalización)**.

1. En la ventana **Where is your code? (¿Dónde está el código?)**, seleccione **Git de Azure Repos (YAML)** y seleccione el repositorio **eShopOnWeb**.

1. En la sección **Configure (Configurar)**, seleccione **Existing Azure Pipelines YAML file (Archivo YAML de Azure Pipelines existente)**. Proporcione la siguiente ruta **/.ado/eshoponweb-cd-aci.yml** y seleccione **Continue (Continuar)**.

1. En la definición de canalización de YAML, personalice lo siguiente:

    - **YOUR-SUBSCRIPTION-ID** por el id. de la suscripción a Azure.
    - **az400eshop-NAME** reemplace NAME para que sea único globalmente.
    - ** YOUR-ACR.azurecr.io** y **ACR-USERNAME** con el servidor de inicio de sesión de ACR (ambos necesitan el nombre de ACR, se pueden revisar en ACR > Claves de acceso).
    - **rg-eshoponweb** con el nombre del grupo de recursos definido antes en el laboratorio.

1. Seleccione **Save and Run (Guardar y ejecutar)** y espere a que la canalización se ejecute correctamente.

    > **Nota**: La implementación puede tardar unos minutos en completarse. La definición de CD consta de las siguientes tareas:
    - **Recursos**: está preparado para desencadenarse automáticamente en función de la finalización de la canalización de CI. También descarga el repositorio para el archivo bicep.
    - **Variables (para la etapa de implementación)** se conecta al grupo de variables para consumir el secreto de Azure Key Vault **acr-secret**
    - **AzureResourceManagerTemplateDeployment** implementa Azure Container Instance (ACI) mediante la plantilla de bicep y proporciona los parámetros de inicio de sesión de ACR para permitir que ACI descargue la imagen de contenedor creada anteriormente desde Azure Container Registry (ACR).

1. La canalización tomará un nombre basado en el nombre del proyecto. Cámbielo para identificar mejor la canalización.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**, seleccione la canalización creada recientemente, seleccione los puntos suspensivos y, después, seleccione la opción **Rename/move (Cambiar nombre/mover)**.

1. Asígnele el nombre **eshoponweb-cd-aci** y seleccione **Save (Guardar)**.

### Ejercicio 2: Eliminación de los recursos del laboratorio de Azure

1. Vaya al grupo de recursos en Azure Portal y haga clic en **Eliminar grupo de recursos**.

    ![Captura de pantalla del botón de eliminación de un grupo de recursos.](media/delete-resource-group.png)

    > [!WARNING]
    > No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

## Revisar

En este laboratorio, ha integrado Azure Key Vault con una canalización de Azure DevOps mediante los pasos siguientes:

- Ha creado una entidad de servicio de Azure para proporcionar acceso a los secretos de Azure Key Vault y autenticar la implementación de Azure desde Azure DevOps.
- Ha ejecutado 2 canalizaciones YAML importadas desde un repositorio de Git.
- Ha configurado la canalización para recuperar la contraseña de Azure Key Vault y pasarla a las tareas posteriores.
