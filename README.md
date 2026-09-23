# BOM Component Analysis

A browser-based **Bill of Materials (BOM) component analysis tool** for evaluating component sourcing risk, lifecycle status, compliance, availability, pricing, and alternate-part information.

The application provides a lightweight interface for maintaining a **Master BOM**, calculating a composite component risk score, and optionally synchronizing component data with **Google Sheets** through a Google Apps Script Web App endpoint.

---

## Features

### Component Management

Maintain component-level BOM information including:

* Manufacturer Part Number (MPN)
* Manufacturer
* Lifecycle status
* Package
* RoHS status
* REACH status
* Lead time
* Distributor count
* Available stock
* Quantity-based pricing
* Functional alternate
* Pin-compatible alternate
* Last verification date

Required fields:

* MPN
* Manufacturer

Components can be added or updated directly from the browser interface.

---

## Risk Analysis

The application automatically calculates component risk using four analysis categories:

| Category               | Weight |
| ---------------------- | -----: |
| Supply availability    |    35% |
| Lifecycle / compliance |    20% |
| Commercial             |    30% |
| Technical fit          |    15% |

The resulting score is converted into a **0–100 composite risk score**.

### Risk Classification

| Risk Score | Classification |
| ---------: | -------------- |
|     `< 30` | LOW            |
|    `30–59` | MEDIUM         |
|     `≥ 60` | HIGH           |

The technical-fit score is currently a placeholder value of `80` until electrical requirements are incorporated into the analysis model.

---

## Supply Analysis

Supply availability is derived from:

* Lead time
* Distributor count
* Stock level

The current implementation applies penalties and bonuses based on these values before constraining the result to a `0–100` range.

---

## Lifecycle & Compliance Analysis

Lifecycle scoring currently considers:

* `Active`
* `NRND`
* `Obsolete`
* `Unknown`

Compliance scoring considers:

* RoHS
* REACH

The lifecycle and compliance components are combined into a lifecycle/compliance score.

---

## Commercial Analysis

Commercial scoring considers:

* Distributor availability
* Stock availability
* Quantity price reduction from 1-piece to 1000-piece pricing

This provides an initial indication of the commercial attractiveness and sourcing condition of a component.

---

## Master BOM

The Master BOM table provides a consolidated view of all stored components.

Displayed fields include:

* MPN
* Manufacturer
* Lifecycle
* RoHS
* REACH
* Package
* Lead time
* Distributor count
* Stock
* 1-piece price
* 10-piece price
* 100-piece price
* 1000-piece price
* Alternate part
* Pin-compatible alternate
* Verification date
* Risk score

The table supports searching by:

* MPN
* Manufacturer
* Package

Double-clicking a component loads its data back into the editing form.

---

## Verification Tracking

Each component includes a **Last Verification Date**.

The dashboard identifies components whose verification date is more than 30 days old:

> `Verification overdue`

This is intended to highlight BOM data that may require re-verification.

---

## Google Sheets Integration

The application can operate entirely in local mode or synchronize data with Google Sheets.

The intended architecture is:

```text
┌────────────────────┐
│   Browser / HTML   │
│                    │
│ Enter / Edit MPN   │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│        API         │
│ Validate + Auth    │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│     Master BOM     │
│    Google Sheet    │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│    Device BOMs     │
│ Reference MPN      │
└─────────┬──────────┘
          │
          ▼
┌────────────────────┐
│    Risk Engine     │
│ Recalculate Risk   │
└────────────────────┘
```

The application currently exposes two API operations:

```text
upsertComponent
getMasterBOM
```

Requests are sent to the configured Google Apps Script Web App endpoint using `POST`.

---

## Device BOM Strategy

The UI provides two synchronization modes:

1. **Reference Master BOM by MPN**
2. **Copy snapshot into device sheet**

This allows the Master BOM to act as the central component reference while individual device BOMs can either reference or copy component information.

---

## Security

No Google service-account credentials or private keys are stored in the browser application.

The frontend expects an external authenticated backend/API endpoint to handle synchronization with Google Sheets.

> **Important:** Never place Google service-account private keys, credentials, API secrets, or other sensitive authentication material directly inside this HTML file.

---

## Running Locally

