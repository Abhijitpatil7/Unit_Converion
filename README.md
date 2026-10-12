# Unit Conversion API

A small ASP.NET Core Web API that converts values between units of length, mass and temperature.

## Solution structure

```
src/
  UnitConversion.Api          Presentation layer: controller, Swagger, startup (DI wiring)
  UnitConversion.Business     Business layer: conversion logic (ConversionService), result model
  UnitConversion.Repository   Data layer: unit data (UnitRepository), Unit model
```

Dependencies: Api -> Business -> Repository (Api also references Repository only to register it in DI).
Flow: Controller -> Service -> Repository.

## Run locally

Requires the [.NET SDK](https://dotnet.microsoft.com/download) (10.0 or later).

```bash
dotnet restore
dotnet run --project src/UnitConversion.Api
```

Swagger UI: http://localhost:5080/swagger

Example request:

```
GET http://localhost:5080/api/convert?category=length&from=meter&to=foot&value=10
```

```json
{ "category": "Length", "from": "meter", "to": "foot", "value": 10, "result": 32.808399 }
```

Units can be given by name or symbol, case-insensitive (`meter`/`m`, `celsius`/`C`, `pound`/`lb`).
Invalid input returns `400` with an error message.

## Design decisions

- **Three layers with one job each.** API handles HTTP, Business holds the conversion rules, Repository owns the data.
- **Convert via a base unit.** Each unit stores a factor (and offset) relative to a base unit per category
  (`base = value * Factor + Offset`). Adding a unit is one line in `UnitRepository.cs`, and temperature
  fits the same formula.
- **Repository interface.** `IUnitRepository` hides where units come from. For hundreds of units, replace the
  hard-coded list with a database-backed implementation; Business and Api stay unchanged.
- **`decimal`** avoids floating-point noise; results are rounded to 6 decimals.
- **Kept small on purpose:** one endpoint, no auth, no database.
