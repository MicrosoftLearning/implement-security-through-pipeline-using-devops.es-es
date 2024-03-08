---
lab:
  title: Configurar una estructura de proyecto y repositorio para admitir canalizaciones seguras
  module: 'Module 1: Configure a project and repository structure to support secure pipelines'
---

# Configurar una estructura de proyecto y repositorio para admitir canalizaciones seguras

En este laboratorio, aprenderá a configurar una estructura de proyecto y repositorio en Azure DevOps para admitir canalizaciones seguras. En este laboratorio se tratan los procedimientos recomendados para organizar proyectos y repositorios, asignar permisos y administrar archivos seguros.

Estos ejercicios duran aproximadamente **30** minutos.

## Antes de comenzar

Necesitará una suscripción de Azure, una organización de Azure DevOps y la aplicación eShopOnWeb para seguir los laboratorios.

- Siga los pasos para [validar el entorno de laboratorio](APL2001_M00_Validate_Lab_Environment.md).

## Instrucciones

### Ejercicio 1: Configuración de una estructura de proyecto segura

En este ejercicio, configurará una estructura de proyecto segura mediante la creación de un nuevo proyecto y la asignación de permisos de proyecto. La separación de responsabilidades y recursos en diferentes proyectos o repositorios con permisos específicos complementa la seguridad.

#### Tarea 1: Crear un nuevo proyecto de equipo

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra la **configuración de la organización** en la esquina inferior izquierda del portal y, después, abra **Proyectos** en la sección General.

1. Seleccione la opción **Nuevo proyecto** y use la siguiente configuración:

   - Nombre: **eShopSecurity**
   - Visibilidad: **Privado**
   - Avanzado: Control de versiones: **Git**
   - Avanzado: Proceso de elemento de trabajo: **Scrum**

   ![Captura de pantalla del cuadro de diálogo del nuevo proyecto con la configuración especificada.](media/new-team-project.png)

1. Haga clic en **Crear** para crear el proyecto.

1. Ahora puede cambiar entre los diferentes proyectos haciendo clic en el icono de Azure DevOps de la esquina superior izquierda del portal de Azure DevOps.

   ![Captura de pantalla de los proyectos de equipo eShopOnWeb y eShopSecurity de Azure DevOps.](media/azure-devops-projects.png)

Puede administrar los permisos y la configuración de cada proyecto de equipo por separado. Para ello, vaya al menú Configuración del proyecto y seleccione el proyecto de equipo adecuado. Si tiene varios usuarios o equipos que trabajan en proyectos diferentes, también puede asignar permisos a cada proyecto por separado.

#### Tarea 2: Crear un nuevo repositorio y asignar permisos de proyecto

1. Seleccione el nombre de la organización en la esquina superior izquierda del portal de Azure DevOps y seleccione el nuevo proyecto **eShopSecurity**.

1. Seleccione el menú **Repositorios**.

1. Seleccione el botón **Inicializar** para inicializar el nuevo repositorio agregando el archivo README.md.

1. Abra el menú **Configuración del proyecto** en la esquina inferior izquierda del portal y seleccione **Repositorios** en la sección Repositorios.

1. Seleccione el nuevo repositorio de **eShopSecurity** y seleccione la pestaña **Seguridad**.

1. Quite los permisos Heredar del elemento primario desactivando el botón de alternancia **Herencia**.

1. Seleccione el grupo **Colaboradores** y seleccione la lista desplegable **Denegar** para todos los permisos excepto **Leer**. Esto impedirá que todos los usuarios del grupo Colaboradores accedan al repositorio.

1. Seleccione el usuario en Usuarios y seleccione el botón **Permitir** para permitir todos los permisos.

   > [!NOTE]
   > Si no ve su nombre en la sección **Usuarios**, escríbalo en el cuadro de texto **Buscar usuarios o grupos** y selecciónelo en la lista de resultados.

   ![Captura de pantalla de la configuración de seguridad del repositorio con permiso de lectura y denegación para todos los demás permisos.](media/repository-security.png)

1. Los cambios se guardarán automáticamente.

Ahora solo el usuario al que asignó permisos y los administradores pueden acceder al repositorio. Esto resulta útil cuando desea permitir que usuarios específicos accedan al repositorio y ejecuten canalizaciones desde el proyecto eShopOnWeb.

### Ejercicio 2: Configuración de una canalización y una estructura de plantilla para admitir canalizaciones seguras

#### Tarea 1: importar y ejecutar la canalización de CI

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra el proyecto **eShopOnWeb** en Azure DevOps.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Seleccione el botón **Crear canalización**.

1. Seleccione **Git de Azure Repos (YAML)**.

1. Selecciona el repositorio **eShopOnWeb**.

1. Seleccione **Archivo YAML de Azure Pipelines existente**.

1. Seleccione el archivo **/.ado/eshoponweb-ci.yml** y seleccione **Continue (Continuar)**.

