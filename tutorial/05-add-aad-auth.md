<!-- markdownlint-disable MD002 MD041 -->

En este ejercicio, extenderá la aplicación desde el ejercicio anterior para admitir la autenticación de inicio de sesión único con Azure AD. Esto es requerido para obtener el token de acceso OAuth necesario para llamar a la API de Microsoft Graph. En este paso, configurará la [biblioteca Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)

> [!IMPORTANT]
> Para evitar almacenar el identificador de aplicación y el secreto en el origen, usará el Administrador de secretos [de .NET](/aspnet/core/security/app-secrets) para almacenar estos valores. El Administrador de secretos solo tiene fines de desarrollo, las aplicaciones de producción deben usar un administrador de secretos de confianza para almacenar secretos.

1. Abra **./appsettings.jsy** reemplace su contenido por lo siguiente.

    :::code language="json" source="../demo/GraphTutorial/appsettings.example.json" highlight="2-8":::

1. Abra la CLI en el directorio donde se encuentra **GraphTutorial.csproj** y ejecute los siguientes comandos, sustituyéndolo por el identificador de la aplicación desde Azure Portal y con el secreto de la `YOUR_APP_ID` `YOUR_APP_SECRET` aplicación.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Implementar el inicio de sesión

En primer lugar, implemente el inicio de sesión único en el código JavaScript de la aplicación. Usará el SDK de [JavaScript](/javascript/api/overview/msteams-client) de Microsoft Teams para obtener un token de acceso que permita que el código JavaScript que se ejecuta en el cliente de Teams realice llamadas AJAX a la API web que implementará más adelante.

1. Abra **./Pages/Index.cshtml** y agregue el siguiente código dentro de la `<script>` etiqueta.

    ```javascript
    (function () {
      if (microsoftTeams) {
        microsoftTeams.initialize();

        microsoftTeams.authentication.getAuthToken({
          successCallback: (token) => {
            // TEMPORARY: Display the access token for debugging
            $('#tab-container').empty();

            $('<code/>', {
              text: token,
              style: 'word-break: break-all;'
            }).appendTo('#tab-container');
          },
          failureCallback: (error) => {
            renderError(error);
          }
        });
      }
    })();

    function renderError(error) {
      $('#tab-container').empty();

      $('<h1/>', {
        text: 'Error'
      }).appendTo('#tab-container');

      $('<code/>', {
        text: JSON.stringify(error, Object.getOwnPropertyNames(error)),
        style: 'word-break: break-all;'
      }).appendTo('#tab-container');
    }
    ```

    Esto llama a `microsoftTeams.authentication.getAuthToken` la autenticación silenciosa como el usuario que ha iniciado sesión en Teams. Normalmente no hay ninguna solicitud de interfaz de usuario implicada, a menos que el usuario tenga que dar su consentimiento. A continuación, el código muestra el token en la pestaña.

1. Guarde los cambios e inicie la aplicación ejecutando el siguiente comando en la CLI.

    ```Shell
    dotnet run
    ```

    > [!IMPORTANT]
    > Si ha reiniciado ngrok y la dirección URL de ngrok ha cambiado, asegúrese de actualizar el valor de ngrok en el siguiente lugar antes **de** probar.
    >
    > - Uri de redireccionamiento en el registro de la aplicación
    > - Uri del identificador de aplicación en el registro de la aplicación
    > - `contentUrl` en manifest.jsen
    > - `validDomains` en manifest.jsen
    > - `resource` en manifest.jsen

1. Cree un archivo ZIP **conmanifest.js,** **color.png** y **outline.png**.

1. En Microsoft Teams, selecciona  Aplicaciones en la barra izquierda, selecciona Upload una aplicación personalizada y, **a** continuación, selecciona Upload para mí o **mis equipos**.

    ![Una captura de pantalla del Upload un vínculo de aplicación personalizada en Microsoft Teams](images/upload-custom-app.png)

1. Vaya al archivo ZIP que creó anteriormente y seleccione **Abrir**.

1. Revise la información de la aplicación y seleccione **Agregar**.

1. La aplicación se abre en Teams y muestra un token de acceso.

