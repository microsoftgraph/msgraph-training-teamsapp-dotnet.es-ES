<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="47baf-101">En este ejercicio, extenderá la aplicación desde el ejercicio anterior para admitir la autenticación de inicio de sesión único con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="47baf-101">In this exercise you will extend the application from the previous exercise to support single sign-on authentication with Azure AD.</span></span> <span data-ttu-id="47baf-102">Esto es requerido para obtener el token de acceso OAuth necesario para llamar a la API de Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="47baf-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="47baf-103">En este paso, configurará la [biblioteca Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="47baf-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="47baf-104">Para evitar almacenar el identificador de aplicación y el secreto en el origen, usará el Administrador de secretos [de .NET](/aspnet/core/security/app-secrets) para almacenar estos valores.</span><span class="sxs-lookup"><span data-stu-id="47baf-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="47baf-105">El Administrador de secretos solo tiene fines de desarrollo, las aplicaciones de producción deben usar un administrador de secretos de confianza para almacenar secretos.</span><span class="sxs-lookup"><span data-stu-id="47baf-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="47baf-106">Abra **./appsettings.jsy** reemplace su contenido por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="47baf-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.example.json" highlight="2-8":::

1. <span data-ttu-id="47baf-107">Abra la CLI en el directorio donde se encuentra **GraphTutorial.csproj** y ejecute los siguientes comandos, sustituyéndolo por el identificador de la aplicación desde Azure Portal y con el secreto de la `YOUR_APP_ID` `YOUR_APP_SECRET` aplicación.</span><span class="sxs-lookup"><span data-stu-id="47baf-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="47baf-108">Implementar el inicio de sesión</span><span class="sxs-lookup"><span data-stu-id="47baf-108">Implement sign-in</span></span>

<span data-ttu-id="47baf-109">En primer lugar, implemente el inicio de sesión único en el código JavaScript de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="47baf-109">First, implement single sign-on in the app's JavaScript code.</span></span> <span data-ttu-id="47baf-110">Usará el SDK de [JavaScript](/javascript/api/overview/msteams-client) de Microsoft Teams para obtener un token de acceso que permita que el código JavaScript que se ejecuta en el cliente de Teams realice llamadas AJAX a la API web que implementará más adelante.</span><span class="sxs-lookup"><span data-stu-id="47baf-110">You will use the [Microsoft Teams JavaScript SDK](/javascript/api/overview/msteams-client) to get an access token which allows the JavaScript code running in the Teams client to make AJAX calls to Web API you will implement later.</span></span>

1. <span data-ttu-id="47baf-111">Abra **./Pages/Index.cshtml** y agregue el siguiente código dentro de la `<script>` etiqueta.</span><span class="sxs-lookup"><span data-stu-id="47baf-111">Open **./Pages/Index.cshtml** and add the following code inside the `<script>` tag.</span></span>

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

    <span data-ttu-id="47baf-112">Esto llama a `microsoftTeams.authentication.getAuthToken` la autenticación silenciosa como el usuario que ha iniciado sesión en Teams.</span><span class="sxs-lookup"><span data-stu-id="47baf-112">This calls the `microsoftTeams.authentication.getAuthToken` to silently authenticate as the user that is signed in to Teams.</span></span> <span data-ttu-id="47baf-113">Normalmente no hay ninguna solicitud de interfaz de usuario implicada, a menos que el usuario tenga que dar su consentimiento.</span><span class="sxs-lookup"><span data-stu-id="47baf-113">There is typically not any UI prompts involved, unless the user has to consent.</span></span> <span data-ttu-id="47baf-114">A continuación, el código muestra el token en la pestaña.</span><span class="sxs-lookup"><span data-stu-id="47baf-114">The code then displays the token in the tab.</span></span>

