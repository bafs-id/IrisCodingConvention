# Refit – Library Template (IRIS Core APIs)

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
This example demonstrates how to integrate Refit into an ASP.NET Core application to call IRIS Core APIs in a strongly-typed and testable way.

### 🧱 Request Models

**`UpdateVehicleMeterStartedRequest.cs`**
```csharp
using IRIS.MobileBackend.Models.AppRequests;
using System.Text.Json.Serialization;

namespace IRIS.MobileBackend.Models.FlightService.AppRequests
{
    public record UpdateVehicleMeterStartedRequest
    {
        [property: JsonPropertyName("start_time"), JsonRequired]
        public new DateTime StartTime { get; init; }

        [property: JsonPropertyName("totalizer_start"), JsonRequired]
        public new decimal TotalizerStart { get; init; }
    }
}
```

**`CoreUpdateVehicleMeterStartedRequest.cs`**
```csharp
using IRIS.MobileBackend.Models.FlightService.AppRequests;
using System.Text.Json.Serialization;

namespace IRIS.MobileBackend.Models.IrisCore.Requests
{
    public record CoreUpdateVehicleMeterStartedRequest(
        [property: JsonPropertyName("start_time")]
        DateTime StartTime,

        [property: JsonPropertyName("totalizer_start")]
        decimal TotalizerStart
    )
    {
        public static CoreUpdateVehicleMeterStartedRequest FromRequest(UpdateVehicleMeterStartedRequest request)
        {
            return new CoreUpdateVehicleMeterStartedRequest(
                StartTime: request.StartTime,
                TotalizerStart: request.TotalizerStart
            );
        }
    }
}
```

---

### 🔌 Refit Interface

**`IIrisCoreApi.cs`**
```csharp
using IRIS.MobileBackend.Models.IrisCore.Requests;
using Refit;

namespace IRIS.MobileBackend.Abstractions
{
    public interface IIrisCoreApi
    {
        [Post("/vehicle_dispatches/{id}/meter_started")]
        Task NotifyVehicleMeterStartedAsync(
            [AliasAs("id")] Guid assignOperatorId,
            [Body] CoreUpdateVehicleMeterStartedRequest request
        );
    }
}
```

---

### 🔌 Interface & Implementation

- #### Service Abstraction

**`IVehicleServiceDetailService.cs`**
```csharp
using IRIS.MobileBackend.Models.FlightService.AppRequests;

namespace IRIS.MobileBackend.Abstractions
{
    public interface IVehicleServiceDetailService
    {
        ValueTask UpdateVehicleMeterStartedAsync(Guid id, UpdateVehicleMeterStartedRequest request);
    }
}
```

- #### Implementation

**`VehicleServiceDetailService.cs`**
```csharp
using IRIS.MobileBackend.Abstractions;
using IRIS.MobileBackend.Models.FlightService.AppRequests;
using IRIS.MobileBackend.Models.IrisCore.Requests;

namespace IRIS.MobileBackend.Services.FlightService
{
    public partial class VehicleServiceDetailService(IIrisCoreApi irisCoreApi) : IVehicleServiceDetailService
    {
        public async ValueTask UpdateVehicleMeterStartedAsync(Guid id, UpdateVehicleMeterStartedRequest request)
        {
            //--- Business Process ---
            var assignOperatorId = xxx; //Get the assign operator ID from the vehicle service detail;

            try
            {
                await irisCoreApi.NotifyVehicleMeterStartedAsync(
                    assignOperatorId: assignOperatorId,
                    request: CoreUpdateVehicleMeterStartedRequest.FromRequest(request)
                );
            }
            catch (Exception ex)
            {
                Serilog.Log.Error(ex, ex.Message);
            }  
        }
    }
}
```

---

### 🔌 Custom DelegatingHandler

**`IrisCoreSetting.cs`**
```csharp
namespace IRIS.MobileBackend.Models.AppModels
{
    public class IrisCoreSetting
    {
        public const string Name = "IrisCore";
        public string Url { get; set; } = string.Empty;
    }
}
```

**`TokenDelegatingHandler.cs`**
```csharp
using System.Net.Http.Headers;

namespace IRIS.MobileBackend.Middlewares
{
    public class TokenDelegatingHandler(IHttpContextAccessor httpContextAccessor) : DelegatingHandler
    {
        protected override async Task<HttpResponseMessage> SendAsync(
            HttpRequestMessage request, 
            CancellationToken cancellationToken
        )
        {
            var tokenArray = httpContextAccessor.HttpContext?.Request.Headers.Authorization.FirstOrDefault()?.Split(" ") ?? [];
            var token = tokenArray.Length > 0 ? tokenArray[^1] : "";
            var context = httpContextAccessor.HttpContext;

            if (!string.IsNullOrEmpty(token))
            {
                request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);
            }
            if (context?.Request.Headers.TryGetValue("X-Agency-Id", out var agencyIdHeader) == true)
            {
                request.Headers.Add("X-Agency-Id", agencyIdHeader.FirstOrDefault());
            }
            if (context?.Request.Headers.TryGetValue("X-Branch-Id", out var branchIdHeader) == true)
            {
                request.Headers.Add("X-Branch-Id", branchIdHeader.FirstOrDefault());
            }

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
var irisCoreSettingOptions = builder.Configuration.GetSection(IrisCoreSetting.Name).Get<IrisCoreSetting>();
builder.Services.AddRefitClient<IIrisCoreApi>()
    .ConfigureHttpClient(c => c.BaseAddress = new Uri(irisCoreSettingOptions.Url))
    .AddHttpMessageHandler<TokenDelegatingHandler>();

//--- Register application services, and custom dependencies ---

var app = builder.Build();

//--- Configure middleware pipeline and endpoints ---

app.Run();
```

---