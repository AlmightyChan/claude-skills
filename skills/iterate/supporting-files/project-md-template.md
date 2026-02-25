# PROJECT.md Template

This template defines the 4-section format for PROJECT.md, the human-written project manifest that the iterate skill reads on session load to orient itself.

**Ownership:** PROJECT.md is human-written and human-maintained. The iterate skill reads it but does not modify it without explicit user permission. If no PROJECT.md exists, the skill offers to generate a draft for user review.

---

## Template

```markdown
# {Project Name}

**Last updated:** {YYYY-MM-DD}

## Overview

{One paragraph describing the project's purpose, target audience, and core value proposition.}

**Tech Stack:** {Primary language/framework, key dependencies, platform targets}

## Feature Map

| Feature | Location | Status |
|---------|----------|--------|
| {Feature name} | {Directory or file path} | {Implemented / Stubbed / Planned} |

## Key Files

- `{path/to/file}` -- {Brief description of what this file does and why it matters}

## Current State

{Paragraph describing where the project is: development phase (early, mid, late), what works, what is stubbed or incomplete, known blockers, and next milestones. Include the last significant milestone reached.}
```

---

## Example

```markdown
# WeatherDash

**Last updated:** 2026-02-15

## Overview

WeatherDash is a macOS menu bar app that displays current weather conditions and 5-day forecasts for saved locations. It targets weather-conscious commuters who want quick, glanceable weather data without opening a browser.

**Tech Stack:** Swift/SwiftUI, macOS 14+, WeatherKit API, SwiftData for persistence

## Feature Map

| Feature | Location | Status |
|---------|----------|--------|
| Menu bar popover | Sources/App/MenuBarView.swift | Implemented |
| Current conditions | Sources/Views/CurrentWeatherView.swift | Implemented |
| 5-day forecast | Sources/Views/ForecastView.swift | Implemented |
| Location search | Sources/Views/LocationSearchView.swift | Stubbed |
| Saved locations | Sources/Models/SavedLocation.swift | Implemented |
| Settings | Sources/Views/SettingsView.swift | Planned |
| Widget extension | WeatherDashWidget/ | Planned |

## Key Files

- `Sources/App/WeatherDashApp.swift` -- App entry point, menu bar configuration, scene setup
- `Sources/Services/WeatherService.swift` -- WeatherKit API client, handles all weather data fetching
- `Sources/Models/SavedLocation.swift` -- SwiftData model for persisted locations
- `Sources/ViewModels/WeatherViewModel.swift` -- Main view model coordinating data flow between service and views

## Current State

WeatherDash is in mid-development. Current conditions and 5-day forecast views are fully implemented and displaying real WeatherKit data. Location search UI exists but is stubbed -- it shows a text field but search results are hardcoded. Settings view has not been started. The widget extension directory exists but contains only the default template. Last milestone: WeatherKit integration completed (2026-02-15). Next milestone: location search with MapKit integration.
```

---

## Field Notes

- **Overview** should be 2-4 sentences. Avoid listing features here -- that is what the Feature Map is for.
- **Tech Stack** is a single line. Include the platform target (e.g., iOS 17+, macOS 14+) and the primary persistence mechanism.
- **Feature Map** status values are: `Implemented` (working), `Stubbed` (UI exists but logic is placeholder), `Planned` (not yet started). Use the Location column to point to the primary file or directory.
- **Key Files** should list 4-8 files. Focus on files a new contributor would need to understand the codebase. Do not list every file.
- **Last updated** is a date field at the top of the file. Update it whenever the manifest is revised. The iterate skill uses this to detect staleness — if the date is significantly older than recent git activity, the skill may suggest a refresh.
- **Current State** should be updated by the developer at natural checkpoints. The iterate skill may flag drift if the manifest contradicts what it observes in the codebase.
