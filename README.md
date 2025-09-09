# BAFS IRIS - Coding Conventions

This repository contains comprehensive coding conventions, architecture guidelines, and library templates for the BAFS IRIS project. All conventions documented here are designed to ensure consistency, maintainability, and best practices across the IRIS ecosystem.

## Table of Contents

### Architecture & Design

- **[System Architecture](./Architecture.md)** - Application layer structure, dispatch service architecture, and overall system design patterns

### Coding Standards

- **[Naming Conventions](./NamingConvention.md)** - **MANDATORY** naming standards for the BAFS IRIS Core API project, including PascalCase/camelCase guidelines, file naming, and code organization rules

### Library Templates & Integration Guides

#### Database & Validation

- **[EF Core](./EFCore.md)** - Entity Framework Core integration guide with NuGet packages, ORM patterns, and database mapping best practices
- **[FluentValidation](./FluentValidator.md)** - FluentValidation library template for building strongly-typed validation rules and ASP.NET Core integration

#### API Integration

- **[Refit with IRIS Core](./RefitWithIRISCore.md)** - Refit integration template for consuming IRIS Core APIs in a strongly-typed and testable way
- **[Refit with Keycloak](./RefitWithKeycloak.md)** - Refit integration template for consuming Keycloak Admin APIs with proper authentication and error handling

## Getting Started

1. **Start with [Naming Conventions](./NamingConvention.md)** - These are mandatory standards that must be followed in all new code
2. **Review [System Architecture](./Architecture.md)** - Understand the overall system structure and layer organization
3. **Choose relevant library templates** based on your project needs:
   - Data access: [EF Core](./EFCore.md)
   - Validation: [FluentValidation](./FluentValidator.md)
   - API integration: [Refit with IRIS Core](./RefitWithIRISCore.md) or [Refit with Keycloak](./RefitWithKeycloak.md)

## Compliance Requirements

- **Naming Conventions**: MANDATORY for all new code
- **Architecture Patterns**: Follow the established layer structure
- **Library Templates**: Use provided patterns for consistency across projects

## Contributing

When adding new conventions or updating existing ones, ensure they align with the established patterns and include practical examples for implementation.
