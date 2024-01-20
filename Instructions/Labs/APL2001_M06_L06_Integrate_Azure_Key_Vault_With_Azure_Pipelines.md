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

En este laboratorio, sacará provecho de la entidad de servicio creada al validar el entorno de laboratorio para lo siguiente:

- Implementar recursos en tu suscripción de Azure.
- Obtener acceso de lectura a los secretos de Azure Key Vault.

## Instrucciones

En este laboratorio, verá cómo puede integrar Azure Key Vault con una canalización de Azure Pipelines mediante los pasos siguientes:

- Crear un almacén de Azure Key Vault para almacenar una contraseña ACR como secreto.
- Configure los permisos para permitir que la entidad de servicio lea el secreto.
- Configurar la canalización para recuperar la contraseña de Azure Key Vault y pasarla a las tareas posteriores.

### Ejercicio 1: Configuración de la canalización de CI para crear el contenedor eShopOnWeb.

En este ejercicio, configurará una canalización de YAML de CI para lo siguiente:

- Crear una instancia de Azure Container Registry para almacenar las imágenes de contenedor
- Uso de Docker Compose para compilar e insertar imágenes de contenedor **eshoppublicapi** y **eshopwebmvc**. Solo se implementará el contenedor **eshopwebmvc**.

#### Tarea 1: Configuración y ejecución de la canalización de CI

En esta tarea, importará una definición de canalización de YAML de CI existente, la modificará y la ejecutará. La canalización creará una instancia de Azure Container Registry (ACR) y compilará/publicará las imágenes de contenedor eShopOnWeb.

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Navegue al proyecto **eShopOnWeb** de Azure DevOps. Vaya a **Canalizaciones > Canalizaciones** y seleccione **Crear canalización**.

1. En la página **¿Dónde está su código?**, seleccione **Git de Azure Repos (YAML)** y seleccione el repositorio **eShopOnWeb**.

1. En la página **Configurar la canalización**, seleccione **Archivo YAML de Azure Pipelines existente**. Proporcione la siguiente ruta de acceso **/.ado/eshoponweb-ci-dockercompose.yml** y seleccione **Continue (Continuar)**.

   ![Captura de pantalla de la selección de canalización del archivo YAML.](media/select-ci-container-compose.png)

1. En la definición de canalización de YAML, en la sección de variables, realice las siguientes acciones:

   - reemplace **AZ400-EWebShop-NAME** por **rg-eshoponweb-docker**
   - establezca el valor de la variable de ubicación en el nombre de una región de Azure que haya usado en los laboratorios anteriores de este curso (por ejemplo, **southcentralus**)
   - reemplace **YOUR-SUBSCRIPTION-ID** por su identificador de suscripción de Azure

1. Seleccione **Guardar y ejecutar** y elija confirmar directamente en la rama principal.

1. Seleccione **Save and Run (Guardar y ejecutar)** de nuevo.

   > [!NOTE]
   > Si decide crear una rama, deberá crear una solicitud de incorporación de cambios para combinar los cambios en la rama principal.

1. Abra la canalización. Si ve el mensaje "Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con Docker Compose a WebApp", seleccione **Ver**, **Permitir** y **Permitir** de nuevo. Esto es necesario para permitir que la canalización cree los recursos de Azure.

   ![Captura de pantalla del permiso de acceso desde la canalización de YAML.](media/pipeline-permit-resource.png)

1. Espere a que se complete la ejecución de canalización. Esto puede tardar unos minutos. La canalización de compilación está formada por las siguientes tareas:

     - **AzureResourceManagerTemplateDeployment** usa **Bicep** para crear una instancia de Azure Container Registry.
     - La tarea de **PowerShell** toma la salida de Bicep (servidor de inicio de sesión acr) y crea una variable de canalización.
     - La tarea **DockerCompose** compila e inserta las imágenes de contenedor para eShopOnWeb en Azure Container Registry.

1. La canalización recibirá de manera predeterminada el nombre en función del nombre del proyecto. Cambie el nombre a **eshoponweb-ci-dockercompose** para identificar mejor la canalización.