1. Haga clic en el botón **Run (Ejecutar)** para ejecutar la canalización.

   > [!NOTE]
   > La canalización tomará un nombre basado en el nombre del proyecto. Cambiará el nombre para identificar la canalización con más facilidad.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)** y seleccione la canalización creada recientemente. Seleccione los puntos suspensivos (...) y, a continuación, seleccione **Cambiar nombre/mover**.

1. Asígnele el nombre **eshoponweb-ci** y seleccione **Guardar**.

#### Tarea 2: Importar y ejecutar la canalización de CD

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Seleccione el botón **New Pipeline (Nueva canalización)**.

1. Seleccione **Git de Azure Repos (YAML)**.

1. Selecciona el repositorio **eShopOnWeb**.

1. Seleccione **Archivo YAML de Azure Pipelines existente**.

1. Seleccione el archivo **/.ado/eshoponweb-cd-webapp-code.yml** y, después, seleccione **Continuar**.

1. En la definición de canalización de YAML en la sección variables, personalice lo siguiente:

   - **AZ400-EWebShop-NAME** con el nombre que prefiera, por ejemplo, **rg-eshoponweb-secure**.
   - **Location** con el nombre de la región de Azure en la que desea implementar los recursos, por ejemplo, **southcentralus**.
   - **YOUR-SUBSCRIPTION-ID** por el id. de la suscripción a Azure.
   - **az400-webapp-NAME** con un nombre único global de la aplicación web que se va a implementar, por ejemplo, la cadena **eshoponweb-lab-secure-** seguida de un número aleatorio de seis dígitos. 

1. Seleccione **Guardar y ejecutar** y elija hacer "commit" directamente en la rama principal.

1. Seleccione **Save and Run (Guardar y ejecutar)** de nuevo.

1. Abra la ejecución de la canalización. Si ve el mensaje “This pipeline needs permission to access a resource before this run can continue to Deploy to WebApp (Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con Implementar en WebApp)”, seleccione **View (Ver)**, **Permit (Permitir)** y **Permit (Permitir)** de nuevo. Esto es necesario para permitir que la canalización cree el recurso de Azure App Service.

   ![Captura de pantalla del permiso de acceso desde la canalización de YAML.](media/pipeline-deploy-permit-resource.png)

1. La implementación puede tardar unos minutos en completarse y esperar a que se ejecute la canalización. La canalización se desencadena después de la finalización de la canalización de CI e incluye las siguientes tareas:

   - **AzureResourceManagerTemplateDeployment**: Implementa la aplicación web de Azure App Service mediante una plantilla de Bicep.
   - **AzureRmWebAppDeployment**: Publica el sitio web en la aplicación web de Azure App Service.

1. La canalización tomará un nombre en función del nombre del proyecto. Vamos a cambiarle el nombre para identificar mejor la canalización.

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)** y seleccione la canalización creada recientemente. Seleccione los puntos suspensivos (...) y, a continuación, seleccione **Cambiar nombre/mover**.

1. Asígnele el nombre **eshoponweb-cd-webapp-code** y seleccione **Guardar**.

Ahora debería tener dos canalizaciones que se ejecutan en el proyecto eShopOnWeb.

![Captura de pantalla de las canalizaciones de CI/CD ejecutadas correctamente.](media/pipeline-successful-executed.png)

#### Tarea 3: Mover las variables de la canalización de CD a una plantilla de YAML

En esta tarea, creará una plantilla de YAML para almacenar las variables usadas en la canalización de CD. Esto le permitirá reutilizar la plantilla en otras canalizaciones.

1. Vaya a **Repositorios** y, después, a **Archivos**.

1. Expanda la carpeta **.ado** y seleccione **Nuevo archivo**.

1. Asigne al archivo el nombre **eshoponweb-secure-variables.yml** y seleccione **Crear**.

1. Agregue la sección variables que se usa en la canalización de CD al nuevo archivo. El archivo debería tener este aspecto:

   ```yaml
   variables:
     resource-group: 'rg-eshoponweb-secure'
     location: 'southcentralus' #the name of the Azure region you want to deploy your resources
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'azure subs' #the name of the service connection to your Azure subscription
     webappname: 'eshoponweb-lab-secure-XXXXXX' #the globally unique name of the web app
   ```

   > [!IMPORTANT]
   > Reemplace los valores de las variables por los valores del entorno (grupo de recursos, ubicación, identificador de suscripción, conexión de servicio de Azure y nombre de la aplicación web).

1. Seleccione **Commit**, en el cuadro de texto de comentario de "commit", escriba `[skip ci]` y, luego, seleccione **Commit**.

   > [!NOTE]
   > Mediante la adición del comentario `[skip ci]` a "commit", evitará la ejecución automática de la canalización que, en este momento, se ejecuta de forma predeterminada después de cada cambio en el repositorio. 

1. En la lista de archivos del repositorio, abra la definición de canalización de **eshoponweb-cd-webapp-code.yml** y reemplace la sección variables por lo siguiente:

   ```yaml
   variables:
     - template: eshoponweb-secure-variables.yml
   ```

