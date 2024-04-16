---
lab:
  title: 'Laboratorio: Administrar la identidad para proyectos y canalizaciones'
  module: 'Module 2: Manage identity for projects, pipelines, and agents'
---

# Laboratorio: Administrar la identidad para proyectos y canalizaciones

Las identidades administradas ofrecen un método seguro para controlar el acceso a los recursos de Azure. Azure controla estas identidades automáticamente, lo que le permite comprobar el acceso a los servicios compatibles con la autenticación de Azure AD. Esto significa que no es necesario insertar credenciales en el código, lo que mejora la seguridad. En Azure DevOps, las identidades administradas pueden autenticar los recursos de Azure dentro de los agentes autohospedados, lo que simplifica el control de acceso sin poner en peligro la seguridad.

En este laboratorio, cree una identidad administrada para usarla en las canalizaciones de YAML mediante Azure DevOps con agentes autohospedados y una identidad administrada.

Estos ejercicios duran aproximadamente **45** minutos.

## Antes de comenzar

Necesitará una suscripción de Azure, una organización de Azure DevOps y la aplicación eShopOnWeb para seguir los laboratorios.

- Siga los pasos para [validar el entorno de laboratorio](APL2001_M00_Validate_Lab_Environment.md).

- Compruebe que tiene una cuenta de Microsoft o de Azure AD con el rol Propietario o Colaborador en la suscripción a Azure. Para más información, consulte [Enumeración de asignaciones de roles de Azure mediante Azure Portal](https://learn.microsoft.com/azure/role-based-access-control/role-assignments-list-portal) y [Ver y asignar roles de administrador en Azure Active Directory](https://learn.microsoft.com/azure/active-directory/roles/manage-roles-portal).

## Instrucciones

### Ejercicio 1: Importación y ejecución de canalizaciones de CI/CD.

En este ejercicio, importará y ejecutará la canalización de CI, configurará la conexión de servicio con la suscripción de Azure y, a continuación, importará y ejecutará la canalización de CD.

#### Tarea 1: (Si ya la ha completado, omita esta tarea) Importar y ejecutar la canalización de CI

Empecemos importando la canalización de CI denominada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra el proyecto **eShopOnWeb** .

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Seleccione el botón **New Pipeline (Nueva canalización)**.

1. Seleccione **Git de Azure Repos (YAML)**.

1. Seleccione el repositorio **eShopOnWeb**.

1. Seleccione **Archivo YAML de Azure Pipelines existente**.

1. Seleccione el archivo **/.ado/eshoponweb-ci.yml** y haga clic en **Continuar**.

1. Haga clic en el botón **Run (Ejecutar)** para ejecutar la canalización.

    > [!NOTE]
    > La canalización tomará un nombre basado en el nombre del proyecto. Cámbielo para identificar mejor la canalización.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**, seleccione la canalización creada recientemente, seleccione los puntos suspensivos y, después, seleccione la opción **Rename/move (Cambiar nombre/mover)**.

1. Asígnele el nombre **eshoponweb-ci** y seleccione **Guardar**.

#### Ejercicio 1: Administración de la conexión de servicio.

Puede crear una conexión desde Azure Pipelines a servicios externos y remotos para ejecutar las tareas de un trabajo.

En esta tarea, creará una entidad de servicio mediante la CLI de Azure, que permitirá a Azure DevOps hacer lo siguiente:

- Implementar recursos en una suscripción de Azure
- Implementar la aplicación eShopOnWeb

> [!NOTE]
> Si ya tiene una entidad de servicio y una conexión de servicio a la suscripción de Azure denominada **azure subs**, puede continuar directamente con la siguiente tarea.

Necesitará una entidad de servicio para implementar recursos de Azure desde Azure Pipelines.

Azure Pipelines crea automáticamente una entidad de servicio cuando se conecta a una suscripción de Azure desde dentro de una definición de canalización o al crear una nueva conexión de servicio desde la página de configuración del proyecto (opción automática). También puede crear manualmente la entidad de servicio desde el portal o mediante la CLI de Azure y volver a usarla en proyectos.

1. Inicie un explorador web, vaya a Azure Portal en `https://portal.azure.com` e inicie sesión con la cuenta de usuario con el rol Propietario en la suscripción de Azure que va a usar en este laboratorio y que tenga el rol Administrador global en el inquilino de Azure AD asociado con esta suscripción.

1. En Azure Portal, seleccione el icono de **Cloud Shell** situado en la parte superior de la página, a la derecha del cuadro de búsqueda.

1. Si se le pide que seleccione **Bash** o **PowerShell**, seleccione **Bash**.

   > [!NOTE]
   > Si es la primera vez que inicia **Cloud Shell** y aparece el mensaje **No tiene ningún almacenamiento montado**, seleccione la suscripción que utiliza en este laboratorio y seleccione **Crear almacenamiento**.

1. En la solicitud de **Bash**, en el panel de **Cloud Shell**, ejecute los siguientes comandos para recuperar los valores del atributo de identificador de suscripción de Azure:

    ```sh
    subscriptionName=$(az account show --query name --output tsv)
    subscriptionId=$(az account show --query id --output tsv)
    echo $subscriptionName
    echo $subscriptionId
    ```

    > [!NOTE]
    > Copie ambos valores en un archivo de texto. Los necesitará más adelante en este laboratorio.

1. En la solicitud de **Bash**, en el panel de **Cloud Shell**, ejecute el siguiente comando para crear una entidad de servicio:

    ```sh
    az ad sp create-for-rbac --name sp-eshoponweb-azdo --role contributor --scopes /subscriptions/$subscriptionId
    ```

    > [!NOTE]
    > El comando generará una salida JSON. Copie los resultados en un archivo de texto. Lo necesitará más adelante en este laboratorio.

1. Después, vaya al proyecto **eShopOnWeb** de Azure DevOps. Haga clic en **Project Settings > Service Connections (under Pipelines) (Configuración del proyecto > Conexiones de servicio (en Canalizaciones)** y **New Service Connection (Nueva conexión de servicio)**.

1. En la hoja **New service connection (Nueva conexión de servicio)**, seleccione **Azure Resource Manager** y, después, seleccione **Next (Siguiente)** (es posible que deba desplazarse hacia abajo).

1. Seleccione **Service Principal (Entidad de servicio) (manual)** y, después, seleccione **Next (Siguiente)**.

1. Rellene los campos vacíos con la información recopilada durante los pasos anteriores:
    - Identificador y nombre de la suscripción.
    - Id. de entidad de servicio (o clientId), Clave (o contraseña) y TenantId.
    - En **Nombre de conexión de servicio**, escriba **azure subs**. Se hará referencia a este nombre en canalizaciones de YAML cuando necesite una conexión de servicio de Azure DevOps para comunicarse con la suscripción de Azure.

1. Seleccione **Verificar y guardar**.

#### Ejercicio 3: Importación y ejecución de la canalización de CD

Ahora, importe la canalización de CD denominada [eshoponweb-cd-webapp-code.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-cd-webapp-code.yml).

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Haga clic en el botón **Nueva canalización**

1. Seleccione **Git de Azure Repos (YAML)**.

1. Seleccione el repositorio **eShopOnWeb**.

1. Seleccione **Archivo YAML de Azure Pipelines existente**.

1. Seleccione el archivo **/.ado/eshoponweb-cd-webapp-code.yml** y, después, seleccione **Continuar**.

1. En la definición de canalización de YAML, establezca la sección variables de la siguiente manera:

    ```YAML
    variables:
      resource-group: 'AZ400-EWebShop-NAME'
      location: 'westeurope'
      templateFile: '.azure/bicep/webapp.bicep'
      subscriptionid: 'YOUR-SUBSCRIPTION-ID'
      azureserviceconnection: 'azure subs'
      webappname: 'az400-webapp-NAME'
    ```

1. En la sección de variables, reemplace los marcadores de posición del comando anterior por los valores siguientes:

   - **AZ400-EWebShop-NAME** con el nombre que prefiera, por ejemplo, **rg-eshoponweb**.
   - **location** con el nombre de la región de Azure que desea implementar los recursos, por ejemplo, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** por el id. de la suscripción a Azure.
   - **az400-webapp-NAME**, con un nombre de aplicación web que se va a implementar con un nombre único global, por ejemplo, **eshoponweb-lab-YOURNAME**.

1. Si están presentes, en la sección de recursos, quite las siguientes entradas:

    ```YAML
    repositories:
      - repository: eShopSecurity
        type: git
        name: eShopSecurity/eShopSecurity #name of the project and repository
    ```

1. Seleccione **Save and Run (Guardar y ejecutar)**, y elija confirmar directamente en la rama principal o crear una nueva rama.

1. Seleccione **Save and Run (Guardar y ejecutar)** de nuevo.

    > [!NOTE]
    > Si decide crear una rama, deberá crear una solicitud de incorporación de cambios para combinar los cambios en la rama principal.

1. Abra la canalización. Si ve el mensaje “This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp (Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con Deploy to WebApp)”, seleccione **View (Ver)**, **Permit (Permitir)** y **Permit (Permitir)** de nuevo. Esto es necesario para permitir que la canalización cree el recurso de Azure App Service.

    ![Captura de pantalla del permiso de acceso desde la canalización de YAML.](media/pipeline-deploy-permit-resource.png)

1. La implementación puede tardar unos minutos en completarse y esperar a que se ejecute la canalización. La definición de CD consta de las siguientes tareas:
      - **Recursos**: está preparado para desencadenarse automáticamente en función de la finalización de la canalización de CI. También descarga el repositorio para el archivo bicep.
      - **AzureResourceManagerTemplateDeployment**: implementa la aplicación web de Azure mediante la plantilla de bicep.

1. La canalización tomará un nombre basado en el nombre del proyecto. Vamos a **cambiarle el nombre** para identificar mejor la canalización.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**, seleccione la canalización creada recientemente, seleccione los puntos suspensivos y, después, seleccione la opción **Rename/move (Cambiar nombre/mover)**.

1. Asígnele el nombre **eshoponweb-cd-webapp-code** y, después, seleccione **Guardar**.

### Ejercicio 2: Creación de una identidad administrada para la conexión de servicio

En este ejercicio, creará una identidad administrada y, después, creará una nueva conexión de servicio para usarla en las canalizaciones de CI/CD.

#### Tarea 1: Creación de una identidad administrada

1. En el explorador, abra Azure Portal desde `https://portal.azure.com`.

1. En el cuadro **Buscar recursos, servicios y documentos (G+/)**, escriba **Identidades administradas** y selecciónelo en la lista desplegable.

    ![Captura de pantalla de la opción de Identidades administradas en Azure Portal.](media/managed-identities.png)

1. Seleccione el botón **Crear identidad administrada**.

1. En el panel **Crear identidad administrada**, rellene la información necesaria:
   - **Suscripción**: indique la suscripción de Azure.
   - **Grupo de recursos**: un grupo de recursos existente o nuevo.
   - **Región**: use la región cercana a la ubicación o disponible para los recursos.
   - **Nombre**: indique el nombre de identidad administrada que prefiera, por ejemplo, **eshoponweb-mi**.

    ![Captura de pantalla del panel de creación de identidades administradas.](media/create-managed-identity.png)

    > [!NOTE]
    > Si no tiene un grupo de recursos, puede crear uno haciendo clic en el vínculo **Crear nuevo**.

1. Seleccione el botón **Revisar y crear** y, después, **Crear**.

#### Tarea 2: Asignación de permisos a la identidad administrada

Ahora, debe asignar los permisos de identidad administrada al grupo de recursos y a los servicios de aplicaciones.

1. En Azure Portal, vaya a la identidad administrada que creó anteriormente.

1. Seleccione la pestaña **Asignaciones de roles de Azure** en el menú izquierdo.

1. Seleccione el botón **Agregar asignación de roles** y realice las siguientes acciones:

    | Configuración | Acción |
    | -- | -- |
    | Lista desplegable de **Ámbito** | Seleccione **Grupo de recursos**. |
    | Lista desplegable de **Suscripción** | Seleccione su suscripción a Azure. |
    | Lista desplegable del **grupo de recursos** | Seleccione el grupo de recursos existente. |
    | Lista desplegable de **Roles** | Seleccione el rol **Colaborador**. |

1. Seleccione el botón **Guardar**.

    ![Captura de pantalla del panel Agregar asignación de roles.](media/add-role-assignment.png)

### Ejercicio 3: Creación de una nueva máquina virtual de Azure mediante el agente autohospedado y la identidad administrada y actualización de la canalización de CI

En este ejercicio, creará una nueva máquina virtual de Azure mediante el agente autohospedado y la identidad administrada que creó en el ejercicio anterior. Después, actualizará la canalización de CI para usar la nueva máquina virtual de Azure.

#### Tarea 1: Creación de una máquina virtual nueva en Azure

1. En el explorador, abra Azure Portal desde `https://portal.azure.com`.

1. En el cuadro **Buscar recursos, servicios y documentos (G+/)**, escriba **Máquinas virtuales** y selecciónelo en la lista desplegable.

1. Seleccione el botón **Crear**.

1. Seleccione **Máquina virtual de Azure con configuración preestablecida**.

    ![Captura de pantalla de la creación de una máquina virtual con configuración preestablecida.](media/create-virtual-machine-preset.png)

1. Seleccione **Dev/Test** como entorno de carga de trabajo y **De uso general** como tipo de carga de trabajo.

1. Seleccione el botón **Continuar para crear una máquina virtual**, en la pestaña **Datos básicos**, realice las siguientes acciones y, después, seleccione la pestaña **Administración**:

    | Configuración | Acción |
    | -- | -- |
    | Lista desplegable de **Suscripción** | Seleccione su suscripción a Azure. |
    | Sección **Grupo de recursos** | Seleccione el grupo de recursos existente o nuevo, por ejemplo, **eshoponweb-resource**. |
    | Cuadro de texto **Nombre de máquina virtual**  | Escriba el nombre que prefiera, por ejemplo, **eshoponweb-vm**. |
    | Lista desplegable de **región** | Seleccione la región cercana a su ubicación o disponible para los recursos, por ejemplo, **Centro-sur de EE. UU**. |
    | Lista desplegable **Opciones de disponibilidad** | Seleccione **No se requiere redundancia de la infraestructura**. |
    | Lista desplegable **Tipo de seguridad** | Seleccione la opción **Máquinas virtuales de inicio seguro**. |
    | Lista desplegable de **imágenes** | Seleccione la imagen de **Windows Server 2019 o 2022 Datacenter** |
    | Lista desplegable de **Tamaño** | Seleccione el tamaño **estándar** más barato para realizar pruebas. |
    | Cuadro de texto **Nombre de usuario** | Escriba el nombre de usuario que prefiera. |
    | Cuadro de texto **Contraseña** | Escriba la contraseña que prefiera. |
    | Sección **Puertos de entrada públicos** | Seleccione **Permitir los puertos seleccionados**. |
    | Lista desplegable **Seleccionar puertos de entrada** | Seleccione **RDP (3389)**. |

1. En la pestaña **Administración**, realice las siguientes acciones y, después, seleccione **Revisar y crear**:
   
    Sección | **Habilitar identidad administrada asignada por el sistema** | Active **la casilla**. Esto permitirá que la máquina virtual use la identidad administrada que creó. | | Sección **Dirección IP pública** | Seleccione **Crear nuevo**, escriba un nombre de su preferencia y, después, seleccione **Aceptar.** |

    > [!IMPORTANT]
    > No omita el paso Ejercicio 5: Eliminación de los recursos del laboratorio de Azure para evitar cargos inesperados.

1. En la pestaña **Revisar y crear**, seleccione **Crear**.

1. Abra la configuración de la máquina virtual, seleccione la pestaña **Identidad** y seleccione el botón **Asignaciones de roles de Azure**.

1. Seleccione el botón **Agregar asignación de roles**.

1. Seleccione el ámbito de la suscripción, la suscripción y el rol **Colaborador**.

1. Seleccione el botón **Guardar**.

#### Tarea 2: Abrir la nueva máquina virtual de Azure e instale el agente autohospedado

1. Abra la nueva máquina virtual de Azure que creó anteriormente mediante la conexión RDP. Puede encontrar la información de conexión en **Información general** marcando el botón **Conectar**.

2. En la máquina virtual de Azure, siga los pasos para instalar el agente en la nueva máquina virtual de Azure del [Ejercicio 1 del laboratorio Configuración de agentes y grupos de agentes para canalizaciones seguras](APL2001_M03_L03_Configure_Agents_And_Agent_Pools_for_Secure_Pipelines.md). Cuando siga las instrucciones, tenga en cuenta los cambios siguientes:

   - Asigne al grupo de agentes el nombre **eShopOnWebSelfPoolManaged** (en lugar de **eShopOnWebSelfPool**) en el paso 5 de la tarea 1.
   - Asigne al agente el nombre **eShopOnWebSelfAgentManaged** (en lugar de **eShopOnWebSelfAgent**) en la tarea 4, paso 3.
   - Seleccione **NT AUTHORITY\NETWORK SERVICE** como cuenta para ejecutar el servicio durante la configuración de la cuenta de usuario en el paso 3 de la tarea 4.

3. Una vez instalado el agente, abra el grupo de agentes en el portal de Azure DevOps y compruebe que el nuevo agente está disponible.

    ![Captura de pantalla del nuevo agente en el nuevo grupo de agentes.](media/new-agent-pool.png)

### Ejercicio 4: Creación de una nueva conexión de servicio mediante la identidad administrada y actualización de la canalización de CD

En este ejercicio, creará una nueva conexión de servicio mediante el método de autenticación de identidad administrada. A continuación, actualizará la canalización de CD para usar la nueva conexión de servicio.

#### Creación de una nueva conexión de servicio

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra el proyecto **eShopOnWeb** y vaya a **Configuración del proyecto > Conexiones de servicio**.

1. Haga clic en el botón **Nueva conexión de servicio** y seleccione **Azure Resource Manager**.

1. En la sección **Método de autenticación**, seleccione **Identidad administrada**.

1. Rellene los campos vacíos con la información recopilada durante los pasos anteriores:
    - Id. de suscripción, Nombre e Id. de inquilino (o clientId).
    - En **Nombre de conexión de servicio**, escriba **azure subs managed**. Se hará referencia a este nombre en canalizaciones de YAML cuando necesite una conexión de servicio de Azure DevOps para comunicarse con la suscripción de Azure.

1. Seleccione **Verificar** y **guardar**.

#### Tarea 2: Actualización de la canalización de CD

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra el proyecto **eShopOnWeb** y vaya a **Canalizaciones > Canalizaciones**.

1. Seleccione la canalización **eshoponweb-cd-webapp-code** y seleccione **Editar**.

1. En la sección variables, actualice la variable **serviceConnection** con el nombre de la conexión de servicio que creó en la tarea anterior, **azure subs managed**.

    ```YAML
          azureserviceconnection: 'azure subs managed'
    ```

1. En la subsección **Trabajos** de la sección **Etapa**, actualice el valor de la propiedad **pool** para hacer referencia al grupo de agentes autohospedado que creó en el ejercicio anterior, **eShopOnWebSelfPoolManaged**, por lo que obtiene el siguiente formato:

    ```YAML    
          jobs:
          - job: Deploy
            pool: eShopOnWebSelfPoolManaged
            steps:
            #download artifacts
            - download: eshoponweb-ci
    ```

1. Seleccione **Guardar**, elija para confirmar directamente en la rama principal o crear una nueva rama.

1. Seleccione **Guardar** otra vez.

    > [!NOTE]
    > Si decide crear una rama, deberá crear una solicitud de incorporación de cambios para combinar los cambios en la rama principal.

1. Seleccione **Ejecutar** la canalización y, a continuación, haga clic en **Ejecutar** de nuevo.

1. Abra la canalización. Si ve el mensaje “This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp (Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con Implementar en WebApp)”, seleccione **View (Ver)**, **Permit (Permitir)** y **Permit (Permitir)** de nuevo. Esto es necesario para permitir que la canalización cree el recurso de Azure App Service.

1. La implementación puede tardar unos minutos en completarse y esperar a que se ejecute la canalización.

1. Debería ver en los registros de canalización que la canalización usa la identidad administrada.

    ![Captura de pantalla de los registros de canalización mediante la identidad administrada.](media/pipeline-logs-managed-identity.png)

Una vez finalizada la canalización, puede ir a Azure Portal y comprobar el nuevo recurso de App Service.

### Ejercicio 5: Eliminación de los recursos del laboratorio de Azure

1. En Azure Portal, abra el grupo de recursos creado y seleccione **Eliminar grupo de recursos** para todos los recursos creados de este laboratorio.

    ![Captura de pantalla del botón de eliminación de un grupo de recursos.](media/delete-resource-group.png)

    > [!WARNING]
    > No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

## Revisar

En este laboratorio, ha aprendido a habilitar dinámicamente la configuración y a administrar las marcas de características.
