---
lab:
  title: Extender una canalización para usar varias plantillas
  module: 'Module 5: Extend a pipeline to use multiple templates'
---

# Extender una canalización para usar varias plantillas

En este laboratorio, explorará la importancia de extender una canalización a varias plantillas y cómo hacerlo mediante Azure DevOps. En este laboratorio se tratan conceptos básicos y procedimientos recomendados para crear una canalización de varias fases, crear una plantilla de variables, crear una plantilla de trabajo y crear una plantilla de fase.

Estos ejercicios duran aproximadamente **20** minutos.

## Antes de comenzar

Necesitará una suscripción de Azure, una organización de Azure DevOps y la aplicación eShopOnWeb para seguir los laboratorios.

- Siga los pasos para [validar el entorno de laboratorio](APL2001_M00_Validate_Lab_Environment.md).

## Instrucciones

### Ejercicio 1: Creación de canalizaciones YAML de varias etapas

#### Tarea 1: Creación de una canalización YAML principal de varias etapas

1. Vaya al portal de Azure DevOps en `https://dev.azure.com` y abra su organización.

1. Abra el proyecto **eShopOnWeb** .

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Seleccione **Crear canalización**.

1. Selecciona **GIT de Azure Repos (YAML)**.

1. Seleccione el repositorio **eShopOnWeb**.

1. Seleccione **Canalización inicial**.

1. Reemplace el contenido del archivo **azure-pipelines.yml** por el código siguiente:

   ```yaml
   trigger:
   - main

   pool:
     vmImage: 'windows-latest'

   stages:
   - stage: Dev
     jobs:
     - job: Build
       steps:
       - script: echo Build
   - stage: Test
     jobs:
     - job: Test
       steps:
       - script: echo Test
   - stage: Production
     jobs:
     - job: Deploy
       steps:
       - script: echo Deploy
   ```

1. Seleccione **Guardar y ejecutar**. Elija confirmar directamente en la rama principal y seleccione **Guardar y ejecutar** de nuevo.

1. Verá la canalización que se ejecuta con las tres etapas (Dev, Test y Production) y los trabajos correspondientes. Espere hasta que finalice la canalización y vuelva a la página **Canalizaciones**.

   ![Captura de pantalla de la canalización que se ejecuta con las tres fases y los trabajos correspondientes](media/eshoponweb-pipeline-multi-stage.png)

1. Seleccione **...** (Más opciones) en el lado derecho de la canalización que acaba de crear y seleccione **Rename/move (Cambiar nombre/mover)**.

1. Cambie el nombre de la canalización a **eShopOnWeb-MultiStage-Main** y seleccione **Save (Guardar)**.

#### Tarea 2: Creación de una plantilla de variables

1. Vaya a **Repos > Files (Repositorio > Archivos)**.

1. Expanda la carpeta **.ado** y haga clic en **New file (Nuevo archivo)**.

1. Asigne al archivo el nombre **eshoponweb-variables.yml** y haga clic en **Create (Crear)**.

1. Agregue el siguiente código al archivo:

   ```yaml
   variables:
     resource-group: 'YOUR-RESOURCE-GROUP-NAME'
     location: 'southcentralus' #name of the Azure region you want to deploy your resources
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
     webappname: 'YOUR-WEB-APP-NAME'
   ```

1. Reemplace los valores de las variables por los valores correctos de su entorno:

   - Reemplace **YOUR-RESOURCE-GROUP-NAME** por el nombre del grupo de recursos que desea usar en este laboratorio, por ejemplo, **rg-eshoponweb-multi**.
   - Establezca el valor de la variable **location** en el nombre de la región de Azure en la que quiere implementar los recursos, por ejemplo, **southcentralus**.
   - Reemplace **YOUR-SUBSCRIPTION-ID** por su identificador de suscripción de Azure.
   - Reemplace **YOUR-AZURE-SERVICE-CONNECTION-NAME** por **azure subs**
   - Reemplace **YOUR-WEB-APP-NAME** por un nombre único global de la aplicación web que se va a implementar, por ejemplo, la cadena **eshoponweb-lab-multi-** seguida de un número aleatorio de seis dígitos.  

1. Seleccione **Confirmar**, en el cuadro de texto de comentario de confirmación, escriba `[skip ci]` y, a continuación, seleccione **Confirmar**.

   > [!NOTE]
   > Mediante la adición del comentario `[skip ci]` a la confirmación, evitará la ejecución automática de la canalización, que, en este momento, se ejecuta de forma predeterminada después de cada cambio en el repositorio. 

#### Tarea 3: Preparación de la canalización para usar plantillas

1. En el portal Azure DevOps, en la página del proyecto **eShopOnWeb**, vaya a **Repositorios**. 

