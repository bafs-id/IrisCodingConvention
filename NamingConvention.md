# BAFS IRIS - Naming Conventions

## Overview

This document establishes the mandatory naming conventions for the BAFS IRIS Core API project based on comprehensive code review findings across 80+ files in Controllers, Services, and Repositories.

**Compliance Level**: MANDATORY - All new code must follow these conventions. Existing code violating these patterns should be refactored during maintenance.

---

## Core Naming Principles

### Primary Casing Styles

Our conventions are built on two primary casing styles that must be applied consistently:

| **Style**      | **Description**                                     | **Usage**                         |
| -------------- | --------------------------------------------------- | --------------------------------- |
| **PascalCase** | Capitalizes first letter of every word              | Public types, methods, properties |
| **camelCase**  | Capitalizes first letter of every word except first | Parameters, local variables       |

#### Identifier Naming Conventions

| Identifier Type        | Casing / Convention                      | Example                                           |
| ---------------------- | ---------------------------------------- | ------------------------------------------------- |
| Namespaces             | PascalCase                               | `namespace CompanyName.ProjectName.FeatureArea`   |
| Files & Directories    | PascalCase                               | `CustomerService.cs`, `/Services`                 |
| Class, Struct, Record  | PascalCase (Noun/Noun Phrase)            | `public class OrderProcessor { }`                 |
| Interface              | PascalCase, prefixed with I              | `public interface ILogger { }`                    |
| Enum                   | PascalCase                               | `public enum Status { Pending, Complete }`        |
| Method                 | PascalCase (Verb/Verb Phrase)            | `public void CalculateTotal() { }`                |
| Property               | PascalCase (Noun/Noun Phrase)            | `public string CustomerName { get; set; }`        |
| Event                  | PascalCase (Verb Phrase)                 | `public event EventHandler StatusChanged;`        |
| Delegate               | PascalCase, EventHandler/Callback suffix | `public delegate void TaskCompletedCallback();`   |
| Public/Const Field     | PascalCase                               | `public const int DefaultTimeout = 5000;`         |
| Private/Internal Field | \_camelCase (underscore prefix)          | `private readonly ILogger _logger;`               |
| Static Field (private) | \_camelCase (underscore prefix)          | `private static int _instanceCount;`              |
| Method Parameter       | camelCase                                | `public void SetName(string newName) { }`         |
| Local Variable         | camelCase                                | `var orderTotal = CalculatePrice();`              |
| Generic Type Parameter | T or TPascalCase                         | `class Repository<TEntity> where TEntity : class` |

### Key Rules

1. **Consistency**: Same naming pattern throughout the entire codebase
2. **Clarity**: Names should clearly indicate purpose and scope
3. **No Abbreviations**: Use full descriptive names instead of shortened versions
4. **Business Context**: Names should reflect the aviation fuel services domain

---

## Identifier Naming Standards

### Classes and Interfaces

```csharp
// ✅ CORRECT: Clear, descriptive class names
public class CustomerService { }
public class FlightScheduleController { }
public class DispatchRepository { }
public class PaymentReconcileService { }

// ✅ CORRECT: Interface naming with I prefix
public interface ICustomerService { }
public interface IFlightScheduleRepository { }
public interface IHeaderService { }
public interface IUserContextService { }

// ❌ WRONG: Abbreviated or unclear names (found in review)
public class CustSrv { }  // Too abbreviated
public class ContractReponsitories { }  // Typo in name
public class BaseRes { }  // Unclear abbreviation
```

### Methods and Properties