1. <span data-ttu-id="47baf-115">Guarde los cambios e inicie la aplicación ejecutando el siguiente comando en la CLI.</span><span class="sxs-lookup"><span data-stu-id="47baf-115">Save your changes and start your application by running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

    > [!IMPORTANT]
    > <span data-ttu-id="47baf-116">Si ha reiniciado ngrok y la dirección URL de ngrok ha cambiado, asegúrese de actualizar el valor de ngrok en el siguiente lugar antes **de** probar.</span><span class="sxs-lookup"><span data-stu-id="47baf-116">If you have restarted ngrok and your ngrok URL has changed, be sure to update the ngrok value in the following place **before** you test.</span></span>
    >
    > - <span data-ttu-id="47baf-117">Uri de redireccionamiento en el registro de la aplicación</span><span class="sxs-lookup"><span data-stu-id="47baf-117">The redirect URI in your app registration</span></span>
    > - <span data-ttu-id="47baf-118">Uri del identificador de aplicación en el registro de la aplicación</span><span class="sxs-lookup"><span data-stu-id="47baf-118">The application ID URI in your app registration</span></span>
    > - <span data-ttu-id="47baf-119">`contentUrl` en manifest.jsen</span><span class="sxs-lookup"><span data-stu-id="47baf-119">`contentUrl` in manifest.json</span></span>
    > - <span data-ttu-id="47baf-120">`validDomains` en manifest.jsen</span><span class="sxs-lookup"><span data-stu-id="47baf-120">`validDomains` in manifest.json</span></span>
    > - <span data-ttu-id="47baf-121">`resource` en manifest.jsen</span><span class="sxs-lookup"><span data-stu-id="47baf-121">`resource` in manifest.json</span></span>

1. <span data-ttu-id="47baf-122">Cree un archivo ZIP **conmanifest.js,** **color.png** y **outline.png**.</span><span class="sxs-lookup"><span data-stu-id="47baf-122">Create a ZIP file with **manifest.json**, **color.png**, and **outline.png**.</span></span>

1. <span data-ttu-id="47baf-123">En Microsoft Teams, selecciona  Aplicaciones en la barra izquierda, selecciona Upload una aplicación personalizada y, **a** continuación, selecciona Upload para mí o **mis equipos**.</span><span class="sxs-lookup"><span data-stu-id="47baf-123">In Microsoft Teams, select **Apps** in the left-hand bar, select **Upload a custom app**, then select **Upload for me or my teams**.</span></span>

    ![Una captura de pantalla del Upload un vínculo de aplicación personalizada en Microsoft Teams](images/upload-custom-app.png)

1. <span data-ttu-id="47baf-125">Vaya al archivo ZIP que creó anteriormente y seleccione **Abrir**.</span><span class="sxs-lookup"><span data-stu-id="47baf-125">Browse to the ZIP file you created previously and select **Open**.</span></span>

1. <span data-ttu-id="47baf-126">Revise la información de la aplicación y seleccione **Agregar**.</span><span class="sxs-lookup"><span data-stu-id="47baf-126">Review the application information and select **Add**.</span></span>

1. <span data-ttu-id="47baf-127">La aplicación se abre en Teams y muestra un token de acceso.</span><span class="sxs-lookup"><span data-stu-id="47baf-127">The application opens in Teams and displays an access token.</span></span>

