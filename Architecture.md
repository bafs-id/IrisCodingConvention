# System Architecture

## Overview

This document describes the three-layer architecture of the system, which consists of:
- **Presentation Layer** - Dispatch/Controller
- **App Layer** - Dispatch Service  
- **Domain Layer** - Dispatch Repository

## Layer Structure

### App Layer - Mobile Service

The mobile service layer contains the core application logic:

**Components:**
- `ServiceOperatorModel` - Handles service operator data and operations
- `MobileService` - Core mobile service implementation (implemented by Kotlin)
- `MobileService.v2.0 REST API` - REST API interface for mobile services

**Request Flow:**
- `CreateServiceDetails(CreateServiceDetailsRequest request)`
- `UpdateServiceDetails(UpdateServiceDetailsRequest request)`

### App Layer - Dispatch Service

The dispatch service layer manages dispatching operations:

**Components:**
- `DispatchModel` - Dispatch data model
- `DispatchService` - Core dispatch service implementation (implemented)
- `DispatchService` - Service interface

**Request Flow:**
- `CreateDispatch(CreateDispatchRequest request)` - Returns status
- `UpdateDispatch(UpdateDispatchService request)` - Returns status

### Domain Layer - Dispatch Repository

The repository layer handles data persistence and retrieval:

**Components:**
- `DispatchEntity` - Entity model for dispatch data
- `DispatchRepository` - Repository interface (implemented)
- `DispatchRepository` - Data access implementation

**Entity Operations:**
- `CreateDispatch(DispatchEntity entity)` - Returns DBContext
- `UpdateDispatch(DispatchEntity entity)` - Returns DBContext

## Communication Flow

1. **Presentation → App Layer**: The Dispatch/Controller sends requests to the Dispatch Service
2. **App Layer → Domain Layer**: The Dispatch Service communicates with the Dispatch Repository for data operations
3. **Mobile Service Integration**: The Mobile Service operates in parallel, handling mobile-specific operations through its REST API

## Key Design Principles

- **Separation of Concerns**: Each layer has distinct responsibilities
- **Dependency Direction**: Dependencies flow downward (Presentation → App → Domain)
- **Interface-based Design**: Services and repositories are defined through interfaces
- **Request/Response Pattern**: Operations use typed request objects and