```csharp
// ✅ CORRECT: Descriptive method names (verb phrases)
public async Task<Customer> GetCustomerByIdAsync(int customerId) { }
public async Task<bool> ValidateFlightScheduleAsync(FlightSchedule schedule) { }
public void UpdateDispatchStatus(int dispatchId, string status) { }
public decimal CalculateFuelQuantity(decimal volume, decimal density) { }

// ✅ CORRECT: Property names (noun phrases)
public string CustomerName { get; set; }
public DateTime FlightDepartureTime { get; set; }
public decimal FuelQuantity { get; set; }
public bool IsActive { get; set; }

// ❌ WRONG: Poor method names (found in review)
public void DoWork() { }  // Too generic
public string GetData() { }  // Unclear what data
public bool Process() { }  // Unclear what's being processed
```

### Fields and Variables

```csharp
// ✅ CORRECT: Private fields with underscore prefix
public class CustomerService
{
    private readonly ICustomerRepository _customerRepository;
    private readonly ILogger<CustomerService> _logger;
    private readonly IUserContextService _userContextService;
    private readonly IHeaderService _headerService;
}

// ✅ CORRECT: Local variables and parameters
public async Task<Customer> ProcessCustomerRequest(CreateCustomerRequest request)
{
    var currentUserId = _userContextService.GetCurrentUserId();
    var existingCustomer = await _customerRepository.GetByEmailAsync(request.Email);
    var validationResult = ValidateCustomerData(request);

    if (validationResult.IsValid)
    {
        var newCustomer = MapToEntity(request);
        return await _customerRepository.CreateAsync(newCustomer);
    }

    throw new HandlingException(validationResult.ErrorMessage);
}

// ❌ WRONG: Poor variable names (found in review)
var _file = ProcessFile(); // Misleading - it's a result, not a file
var _baseRes = new BaseResponse(); // Abbreviated and unclear
string e = exception.Message; // Single letter variables
var BranchId = GetBranchId(); // Wrong casing for local variable
```

### Constants and Enums

```csharp
// ✅ CORRECT: Constant naming
public static class ServiceConstants
{
    public const int DefaultTimeoutSeconds = 30;
    public const char EmailDelimiter = ';';
    public const string ActiveStatus = "ACTIVE";
    public const string InactiveStatus = "INACTIVE";
}

public static class HeaderConstants
{
    public const string AgencyId = "X-Agency-Id";
    public const string BranchId = "X-Branch-Id";
    public const string UserId = "X-User-Id";
}

// ✅ CORRECT: Enum naming
public enum FlightStatus
{
    Scheduled,
    InProgress,
    Completed,
    Cancelled,
    Delayed
}

public enum FuelType
{
    JetA1,
    JetA,
    AvGas100LL,
    Diesel
}

// ❌ WRONG: Poor constant/enum names
public const int TIMEOUT = 30; // All caps not needed
public const string stat = "ACTIVE"; // Abbreviated
public enum Status { A, B, C } // Meaningless values
```

---

## Common Naming Issues Found

Based on the comprehensive review of 80+ files, these are the most common naming violations:

### Issue #1: Misleading Variable Names

```csharp
// ❌ FOUND IN REVIEW: Variables that don't represent what they contain
bool _file = service.ProcessFile(); // _file contains a boolean result, not a file
var data = GetCustomers(); // Generic "data" - should be "customers"
var result = CalculatePrice(); // Should be "calculatedPrice" or "totalPrice"

// ✅ CORRECT: Descriptive names that match content
bool processingResult = service.ProcessFile();
var customers = GetCustomers();
var totalPrice = CalculatePrice();
```

### Issue #2: Abbreviated Service Names

```csharp
// ❌ FOUND IN REVIEW: Abbreviated and unclear service references
var _baseRes = new BaseResponseService(); // Should be _baseResponseService
var _extFile = new ExternalFileService(); // Should be _externalFileService
var _custSvc = customerService; // Should be _customerService

// ✅ CORRECT: Full descriptive names
var _baseResponseService = new BaseResponseService();
var _externalFileService = new ExternalFileService();
var _customerService = customerService;
```

### Issue #3: Single Letter Variables