1. Una vez completada la ejecución de canalización, use el explorador web para ir a Azure Portal, abra el grupo de recursos **rg-eshoponweb-docker** y seleccione la entrada que representa a la instancia de Azure Container Registry (ACR) implementada por la canalización.

   > [!NOTE]
   > Para ver repositorios en el registro, debe conceder a su cuenta de usuario un rol que proporcione dicho acceso. Usará para este propósito el rol AcrPull.

1. En la página Container registry (registro de contenedor), seleccione **Access control (Control de acceso) (IAM)**, seleccione **+ Add (+ Agregar)** y, en la lista desplegable, seleccione **Add role assignment (Agregar asignación de rol)**.

1. En la pestaña **Role (Rol)** de la página **Add role assignment (Agregar asignación de roles)**, seleccione **AcrPull** y, a continuación, seleccione **Next (Siguiente)**.

1. En la pestaña **Members (Miembros)**, haga clic en **+Select members (+ Seleccionar miembros)**, seleccione su cuenta de usuario, haga clic en **Select (Seleccionar)** y, a continuación, seleccione **Next (Siguiente)**.

1. Seleccione **Review + assign (Revisar y asignar)** y, una vez completada correctamente la asignación, actualice la página del explorador.

1. De nuevo en la página Container registry (Registro de contenedor), en la barra de menús vertical de la izquierda, en la sección **Services (Servicios)**, seleccione **Repositories (Repositorios)**.

1. Compruebe que el registro contiene las imágenes **eshoppublicapi** y **eshopwebmvc**. Solo usará **eshopwebmvc** en la fase de implementación.

   ![Captura de pantalla de las imágenes de contenedor de ACR de Azure Portal.](media/azure-container-registry.png)

1. Seleccione **Claves de acceso**, habilite la casilla **Usuario administrador** y copie el valor de **contraseña**, que se usará en la siguiente tarea, ya que lo agregará como secreto a Azure Key Vault.

   ![Captura de pantalla de la contraseña de ACR desde la configuración Claves de acceso.](media/acr-password.png)

1. En la misma página, registre el valor de **Nombre del registro**. Lo necesitará más adelante en este laboratorio.

#### Tarea 2: crear una instancia del Almacén de claves de Azure

En esta tarea, crearás un almacén de claves de Azure mediante Azure Portal.

En este escenario de laboratorio, habrá una instancia de Azure Container Instance (ACI) que extrae y ejecuta una imagen de contenedor almacenada en Azure Container Registry (ACR). Tenemos previsto almacenar la contraseña de ACR como un secreto en Azure Key Vault.

1. En Azure Portal, en el cuadro de texto **Buscar recursos, servicios y documentos**, escriba **Almacén de claves** y presione la tecla **Entrar**.

1. En la hoja **Almacén de claves**, seleccione **Crear > Almacén de claves**.

1. En la pestaña **Datos básicos** de la hoja **Crear un almacén de claves**, especifique la siguiente configuración y seleccione **Siguiente**:

   | Configuración | Valor |
   | --- | --- |
   | Suscripción | nombre de la suscripción de Azure que usa en este laboratorio |
   | Resource group | el nombre del grupo de recursos **rg-eshoponweb-docker** |
   | Nombre del almacén de claves | cualquier nombre válido único, como **ewebshop-kv-** seguido de un número aleatorio de seis dígitos |
   | Region | la misma región de Azure que eligió anteriormente en este laboratorio |
   | Plan de tarifa | **Estándar** |
   | Días durante los cuales se conservarán los almacenes eliminados | **7**
           |
   | Protección de purgas | **Deshabilitar la protección de purgas** |

1. En la pestaña **Configuración de acceso** de la hoja **Crear almacén de claves**, en la sección **Modelo de permisos**, seleccione **Directiva de acceso del almacén**. 

