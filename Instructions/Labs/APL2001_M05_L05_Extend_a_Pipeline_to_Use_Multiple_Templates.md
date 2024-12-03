---
lab:
  title: Extensión de una canalización para usar varias plantillas
  module: 'Module 5: Extend a pipeline to use multiple templates'
---

# Extensión de una canalización para usar varias plantillas

En este laboratorio, explorarás la importancia de extender una canalización a varias plantillas y cómo hacerlo mediante Azure DevOps. En este laboratorio se tratan conceptos básicos y procedimientos recomendados para crear una canalización de varias fases, crear una plantilla de variables, crear una plantilla de trabajo y crear una plantilla de fase.

Estos ejercicios duran aproximadamente **20** minutos.

## Antes de comenzar

Necesitarás una suscripción a Azure, una organización de Azure DevOps y la aplicación eShopOnWeb para seguir los laboratorios.

- Sigue los pasos para [validar el entorno de laboratorio](APL2001_M00_Validate_Lab_Environment.md).

## Instrucciones

### Ejercicio 1: Creación de canalizaciones YAML de varias fases

En este ejercicio, crearás una canalización YAML de varias fases en Azure DevOps.

#### Tarea 1: Creación de una canalización YAML principal de varias fases

1. Ve al portal de Azure DevOps en `https://aex.dev.azure.com` y abre tu organización.

1. Abre el proyecto **eShopOnWeb** .

1. Ve a **Canalizaciones > Canalizaciones**.

1. Haz clic en el botón **Nueva canalización**.

1. Selecciona **GIT de Azure Repos (YAML)**.

1. Selecciona el repositorio **eShopOnWeb**.

1. Selecciona **Canalización inicial**.

1. Reemplaza el contenido del archivo **azure-pipelines.yml** por el código siguiente:

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

1. Selecciona **Guardar y ejecutar**. Elige confirmar directamente en la rama principal y selecciona **Guardar y ejecutar** de nuevo.

1. Verás la canalización que se ejecuta con las tres fases (Desarrollo, Pruebas y Producción) y los trabajos correspondientes. Espera hasta que finalice la canalización y vuelve a la página **Canalizaciones**.

   ![Captura de pantalla de la canalización que se ejecuta con las tres fases y los trabajos correspondientes](media/eshoponweb-pipeline-multi-stage.png)

1. Selecciona **...** (Más opciones) en el lado derecho de la canalización que acabas de crear y selecciona **Rename/move (Cambiar nombre/mover)**.

1. Cambia el nombre de la canalización a **eShopOnWeb-MultiStage-Main** y selecciona **Guardar**.

#### Tarea 2: Creación de una plantilla de variables

1. Ve a **Repositorio > Archivos**.

1. Expande la carpeta **.ado** y haz clic en **Nuevo archivo**.

1. Asigna al archivo el nombre **eshoponweb-variables.yml** y haz clic en **Crear**.

1. Agrega el siguiente código al archivo:

   ```yaml
   variables:
     resource-group: 'YOUR-RESOURCE-GROUP-NAME'
     location: 'centralus'
     templateFile: 'infra/webapp.bicep'
     subscriptionid: 'YOUR-SUBSCRIPTION-ID'
     azureserviceconnection: 'YOUR-AZURE-SERVICE-CONNECTION-NAME'
     webappname: 'YOUR-WEB-APP-NAME'
   ```

1. Reemplaza los valores de las variables por los valores correctos de tu entorno:

   - Reemplaza **YOUR-RESOURCE-GROUP-NAME** por el nombre del grupo de recursos que deseas usar en este laboratorio, por ejemplo, **rg-eshoponweb-multi**.
   - Establece el valor de la variable **location** en el nombre de la región de Azure en la que quieres implementar los recursos, por ejemplo, **centralus**.
   - Reemplaza **YOUR-SUBSCRIPTION-ID** por tu identificador de suscripción a Azure.
   - Reemplaza **YOUR-AZURE-SERVICE-CONNECTION-NAME** por **azure subs**
   - Reemplaza **YOUR-WEB-APP-NAME** por un nombre único global de la aplicación web que se va a implementar, por ejemplo, la cadena **eshoponweb-lab-multi-123456** seguida de un número aleatorio de seis dígitos.  