```csharp
// ❌ FOUND IN REVIEW: Single letter or meaningless variable names
string e = exception.Message; // Should be exceptionMessage
var i = GetIndex(); // Should be currentIndex
var x = CalculateValue(); // Should be calculatedValue

// ✅ CORRECT: Descriptive single-purpose names
string exceptionMessage = exception.Message;
var currentIndex = GetIndex();
var calculatedValue = CalculateValue();
```

### Issue #4: Wrong Casing for Scope

```csharp
// ❌ FOUND IN REVIEW: Wrong casing for variable scope
var BranchId = GetBranchId(); // Local variable should be camelCase
var AgencyId = headerService.GetAgencyId(); // Local variable should be camelCase

// ✅ CORRECT: Proper casing based on scope
var branchId = GetBranchId();
var agencyId = headerService.GetAgencyId();
```

---

## Project-Specific Patterns

### IRIS Aviation Domain Naming

```csharp
// ✅ CORRECT: Aviation fuel services domain naming
public class FlightScheduleService
{
    public async Task<Dispatch> CreateFuelDispatchAsync(CreateDispatchRequest request) { }
    public async Task<decimal> CalculateFuelQuantityAsync(FuelCalculationRequest request) { }
    public async Task<Vehicle> AssignFuelVehicleAsync(int flightId, int vehicleId) { }
}

public class AircraftService
{
    public async Task<AircraftType> GetAircraftTypeAsync(string aircraftCode) { }
    public async Task<decimal> GetFuelCapacityAsync(int aircraftTypeId) { }
    public async Task<Stand> GetAssignedStandAsync(int flightId) { }
}

// ✅ CORRECT: Business entity naming
public class FuelDispatch
{
    public int DispatchId { get; set; }
    public int FlightId { get; set; }
    public decimal RequestedQuantity { get; set; }
    public decimal DeliveredQuantity { get; set; }
    public DateTime DispatchTime { get; set; }
    public string VehicleRegistration { get; set; }
    public decimal FuelDensity { get; set; }
}
```

### Controller Action Naming

```csharp
// ✅ CORRECT: RESTful controller action naming
[Route("api/v1/flights")]
public class FlightController : ControllerAbstract
{
    [HttpGet]
    public async Task<IActionResult> GetFlights() { }

    [HttpGet("{id}")]
    public async Task<IActionResult> GetFlight(int id) { }

    [HttpPost]
    public async Task<IActionResult> CreateFlight([FromBody] CreateFlightRequest request) { }

    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateFlight(int id, [FromBody] UpdateFlightRequest request) { }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteFlight(int id) { }

    [HttpPost("{id}/dispatch")]
    public async Task<IActionResult> CreateFlightDispatch(int id, [FromBody] CreateDispatchRequest request) { }
}

// ❌ WRONG: Generic or unclear action names
public async Task<IActionResult> DoSomething() { }
public async Task<IActionResult> ProcessRequest() { }
public async Task<IActionResult> HandleData() { }
```

### Request/Response Model Naming

```csharp
// ✅ CORRECT: Clear request/response model naming
public class CreateCustomerRequest
{
    public string CustomerName { get; set; }
    public string EmailAddress { get; set; }
    public string PhoneNumber { get; set; }
}

public class CustomerDto
{
    public int CustomerId { get; set; }
    public string CustomerName { get; set; }
    public string EmailAddress { get; set; }
    public DateTime CreatedAt { get; set; }
    public string CreatedBy { get; set; }
}

public class UpdateFlightScheduleRequest
{
    public DateTime DepartureTime { get; set; }
    public DateTime ArrivalTime { get; set; }
    public string AircraftRegistration { get; set; }
}

// ❌ WRONG: Generic or unclear model names
public class Data { }
public class Request { }
public class Info { }
public class Model { }
```

---

## File and Folder Naming

### File Naming Standards