1. Seleccione **Commit**, acepte el comentario predeterminado y, luego, seleccione **Commit** para volver a ejecutar la canalización.

1. Compruebe que la ejecución de canalización se haya completado correctamente. 

Ahora tiene una plantilla de YAML con las variables usadas en la canalización de CD. Puede reutilizar esta plantilla en otras canalizaciones en escenarios en los que necesite implementar los mismos recursos. Además, el equipo de operaciones puede controlar el grupo de recursos y la ubicación donde se implementan los recursos y otra información de los valores de la plantilla y no es necesario realizar ningún cambio en la definición de la canalización.

#### Tarea 4: Mover las plantillas de YAML a un repositorio y proyecto independientes

En esta tarea, moverá las plantillas de YAML a un repositorio y proyecto independientes.

1. En el proyecto eShopSecurity, vaya a **Repositorio > Archivos**.

1. Cree un nuevo archivo denominado **eshoponweb-secure-variables.yml**.

1. Copie el contenido del archivo **.ado/eshoponweb-secure-variables.yml** del repositorio eShopOnWeb al nuevo archivo.

1. Confirme los cambios.

1. Abra la definición de canalización **eshoponweb-cd-webapp-code.yml** en el repositorio eShopOnWeb.

1. Agregue lo siguiente a la sección de recursos:

   ```yaml
     repositories:
       - repository: eShopSecurity
         type: git
         name: eShopSecurity/eShopSecurity #name of the project and repository
   ```

1. Reemplace la sección variables por el contenido siguiente:

   ```yaml
   variables:
     - template: eshoponweb-secure-variables.yml@eShopSecurity #name of the template and repository
   ```

   ![Captura de pantalla de la definición de canalización con las nuevas variables y secciones de recursos.](media/pipeline-variables-resource-section.png)

1. Seleccione **Commit**, acepte el comentario predeterminado y, luego, seleccione **Commit** para volver a ejecutar la canalización.

1. Vaya a la ejecución de canalización y compruebe que la canalización use el archivo YAML desde el repositorio de eShopSecurity.

   ![Captura de pantalla de la ejecución de la canalización mediante la plantilla de YAML del repositorio eShopSecurity.](media/pipeline-execution-using-template.png)

Ahora tiene el archivo YAML en un repositorio y un proyecto independientes. Puede reutilizar este archivo en otras canalizaciones en escenarios en los que necesite implementar los mismos recursos. Además, el equipo de operaciones puede controlar el grupo de recursos, la ubicación, la seguridad, dónde se implementan los recursos y otra información modificando los valores en el archivo YAML y no es necesario realizar ningún cambio en la definición de canalización.

### Ejercicio 2: Limpieza de recursos de Azure y Azure DevOps

En este ejercicio, quitará los recursos de Azure y Azure DevOps creados en este laboratorio.

#### Tarea 1: Eliminación de recursos de Azure

1. En Azure Portal, vaya al grupo de recursos **rg-eshoponweb-secure**, que contiene los recursos implementados, y seleccione **Eliminar grupo de recursos** para eliminar todos los recursos creados en este laboratorio.

   ![Captura de pantalla del botón de eliminación de un grupo de recursos.](media/delete-resource-group.png)

   > [!WARNING]
   > No olvide quitar los recursos de Azure recién creados que ya no use. La eliminación de los recursos sin usar garantiza que no verá cargos inesperados.

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

1. En el panel **Crear un repositorio**, en el cuadro de texto **Nombre del repositorio**, escriba **eShopOnWeb**, desactive la casilla **Agregar un archivo LÉAME** y seleccione **Crear**.

1. De vuelta en el panel **Todos los repositorios**, mantenga el puntero sobre el extremo derecho de la entrada del repositorio **eShopOnWeb_old** hasta que aparezca el icono de puntos suspensivos **Más opciones**; selecciónelo y, en el menú **Más opciones**, seleccione **Eliminar**.  

1. En la ventana **Eliminar repositorio eShopOnWeb_old**, escriba **eShopOnWeb_old** y seleccione **Eliminar**.

1. En el menú de navegación izquierdo del portal de Azure DevOps, seleccione **Repositorios**.

1. En el panel **eShopOnWeb está vacío. Agregue código.**, seleccione **Importar un repositorio**.

1. En la ventana **Importar un repositorio de Git**, pegue la siguiente dirección URL `https://github.com/MicrosoftLearning/eShopOnWeb` y seleccione **Importar**:

## Revisar

En este laboratorio, aprendió a configurar una estructura de proyecto y repositorio en Azure DevOps para admitir canalizaciones seguras. Al administrar los permisos de forma eficaz, puede asegurarse de que los usuarios adecuados tengan acceso a los recursos que necesitan al tiempo que mantienen la seguridad e integridad de las canalizaciones y procesos de DevOps.