1. Selecciona **Commit**, en el cuadro de texto de comentario de "commit", escribe `[skip ci]` y, luego, selecciona **Commit**.

   > **Nota**: mediante la adición del comentario `[skip ci]` a hacer "commit", evitarás la ejecución automática de la canalización que, en este momento, se ejecuta de forma predeterminada después de cada cambio en el repositorio. 

#### Tarea 3: Preparación de la canalización para usar plantillas

1. En el portal Azure DevOps, en la página del proyecto **eShopOnWeb**, ve a **Repositorios**.

1. En el directorio raíz del repositorio, selecciona **azure-pipelines.yml** que contiene la definición de la canalización **eShopOnWeb-MultiStage-Main**.

1. Haz clic en el botón **Editar**.

1. Reemplaza el contenido del archivo **azure-pipelines.yml** por el código siguiente:

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

1. Selecciona **Commit**, en el cuadro de texto de hacer "commit", escribe `[skip ci]` y, luego, selecciona **Commit**.

#### Tarea 4: Actualización de plantillas de CI/CD

1. En el **Repositorio** del proyecto **eShopOnWeb**, selecciona el directorio **.ado** y selecciona el archivo **eshoponweb-ci.yml**.

1. Haz clic en el botón **Editar**.

1. Quita todo lo que está encima de la sección **trabajos**.

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

1. Selecciona **Commit**, en el cuadro de texto de comentario de "commit", escribe `[skip ci]` y, a continuación, selecciona **Commit**.

1. En el **Repositorio** del proyecto **eShopOnWeb**, selecciona el directorio **.ado** y selecciona el archivo **eshoponweb-cd-webapp-code.yml**.

1. Haz clic en el botón **Editar**.

1. Quita todo lo que está encima de la sección **trabajos**.

   ```yaml
    # NAME THE PIPELINE SAME AS FILE (WITHOUT ".yml") #
    # Trigger CD when CI executed successfully
    
    resources:
      pipelines:
        - pipeline: eshoponweb-ci
          source: eshoponweb-ci # given pipeline name
          trigger: true
    
    repositories:
      - repository: eShopSecurity
        type: git
        name: eShopSecurity/eShopSecurity # name of the project and repository
    
    variables:
      - template: eshoponweb-secure-variables.yml@eShopSecurity # name of the template and repository
    
    stages:
      - stage: Test
        displayName: Testing WebApp
        jobs:
          - deployment: Test
            pool: eShopOnWebSelfPool
            environment: Test
            strategy:
              runOnce:
                deploy:
                  steps:
                    - script: echo Hello world! Testing environments!
    
      - stage: Deploy
        displayName: Deploy to WebApp
   ```

1. Reemplaza el contenido existente del paso **#download artifacts** por:

   ```yaml
       - download: current
         artifact: Website
       - download: current
         artifact: Bicep
   ```

1. Selecciona **Commit**, en el cuadro de texto de hacer "commit", escribe `[skip ci]` y, luego, selecciona **Commit**.

#### Tarea 5: Ejecución de la canalización principal

1. Ve a **Canalizaciones > Canalizaciones**.

1. Abre la canalización **eShopOnWeb-MultiStage-Main**.

1. Selecciona **Ejecutar canalización**.

   > **Nota**: si recibes un mensaje que indica que la canalización necesita permiso para acceder a un recurso antes de que la ejecución pueda continuar, selecciona **Ver** y, a continuación, selecciona **Permitir** y **Permitir** de nuevo para permitir que se ejecute la canalización.

   > **Nota**: si se produce un error en los trabajos de la fase de implementación, ve a la página de ejecución de la canalización y selecciona **Volver a ejecutar los trabajos con errores***.

1. Espera hasta que finalice la canalización y comprueba los resultados.

   ![Captura de pantalla de la canalización que se ejecuta con las tres fases y los trabajos correspondientes](media/multi-stage-completed.png)

> [!IMPORTANT]
> Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisión

En este laboratorio, has aprendido a ampliar una canalización en varias plantillas mediante Azure DevOps. En este laboratorio se han tratado conceptos básicos y procedimientos recomendados para crear una canalización de varias fases, crear una plantilla de variables, crear una plantilla de trabajo y crear una plantilla de fase.