```csharp
// ✅ CORRECT: File names match primary class
CustomerService.cs          // Contains CustomerService class
ICustomerRepository.cs       // Contains ICustomerRepository interface
FlightScheduleController.cs  // Contains FlightScheduleController class
CreateCustomerRequest.cs     // Contains CreateCustomerRequest class
HandlingException.cs         // Contains HandlingException class

// ❌ WRONG: File names don't match content or are abbreviated
CustSvc.cs                   // Should be CustomerService.cs
Repo.cs                      // Should match the actual repository name
Utils.cs                     // Too generic
```

### Folder Organization

```
Controllers/
├── AircraftTypeController.cs
├── CustomerController.cs
├── FlightScheduleController.cs
├── DispatchController.cs
└── VehicleController.cs

Services/
├── CustomerService.cs
├── FlightScheduleService.cs
├── DispatchService.cs
├── ValidationService.cs
└── NotificationService.cs

Repositories/
├── CustomerRepository.cs
├── FlightScheduleRepository.cs
├── DispatchRepository.cs
└── VehicleRepository.cs

Models/
├── Requests/
│   ├── CreateCustomerRequest.cs
│   ├── UpdateCustomerRequest.cs
│   └── CreateFlightRequest.cs
├── Responses/
│   ├── CustomerDto.cs
│   ├── FlightDto.cs
│   └── DispatchDto.cs
└── Entities/
    ├── Customer.cs
    ├── Flight.cs
    └── Dispatch.cs
```

---

## Database and Entity Naming

### Entity Class Naming

```csharp
// ✅ CORRECT: Entity naming matching database tables
public class Customer
{
    public int CustomerId { get; set; }
    public string CustomerName { get; set; }
    public string EmailAddress { get; set; }
    public DateTime CreatedAt { get; set; }
    public string CreatedBy { get; set; }
}

public class FlightSchedule
{
    public int FlightId { get; set; }
    public string FlightNumber { get; set; }
    public DateTime DepartureTime { get; set; }
    public DateTime ArrivalTime { get; set; }
    public int AircraftTypeId { get; set; }
    public AircraftType AircraftType { get; set; }
}

// ✅ CORRECT: Navigation properties
public class Dispatch
{
    public int DispatchId { get; set; }
    public int FlightId { get; set; }
    public Flight Flight { get; set; }
    public int VehicleId { get; set; }
    public Vehicle Vehicle { get; set; }
    public List<DispatchItem> DispatchItems { get; set; }
}
```

### Repository Method Naming

```csharp
// ✅ CORRECT: Repository method naming patterns
public interface ICustomerRepository
{
    Task<Customer?> GetByIdAsync(int customerId);
    Task<Customer?> GetByEmailAsync(string email);
    Task<List<Customer>> GetAllActiveAsync();
    Task<List<Customer>> GetByCreatedDateAsync(DateTime date);
    Task<Customer> CreateAsync(Customer customer);
    Task<Customer> UpdateAsync(Customer customer);
    Task<bool> DeleteAsync(int customerId);
    Task<bool> ExistsAsync(int customerId);
    Task<bool> ExistsWithEmailAsync(string email);
}

// ❌ WRONG: Generic or unclear repository methods
Task<Customer> Get(int id); // Should be GetByIdAsync
Task<List<Customer>> All(); // Should be GetAllAsync
Task Save(Customer customer); // Should be CreateAsync or UpdateAsync
Task Remove(int id); // Should be DeleteAsync
```

---

## API and HTTP Naming

### Route Naming

```csharp
// ✅ CORRECT: RESTful route naming
[Route("api/v1/customers")]           // Plural nouns for collections
[Route("api/v1/flights")]             // Clear business entities
[Route("api/v1/dispatches")]          // Consistent plural naming
[Route("api/v1/aircraft-types")]      // Kebab-case for compound words

// ✅ CORRECT: Action-specific routes
[HttpPost("api/v1/flights/{id}/dispatch")]        // Action on specific resource
[HttpPut("api/v1/customers/{id}/activate")]       // State change actions
[HttpGet("api/v1/dispatches/{id}/details")]       // Detailed views

// ❌ WRONG: Inconsistent or unclear routes
[Route("api/v1/customer")]            // Should be plural
[Route("api/v1/data")]                // Too generic
[Route("api/v1/stuff")]               // Meaningless
```