1. En el directorio raíz del repositorio, seleccione **azure-pipelines.yml** que contiene la definición de la canalización **eShopOnWeb-MultiStage-Main**.

1. Seleccione **Editar**.

1. Reemplace el contenido del archivo **azure-pipelines.yml** por el código siguiente:

   ```yaml
   trigger:
   - main
   variables:
   - template: .ado/eshoponweb-variables.yml
   
   stages:
   - stage: Dev
     jobs:
     - template: .ado/eshoponweb-ci.yml
   - stage: Test
     jobs:
     - template: .ado/eshoponweb-cd-webapp-code.yml
   - stage: Production
     jobs:
     - job: Deploy
       steps:
       - script: echo Deploy to Production or Swap
   ```

1. Seleccione **Confirmar**, en el cuadro de texto de comentario de confirmación, escriba `[skip ci]` y, a continuación, seleccione **Confirmar**.

#### Tarea 4: Actualización de plantillas de CI/CD

1. En el **Repositorio** del proyecto **eShopOnWeb**, seleccione el directorio **.ado** y seleccione el archivo **eshoponweb-ci.yml**.

1. Quite todo lo que está encima de la sección **jobs (trabajos)**.

   ```yaml
   #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
   # trigger:
   # - main
   
   resources:
     repositories:
       - repository: self
         trigger: none
   
   stages:
   - stage: Build
     displayName: Build .Net Core Solution
   ```

1. Seleccione **Confirmar**, en el cuadro de texto de comentario de confirmación, escriba `[skip ci]` y, a continuación, seleccione **Confirmar**.

1. En el **Repositorio** del proyecto **eShopOnWeb**, seleccione el directorio **.ado** y seleccione el archivo **eshoponweb-cd-webapp-code.yml**.

1. Quite todo lo que está encima de la sección **jobs (trabajos)**.

   ```yaml
   #NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml")
   
   # Trigger CD when CI executed successfully
   resources:
     pipelines:
       - pipeline: eshoponweb-ci
         source: eshoponweb-ci # given pipeline name
         trigger: true

   variables:
     resource-group: 'rg-eshoponweb'
     location: 'southcentralus'
     templateFile: '.azure/bicep/webapp.bicep'
     subscriptionid: ''
     azureserviceconnection: 'azure subs'
     webappname: 'eshoponweb-lab'
     # webappname: 'webapp-windows-eshop'
   
   stages:
   - stage: Deploy
     displayName: Deploy to WebApp`
   ```

1. Reemplace el contenido existente del paso **#download artifacts** por:

   ```yaml
       - download: current
         artifact: Website
       - download: current
         artifact: Bicep
   ```

1. Seleccione **Confirmar**, en el cuadro de texto de comentario de confirmación, escriba `[skip ci]` y, a continuación, seleccione **Confirmar**.

#### Tarea 5: Ejecución de la canalización principal

1. Vaya a **Pipelines (Canalizaciones) > Pipelines (Canalizaciones)**.

1. Abra la canalización **eShopOnWeb-MultiStage-Main**.

1. Seleccione **Run pipeline** (Ejecutar canalización).

1. Una vez que la canalización llegue a la fase de **implementación** en el entorno de **prueba**, abra la canalización y anote el mensaje "Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con la prueba". Seleccione **Ver** y, después, **Permitir** para que la canalización pueda ejecutarse.

   > [!NOTE]
   > Si se produce un error en los trabajos de la fase de implementación, vaya a la página de ejecución de la canalización y seleccione **Volver a ejecutar los trabajos con errores***.

1. Una vez que la canalización llegue a la fase de **implementación** en el entorno de **producción**, abra la canalización y anote el mensaje "Esta canalización necesita permiso para acceder a un recurso antes de que esta ejecución pueda continuar con la fase de producción". Seleccione **Ver** y, después, **Permitir** para que la canalización pueda ejecutarse.

1. Espere hasta que finalice la canalización y compruebe los resultados.

   ![Captura de pantalla de la canalización que se ejecuta con las tres fases y los trabajos correspondientes](media/multi-stage-completed.png)

### Ejercicio 2: Limpieza de recursos de Azure y Azure DevOps

En este ejercicio, quitará los recursos de Azure y Azure DevOps creados en este laboratorio.

#### Tarea 1: Eliminación de recursos de Azure

1. En Azure Portal, vaya al grupo de recursos **rg-eshoponweb-multi** que contiene los recursos implementados y seleccione **Eliminar grupo de recursos** para eliminar todos los recursos creados en este laboratorio.

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

En este laboratorio, ha aprendido a ampliar una canalización en varias plantillas mediante Azure DevOps. En este laboratorio se han tratado conceptos básicos y procedimientos recomendados para crear una canalización de varias fases, crear una plantilla de variables, crear una plantilla de trabajo y crear una plantilla de fase.