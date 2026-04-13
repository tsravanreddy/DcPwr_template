# Driver-Stub

This directory contains the skeleton driver implementation for DC Power instruments.

## Purpose

This stub provides:
- Basic driver class structure
- Method signatures for common DC Power operations
- Placeholder implementation points

## When Creating a Concrete Driver

1. Rename this directory to match your driver (e.g., `N6705C-Driver`)
2. Update namespace in `.csproj` and `.cs` files
3. Uncomment and configure the InstrumentFoundation NuGet package reference
4. Implement actual SCPI commands and instrument logic
5. Add instrument-specific methods and properties

## Template Characteristics

- ✅ Compiles successfully (no implementation)
- ✅ Demonstrates expected structure
- ✅ Provides code organization guidance
- ❌ Does NOT connect to actual instruments
- ❌ Does NOT deploy to NuGet (template mode)