1. En la sección **Directivas de acceso**, seleccione **+ Crear** para configurar una nueva directiva.

   > **Nota**: Debe proteger el acceso a los almacenes de claves permitiendo el acceso unicamente a aplicaciones y usuarios autorizados. Para acceder a los datos del almacén, deberás facilitar permisos de lectura (Obtiene o enumera) a la entidad de servicio creada anteriormente que usarás para la autenticación en la canalización.

   - En la hoja **Permiso**, marque los permisos **Obtener** y **enumerar** en de **Permiso secreto**. Seleccione **Siguiente**.
   - En la hoja **Entidad de seguridad**, busque la entidad de servicio que creó al validar el entorno de laboratorio, ya sea mediante su identificador o su nombre. Seleccione **Siguiente** y luego **Siguiente** de nuevo.
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
   | Value | Contraseña de acceso de ACR copiada en la tarea anterior |

#### Tarea 3: Creación de un grupo de variables conectado a Azure Key Vault

En esta tarea, creará un grupo de variables en Azure DevOps que recuperará el secreto de contraseña de ACR de Key Vault mediante la Conexión de servicio (entidad de servicio).

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Navegue al proyecto **eShopOnWeb** de Azure DevOps.

1. En el panel de navegación vertical del portal de Azure DevOps, seleccione **Pipelines (Canalizaciones) > Library (Biblioteca)**. Seleccione **+ Variable Group (+ Grupo de variables)**.

1. En la hoja **New variable group (Nuevo grupo de variables)**, configure las opciones siguientes:

   | Configuración | Valor |
   | --- | --- |
   | Nombre del grupo de variables | **eshopweb-vg** |
   | Vincular secretos del almacén de claves de Azure como variables | **enable** |
   | Suscripción de Azure | **Conexión de servicio de Azure disponible > suscripción de Azure** |
   | Nombre del almacén de claves | el nombre que asignó a la instancia de Azure Key Vault en la tarea anterior |

1. En **Variables**, seleccione **+Add (+ Agregar)** y seleccione el secreto **acr-secret**. Seleccione **Aceptar**.

1. Seleccione **Guardar**.

   ![Captura de pantalla de la creación del grupo de variables.](media/vg-create.png)

#### Tarea 4: Configuración de la canalización de CD para implementar el contenedor en Azure Container Instance (ACI)

En esta tarea, importará una canalización de CD, la personalizará y la ejecutará para implementar la imagen de contenedor creada antes en una instancia de Azure Container Instance.

1. En el portal de Azure DevOps que muestra el proyecto **eShopOnWeb**, seleccione **Canalizaciones > Canalizaciones** y, a continuación, seleccione **Nueva canalización**.

1. En la página **¿Dónde está su código?**, seleccione **Git de Azure Repos (YAML)** y, a continuación, seleccione el repositorio **eShopOnWeb**.

1. En la página **Configurar la canalización**, seleccione **Archivo YAML de Azure Pipelines existente**. Proporcione la ruta de acceso **/.ado/eshoponweb-cd-aci.yml** y seleccione **Continuar**.

1. En la definición de canalización de YAML, en la sección de variable, realice las siguientes acciones:

   - establezca el valor de la variable de ubicación en el nombre de una región de Azure que usó anteriormente en este laboratorio
   - reemplace **YOUR-SUBSCRIPTION-ID** por su identificador de suscripción de Azure
   - reemplace **az400eshop-NAME** por un nombre único global de la instancia de Azure Container Instance que se va a implementar, por ejemplo, la cadena **eshoponweb-lab-docker,** seguida de un número aleatorio de seis dígitos. 
   - reemplace **YOUR-ACR** y **ACR-USERNAME** por el nombre del registro de ACR que registró anteriormente en este laboratorio.
   - reemplace **AZ400-EWebShop-NAME** por el nombre del grupo de recursos que creó anteriormente en este laboratorio (**rg-eshoponweb-docker**).

1. Seleccione **Guardar y ejecutar** y, a continuación, seleccione **Guardar y ejecutar** de nuevo.

