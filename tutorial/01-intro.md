<!-- markdownlint-disable MD002 MD041 -->

Este tutorial te enseña a crear una aplicación Microsoft Teams con ASP.NET Core y la API de Microsoft Graph para recuperar información de calendario para un usuario.

> [!TIP]
> Si prefiere descargar el tutorial completado, puede descargar o clonar el [repositorio GitHub archivo](https://github.com/microsoftgraph/msgraph-training-teamsapp-dotnet). Consulta el archivo README en la carpeta **de demostración** para obtener instrucciones sobre cómo configurar la aplicación con un identificador de aplicación y un secreto.

## <a name="prerequisites"></a>Requisitos previos

Antes de iniciar este tutorial, debe tener lo siguiente instalado en el equipo de desarrollo.

- [SDK de .NET](https://dotnet.microsoft.com/download).
- [ngrok](https://ngrok.com/)

También debe tener una cuenta laboral o educativa de Microsoft en un espacio empresarial Microsoft 365 que haya habilitado la instalación [Teams aplicación local.](/microsoftteams/platform/concepts/build-and-test/prepare-your-o365-tenant#enable-custom-teams-apps-and-turn-on-custom-app-uploading) Si no tienes una cuenta laboral o educativa de Microsoft o tu organización no ha habilitado la instalación local de aplicaciones de Teams personalizadas, puedes registrarte en el Programa de desarrolladores de [Microsoft 365](https://developer.microsoft.com/office/dev-program) para obtener una suscripción Office 365 desarrollador gratuita.

> [!NOTE]
> Este tutorial se escribió con .NET SDK versión 5.0.302. Los pasos de esta guía pueden funcionar con otras versiones, pero eso no se ha probado.

## <a name="feedback"></a>Comentarios

Proporcione cualquier comentario sobre este tutorial en el [repositorio GitHub archivo](https://github.com/microsoftgraph/msgraph-training-teamsapp-dotnet).