This project is a standalone HTML application.

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<repository-name>.git
cd <repository-name>
```

### 2. Open the application

Open:

```text
BOM_Component_Analysis.html
```

in a modern web browser.

No build system or package installation is required for the current frontend implementation.

---

## Local Mode

The application can be used without configuring a Google Sheets endpoint.

In local mode:

* Components are maintained in the browser session.
* Risk calculations run locally.
* The Master BOM table is updated locally.
* Google Sheets synchronization remains disabled until an endpoint is configured.

The application starts with demonstration component records so that the analysis interface can be tested immediately.

---

## Google Apps Script Configuration

To enable synchronization:

1. Deploy a Google Apps Script Web App.
2. Configure its endpoint in the **Google Sheets synchronization** section.
3. Set the Master BOM sheet name.
4. Use the synchronization controls to push or pull BOM data.

Configure the endpoint in:

```text
Google Apps Script Web App endpoint
```

Example format:

```text
https://script.google.com/macros/s/YOUR_DEPLOYMENT_ID/exec
```

The frontend sends requests in the following general structure:

```json
{
  "action": "upsertComponent",
  "masterSheet": "Master BOM",
  "payload": {
    "mpn": "TPS62160DQCR",
    "manufacturer": "Texas Instruments"
  }
}
```

For retrieval:

```json
{
  "action": "getMasterBOM",
  "masterSheet": "Master BOM",
  "payload": {}
}
```

The backend is expected to return JSON responses. The pull operation expects the response to contain a `data` array.

---

## Risk Engine

The current risk engine is implemented entirely in JavaScript.

Conceptually:

```text
Technical Fit
      │
      ├── 15%
      │
Supply Availability
      │
      ├── 35%
      │
Lifecycle / Compliance
      │
      ├── 20%
      │
Commercial
      │
      └── 30%
             │
             ▼
      Composite Risk
        0 – 100
```

The current composite calculation is based on:

```text
Risk =
100 -
(
    Technical Fit × 0.15
  + Supply × 0.35
  + Lifecycle/Compliance × 0.20
  + Commercial × 0.30
)
```

The resulting value is constrained to the `0–100` range.

---

## Example Data

The current demo dataset contains example components such as:

```text
TPS62160DQCR
Texas Instruments
Active
VSON-10
```

```text
MP1584EN
Monolithic Power Systems
Active
SOIC-8
```

```text
ABC-OLD-1
Example Semiconductor
NRND
QFN
```

These records are intended for demonstrating the interface and risk-analysis workflow.

---

## User Interface

The application is organized into the following sections:

### Dashboard Metrics

* Master BOM components
* High-risk components
* Average risk
* Verification overdue

### Component Data

Form for creating and updating component records.

### Risk Analysis

Displays:

* Technical fit
* Supply availability
* Lifecycle / compliance
* Commercial score
* Composite risk

### Data Flow

Shows the intended relationship between the frontend, API, Master BOM, Device BOMs, and risk engine.

### Master BOM

Searchable component table containing the current Master BOM dataset.

### Google Sheets Synchronization

Configuration and controls for remote synchronization.

### Activity Log

Displays application events such as:

* Component updates
* Risk recalculations
* Validation errors
* Synchronization status
* Pull operations

---

## Current Limitations

The current implementation is intentionally lightweight and has several areas that can be expanded:

* Technical-fit scoring is currently fixed at `80`.
* Electrical specifications are not yet part of the risk model.
* The frontend primarily provides the browser-side application layer.
* Google Sheets synchronization requires a separately deployed backend endpoint.
* Authentication and authorization logic are not implemented inside the HTML frontend.
* Component data quality depends on the accuracy and freshness of supplier information entered into the system.

---

## Intended Use

This tool is designed as a foundation for electronics BOM engineering workflows where component decisions need to account for more than electrical functionality alone.

Potential use cases include:

* New product BOM review
* Component sourcing assessment
* Alternate-part evaluation
* Obsolescence monitoring
* Procurement risk review
* Cost-down analysis
* BOM verification
* Centralized component databases

---

## Roadmap

Potential future extensions include:

* Electrical specification-based technical-fit scoring
* Datasheet-driven component verification
* Automated distributor/stock-price data collection
* Manufacturer lifecycle-status verification
* Automated alternate-part discovery
* Pin-compatible replacement analysis
* Multi-BOM dependency analysis
* Historical component pricing
* Supply-risk trend tracking
* Automated BOM import/export
* Revision history
* User authentication and role-based access
* Backend database integration
* Automated component verification workflows

---

## Project Structure

The current implementation is intentionally compact:

```text
.
├── BOM_Component_Analysis.html
└── README.md
```

The frontend currently contains the HTML structure, CSS, user interface logic, BOM data handling, risk engine, search functionality, and API integration logic in a single file.

---

## License

No license is currently specified for this project.

Add an appropriate license before distributing or publishing the repository for external use.

---

## Status

**Development / Prototype**

The current implementation provides the core BOM component management and risk-analysis workflow, with Google Sheets synchronization exposed as an optional integration layer.
