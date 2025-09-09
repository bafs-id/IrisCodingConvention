# System Architecture

##### App Layer - Dispatch Service

The dispatch service layer manages dispatching operations:

**Components:**

- `DispatchModel` - Dispatch data model
- `IDispatchService` - Service interface
- `DispatchService` - Core dispatch service implementation

**Example Functions:**

- `CreateDispatch(CreateDispatchRequest request)`
- `UpdateDispatch(UpdateDispatchRequest request)`

## Layer Structure

<img width="3230" height="918" alt="arch" src="https://github.com/user-attachments/assets/bb8bb58a-e779-4fc6-8bf3-3c6f697469d0" />

### App Layer - Mobile Service

The mobile service layer contains the core application logic:

**Components:**

- `ServiceDetailsModel` - Handles service details data and operations
- `IMobileService` - Core mobile service interface (implemented by Refit)

**Example Functions:**

- `CreateServiceDetails(CreateServiceDetailsRequest request)`
- `UpdateServiceDetails(UpdateServiceDetailsRequest request)`

### App Layer - Dispatch Service

The dispatch service layer manages dispatching operations:

**Components:**

- `DispatchModel` - Dispatch data model
- `IDispatchService` - Service interface
- `DispatchService` - Core dispatch service implementation

**Example Functions:**

- `CreateDispatch(CreateDispatchRequest request)`
- `UpdateDispatch(UpdateDispatchService request)`

### Domain Layer - Dispatch Repository

The repository layer handles data persistence and retrieval:

**Components:**

- `DispatchEntity` - Entity model for dispatch data
- `IDispatchRepository` - Repository interface
- `DispatchRepository` - Data access implementation

**Entity Operations:**

- `CreateDispatch(DispatchEntity entity)`
- `UpdateDispatch(DispatchEntity entity)`

## Communication Flow

1. **Presentation → App Layer**: The Dispatch/Controller sends requests to the Dispatch Service
2. **App Layer → Domain Layer**: The Dispatch Service communicates with the Dispatch Repository for data operations
3. **App Layer → App Layer**: The Dispatch Service can also call the Mobile Service for mobile-specific operations
4. **Service Integration**: The Mobile Service operates both independently and as a dependency for the Dispatch Service
