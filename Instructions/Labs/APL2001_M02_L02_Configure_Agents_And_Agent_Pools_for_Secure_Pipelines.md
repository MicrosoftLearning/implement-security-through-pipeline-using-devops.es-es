---
lab:
  title: 'Laboratorio: Configurar agentes y grupos de agentes para canalizaciones seguras'
  module: 'Module 2: Configure secure access to pipeline resources'
---

# Laboratorio: Configurar agentes y grupos de agentes para canalizaciones seguras

En este laboratorio, aprenderá a configurar agentes y grupos de agentes de Azure DevOps y a administrar permisos para esos grupos. Los grupos de agentes de Azure DevOps proporcionan los recursos para ejecutar las canalizaciones de compilación y versión.

Estos ejercicios duran aproximadamente **25** minutos.

## Antes de comenzar

Necesitarás una suscripción a Azure, una organización de Azure DevOps y la aplicación eShopOnWeb para seguir los laboratorios.

- Sigue los pasos para [validar el entorno de laboratorio](APL2001_M00_Validate_Lab_Environment.md).

## Instrucciones

Creará agentes y configurará agentes autohospedados mediante Windows. Si desea configurar agentes en Linux o MacOS, siga las instrucciones de la [documentación de Azure DevOps](https://docs.microsoft.com/azure/devops/pipelines/agents/v2-linux).

Durante la configuración, tenga en cuenta lo siguiente:

- **Mantener agentes independientes por proyecto**: cada agente solo se puede asociar a un grupo. Aunque el uso compartido de grupos de agentes entre proyectos puede ahorrar en costes de infraestructura, también crea el riesgo de movimiento lateral. Por lo tanto, es mejor tener grupos de agentes independientes con agentes dedicados para cada proyecto para evitar la contaminación cruzada.
- **Uso de cuentas con pocos privilegios para los agentes en ejecución**: la ejecución de un agente bajo una identidad con acceso directo a los recursos de Azure DevOps puede suponer amenazas de seguridad. El funcionamiento del agente bajo una cuenta local sin privilegios, como el servicio de red, es aconsejable, lo que minimiza el riesgo.
- **Tenga cuidado con los nombres de grupo engañosos**: el grupo “Cuentas de servicio de recopilación de proyectos” en Azure DevOps conlleva un riesgo de seguridad potencial. La ejecución de agentes mediante una identidad que forma parte de este grupo y respaldada por Azure AD puede poner en peligro la seguridad de toda la organización de Azure DevOps.
- **Evite cuentas con privilegios elevados para agentes autohospedados**: el uso de cuentas con privilegios elevados para ejecutar agentes autohospedados, especialmente para acceder a secretos o entornos de producción, puede exponer el sistema a amenazas graves si una canalización está en peligro.
- **Priorice la seguridad**: para proteger los sistemas, use la cuenta con privilegios mínimos para ejecutar agentes autohospedados. Por ejemplo, use la cuenta de la máquina o una identidad de servicio administrada. También es aconsejable permitir que Azure Pipelines controle el acceso a secretos y entornos.

### Ejercicio 1: Creación de agentes y configuración de grupos de agentes

En este ejercicio, creará una máquina virtual (VM) de Azure y la usará para crear un agente y configurar grupos de agentes.

#### Tarea 1: Creación de una VM de Azure y conexión a ella

1. En el explorador, abra Azure Portal desde `https://portal.azure.com`. Si se le solicita, inicie sesión con una cuenta con el rol Propietario en su suscripción de Azure.

1. En el cuadro **Buscar recursos, servicios y documentos (G+/)**, escriba **Máquinas virtuales** y selecciónelo en la lista desplegable.

1. Seleccione el botón **Crear**.

1. Seleccione **Máquina virtual de Azure con configuración preestablecida**.

    ![Captura de pantalla de la creación de una máquina virtual con configuración preestablecida.](media/create-virtual-machine-preset.png)

1. Seleccione **Dev/Test** como entorno de carga de trabajo y **De uso general** como tipo de carga de trabajo.

1. Seleccione el botón **Seguir creando una VM**; en la pestaña **Aspectos básicos**, realice las siguientes acciones y, luego, seleccione **Administración**:

   | Configuración | Acción |
   | -- | -- |
   | Lista desplegable de **Suscripción** | Seleccione su suscripción a Azure. |
   | Sección **Grupo de recursos** | Cree un grupo de recursos denominado **rg-eshoponweb-agentpool**. |
   | Cuadro de texto **Nombre de máquina virtual**  | Escriba el nombre que prefiera, por ejemplo, **eshoponweb-vm**. |
   | Lista desplegable de **región** | Puedes elegir tu región de [Azure](https://azure.microsoft.com/explore/global-infrastructure/geographies) más cercana. Por ejemplo, "eastus", "eastasia", "westus", etc. |
   | Lista desplegable **Opciones de disponibilidad** | Seleccione **No se requiere redundancia de la infraestructura**. |
   | Lista desplegable **Tipo de seguridad** | Seleccione la opción **Máquinas virtuales de inicio seguro**. |
   | Lista desplegable de **imágenes** | Seleccione la imagen **Windows Server 2022 Datacenter: Azure Edition - x64 Gen2**. |
   | Lista desplegable de **Tamaño** | Seleccione el tamaño **estándar** más barato para realizar pruebas. |
   | Cuadro de texto **Nombre de usuario** | Escriba el nombre de usuario que prefiera. |
   | Cuadro de texto **Contraseña** | Escriba la contraseña que prefiera. |
   | Sección **Puertos de entrada públicos** | Seleccione **Permitir los puertos seleccionados**. |
   | Lista desplegable **Seleccionar puertos de entrada** | Seleccione **RDP (3389)**. |

1. En la pestaña **Administración**, en la sección **Identidad**, seleccione la casilla de verificación **Habilitar identidad administrada asignada por el sistema** y, luego, seleccione **Revisar y crear**:

1. En la pestaña **Revisar y crear**, selecciona **Crear**.

   > **Nota**: espera a que se complete el proceso de aprovisionamiento. Este proceso tardará alrededor de 2 minutos.

1. En Azure Portal, vaya a la página que muestra la configuración de la VM de Azure recién creada.

1. En la página de VM de Azure, selecciona **Conectar**, en el menú desplegable, selecciona **Conectar** y después selecciona **Descargar archivo RDP**.

1. Usa el archivo RDP descargado para establecer una sesión de Escritorio remoto en el sistema operativo que se ejecuta en la VM de Azure.

#### Tarea 2: Creación de un grupo de agentes

1. En la sesión de Escritorio remoto a la VM de Azure, inicie el explorador web de Microsoft Edge.

1. En el explorador web, vaya al portal de Azure DevOps en `https://aex.dev.azure.com` e inicie sesión para acceder a la organización.

   > **Nota**: Si es la primera vez que accedes al portal de Azure DevOps, es posible que tengas que crear tu perfil.

1. Abra el proyecto **eShopOnWeb** y seleccione **Configuración del proyecto** en el menú inferior izquierdo.

1. En **Canalizaciones > Grupos de agentes**, seleccione el botón **Agregar grupo**.

1. Elija el tipo de grupo **autohospedado**.

1. Proporcione un nombre para el grupo de agentes, como **eShopOnWebSelfPool** y agregue una descripción opcional.

1. Seleccione **Conceder permiso de acceso a todas las canalizaciones**.

   ![Captura de pantalla que muestra cómo agregar opciones de grupo de agentes con el tipo autohospedado.](media/create-new-agent-pool-self-hosted-agent.png)

   > **Nota:** No se recomienda conceder permiso de acceso a todas las canalizaciones para entornos de producción. Solo se usa en este laboratorio para simplificar la configuración de la canalización.

1. Seleccione el botón **Crear** para crear el grupo de agentes.

#### Tarea 3: Descarga y extracción de los archivos de instalación del agente

1. En el portal de Azure DevOps, seleccione el grupo de agentes recién creado y, luego, seleccione la pestaña **Agentes**.

1. Seleccione el botón **Nuevo agente** y, a continuación, el botón **Descargar** de la nueva ventana emergente **Descargar agente**.

   > **Nota**: Sigue las instrucciones de instalación para instalar el agente y anota la versión descargada en el nombre de archivo (por ejemplo: vsts-agent-win-x64-3.246.0.zip).

1. Inicie una sesión de PowerShell y ejecute los siguientes comandos para crear una carpeta denominada **agente**.

   ```powershell
   mkdir agent ; cd agent        
   ```

   > **Nota**: Asegúrate de que estás en la carpeta donde deseas instalar el agente, por ejemplo, C:\agent.

1. Ejecute el siguiente comando para extraer el contenido de los archivos del instalador del agente descargados:

   ```powershell
   Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$HOME\Downloads\vsts-agent-win-x64-3.246.0.zip", "$PWD")
   ```

   > **Nota**: Si has descargado el agente en otra ubicación (o la versión descargada difiere), ajusta el comando anterior en consecuencia.

#### Tarea 4: Creación de un token PAT

> **Nota**: Antes de configurar el agente, debes crear un token PAT (a menos que tengas uno existente). Para crear un token PAT, siga estos pasos:

1. En la sesión de Escritorio remoto de la VM de Azure, abre otra ventana del explorador, ve al portal de Azure DevOps en `https://aex.dev.azure.com` y accede a tu organización y proyecto.

1. Seleccione **Configuración de usuario** en el menú superior derecho (directamente a la izquierda del icono de avatar del usuario).

1. Seleccione el elemento de menú **Token de acceso personal**.

   ![Captura de pantalla que muestra el menú Tokens de acceso personal.](media/personal-access-token-menu.png)

1. Seleccione el botón **Nuevo token**.

1. Proporcione un nombre para el token, como **eShopOnWebToken**.

1. Seleccione la organización de Azure DevOps para la que quiere usar el token.

1. Establezca la fecha de expiración del token (solo se usa para configurar el agente).

1. Seleccione el ámbito definido personalizado.

1. Seleccione mostrar todos los ámbitos.

1. Seleccione el ámbito **Grupos de agentes (Leer y administrar).**

1. Seleccione el botón **Crear** para crear el token.

1. Copie el valor del token y guárdelo en un lugar seguro (no podrá volver a verlo. Solo puede regenerar el token).

   ![Captura de pantalla que muestra la configuración del token de acceso personal.](media/personal-access-token-configuration.png)

   > [!IMPORTANT]
   > Use la opción de privilegios mínimos, **Grupos de agentes (leer y administrar)**, solo para la configuración del agente. Además, asegúrese de establecer la fecha de expiración mínima para el token si ese es el único propósito del token. Puede crear otro token con los mismos privilegios si necesita volver a configurar el agente.

#### Tarea 5: Configuración del agente

1. En la sesión de Escritorio remoto a la VM de Azure, vuelva a la ventana de PowerShell. Si es necesario, cambie del directorio actual a aquél en el que extrajo los archivos de instalación del agente anteriormente en este ejercicio.

1. Para configurar el agente a fin de que se ejecute desatendido, invoque el siguiente comando:

   ```powershell
   .\config.cmd
   ```

   > **Nota**: Si deseas ejecutar el agente de forma interactiva, usa `.\run.cmd` en su lugar.

1. Para configurar el agente, realice las siguientes acciones cuando se le solicite:

   - Escriba la dirección URL de la organización de Azure DevOps (**dirección URL del servidor**) en el formato `https://aex.dev.azure.com`{nombre de la organización}.
   - Acepte el tipo de autenticación predeterminado (**PAT**).
   - Escriba el valor del token PAT que creó en el paso anterior.
   - Escribe el nombre del grupo de agentes **`eShopOnWebSelfPool`** que has creado anteriormente en este ejercicio.
   - Escribe el nombre del agente **`eShopOnWebSelfAgent`**.
   - Acepte la carpeta de trabajo del agente predeterminada (_work).
   - Escriba **Y** para configurar el agente a fin de que se ejecute como servicio.
   - Escriba **Y** para habilitar SERVICE_SID_TYPE_UNRESTRICTED para el servicio del agente.
   - Escriba **NT AUTHORITY\SYSTEM** para establecer el contexto de seguridad para el servicio.

   > [!IMPORTANT]
   > En general, debe seguir el principio de privilegios mínimos al configurar el contexto de seguridad del servicio.

   - Acepte la opción predeterminada (**N**) para permitir que el servicio se inicie inmediatamente después de finalizar la configuración.

   ![Captura de pantalla que muestra la configuración del agente.](media/agent-configuration.png)

   > **Nota**: El proceso de configuración del agente tardará unos minutos en completarse. Una vez hecho esto, verás un mensaje que indica que el agente se está ejecutando como servicio.

   > [!IMPORTANT] Si ves un mensaje de error que indica que el agente no se está ejecutando, es posible que tengas que iniciar el servicio manualmente. Para ello, abre el applet **Servicios** del panel de control de Windows, busca el servicio denominado **Agente de Azure DevOps (eShopOnWebSelfAgent)** e inícialo.

   > [!IMPORTANT] Si el agente no se inicia, es posible que tengas que elegir una carpeta diferente para el directorio de trabajo del agente. Para ello, vuelve a ejecutar el script de configuración del agente y elige otra carpeta.

1. Para comprobar el estado del agente, cambie al explorador web que muestra el portal de Azure DevOps, vaya al grupo de agentes y haga clic en la pestaña **Agentes**. Debería ver el nuevo agente en la lista.

   ![Captura de pantalla que muestra el estado del agente.](media/agent-status.png)

   > **Nota**: Para obtener más información sobre los agentes de Windows, consulta: [Agentes de Windows autohospedados](https://learn.microsoft.com/azure/devops/pipelines/agents/windows-agent)

   > [!IMPORTANT]
   > Para que el agente pueda compilar e implementar recursos de Azure desde las canalizaciones de Azure DevOps (que configurará en los próximos laboratorios), debe instalar la CLI de Azure en el sistema operativo de la máquina virtual de Azure que hospeda el agente.

1. Inicie un explorador web y vaya a la página [Instalación de la CLI de Azure en Windows](https://learn.microsoft.com/cli/azure/install-azure-cli-windows?tabs=azure-cli#install-or-update).

1. Descargue e instale la CLI de Azure.

1. (Opcional) Si lo prefieres, ejecuta el siguiente comando de PowerShell para instalar la CLI de Azure:

   ```powershell
   $ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi; Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'; Remove-Item .\AzureCLI.msi
   ```

   > **Nota**: Si usas una versión diferente de la CLI de Azure, es posible que tengas que ajustar el comando anterior en consecuencia.

1. En el explorador web, ve a la página del instalador del SDK de Microsoft .NET 8.0 en `https://dotnet.microsoft.com/en-us/download/dotnet/thank-you/sdk-8.0.403-windows-x64-installer`.

   > [!IMPORTANT]
   > Debes instalar el SDK de .NET 8.0 (o superior) en la VM de Azure que hospeda el agente. Esto es necesario para compilar la aplicación eShopOnWeb en los próximos laboratorios. Cualquier otra herramienta o SDK necesarios para la compilación de la aplicación también debe instalarse en la VM de Azure.

1. Descarga e instala el SDK de Microsoft .NET 8.0.

### Ejercicio 2: Creación y configuración de roles de seguridad

En este ejercicio, configurará la seguridad para el grupo de agentes.

#### Tarea 3: Creación de un grupo de seguridad de red

1. En la sesión de Escritorio remoto de la VM de Azure, en el explorador web que muestra el portal de Azure DevOps, en el panel **Configuración del proyecto**, en la sección **General**, selecciona **Permisos**.

1. Seleccione el botón **Nuevo grupo**.

1. Proporcione un nombre para el grupo, como **eShopOnWeb Security Group**.

1. Seleccione el botón **Crear** para crear el grupo.

   ![Captura de pantalla que muestra la creación del grupo de seguridad de red](media/create-security-group.png)

#### Tarea 4: Configuración del grupo de seguridad

1. En la ventana del explorador web que muestra el portal de Azure Devops, seleccione el nuevo grupo para mostrar la pestaña **Permisos**.

1. Deniegue los permisos innecesarios para el grupo, como **Cambiar nombre de proyecto de equipo**, **Eliminar definitivamente elementos de trabajo**o cualquier otro permiso que no quiera que tenga el grupo, ya que se supone que solo se usará para el grupo de agentes.

   ![Una captura de pantalla que muestra la configuración del grupo de seguridad.](media/security-group-settings.png)

   > [!IMPORTANT]
   > Si deja permisos que no desea que el grupo tenga, los scripts o las tareas que se ejecutan en el agente pueden usar los permisos de grupo para realizar acciones que no desea que realicen.

#### Tarea 3: Administración de permisos del grupo de agentes

En esta tarea, administrará los permisos del grupo de agentes.

1. En la ventana del explorador web que muestra el portal de Azure Devops, en la **Configuración del proyecto** del proyecto **eShopOnWeb**, en la sección **Canalizaciones**, seleccione **Grupos de agentes**.

1. Seleccione el grupo de agentes **eShopOnWebSelfPool**.

1. En la vista de detalles del grupo de agentes, seleccione la pestaña **Seguridad**.

1. Seleccione el botón **Agregar** y agregue el nuevo grupo **eShopOnWeb Security Group** a los permisos de usuario del grupo de agentes.

1. Elija el rol adecuado para el usuario o grupo, como lector, usuario o administrador del grupo de agentes. En este caso, seleccione **Usuario**.

1. Seleccione **Agregar** para aplicar los permisos.

   ![Captura de pantalla que muestra la configuración de seguridad del grupo de agentes.](media/agent-pool-security.png)

Ya tiene todo listo para usar de forma segura el grupo de agentes en las canalizaciones. Puedes usar el nuevo grupo para agregar usuarios y administrar permisos para el grupo de agentes. Puedes volver a configurar el agente autohospedado instalado utilizando el nuevo grupo para asegurarte de que el agente tiene los permisos necesarios para ejecutar las canalizaciones y nada más. Por ejemplo, puedes agregar un usuario al grupo y configurar el agente para que se ejecute como ese usuario.

Para obtener más información sobre los grupos de agentes, consulte [Grupos de agentes](https://learn.microsoft.com/azure/devops/pipelines/agents/pools-queues).

> [!IMPORTANT]
> Recuerda eliminar los recursos creados en Azure Portal para evitar cargos innecesarios.

## Revisión

En este laboratorio, aprenderá a configurar agentes autohospedados y grupos de agentes de Azure DevOps y a administrar permisos para esos grupos. Al administrar los permisos de forma eficaz, puede asegurarse de que los usuarios adecuados tengan acceso a los recursos que necesitan al tiempo que mantienen la seguridad e integridad de los procesos de DevOps.
