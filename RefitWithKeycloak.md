# Refit – Library Template (Keycloak Admin APIs)

## 📦 NuGet

- **Package Ids:**
  - `Refit`
  - `Refit.HttpClientFactory` *(recommended for DI integration)*
- **Docs:** [https://github.com/reactiveui/refit](https://github.com/reactiveui/refit)

---

## 📖 Description

Refit is a REST library for .NET that turns your REST API into a live interface using attributes.

---

## 💻 Example Usage (C#)
This example demonstrates how to integrate Refit into an ASP.NET Core application to call Keycloak Admin APIs in a strongly-typed and testable way.

### 🧱 Response Model

**`KeycloakUserResponse.cs`**
```csharp
namespace IRIS.MobileBackend.Models.Keycloak.AppModels
{
    public class KeycloakUserResponse
    {
        public Guid Id { get; set; }
        public string Username { get; set; } = string.Empty;
        public string FirstName { get; set; } = string.Empty;
        public string LastName { get; set; } = string.Empty;
        public string Email { get; set; } = string.Empty;
        public bool Enabled { get; set; }
        public bool EmailVerified { get; set; }
    }
}
```

---

### 🔌 Refit Interface

**`IKeycloakAdminApi.cs`**
```csharp
using Refit;

namespace IRIS.MobileBackend.Abstractions
{
    public interface IKeycloakAdminApi
    {
        [Get("/users")]
        Task<ApiResponse<List<KeycloakUserResponse>>> GetUsersAsync(
            [Query] int? max = 100,
            [Query] string? username = null
        );

        [Get("/users/?username={username}")]
        Task<ApiResponse<List<KeycloakUserResponse>>> GetUserByUsernameAsync(
            [AliasAs("username")] string username
        );
    }
}
```

---

### 🔌 Interface & Implementation

- #### Repository Abstraction

**`IKeycloakAdminApiRepository.cs`**
```csharp
namespace IRIS.MobileBackend.Abstractions
{
    public interface IKeycloakAdminApiRepository
    {
        Task<List<KeycloakUserResponse>> GetUsersAsync(int? max = 100, string? username = null);
    }
}
```

- #### Implementation

**`KeycloakAdminApiRepository.cs`**
```csharp
using IRIS.MobileBackend.Abstractions;

namespace IRIS.MobileBackend.Repositories.Keycloak
{
    public class KeycloakAdminApiRepository(
        IKeycloakAdminApi keycloakAdminApi
    ) : IKeycloakAdminApiRepository
    {
        public async Task<List<KeycloakUserResponse>> GetUsersAsync(
            int? max = 100,
            string? username = null
        )
        {
            try
            {
                var response = await keycloakAdminApi.GetUsersAsync(
                    max: max,
                    username: username
                );

                if (response != null && response.Error != null)
                {
                    throw response.Error;
                }

                return response?.Content ?? [];
            }
            catch (Refit.ApiException ex)
            {
                throw new HttpRequestException(
                    $"failed to get users : {ex.Message} {ex.Content}"
                );
            }
            catch (Exception ex)
            {
                throw new HttpRequestException(
                    $"failed to get users : {ex.Message}"
                );
            }
        }
    }
}
```

---

### 🔌 Custom DelegatingHandler

**`KeycloakSetting.cs`**
```csharp
namespace IRIS.MobileBackend.Models.AppModels
{
    public class KeycloakSetting
    {
        public const string Name = "Keycloak";
        public string AdminRestApiBaseUrl { get; set; } = string.Empty;
        public string AdminRestApiClientId { get; set; } = string.Empty;
        public string AdminRestApiClientSecret { get; set; } = string.Empty;
        public string AdminRestApiGrantType { get; set; } = string.Empty;
    }
}
```

**`AccessTokenRequest.cs`**
```csharp
using System.Text.Json.Serialization;

namespace IRIS.MobileBackend.Models.Keycloak.AppRequests
{
    public class AccessTokenRequest
    {
        [property: JsonPropertyName("client_id"), JsonRequired]
        public string? ClientId { get; set; }

        [property: JsonPropertyName("client_secret"), JsonRequired]
        public string? ClientSecret { get; set; }

        [property: JsonPropertyName("grant_type"), JsonRequired]
        public string? GrantType { get; set; }
    }
}
```

**`KeycloakTokenDelegatingHandler.cs`**
```csharp
using IRIS.MobileBackend.Abstractions;
using IRIS.MobileBackend.Models.AppModels;
using IRIS.MobileBackend.Models.Keycloak.AppRequests;
using Microsoft.Extensions.Options;
using System.Net.Http.Headers;

namespace IRIS.MobileBackend.Middlewares
{
    public class KeycloakTokenDelegatingHandler(
        IOptions<KeycloakSetting> keycloakSetting,
        IKeycloakAuthApi keycloakAuthApi
    ) : DelegatingHandler
    {
        protected override async Task<HttpResponseMessage> SendAsync(
            HttpRequestMessage request,
            CancellationToken cancellationToken
        )
        {
            var response = await keycloakAuthApi.GetRealmManagementTokenAsync(
                new AccessTokenRequest()
                {
                    ClientId = keycloakSetting.Value.AdminRestApiClientId,
                    ClientSecret = keycloakSetting.Value.AdminRestApiClientSecret,
                    GrantType = keycloakSetting.Value.AdminRestApiGrantType,
                }
            );

            if (response.Error != null)
            {
                throw response.Error;
            }      

            request.Headers.Authorization = new AuthenticationHeaderValue(
                "Bearer",
                response.Content?.AccessToken
            );

            return await base.SendAsync(request, cancellationToken);
        }
    }
}
```

---

### 🧩 DI Registration

**`Program.cs`**
```csharp
using Refit;
using IRIS.MobileBackend.Abstractions;
using IRIS.MobileBackend.Models.AppModels;
using IRIS.MobileBackend.Middlewares;

var builder = WebApplication.CreateBuilder(args);
var keycloakSettingOptions = builder.Services.Configure<KeycloakSetting>(KeycloakSetting.Name);
builder.Services
    .AddRefitClient<IKeycloakAdminApi>()
    .ConfigureHttpClient(c => c.BaseAddress = new Uri(keycloakSettingOptions.AdminRestApiBaseUrl))
    .AddHttpMessageHandler<KeycloakTokenDelegatingHandler>();

//--- Register application services, and custom dependencies ---

var app = builder.Build();

//--- Configure middleware pipeline and endpoints ---

app.Run();
```

---