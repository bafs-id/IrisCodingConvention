# FluentValidation – Library Template

## 📦 NuGet

- **Package Ids:**

  * `FluentValidation`
  * `FluentValidation.AspNetCore` *(for ASP.NET Core auto-validation & MVC integration)*
  * `FluentValidation.DependencyInjectionExtensions` *(for `AddValidatorsFromAssembly*` helpers)*
- **Docs:** [https://docs.fluentvalidation.net/](https://docs.fluentvalidation.net/)
  
---

## 📖 Description

FluentValidation is a popular .NET library for building strongly-typed validation rules. It provides a fluent API to validate DTOs/commands and integrates with ASP.NET Core for automatic model validation and client-side adapters.

---

## 💻 Example Usage (C#)

This example demonstrates how to build a clean API endpoint in ASP.NET Core with request validation using FluentValidation.

### 🧱 Request Model

**`CreateDrNumberRequest.cs`**
```csharp
using System.Text.Json.Serialization;

namespace IRIS.MobileBackend.Models.DrNumber.AppRequests
{
    public record CreateDrNumberRequest(
        [property: JsonPropertyName("number"), JsonRequired]
        int Number,

        [property: JsonPropertyName("generated_at"), JsonRequired]
        DateTime GeneratedAt,

        [property: JsonPropertyName("vehicle_id"), JsonRequired]
        int VehicleId,

        [property: JsonPropertyName("service_type"), JsonRequired]
        string ServiceTypeRefcode
    );
}
```

---

### 🔌 Implementation (Fluent Validator)

**`CreateDrNumberValidator.cs`**
```csharp
using FluentValidation;
using IRIS.MobileBackend.Models.DrNumber.AppRequests;

namespace IRIS.MobileBackend.Validators
{
    public class CreateDrNumberValidator : AbstractValidator<CreateDrNumberRequest>
    {
        public CreateDrNumberValidator()
        {
            RuleFor(x => x.GeneratedAt.Date)
               .LessThanOrEqualTo(DateTime.UtcNow.Date);

            RuleFor(x => x.VehicleId)
                .NotNull()
                .NotEmpty();

            RuleFor(x => x.ServiceTypeRefcode)
                .NotNull()
                .MaximumLength(20);

            RuleFor(x => x.Number).GreaterThan(0);
        }
    }
}

```

---

### 🔌 Custom ApiBehaviorOptions

**`ApiBehaviorOptionsSetup`**
```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.ModelBinding;
using Microsoft.Extensions.Options;

namespace IRIS.MobileBackend.Middlewares
{
    public class ApiBehaviorOptionsSetup : IConfigureOptions<ApiBehaviorOptions>
    {
        public void Configure(ApiBehaviorOptions options)
        {
            options.InvalidModelStateResponseFactory = context =>
            {
                var errors = context.ModelState
                    .Where(x => x.Value?.Errors.Count > 0)
                    .SelectMany(x => x.Value?.Errors ?? Enumerable.Empty<ModelError>())
                    .Select(x => x.ErrorMessage).ToArray();

                var messages = $"One or more validation errors occurred.{Environment.NewLine}{string.Join(Environment.NewLine, errors)}";
                return new BadRequestObjectResult(messages);
            };
        }
    }
}
```

---

### 🧩 DI Registration

**`Program.cs`**
```csharp
using FluentValidation;
using FluentValidation.AspNetCore;
using IRIS.MobileBackend.Middlewares;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddFluentValidationAutoValidation()
    .AddFluentValidationClientsideAdapters();
builder.Services.ConfigureOptions<ApiBehaviorOptionsSetup>();

//--- Register application services, and custom dependencies ---

var app = builder.Build();

//--- Configure middleware pipeline and endpoints ---

app.Run();
```

---

### 🎮 Controller Example

**`Controllers/DrNumberController.cs`**
```csharp
using IRIS.MobileBackend.Abstractions;
using IRIS.MobileBackend.Models.DrNumber.AppRequests;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace IRIS.MobileBackend.Controllers
{
    [Route("dr_number")]
    [ApiController]
    [Authorize()]
    public class DrNumberController(
        IDrNumberService drNumberService
    ) : ControllerBase
    {
        [HttpPost()]
        public async ValueTask<IActionResult> CreateDrNumber(
            [FromBody] CreateDrNumberRequest request
        )
        {
            //--- Call Create DR Number Service---
            return Created();
        }
    }
}
```

---