### HTTP Method and Action Alignment

```csharp
// ✅ CORRECT: HTTP method and action name alignment
[HttpGet]
public async Task<IActionResult> GetCustomers() { } // GET for retrieval

[HttpPost]
public async Task<IActionResult> CreateCustomer([FromBody] CreateCustomerRequest request) { } // POST for creation

[HttpPut("{id}")]
public async Task<IActionResult> UpdateCustomer(int id, [FromBody] UpdateCustomerRequest request) { } // PUT for full update

[HttpPatch("{id}")]
public async Task<IActionResult> PatchCustomer(int id, [FromBody] JsonPatchDocument<UpdateCustomerRequest> patch) { } // PATCH for partial update

[HttpDelete("{id}")]
public async Task<IActionResult> DeleteCustomer(int id) { } // DELETE for removal
```

---

## Validation and Verification

### Common Validation Checks

```csharp
// ✅ VERIFY: No abbreviations
CustomerService ✓         // Not CustSvc
FlightScheduleRepository ✓ // Not FlightRepo
BaseResponseService ✓     // Not BaseRes

// ✅ VERIFY: Proper scope casing
private readonly ILogger _logger; ✓        // Private field
var customerId = request.Id; ✓             // Local variable
public string CustomerName { get; set; } ✓ // Public property

// ✅ VERIFY: Business context clarity
CreateDispatchRequest ✓    // Clear what's being created
FuelCalculationService ✓   // Clear business purpose
AircraftTypeRepository ✓   // Clear entity and layer
```

---

## Examples from IRIS Project

### Real Examples from Code Review

```csharp
// ❌ FOUND IN REVIEW: Poor naming from actual files
private readonly BaseResponseService _baseRes; // Should be _baseResponseService
private readonly ExternalFileService _extFile; // Should be _externalFileService
bool _file = service.ProcessFile(); // Should be processingResult
string e = exception.Message; // Should be exceptionMessage

// ✅ CORRECTED: Following IRIS naming conventions
private readonly BaseResponseService _baseResponseService;
private readonly ExternalFileService _externalFileService;
bool processingResult = service.ProcessFile();
string exceptionMessage = exception.Message;
```

### Project-Specific Service Examples

```csharp
// ✅ ACTUAL IRIS SERVICES: Following proper naming
public class DispatchService : IDispatchService
{
    private readonly IDispatchRepository _dispatchRepository;
    private readonly IFlightScheduleService _flightScheduleService;
    private readonly IVehicleService _vehicleService;
    private readonly IUserContextService _userContextService;

    public async Task<Dispatch> CreateFuelDispatchAsync(CreateDispatchRequest request)
    {
        var currentUserId = _userContextService.GetCurrentUserId();
        var flightSchedule = await _flightScheduleService.GetByIdAsync(request.FlightId);
        var assignedVehicle = await _vehicleService.GetAvailableVehicleAsync(request.VehicleType);

        var newDispatch = new Dispatch
        {
            FlightId = request.FlightId,
            VehicleId = assignedVehicle.VehicleId,
            RequestedQuantity = request.FuelQuantity,
            CreatedBy = currentUserId.ToString(),
            CreatedAt = DateTime.UtcNow
        };

        return await _dispatchRepository.CreateAsync(newDispatch);
    }
}
```

---

**This document is MANDATORY for all IRIS Core API development. All naming must follow these conventions before code merge.**

_Document Version: 1.0_  
_Last Updated: August 13, 2025_  
_Based on: Comprehensive review of 80+ files in Controllers, Services, and Repositories_
