---
lab:
  title: Configuración de canalizaciones para usar variables y parámetros de forma segura
  module: 'Module 7: Configure pipelines to securely use variables and parameters'
---

# Configuración de canalizaciones para usar variables y parámetros de forma segura

En este laboratorio, aprenderás a configurar canalizaciones para usar de forma segura variables y parámetros.

Estos ejercicios duran aproximadamente **20** minutos.

## Antes de comenzar

Necesitarás una suscripción a Azure, una organización de Azure DevOps y la aplicación eShopOnWeb para seguir los laboratorios.

- Sigue los pasos para [validar el entorno de laboratorio](APL2001_M00_Validate_Lab_Environment.md).

## Instrucciones

### Ejercicio 1: Garantizar los tipos de parámetros y variables

#### Tarea 1: (omitir si ya la has completado) Importación y ejecución de la canalización de CI

Empecemos importando la canalización de CI denominada [eshoponweb-ci.yml](https://github.com/MicrosoftLearning/eShopOnWeb/blob/main/.ado/eshoponweb-ci.yml).

1. Ve al portal de Azure DevOps en `https://aex.dev.azure.com` y abre tu organización.

1. Abre el proyecto **eShopOnWeb** en Azure DevOps.

1. Ve a **Canalizaciones > Canalizaciones**.

1. Selecciona el botón **Crear canalización**.

1. Selecciona **Git de Azure Repos (YAML)**.

1. Selecciona el repositorio **eShopOnWeb**.

1. Selecciona el **archivo YAML de Azure Pipelines existente**.

1. Selecciona el archivo **/.ado/eshoponweb-ci.yml** y haz clic en **Continuar**.

1. Haz clic en el botón **Ejecutar** para ejecutar la canalización.

   > **Nota**: la canalización adoptará un nombre en función del nombre del proyecto. Cambiarás el nombre para identificar la canalización con más facilidad.

1. Ve a **Canalizaciones > Canalizaciones** y selecciona la canalización creada recientemente. Selecciona los puntos suspensivos (...) y, a continuación, selecciona **Cambiar nombre/mover**.

1. Asígnale el nombre **eshoponweb-ci** y selecciona **Guardar**.

#### Tarea 2: Garantizar tipos de parámetros para canalizaciones YAML

En esta tarea, establecerás los tipos de parámetro y parámetro para la canalización.

1. Ve a **Canalizaciones > Canalizaciones** y selecciona la canalización **eshoponweb-ci**.

1. Selecciona **Editar**.

1. Agrega los siguientes parámetros a la parte superior de las secciones de trabajo del archivo YAML:

   ```yaml
   parameters:
   - name: dotNetProjects
     type: string
     default: '**/*.sln'
   - name: testProjects
     type: string
     default: 'tests/UnitTests/*.csproj'

   jobs:
   - job: Build
     pool: eShopOnWebSelfPool
     steps:

   ```

1. Reemplaza las rutas de acceso codificadas de forma rígida en las tareas “Restaurar”, “Compilar” y “Probar” por los parámetros que acabas de crear.

   - **Reemplazar proyectos**: `**/*.sln` por proyectos: `${{ parameters.dotNetProjects }}` en las tareas `Restore` y `Build`.
   - **Reemplazar proyectos**: `tests/UnitTests/*.csproj` por proyectos: `${{ parameters.testProjects }}` en la tarea `Test`

   Las tareas "Restaurar", "Compilar" y "Probar" en la sección de pasos del archivo YAML deben tener este aspecto:

    ```yaml
    steps:
    - task: DotNetCoreCLI@2
      displayName: Restore
      inputs:
        command: 'restore'
        projects: ${{ parameters.dotNetProjects }}
        feedsToUse: 'select'
    
    - task: DotNetCoreCLI@2
      displayName: Build
      inputs:
        command: 'build'
        projects: ${{ parameters.dotNetProjects }}
    
    - task: DotNetCoreCLI@2
      displayName: Test
      inputs:
        command: 'test'
        projects: ${{ parameters.testProjects }}
    
    ```

1. Haz clic en **Validar y guardar** para guardar los cambios y, a continuación, haz clic en **Guardar**.

1. Ve a **Canalizaciones > Canalizaciones** y abre la canalización **eshoponweb-ci** que se ejecuta automáticamente.

1. Comprueba que la ejecución de canalización se completa correctamente.

   ![Captura de pantalla de la ejecución de canalización con parámetros.](media/pipeline-parameters-run.png)

#### Tarea 3: Protección de variables y parámetros

En esta tarea, protegerás las variables y los parámetros de la canalización mediante grupos de variables.

1. Ve a **Canalizaciones > Biblioteca**.

1. Selecciona el botón **+ Grupo de variables** para crear un nuevo grupo de variables denominado `BuildConfigurations`.

1. Agrega una variable denominada `buildConfiguration` y establece su valor en `Release`.

1. Guarda el grupo de variables.

   ![Captura de pantalla del grupo de variables con BuildConfigurations.](media/eshop-variable-group.png)

1. Selecciona el botón **Permisos de canalización** y, luego, el botón **+** para agregar una nueva canalización.

1. Selecciona la canalización **eshoponweb-ci** para permitir que la canalización use el grupo de variables.

   ![Captura de pantalla de los permisos de canalización.](media/pipeline-permissions.png)

   > **Nota**: también puede establecer usuarios o grupos específicos para poder editar el grupo de variables haciendo clic en el botón **Seguridad**.

1. Ve a **Canalizaciones > Canalizaciones**.

1. Abre la canalización **eshoponweb-ci** y selecciona **Editar**.

1. En la parte superior del archivo yml, justo debajo de los parámetros, haz referencia al grupo de variables agregando lo siguiente:

   ```yaml
   variables:
     - group: BuildConfigurations
   ```

1. En la tarea "Compilar", agrega el parámetro de configuración a la tarea para utilizar la configuración de compilación del grupo de variables.

    ```yaml
            command: 'build'
            projects: ${{ parameters.dotNetProjects }}
            configuration: $(buildConfiguration)
    ```

1. Haz clic en **Validar y guardar** para guardar los cambios y, a continuación, haz clic en **Guardar**.

1. Abre la ejecución de canalización **eshoponweb-ci**. Debe ejecutarse correctamente con la configuración de compilación establecida en "Liberar". Para comprobarlo, examina los registros de la tarea “Compilar”.

> **Nota**: si no ves la configuración de compilación establecida en "Liberar" en los registros, habilita el diagnóstico del sistema y vuelve a ejecutar la canalización para ver el valor de configuración.

> **Nota**: siguiendo este enfoque, puedes proteger las variables y los parámetros mediante el uso de grupos de variables sin tener que codificarlos de forma rígida en archivos YAML.

#### Tarea 4: Validación de variables y parámetros obligatorios

En esta tarea, validarás las variables obligatorias antes de que se ejecute la canalización.

1. Ve a **Canalizaciones > Canalizaciones**.

1. Abre la canalización **eshoponweb-ci** y selecciona **Editar**.

1. En la sección de pasos, al principio (siguiendo la línea **steps:**), agrega una nueva tarea de script para validar las variables obligatorias antes de que se ejecute la canalización.

    ```yaml
    - script: |
        IF NOT DEFINED buildConfiguration (
          ECHO Error: buildConfiguration variable is not set
          EXIT /B 1
        )
      displayName: 'Validate Variables'
     ```

    > **Nota**: se trata de una validación sencilla para comprobar si la variable está establecida. Si no se establecen las variables, se producirá un error en el script y se detendrá la canalización. Puedes agregar una validación más compleja para comprobar el valor de la variable o si se estableció en un valor específico.

1. Haz clic en **Validar y guardar** para guardar los cambios y, a continuación, haz clic en **Guardar**.

1. Abre la ejecución de canalización **eshoponweb-ci**. Se ejecutará correctamente porque la variable buildConfiguration se establece en el grupo de variables.

1. Para probar la validación, quita la variable buildConfiguration del grupo de variables o elimine el grupo de variables, o cambia el nombre de la variable, y vuelve a ejecutar la canalización. No debería completarse y debería aparecer el siguiente error:

    ```yaml
    Error: buildConfiguration variable is not set   
    ```

    ![Captura de pantalla de la ejecución de canalización con error de validación.](media/pipeline-validation-fail.png)

1. Vuelve a agregar la variable buildConfiguration al grupo de variables y vuelve a ejecutar la canalización. Debería ejecutarse correctamente.

> [!IMPORTANT]
> Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisión

En este laboratorio, aprendiste a configurar canalizaciones para usar de forma segura variables y parámetros, y cómo validar las variables y parámetros obligatorios.