<span data-ttu-id="47baf-128">Si copia el token, puede pegarlo [en](https://jwt.ms)jwt.ms .</span><span class="sxs-lookup"><span data-stu-id="47baf-128">If you copy the token, you can paste it into [jwt.ms](https://jwt.ms).</span></span> <span data-ttu-id="47baf-129">Compruebe que la audiencia (la notificación) es su identificador de aplicación y que el único ámbito `aud` (la `scp` notificación) es el `access_as_user` ámbito de la API que creó.</span><span class="sxs-lookup"><span data-stu-id="47baf-129">Verify that the audience (the `aud` claim) is your application ID, and the only scope (the `scp` claim) is the `access_as_user` API scope you created.</span></span> <span data-ttu-id="47baf-130">Esto significa que este token no concede acceso directo a Microsoft Graph!</span><span class="sxs-lookup"><span data-stu-id="47baf-130">That means that this token does not grant direct access to Microsoft Graph!</span></span> <span data-ttu-id="47baf-131">En su lugar, la API web que implementará [](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) pronto tendrá que intercambiar este token mediante el flujo en nombre del usuario para obtener un token que funcionará con las llamadas Graph Microsoft.</span><span class="sxs-lookup"><span data-stu-id="47baf-131">Instead, the Web API you will implement soon will need to exchange this token using the [on-behalf-of flow](/azure/active-directory/develop/v2-oauth2-on-behalf-of-flow) to get a token that will work with Microsoft Graph calls.</span></span>

## <a name="configure-authentication-in-the-aspnet-core-app"></a><span data-ttu-id="47baf-132">Configurar la autenticación en la ASP.NET Core aplicación</span><span class="sxs-lookup"><span data-stu-id="47baf-132">Configure authentication in the ASP.NET Core app</span></span>

<span data-ttu-id="47baf-133">Comience agregando los servicios de la plataforma Microsoft Identity a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="47baf-133">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="47baf-134">Abra el **archivo ./Startup.cs** y agregue la siguiente `using` instrucción a la parte superior del archivo.</span><span class="sxs-lookup"><span data-stu-id="47baf-134">Open the **./Startup.cs** file and add the following `using` statement to the top of the file.</span></span>

    ```csharp
    using Microsoft.Identity.Web;
    ```

1. <span data-ttu-id="47baf-135">Agregue la siguiente línea justo antes de `app.UseAuthorization();` la línea de la `Configure` función.</span><span class="sxs-lookup"><span data-stu-id="47baf-135">Add the following line just before the `app.UseAuthorization();` line in the `Configure` function.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="47baf-136">Agregue la siguiente línea justo después de `endpoints.MapRazorPages();` la línea de la `Configure` función.</span><span class="sxs-lookup"><span data-stu-id="47baf-136">Add the following line just after the `endpoints.MapRazorPages();` line in the `Configure` function.</span></span>

    ```csharp
    endpoints.MapControllers();
    ```

1. <span data-ttu-id="47baf-137">Reemplace la función `ConfigureServices` existente por lo siguiente.</span><span class="sxs-lookup"><span data-stu-id="47baf-137">Replace the existing `ConfigureServices` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="ConfigureServicesSnippet":::

    <span data-ttu-id="47baf-138">Este código configura la aplicación para permitir que las llamadas a las API web se autentiquen en función del token portador JWT en el `Authorization` encabezado.</span><span class="sxs-lookup"><span data-stu-id="47baf-138">This code configures the application to allow calls to Web APIs to be authenticated based on the JWT bearer token in the `Authorization` header.</span></span> <span data-ttu-id="47baf-139">También agrega los servicios de adquisición de tokens que pueden intercambiar ese token a través del flujo en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="47baf-139">It also adds the token acquisition services that can exchange that token via the on-behalf-of flow.</span></span>

## <a name="create-the-web-api-controller"></a><span data-ttu-id="47baf-140">Crear el controlador de API web</span><span class="sxs-lookup"><span data-stu-id="47baf-140">Create the Web API controller</span></span>

1. <span data-ttu-id="47baf-141">Cree un nuevo directorio en la raíz del proyecto denominado **Controladores**.</span><span class="sxs-lookup"><span data-stu-id="47baf-141">Create a new directory in the root of the project named **Controllers**.</span></span>

1. <span data-ttu-id="47baf-142">Cree un nuevo archivo en el **directorio ./Controllers** denominado **CalendarController.cs** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="47baf-142">Create a new file in the **./Controllers** directory named **CalendarController.cs** and add the following code.</span></span>

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

    <span data-ttu-id="47baf-143">Esto implementa una API web ( ) a `GET /calendar` la que se puede llamar desde la Teams pestaña. Por ahora, simplemente intenta intercambiar el token de portador por un token Graph.</span><span class="sxs-lookup"><span data-stu-id="47baf-143">This implements a Web API (`GET /calendar`) that can be called from the Teams tab. For now it simply tries to exchange the bearer token for a Graph token.</span></span> <span data-ttu-id="47baf-144">La primera vez que un usuario carga la pestaña, se producirá un error porque aún no han dado su consentimiento para permitir el acceso de la aplicación a Microsoft Graph en su nombre.</span><span class="sxs-lookup"><span data-stu-id="47baf-144">The first time a user loads the tab, this will fail because they have not yet consented to allow the app access to Microsoft Graph on their behalf.</span></span>

1. <span data-ttu-id="47baf-145">Abra **./Pages/Index.cshtml** y reemplace la `successCallback` función por la siguiente.</span><span class="sxs-lookup"><span data-stu-id="47baf-145">Open **./Pages/Index.cshtml** and replace the `successCallback` function with the following.</span></span>

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

    <span data-ttu-id="47baf-146">Esto llamará a la API web y mostrará la respuesta.</span><span class="sxs-lookup"><span data-stu-id="47baf-146">This will call the Web API and display the response.</span></span>

1. <span data-ttu-id="47baf-147">Guarde los cambios y reinicie la aplicación.</span><span class="sxs-lookup"><span data-stu-id="47baf-147">Save your changes and restart the app.</span></span> <span data-ttu-id="47baf-148">Actualice la pestaña en Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="47baf-148">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="47baf-149">La página debe mostrar `consent_required` .</span><span class="sxs-lookup"><span data-stu-id="47baf-149">The page should display `consent_required`.</span></span>

1. <span data-ttu-id="47baf-150">Revise el resultado del registro en la CLI.</span><span class="sxs-lookup"><span data-stu-id="47baf-150">Review the log output in your CLI.</span></span> <span data-ttu-id="47baf-151">Observe dos cosas.</span><span class="sxs-lookup"><span data-stu-id="47baf-151">Notice two things.</span></span>

    - <span data-ttu-id="47baf-152">Una entrada como `Authenticated user: MeganB@contoso.com` .</span><span class="sxs-lookup"><span data-stu-id="47baf-152">An entry like `Authenticated user: MeganB@contoso.com`.</span></span> <span data-ttu-id="47baf-153">La API web ha autenticado al usuario en función del token enviado con la solicitud de API.</span><span class="sxs-lookup"><span data-stu-id="47baf-153">The Web API has authenticated the user based on the token sent with the API request.</span></span>
    - <span data-ttu-id="47baf-154">Una entrada como `AADSTS65001: The user or administrator has not consented to use the application with ID...` .</span><span class="sxs-lookup"><span data-stu-id="47baf-154">An entry like `AADSTS65001: The user or administrator has not consented to use the application with ID...`.</span></span> <span data-ttu-id="47baf-155">Esto se espera, ya que aún no se ha solicitado al usuario el consentimiento para los ámbitos de permisos Graph Microsoft solicitados.</span><span class="sxs-lookup"><span data-stu-id="47baf-155">This is expected, since the user has not yet been prompted to consent for the requested Microsoft Graph permission scopes.</span></span>

## <a name="implement-consent-prompt"></a><span data-ttu-id="47baf-156">Implementar solicitud de consentimiento</span><span class="sxs-lookup"><span data-stu-id="47baf-156">Implement consent prompt</span></span>

<span data-ttu-id="47baf-157">Dado que la API web no puede preguntar al usuario, la Teams tendrá que implementar un mensaje.</span><span class="sxs-lookup"><span data-stu-id="47baf-157">Because the Web API cannot prompt the user, the Teams tab will need to implement a prompt.</span></span> <span data-ttu-id="47baf-158">Esto solo tendrá que hacerse una vez para cada usuario.</span><span class="sxs-lookup"><span data-stu-id="47baf-158">This will only need to be done once for each user.</span></span> <span data-ttu-id="47baf-159">Una vez que un usuario da su consentimiento, no es necesario volver a confirmarlo a menos que revoque explícitamente el acceso a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="47baf-159">Once a user consents, they do not need to reconsent unless they explicitly revoke access to your application.</span></span>

1. <span data-ttu-id="47baf-160">Cree un nuevo archivo en el directorio **./Pages** denominado **Authenticate.cshtml.cs** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="47baf-160">Create a new file in the **./Pages** directory named **Authenticate.cshtml.cs** and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Pages/Authenticate.cshtml.cs" id="AuthenticateModelSnippet":::

1. <span data-ttu-id="47baf-161">Cree un nuevo archivo en el directorio **./Pages** denominado **Authenticate.cshtml** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="47baf-161">Create a new file in the **./Pages** directory named **Authenticate.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/Authenticate.cshtml":::

1. <span data-ttu-id="47baf-162">Cree un nuevo archivo en el directorio **./Pages** denominado **AuthComplete.cshtml** y agregue el siguiente código.</span><span class="sxs-lookup"><span data-stu-id="47baf-162">Create a new file in the **./Pages** directory named **AuthComplete.cshtml** and add the following code.</span></span>

    :::code language="razor" source="../demo/GraphTutorial/Pages/AuthComplete.cshtml":::

1. <span data-ttu-id="47baf-163">Abra **./Pages/Index.cshtml** y agregue las siguientes funciones dentro de la `<script>` etiqueta.</span><span class="sxs-lookup"><span data-stu-id="47baf-163">Open **./Pages/Index.cshtml** and add the following functions inside the `<script>` tag.</span></span>

    :::code language="javascript" source="../demo/GraphTutorial/Pages/Index.cshtml" id="LoadUserCalendarSnippet":::

1. <span data-ttu-id="47baf-164">Agregue la siguiente función dentro de la `<script>` etiqueta para mostrar un resultado correcto de la API web.</span><span class="sxs-lookup"><span data-stu-id="47baf-164">Add the following function inside the `<script>` tag to display a successful result from the Web API.</span></span>

    ```javascript
    function renderCalendar(events) {
      $('#tab-container').empty();

      $('<pre/>').append($('<code/>', {
        text: JSON.stringify(events, null, 2),
        style: 'word-break: break-all;'
      })).appendTo('#tab-container');
    }
    ```

1. <span data-ttu-id="47baf-165">Reemplace el existente `successCallback` por el código siguiente.</span><span class="sxs-lookup"><span data-stu-id="47baf-165">Replace the existing `successCallback` with the following code.</span></span>

    ```javascript
    successCallback: (token) => {
      loadUserCalendar(token, (events) => {
        renderCalendar(events);
      });
    }
    ```

1. <span data-ttu-id="47baf-166">Guarde los cambios y reinicie la aplicación.</span><span class="sxs-lookup"><span data-stu-id="47baf-166">Save your changes and restart the app.</span></span> <span data-ttu-id="47baf-167">Actualice la pestaña en Microsoft Teams.</span><span class="sxs-lookup"><span data-stu-id="47baf-167">Refresh the tab in Microsoft Teams.</span></span> <span data-ttu-id="47baf-168">Debe obtener una ventana emergente que pida su consentimiento a los ámbitos de permisos de Graph Microsoft.</span><span class="sxs-lookup"><span data-stu-id="47baf-168">You should get a pop-up window asking for consent to the Microsoft Graph permissions scopes.</span></span> <span data-ttu-id="47baf-169">Después de aceptar, la pestaña debe mostrar `{ "status": "OK" }` .</span><span class="sxs-lookup"><span data-stu-id="47baf-169">After accepting, the tab should display `{ "status": "OK" }`.</span></span>

    > [!NOTE]
    > <span data-ttu-id="47baf-170">Si se muestra la pestaña , deshabilite los bloqueadores de elementos `"FailedToOpenWindow"` emergentes en el explorador y vuelva a cargar la página.</span><span class="sxs-lookup"><span data-stu-id="47baf-170">If the tab displays `"FailedToOpenWindow"`, please disable pop-up blockers in your browser and reload the page.</span></span>

1. <span data-ttu-id="47baf-171">Revise el resultado del registro.</span><span class="sxs-lookup"><span data-stu-id="47baf-171">Review the log output.</span></span> <span data-ttu-id="47baf-172">Debería ver la `Access token for Graph` entrada.</span><span class="sxs-lookup"><span data-stu-id="47baf-172">You should see the `Access token for Graph` entry.</span></span> <span data-ttu-id="47baf-173">Si analiza ese token, observará que contiene los ámbitos de microsoft Graph configurados **enappsettings.jsen**.</span><span class="sxs-lookup"><span data-stu-id="47baf-173">If you parse that token, you'll notice that it contains the Microsoft Graph scopes configured in **appsettings.json**.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="47baf-174">Almacenar y actualizar tokens</span><span class="sxs-lookup"><span data-stu-id="47baf-174">Storing and refreshing tokens</span></span>

<span data-ttu-id="47baf-175">En este momento, la aplicación tiene un token de acceso, que se envía en el `Authorization` encabezado de las llamadas API.</span><span class="sxs-lookup"><span data-stu-id="47baf-175">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="47baf-176">Este es el token que permite a la aplicación obtener acceso a Microsoft Graph en nombre del usuario.</span><span class="sxs-lookup"><span data-stu-id="47baf-176">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="47baf-177">Sin embargo, este token es de corta duración.</span><span class="sxs-lookup"><span data-stu-id="47baf-177">However, this token is short-lived.</span></span> <span data-ttu-id="47baf-178">El token expira una hora después de su emisión.</span><span class="sxs-lookup"><span data-stu-id="47baf-178">The token expires an hour after it is issued.</span></span> <span data-ttu-id="47baf-179">Aquí es donde el token de actualización resulta útil.</span><span class="sxs-lookup"><span data-stu-id="47baf-179">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="47baf-180">El token de actualización permite a la aplicación solicitar un nuevo token de acceso sin requerir que el usuario vuelva a iniciar sesión.</span><span class="sxs-lookup"><span data-stu-id="47baf-180">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="47baf-181">Dado que la aplicación usa la biblioteca Microsoft.Identity.Web, no es necesario implementar ninguna lógica de actualización o almacenamiento de tokens.</span><span class="sxs-lookup"><span data-stu-id="47baf-181">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="47baf-182">La aplicación usa la memoria caché de tokens en memoria, lo que es suficiente para las aplicaciones que no necesitan conservar tokens cuando se reinicia la aplicación.</span><span class="sxs-lookup"><span data-stu-id="47baf-182">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="47baf-183">En su lugar, las aplicaciones de producción pueden usar las opciones de caché [distribuida](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) en la biblioteca Microsoft.Identity.Web.</span><span class="sxs-lookup"><span data-stu-id="47baf-183">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="47baf-184">El método controla la expiración y actualización del token `GetAccessTokenForUserAsync` por usted.</span><span class="sxs-lookup"><span data-stu-id="47baf-184">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="47baf-185">Primero comprueba el token almacenado en caché y, si no ha expirado, lo devuelve.</span><span class="sxs-lookup"><span data-stu-id="47baf-185">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="47baf-186">Si ha expirado, usa el token de actualización en caché para obtener uno nuevo.</span><span class="sxs-lookup"><span data-stu-id="47baf-186">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="47baf-187">**GraphServiceClient que los** controladores obtienen a través de la inserción de dependencias está preconfigurado con un proveedor de autenticación que lo `GetAccessTokenForUserAsync` usa.</span><span class="sxs-lookup"><span data-stu-id="47baf-187">The **GraphServiceClient** that controllers get via dependency injection is pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