Si copia el token, puede pegarlo [en](https://jwt.ms)jwt.ms . Compruebe que la audiencia (la notificación) es su identificador de aplicación y que el único ámbito `aud` (la `scp` notificación) es el `access_as_user` ámbito de la API que creó. Esto significa que este token no concede acceso directo a Microsoft Graph! En su lugar, la API web que implementará [](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) pronto tendrá que intercambiar este token mediante el flujo en nombre del usuario para obtener un token que funcionará con las llamadas Graph Microsoft.

## <a name="configure-authentication-in-the-aspnet-core-app"></a>Configurar la autenticación en la ASP.NET Core aplicación

Comience agregando los servicios de la plataforma Microsoft Identity a la aplicación.

1. Abra el **archivo ./Startup.cs** y agregue la siguiente `using` instrucción a la parte superior del archivo.

    ```csharp
    using Microsoft.Identity.Web;
    ```

1. Agregue la siguiente línea justo antes de `app.UseAuthorization();` la línea de la `Configure` función.

    ```csharp
    app.UseAuthentication();
    ```

1. Agregue la siguiente línea justo después de `endpoints.MapRazorPages();` la línea de la `Configure` función.

    ```csharp
    endpoints.MapControllers();
    ```

1. Reemplace la función `ConfigureServices` existente por lo siguiente.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="ConfigureServicesSnippet":::

    Este código configura la aplicación para permitir que las llamadas a las API web se autentiquen en función del token portador JWT en el `Authorization` encabezado. También agrega los servicios de adquisición de tokens que pueden intercambiar ese token a través del flujo en nombre del usuario.

## <a name="create-the-web-api-controller"></a>Crear el controlador de API web

1. Cree un nuevo directorio en la raíz del proyecto denominado **Controladores**.

1. Cree un nuevo archivo en el **directorio ./Controllers** denominado **CalendarController.cs** y agregue el siguiente código.

    ```csharp
    using System;
    using System.Collections.Generic;
    using System.Net;
    using System.Threading.Tasks;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Http;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.Resource;
    using Microsoft.Graph;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        [ApiController]
        [Route("[controller]")]
        [Authorize]
        public class CalendarController : ControllerBase
        {
            private static readonly string[] apiScopes = new[] { "access_as_user" };

            private readonly GraphServiceClient _graphClient;
            private readonly ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<CalendarController> _logger;

            public CalendarController(ITokenAcquisition tokenAcquisition, GraphServiceClient graphClient, ILogger<CalendarController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _graphClient = graphClient;
                _logger = logger;
            }

            [HttpGet]
            public async Task<ActionResult<string>> Get()
            {
                // This verifies that the access_as_user scope is
                // present in the bearer token, throws if not
                HttpContext.VerifyUserHasAnyAcceptedScope(apiScopes);

                // To verify that the identity libraries have authenticated
                // based on the token, log the user's name
                _logger.LogInformation($"Authenticated user: {User.GetDisplayName()}");

                try
                {
                    // TEMPORARY
                    // Get a Graph token via OBO flow
                    var token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(new[]{
                            "User.Read",
                            "MailboxSettings.Read",
                            "Calendars.ReadWrite" });

                    // Log the token
                    _logger.LogInformation($"Access token for Graph: {token}");
                    return Ok("{ \"status\": \"OK\" }");
                }
                catch (MicrosoftIdentityWebChallengeUserException ex)
                {
                    _logger.LogError(ex, "Consent required");
                    // This exception indicates consent is required.
                    // Return a 403 with "consent_required" in the body
                    // to signal to the tab it needs to prompt for consent
                    return new ContentResult {
                        StatusCode = (int)HttpStatusCode.Forbidden,
                        ContentType = "text/plain",
                        Content = "consent_required"
                    };
                }
                catch (Exception ex)
                {
                    _logger.LogError(ex, "Error occurred");
                    throw;
                }
            }
        }
    }
    ```

    Esto implementa una API web ( ) a `GET /calendar` la que se puede llamar desde la Teams pestaña. Por ahora, simplemente intenta intercambiar el token de portador por un token Graph. La primera vez que un usuario carga la pestaña, se producirá un error porque aún no han dado su consentimiento para permitir el acceso de la aplicación a Microsoft Graph en su nombre.

1. Abra **./Pages/Index.cshtml** y reemplace la `successCallback` función por la siguiente.

    ```javascript
    successCallback: (token) => {
      // TEMPORARY: Call the Web API
      fetch('/calendar', {
        headers: {
          'Authorization': `Bearer ${token}`
        }
      }).then(response => {
        response.text()
          .then(body => {
            $('#tab-container').empty();
            $('<code/>', {
              text: body
            }).appendTo('#tab-container');
          });
      }).catch(error => {
        console.error(error);
        renderError(error);
      });
    }
    ```

    Esto llamará a la API web y mostrará la respuesta.

1. Guarde los cambios y reinicie la aplicación. Actualice la pestaña en Microsoft Teams. La página debe mostrar `consent_required` .

1. Revise el resultado del registro en la CLI. Observe dos cosas.

    - Una entrada como `Authenticated user: MeganB@contoso.com` . La API web ha autenticado al usuario en función del token enviado con la solicitud de API.
    - Una entrada como `AADSTS65001: The user or administrator has not consented to use the application with ID...` . Esto se espera, ya que aún no se ha solicitado al usuario el consentimiento para los ámbitos de permisos Graph Microsoft solicitados.

## <a name="implement-consent-prompt"></a>Implementar solicitud de consentimiento

Dado que la API web no puede preguntar al usuario, la Teams tendrá que implementar un mensaje. Esto solo tendrá que hacerse una vez para cada usuario. Una vez que un usuario da su consentimiento, no es necesario volver a confirmarlo a menos que revoque explícitamente el acceso a la aplicación.

1. Cree un nuevo archivo en el directorio **./Pages** denominado **Authenticate.cshtml.cs** y agregue el siguiente código.

    :::code language="csharp" source="../demo/GraphTutorial/Pages/Authenticate.cshtml.cs" id="AuthenticateModelSnippet":::

1. Cree un nuevo archivo en el directorio **./Pages** denominado **Authenticate.cshtml** y agregue el siguiente código.

    :::code language="razor" source="../demo/GraphTutorial/Pages/Authenticate.cshtml":::

1. Cree un nuevo archivo en el directorio **./Pages** denominado **AuthComplete.cshtml** y agregue el siguiente código.

    :::code language="razor" source="../demo/GraphTutorial/Pages/AuthComplete.cshtml":::

1. Abra **./Pages/Index.cshtml** y agregue las siguientes funciones dentro de la `<script>` etiqueta.

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="LoadUserCalendarSnippet":::

1. Agregue la siguiente función dentro de la `<script>` etiqueta para mostrar un resultado correcto de la API web.

    ```javascript
    function renderCalendar(events) {
      $('#tab-container').empty();

      $('<pre/>').append($('<code/>', {
        text: JSON.stringify(events, null, 2),
        style: 'word-break: break-all;'
      })).appendTo('#tab-container');
    }
    ```

1. Reemplace el existente `successCallback` por el código siguiente.

    ```javascript
    successCallback: (token) => {
      loadUserCalendar(token, (events) => {
        renderCalendar(events);
      });
    }
    ```

1. Guarde los cambios y reinicie la aplicación. Actualice la pestaña en Microsoft Teams. Debe obtener una ventana emergente que pida su consentimiento a los ámbitos de permisos de Graph Microsoft. Después de aceptar, la pestaña debe mostrar `{ "status": "OK" }` .

    > [!NOTE]
    > Si se muestra la pestaña , deshabilite los bloqueadores de elementos `"FailedToOpenWindow"` emergentes en el explorador y vuelva a cargar la página.

1. Revise el resultado del registro. Debería ver la `Access token for Graph` entrada. Si analiza ese token, observará que contiene los ámbitos de microsoft Graph configurados **enappsettings.jsen**.

## <a name="storing-and-refreshing-tokens"></a>Almacenar y actualizar tokens

En este momento, la aplicación tiene un token de acceso, que se envía en el `Authorization` encabezado de las llamadas API. Este es el token que permite a la aplicación obtener acceso a Microsoft Graph en nombre del usuario.

Sin embargo, este token es de corta duración. El token expira una hora después de su emisión. Aquí es donde el token de actualización resulta útil. El token de actualización permite a la aplicación solicitar un nuevo token de acceso sin requerir que el usuario vuelva a iniciar sesión.

Dado que la aplicación usa la biblioteca Microsoft.Identity.Web, no es necesario implementar ninguna lógica de actualización o almacenamiento de tokens.

La aplicación usa la memoria caché de tokens en memoria, lo que es suficiente para las aplicaciones que no necesitan conservar tokens cuando se reinicia la aplicación. En su lugar, las aplicaciones de producción pueden usar las opciones de caché [distribuida](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) en la biblioteca Microsoft.Identity.Web.

El método controla la expiración y actualización del token `GetAccessTokenForUserAsync` por usted. Primero comprueba el token almacenado en caché y, si no ha expirado, lo devuelve. Si ha expirado, usa el token de actualización en caché para obtener uno nuevo.

**GraphServiceClient que los** controladores obtienen a través de la inserción de dependencias está preconfigurado con un proveedor de autenticación que lo `GetAccessTokenForUserAsync` usa.