1. Abra la canalización y observe el mensaje "Esta canalización necesita permiso para acceder a 2 recursos antes de que esta ejecución pueda continuar con Docker Compose a ACI". Seleccione **Ver** y, a continuación, seleccione **Permitir** dos veces (para cada recurso) para permitir que se ejecute la canalización.

1. Espere a que se complete la ejecución de canalización. Esto puede tardar unos minutos. La definición de compilación consta de una sola tarea **AzureResourceManagerTemplateDeployment**, que implementa Azure Container Instance (ACI) mediante una plantilla de Bicep y proporciona los parámetros de inicio de sesión de ACR para permitir que ACI descargue la imagen de contenedor creada anteriormente.

1. La canalización tomará un nombre en función del nombre del proyecto. Cambie el nombre a **eshoponweb-cd-aci** para facilitar la identificación de su propósito.

### Ejercicio 2: Limpieza de recursos de Azure y Azure DevOps

En este ejercicio, quitará los recursos de Azure y Azure DevOps creados en este laboratorio.

#### Tarea 1: Eliminación de recursos de Azure

1. En Azure Portal, vaya al grupo de recursos **rg-eshoponweb-docker** que contiene recursos implementados y seleccione **Eliminar grupo de recursos** para eliminar todos los recursos creados en este laboratorio.

#### Tarea 2: Eliminación de canalizaciones de Azure DevOps

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra el proyecto **eShopOnWeb** .

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Vaya a **Canalizaciones > Canalizaciones** y elimine las canalizaciones existentes.

#### Tarea 3: Volver a crear el repositorio de Azure DevOps

1. En el portal de Azure DevOps, en el proyecto **eShopOnWeb**, seleccione **Configuración del proyecto** en la esquina inferior izquierda.

1. En el menú vertical **Configuración del proyecto** en el lado izquierdo, en la sección **Repos**, seleccione**Repositorios**.

1. En el panel **Todos los repositorios**, mantenga el puntero sobre el extremo derecho de la entrada del repositorio **eShopOnWeb** hasta que aparezca el icono de puntos suspensivos de **Más opciones**; selecciónelo y, en el menú **Más opciones**, seleccione **Cambiar nombre**.  

1. En la ventana **Cambiar nombre del repositorio eShopOnWeb**, en el cuadro de texto **Nombre del repositorio**, escriba **eShopOnWeb_old** y seleccione **Cambiar nombre**.

1. Nuevamente en el panel **Todos los repositorios**, seleccione **+ Crear**.

1. En el panel **Crear un repositorio**, en el cuadro de texto **Nombre del repositorio**, escriba **eShopOnWeb**, desactive la casilla **Agregar un archivo README** y seleccione **Crear**.

1. De vuelta en el panel **Todos los repositorios**, mantenga el puntero sobre el extremo derecho de la entrada del repositorio **eShopOnWeb_old** hasta que aparezca el icono de puntos suspensivos **Más opciones**; selecciónelo y, en el menú **Más opciones**, seleccione **Eliminar**.  

1. En la ventana **Eliminar repositorio eShopOnWeb_old**, escriba **eShopOnWeb_old** y seleccione **Eliminar**.

1. En el menú de navegación izquierdo del portal de Azure DevOps, seleccione **Repositorios**.

1. En el panel **eShopOnWeb está vacío. Agregue código.**, seleccione **Importar un repositorio**.

1. En la ventana **Importar un repositorio de Git**, pegue la siguiente dirección URL `https://github.com/MicrosoftLearning/eShopOnWeb` y seleccione **Importar**:

## Revisar

En este laboratorio, integrarás el Almacén de claves de Azure con una canalización de Azure DevOps siguiendo estos pasos:

- Se usa una entidad de servicio de Azure para proporcionar acceso a los secretos de Azure Key Vault y para proporcionar acceso a los recursos de Azure desde Azure DevOps.
- Ejecuta dos canalizaciones de YAML importadas desde un repositorio de Git.
- Canalización configurada para recuperar la contraseña de Azure Key Vault mediante un grupo de variables y volver a usarla en tareas posteriores.
