# Aircraft Design Program - Product Requirements Document

## 1. Introduction

### 1.1 Purpose
To create a streamlined aircraft design management system that allows engineers to efficiently create, track, and iterate on aircraft designs while providing essential testing capabilities and component management to support the full aircraft design lifecycle.

### 1.2 Scope
This system will provide comprehensive functionality for aircraft design management including design creation and versioning, component tracking with material specifications, and critical testing capabilities to validate design performance and safety requirements.

## 2. Global API Specifications

### 2.1 Core Requirements:
- HTTP only (no HTTPS)
- No authentication required
- No pagination
- No concurrency handling
- No rate-limiting
- No performance requirements
- Resource IDs should be UUIDs
- When a resource ID is invalid or not found, always return 404 (no validation of UUID format)
- Resources: designs, tests, components
- Complex aspects: Manage design creation, versioning, and approval workflows. Track component specifications and material details. Conduct testing to validate design performance and safety.

### 2.2 Base URL Configuration
- **Exact Base URL**: `/api/v1`
- **Versioning Strategy**: Explicit version in URL to support future API evolution
- **Global URL Pattern**: 
  - Parent Resources: `/{resource-type}`
  - Child/Nested Resources: `/{parent-resource}/{parent-id}/{child-resource}`
  - Special Operations: `/{resource-type}/{operation}`

### 2.3 Timestamp Management
- **Timestamp Type**: Server-managed
- **Timestamp Fields**:
  - `created_at`: Immutable creation timestamp
  - `updated_at`: Dynamically updated on any resource modification
  - `deleted_at`: Dynamically updated on any resource deletion
- **Format**: ISO 8601 Extended Format (e.g., "2024-02-03T15:30:45.123Z")
- **Time Zone**: Always stored in UTC
- **Precision**: Millisecond-level accuracy

### 2.4 Error Handling

#### 2.4.1 Error Response Schema when we have multiple errors for a particular status code
```json
{
  "error_id": "ERROR_CODE",
  "errors":[
    {
      "id": "DOMAIN_SPECIFIC_ERROR_CODE",
      "field": "FIELD_NAME",
      "message": "ERROR_MESSAGE"
    }
  ]
}
```
#### 2.4.2 Error Response Schema when we have a single error for a particular status code
```json
{
  "error_id": "ERROR_CODE",
  "message": "ERROR_MESSAGE"
}
```

#### 2.4.3 Error examples

- `400 Bad Request`:

```json
{
    "error_id": "VALIDATION_ERROR",
    "errors": [
        {
            "id": "invalid_field_name_1",
            "field": "field_name_1",
            "message": "Descriptive error message explaining the validation issue."
        },
        {
            "id": "invalid_field_name_2",
            "field": "field_name_2",
            "message": "Descriptive error message explaining the validation issue."
        }
    ]
}
```

- `404 Not Found`:

```json
{
    "error_id": "RESOURCE_NOT_FOUND",
    "message": "Descriptive message about the missing resource."
}
```

- `409 Conflict`:

```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Descriptive message explaining the conflict."
}
```

**Error Handling Principles**:
- Always return appropriate HTTP status codes
- Include unique error identifiers for log tracking
- Maintain consistency across all endpoints

### 2.5 Field Validation Principles
- **String Fields**:
  - Length constraints explicitly defined
  - Trimmed of leading/trailing whitespaces
  - Cannot be empty unless explicitly allowed

- **Numeric Fields**:
  - Precise range constraints
  - Non-negative unless explicitly specified

- **Enum Fields**:
  - Strict matching against predefined values
  - Case-sensitive matching
  - No default values unless specified

## 3. Comprehensive Resource Models

### 3.1 Design Model
```json
{
  "design_id": "uuid",
  "name": "string",
  "status": "DRAFT|APPROVED|REJECTED",
  "version": "string",
  "description": "string",
  "specifications": {
    "wingspan": "decimal",
    "length": "decimal",
    "weight": "decimal",
    "engine_count": "integer",
    "unit_of_measure_weight": "string",
    "unit_of_measure_dimensions": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### Field Specifications
| Field | Type | Required | Constraints | Mutability | Default | Description |
|-------|------|----------|-------------|------------|---------|-------------|
| `design_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the aircraft design that remains consistent throughout its lifecycle, used in all API operations and cross-references from components and tests |
| `name` | string | Yes | Length: 1-100 | Mutable | None | Official name of the aircraft design that will appear in technical documentation, engineering reports, and project planning materials |
| `status` | enum | Yes | [`DRAFT`, `APPROVED`, `REJECTED`] | Mutable | `DRAFT` | Current state of the design in the workflow that determines its visibility, modifiability, and readiness for production consideration |
| `version` | string | Yes | Length: 1-50 | Mutable | None | Version identifier that tracks design iterations and evolution, critical for maintaining design history and comparing changes between iterations |
| `description` | string | No | Length: 1-2000 | Mutable | None | Detailed explanation of the aircraft's purpose, key features, and design philosophy, used for contextual understanding and documentation purposes |
| `specifications.wingspan` | decimal | Yes | Min: 0 | Mutable | None | Distance from wingtip to wingtip in meters, a critical aerodynamic parameter that affects lift, stability, and airport compatibility |
| `specifications.length` | decimal | Yes | Min: 0 | Mutable | None | Total length of the aircraft from nose to tail in meters, important for determining storage requirements and runway compatibility |
| `specifications.weight` | decimal | Yes | Min: 0 | Mutable | None | Empty weight of the aircraft in kilograms, a fundamental parameter affecting performance, fuel consumption, and payload capacity |
| `specifications.engine_count` | integer | Yes | Range: 0-10 | Mutable | 1 | Number of engines on the aircraft, determines thrust capacity, redundancy, and maintenance requirements |
| `specifications.unit_of_measure_weight` | string | Yes | Length: 1-500 | Mutable | None | Unit of measure for the weight, used for storage, transportation, and assembly planning |
| `specifications.unit_of_measure_dimensions` | string | Yes | Length: 1-500 | Mutable | None | Unit of measure for the dimensions, used for storage, transportation, and assembly planning |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the design record was first created, used for audit trails and chronological ordering in design evolution |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any design field, used for tracking changes and design history |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | Null | When populated, indicates the design has been soft-deleted and should be excluded from active development while maintaining historical record |

### 3.2 Component Model
```json
{
  "component_id": "uuid",
  "design_id": "uuid",
  "name": "string",
  "component_type": "STRUCTURAL|ELECTRICAL|HYDRAULIC|AVIONICS|OTHER",
  "specifications": {
    "material": "string",
    "weight": "decimal",
    "unit_of_measure_weight": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### Field Specifications
| Field | Type | Required | Constraints | Mutability | Default | Description |
|-------|------|----------|-------------|------------|---------|-------------|
| `component_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the component that remains consistent throughout its lifecycle, used in all API operations and cross-references |
| `design_id` | uuid | Yes | Must reference existing design | Never | None | Reference to the parent aircraft design this component belongs to, establishing the hierarchical relationship between designs and their components. Relationship: Design |
| `name` | string | Yes | Length: 1-100 | Mutable | None | Descriptive name of the component used in technical documentation, parts catalogs, and assembly instructions |
| `component_type` | enum | Yes | [`STRUCTURAL`, `ELECTRICAL`, `HYDRAULIC`, `AVIONICS`, `OTHER`] | Mutable | `STRUCTURAL` | Classification of the component that determines its function, integration requirements, and relevant engineering standards |
| `specifications.material` | string | Yes | Length: 1-500 | Mutable | None | Primary material composition of the component, critical for weight calculations, stress analysis, and manufacturing planning |
| `specifications.weight` | decimal | Yes | Min: 0 | Mutable | None | Weight of component in kilograms, important for overall aircraft weight calculations, balance determination, and performance analysis |
| `specifications.unit_of_measure_weight` | string | Yes | Length: 1-500 | Mutable | None | Unit of measure for the weight, used for storage, transportation, and assembly planning |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the component record was first created, used for audit trails and component development tracking |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any component field, used for tracking component revisions and change history |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | Null | When populated, indicates the component has been soft-deleted and should be excluded from active development while maintaining historical record |


### 3.3 Test Model
```json
{
  "test_id": "uuid",
  "design_id": "uuid",
  "status": "PLANNED|COMPLETED",
  "test_type": "AERODYNAMIC|STRUCTURAL|SYSTEMS|OTHER",
  "results": {
    "outcome": "PASS|FAIL",
    "notes": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### Field Specifications
| Field | Type | Required | Constraints | Mutability | Default | Description |
|-------|------|----------|-------------|------------|---------|-------------|
| `test_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the test that remains consistent throughout its lifecycle, used in all API operations and cross-references.|
| `design_id` | uuid | Yes | Must reference existing design | Never | None | Reference to the aircraft design being tested, establishing traceability between test results and specific design versions. Relationship: Design |
| `status` | enum | Yes | [`PLANNED`, `COMPLETED`] | Mutable | `PLANNED` | Current state of the test in the workflow that determines whether it is scheduled, successfully finished, or terminated due to issues |
| `test_type` | enum | Yes | [`AERODYNAMIC`, `STRUCTURAL`, `SYSTEMS`, `OTHER`] | Mutable | `AERODYNAMIC` | Classification of the test methodology and focus area, determining required equipment, expertise, and validation criteria |
| `results.outcome` | enum | No | [`PASS`, `FAIL`] | Mutable | None | Final determination of test success or failure, critical for design approval decisions |
| `results.notes` | string | No | Length: 1-2000 | Mutable | None | Detailed observations, measurements, and analysis from the test execution, providing context and explanation for the outcome |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the test record was first created, used for audit trails and test scheduling chronology |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any test field, used for tracking test evolution and result updates |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | Null | When populated, indicates the test has been soft-deleted and should be excluded from active consideration while maintaining historical record |


#### Field Specifications

| Field | Type | Required | Constraints | Mutability | Default | Relationships |
|-------|------|----------|-------------|------------|---------|--------------|
| `component_id` | uuid | Yes | Immutable unique identifier | Never | None | None |
| `design_id` | uuid | Yes | Must reference existing design | Never | None | None |
| `name` | string | Yes | Length: 1-100 | Mutable | None | None |
| `component_type` | enum | Yes | [STRUCTURAL, ELECTRICAL, HYDRAULIC, AVIONICS, OTHER] | Mutable | STRUCTURAL | None |
| `specifications.material` | string | Yes | Length: 1-500 | Mutable | None | None |
| `specifications.weight` | decimal | Yes | Min: 0 | Mutable | None | None |
| `specifications.dimensions` | string | Yes | Length: 1-500 | Mutable | None | None |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | None |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | None |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | Null | None |


## 4. Complex Business Rules

### 4.1 Conditional Field Requirements

#### Design Model
- `version` must be incremented when any specification is changed
- `description` becomes required when transitioning from "DRAFT" to "APPROVED"
- `specifications.wingspan`, `specifications.length` must be provided in same unit of measure as `specifications.unit_of_measure_dimensions`.

#### Component Model
- Components with duplicate `name` values within the same `design_id` are allowed only when in different `component_type` categories

#### Test Model
- `results.outcome` required only when `status` is "COMPLETED".
- `test_type` cannot be modified once the test `status` is "COMPLETED".

### 4.2 State Machine Transitions

#### 4.2.1 Design Status Transitions
- From "DRAFT" to "APPROVED"
- From "DRAFT" to "REJECTED"
- From "REJECTED" to "DRAFT"
- Once "APPROVED", the status cannot be changed

#### 4.2.2 Test Status Transitions
- From "PLANNED" to "COMPLETED"
- From "COMPLETED" to "PLANNED" (When test fails, it must be retested)

### 4.3 Cross-Resource Validation Rules

#### 4.3.1 Design-Component Validation
- Components can only reference designs that are not soft-deleted (no `metadata.deleted_at` value)
- When a design's `status` changes to "APPROVED", any modification to its associated components is prohibited
- Sum of all component weights must not exceed the design's `specifications.weight`, keeping in mind that the unit of measure for the weight may be different from the unit of measure for the dimensions.

#### 4.3.2 Design-Test Validation
- Tests can only reference designs that are not soft-deleted (no `metadata.deleted_at` value)
- When a design's `status` changes to "APPROVED", all its tests with `status` "PLANNED" must first be updated to "COMPLETED" and the `results.outcome` must be "PASS".

### 4.4 Multi-Step Operations

#### 4.4.1 Design Approval Process
1. **Initial Creation**
   - Create design record with `status` "DRAFT"
   - Define all required attributes including `name`, `version`, and specifications

2. **Component Definition**
   - Add all necessary components to the design
   - Ensure all components have proper material specifications
   - Verify total component weight is less than or equal to the design's `specifications.weight`

3. **Testing Execution**
   - Create required tests based on design characteristics
   - Complete all tests with documented results
   - Verify all tests have "COMPLETED" status.

4. **Final Design Approval**
   - Ensure description is provided
   - Verify all COMPLETED tests have `results.outcome` "PASS"
   - Change design `status` to "APPROVED"

**Rollback Scenarios:**
- If any test fails with critical issues, design must remain in "DRAFT" status until issues are resolved and status of the test is changed to "PLANNED" to be retested.
- If component weight is found to be inconsistent, design must remain in "DRAFT" status until issues are resolved.

### 4.5 Deletion Behavior

#### 4.5.1 Design Model Deletion Rules
- Design deletion MUST be prevented if:
  - Design has `status` "APPROVED" and has referenced components.

- When a design is marked for deletion (`metadata.deleted_at` populated):
  - It should no longer appear in standard design listings
  - It remains in the database for historical reference and cannot be modified.
  - If the Design is deleted when in "DRAFT" status, components should de-reference the design.

#### 4.5.2 Component Model Deletion Rules
- Component deletion MUST be prevented if:
  - It is referenced by an "APPROVED" design.

- When a component is marked for deletion (`metadata.deleted_at` populated):
  - Component remains in database for historical reference
  - Design weight calculation should exclude deleted component.
  - Component is excluded from standard component listings.

#### 4.5.3 Test Model Deletion Rules
- Test deletion MUST be prevented if:
  - Test has `status` "COMPLETED" with `results.outcome` "PASS" and is required for design approval.

- When a test is marked for deletion (`metadata.deleted_at` populated):
  - Test results remain available for historical reference
  - Test is excluded from standard test listings

### 4.6 Deletion Audit Requirements

**Audit Trail:**

- All deletion attempts (successful or prevented) must be logged with:
  - Timestamp of deletion attempt (ISO 8601 format)
  - User or system identifier that initiated the deletion
  - Resource ID and type

**Retention Period:**

- Deletion audit logs must be retained for a minimum of 7 years
- Deletion of critical infrastructure components must be retained for 15 years
- Audit logs cannot be deleted or modified once created

**Regulatory Compliance:**

- Deletion of certain resources may require regulatory notification
- Critical infrastructure deletions may require additional approval workflow
- Mass deletions (more than 10 resources of the same type within 24 hours) require higher level authorization

## 5. Comprehensive API Endpoints

### 5.1 Design Endpoints

#### 5.1.1 Create Design
**URL:** POST /api/v1/designs

**Request Body Schema:**
```json
{
  "name": "string", // Required - Official name of the aircraft design
  "description": "string", // Optional - Detailed explanation of the aircraft
  "status": "DRAFT", // Required - Initial status of the design
  "specifications": { // Required object
    "wingspan": "decimal", // Required - Distance from wingtip to wingtip
    "length": "decimal", // Required - Total length of the aircraft
    "weight": "decimal", // Required - Empty weight of the aircraft
    "engine_count": "integer", // Required - Number of engines
    "unit_of_measure_weight": "string", // Required - Unit of measure for weight
    "unit_of_measure_dimensions": "string" // Required - Unit of measure for dimensions
  }
}
```

**Success Response (201 Created):**
```json
{
  "design_id": "uuid",
  "name": "string",
  "status": "DRAFT",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_NAME",
      "field": "name",
      "message": "Name must be between 1 and 100 characters."
    },
    {
      "id": "MISSING_WINGSPAN",
      "field": "specifications.wingspan",
      "message": "Wingspan is required for design creation along with unit of measure dimensions."
    },
    {
      "id": "MISSING_LENGTH",
      "field": "specifications.length",
      "message": "Length is required for design creation along with unit of measure dimensions."
    },
    {
      "id": "MISSING_ENGINE_COUNT",
      "field": "specifications.engine_count",
      "message": "Engine count is required for design creation."
    }
  ]
}
```
- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "`specifications.weight` is provided, but `specifications.unit_of_measure_weight` is not provided."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.1.2 Get Design
**URL:** GET /api/v1/designs/{designId}

**Success Response (200 OK):**
```json
{
  "design_id": "uuid",
  "name": "string",
  "status": "DRAFT|APPROVED|REJECTED",
  "version": "string",
  "description": "string",
  "specifications": {
    "wingspan": "decimal",
    "length": "decimal",
    "weight": "decimal",
    "engine_count": "integer",
    "unit_of_measure_weight": "string",
    "unit_of_measure_dimensions": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Design with ID {designId} not found."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.1.3 Update Design
**URL:** PATCH /api/v1/designs/{designId}

**Request Body Schema:**
```json
{
  "name": "string", // Optional - Updated name of the aircraft design
  "description": "string", // Optional - Updated description
  "status": "DRAFT|APPROVED|REJECTED", // Optional - Updated status
  "specifications": { // Optional object
    "wingspan": "decimal", // Optional - Updated wingspan
    "length": "decimal", // Optional - Updated length
    "weight": "decimal", // Optional - Updated weight
    "engine_count": "integer", // Optional - Updated engine count
    "unit_of_measure_weight": "string", // Optional - Updated unit of measure for weight
    "unit_of_measure_dimensions": "string" // Optional - Updated unit of measure for dimensions
  }
}
```

**Success Response (200 OK):**
```json
{
  "design_id": "uuid",
  "name": "string",
  "status": "DRAFT|APPROVED|REJECTED",
  "version": "string",
  "description": "string",
  "specifications": {
    "wingspan": "decimal",
    "length": "decimal",
    "weight": "decimal",
    "engine_count": "integer",
    "unit_of_measure_weight": "string",
    "unit_of_measure_dimensions": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_STATUS_TRANSITION",
      "field": "status",
      "message": "Cannot transition from APPROVED to DRAFT."
    },
    {
      "id": "MISSING_DESCRIPTION",
      "field": "description",
      "message": "Description is required when status is APPROVED."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Design with ID {designId} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot approve design with tests still in PLANNED status."
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "`specifications.weight` is provided, but `specifications.unit_of_measure_weight` is not provided."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.1.4 Delete Design
**URL:** DELETE /api/v1/designs/{designId}

**Success Response (204 No Content)**

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Design with ID {designId} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot delete design with APPROVED status that has components."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.1.5 List Designs
**URL:** GET /api/v1/designs

**Query Parameters:**
- status (optional): Filter by status (DRAFT, APPROVED, REJECTED)
- include_deleted (optional): Include soft-deleted designs

**Success Response (200 OK):**
```json
{
  "designs": [
    {
      "design_id": "uuid",
      "name": "string",
      "status": "DRAFT|APPROVED|REJECTED",
      "version": "string",
      "metadata": {
        "created_at": "timestamp",
        "updated_at": "timestamp",
        "deleted_at": null
      }
    }
  ],
  "count": "integer"
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Invalid query parameter value."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

### 5.2 Component Endpoints

#### 5.2.1 Create Component
**URL:** POST /api/v1/components

**Request Body Schema:**
```json
{
  "design_id": "uuid", // Required - Reference to parent design
  "name": "string", // Required - Descriptive name of the component
  "component_type": "STRUCTURAL|ELECTRICAL|HYDRAULIC|AVIONICS|OTHER", // Required - Classification of the component
  "specifications": { // Required object
    "material": "string", // Required - Primary material composition
    "weight": "decimal", // Required - Weight of component
    "unit_of_measure_weight": "string" // Required - Unit of measure for weight
  }
}
```

**Success Response (201 Created):**
```json
{
  "component_id": "uuid",
  "name": "string",
  "component_type": "STRUCTURAL|ELECTRICAL|HYDRAULIC|AVIONICS|OTHER",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_DESIGN_ID",
      "field": "design_id",
      "message": "Design ID is required."
    },
    {
      "id": "INVALID_COMPONENT_TYPE",
      "field": "component_type",
      "message": "Component type must be one of: STRUCTURAL, ELECTRICAL, HYDRAULIC, AVIONICS, OTHER"
    },
    {
      "id": "INVALID_WEIGHT",
      "field": "specifications.weight",
      "message": "Weight must be greater than 0."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Referenced design with ID {design_id} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "DUPLICATE_COMPONENT_NAME",
      "field": "name",
      "message": "Duplicate component name detected within the same design and component type. Each component name must be unique within its component type for a given design. Duplicate names are only allowed if the components belong to different component type categories."
    },
    {
      "id": "REJECTED_DESIGN",
      "field": "design_id",
      "message": "Cannot create component for a design with REJECTED status."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.2.2 Get Component
**URL:** GET /api/v1/components/{componentId}

**Success Response (200 OK):**
```json
{
  "component_id": "uuid",
  "design_id": "uuid",
  "name": "string",
  "component_type": "STRUCTURAL|ELECTRICAL|HYDRAULIC|AVIONICS|OTHER",
  "specifications": {
    "material": "string",
    "weight": "decimal",
    "unit_of_measure_weight": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Component with ID {componentId} not found."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.2.3 Update Component
**URL:** PATCH /api/v1/components/{componentId}

**Request Body Schema:**
```json
{
  "name": "string", // Optional - Updated component name
  "component_type": "STRUCTURAL|ELECTRICAL|HYDRAULIC|AVIONICS|OTHER", // Optional - Updated component type
  "specifications": { // Optional object
    "material": "string", // Optional - Updated material
    "weight": "decimal", // Optional - Updated weight
    "unit_of_measure_weight": "string" // Optional - Updated unit of measure
  }
}
```

**Success Response (200 OK):**
```json
{
  "component_id": "uuid",
  "design_id": "uuid",
  "name": "string",
  "component_type": "STRUCTURAL|ELECTRICAL|HYDRAULIC|AVIONICS|OTHER",
  "specifications": {
    "material": "string",
    "weight": "decimal",
    "unit_of_measure_weight": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_WEIGHT",
      "field": "specifications.weight",
      "message": "Weight must be greater than 0."
    },
    {
      "id": "INVALID_COMPONENT_TYPE",
      "field": "component_type",
      "message": "Component type must be one of: STRUCTURAL, ELECTRICAL, HYDRAULIC, AVIONICS, OTHER"
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Component with ID {componentId} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "DUPLICATE_COMPONENT_NAME",
      "field": "name",
      "message": "Duplicate component name detected within the same design and component type. Each component name must be unique within its component type for a given design. Duplicate names are only allowed if the components belong to different component type categories."
    },
    {
      "id": "IMMUTABLE_COMPONENT",
      "field": "component_id",
      "message": "Design with ID {designId} is in APPROVED status. Cannot modify component fields."
    },
    {
      "id": "COMPONENT_WEIGHT_EXCEEDS_DESIGN_WEIGHT",
      "field": "specifications.weight",
      "message": "Total component weight exceeds the `specifications.weight`. Please verify that component weights associated with the design are correct and update the design specifications accordingly."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.2.4 Delete Component
**URL:** DELETE /api/v1/components/{componentId}

**Success Response (204 No Content)**

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Component with ID {componentId} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot delete component referenced by an APPROVED design."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.2.5 List Components by Design
**URL:** GET /api/v1/designs/{designId}/components

**Query Parameters:**
- component_type (optional): Filter by component type
- include_deleted (optional): Include soft-deleted components

**Success Response (200 OK):**
```json
{
  "components": [
    {
      "component_id": "uuid",
      "name": "string",
      "component_type": "STRUCTURAL|ELECTRICAL|HYDRAULIC|AVIONICS|OTHER",
      "specifications": {
        "material": "string",
        "weight": "decimal",
        "unit_of_measure_weight": "string"
      },
      "metadata": {
        "created_at": "timestamp",
        "updated_at": "timestamp",
        "deleted_at": "timestamp"
      }
    }
  ],
  "count": "integer"
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Design with ID {designId} not found."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

### 5.3 Test Endpoints

#### 5.3.1 Create Test
**URL:** POST /api/v1/tests

**Request Body Schema:**
```json
{
  "design_id": "uuid", // Required - Reference to parent design
  "test_type": "AERODYNAMIC|STRUCTURAL|SYSTEMS|OTHER", // Required - Classification of the test
  "status": "PLANNED" // Required - Initial test status
}
```

**Success Response (201 Created):**
```json
{
  "test_id": "uuid",
  "design_id": "uuid",
  "test_type": "AERODYNAMIC|STRUCTURAL|SYSTEMS|OTHER",
  "status": "PLANNED",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_DESIGN_ID",
      "field": "design_id",
      "message": "Design ID is required."
    },
    {
      "id": "INVALID_TEST_TYPE",
      "field": "test_type",
      "message": "Test type must be one of: AERODYNAMIC, STRUCTURAL, SYSTEMS, OTHER"
    },
    {
      "id": "MISSING_TEST_TYPE",
      "field": "test_type",
      "message": "Test type is required."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Referenced design with ID {design_id} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot create test for a deleted design."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.3.2 Get Test
**URL:** GET /api/v1/tests/{testId}

**Success Response (200 OK):**
```json
{
  "test_id": "uuid",
  "design_id": "uuid",
  "status": "PLANNED|COMPLETED",
  "test_type": "AERODYNAMIC|STRUCTURAL|SYSTEMS|OTHER",
  "results": {
    "outcome": "PASS|FAIL",
    "notes": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Test with ID {testId} not found."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.3.3 Update Test
**URL:** PATCH /api/v1/tests/{testId}

**Request Body Schema:**
```json
{
  "status": "PLANNED|COMPLETED|FAILED", // Optional - Updated test status
  "test_type": "AERODYNAMIC|STRUCTURAL|SYSTEMS|OTHER", // Optional - Updated test type
  "results": { // Optional object
    "outcome": "PASS|FAIL", // Required if status is COMPLETED - Test outcome
    "notes": "string" // Optional - Detailed observations
  }
}
```

**Success Response (200 OK):**
```json
{
  "test_id": "uuid",
  "design_id": "uuid",
  "status": "PLANNED|COMPLETED",
  "test_type": "AERODYNAMIC|STRUCTURAL|SYSTEMS|OTHER",
  "results": {
    "outcome": "PASS|FAIL",
    "notes": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_OUTCOME",
      "field": "results.outcome",
      "message": "Outcome is required when status is COMPLETED."
    },
    {
      "id": "MISSING_DESIGN_ID",
      "field": "design_id",
      "message": "Design ID is required."
    },
    {
      "id": "INVALID_TEST_TYPE",
      "field": "test_type",
      "message": "Test type must be one of: AERODYNAMIC, STRUCTURAL, SYSTEMS, OTHER"
    },
    {
      "id": "IMMUTABLE_TEST_TYPE",
      "field": "test_type",
      "message": "Cannot modify test type when status is COMPLETED."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Test with ID {testId} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot modify test type when status is COMPLETED."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.3.4 Delete Test
**URL:** DELETE /api/v1/tests/{testId}

**Success Response (204 No Content)**

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Test with ID {testId} not found."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot delete test as it is required for design approval."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

#### 5.3.5 List Tests by Design
**URL:** GET /api/v1/designs/{designId}/tests

**Query Parameters:**
- status (optional): Filter by status (PLANNED, COMPLETED)
- test_type (optional): Filter by test type
- include_deleted (optional): Include soft-deleted tests

**Success Response (200 OK):**
```json
{
  "tests": [
    {
      "test_id": "uuid",
      "status": "PLANNED|COMPLETED",
      "test_type": "AERODYNAMIC|STRUCTURAL|SYSTEMS|OTHER",
      "results": {
        "outcome": "PASS|FAIL"
      },
      "metadata": {
        "created_at": "timestamp",
        "updated_at": "timestamp",
        "deleted_at": "timestamp"
      }
    }
  ],
  "count": "integer"
}
```

**Possible Error Responses:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Design with ID {designId} not found."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred while processing your request."
}
```

## 6. Example Workflows

### 6.1 Aircraft Design Approval Process

This example illustrates the complete workflow for creating, testing, and approving an aircraft design.

#### Step 1: Create Initial Design

**Request:**
```
POST /api/v1/designs
```
```json
{
  "name": "Sky Cruiser 700",
  "description": "Medium-range commercial aircraft with high fuel efficiency",
  "specifications": {
    "wingspan": 35.8,
    "length": 39.5,
    "weight": 42500,
    "engine_count": 2,
    "unit_of_measure_weight": "kg",
    "unit_of_measure_dimensions": "m"
  }
}
```

**Response (201 Created):**
```json
{
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "name": "Sky Cruiser 700",
  "status": "DRAFT",
  "metadata": {
    "created_at": "2023-06-15T10:30:45.123Z"
  }
}
```

#### Step 2: Add Components to Design

**Request:**
```
POST /api/v1/components
```
```json
{
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "name": "Main Wing Assembly",
  "component_type": "STRUCTURAL",
  "specifications": {
    "material": "Carbon fiber composite",
    "weight": 5200,
    "unit_of_measure_weight": "kg"
  }
}
```

**Response (201 Created):**
```json
{
  "component_id": "a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6",
  "name": "Main Wing Assembly",
  "component_type": "STRUCTURAL",
  "metadata": {
    "created_at": "2023-06-15T10:45:12.456Z"
  }
}
```

**Request:**
```
POST /api/v1/components
```
```json
{
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "name": "Turbofan Engine",
  "component_type": "OTHER",
  "specifications": {
    "material": "Titanium alloy, steel, composites",
    "weight": 2300,
    "unit_of_measure_weight": "kg"
  }
}
```

**Response (201 Created):**
```json
{
  "component_id": "f1e2d3c4-b5a6-7f8e-9d0c-b1a2c3d4e5f6",
  "name": "Turbofan Engine",
  "component_type": "OTHER",
  "metadata": {
    "created_at": "2023-06-15T10:50:28.789Z"
  }
}
```

#### Step 3: Create Required Tests

**Request:**
```
POST /api/v1/tests
```
```json
{
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "test_type": "AERODYNAMIC",
  "status": "PLANNED"
}
```

**Response (201 Created):**
```json
{
  "test_id": "c4d5e6f7-a8b9-0c1d-2e3f-4a5b6c7d8e9f",
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "test_type": "AERODYNAMIC",
  "status": "PLANNED",
  "metadata": {
    "created_at": "2023-06-16T09:15:33.654Z"
  }
}
```

**Request:**
```
POST /api/v1/tests
```
```json
{
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "test_type": "STRUCTURAL",
  "status": "PLANNED"
}
```

**Response (201 Created):**
```json
{
  "test_id": "e5f6g7h8-i9j0-k1l2-m3n4-o5p6q7r8s9t0",
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "test_type": "STRUCTURAL",
  "status": "PLANNED",
  "metadata": {
    "created_at": "2023-06-16T09:20:45.321Z"
  }
}
```

#### Step 4: Execute and Update Tests

**Request:**
```
PATCH /api/v1/tests/c4d5e6f7-a8b9-0c1d-2e3f-4a5b6c7d8e9f
```
```json
{
  "status": "COMPLETED",
  "results": {
    "outcome": "PASS",
    "notes": "Aerodynamic efficiency meets requirements. Lift/drag ratio exceeds minimum threshold by 8.3%."
  }
}
```

**Response (200 OK):**
```json
{
  "test_id": "c4d5e6f7-a8b9-0c1d-2e3f-4a5b6c7d8e9f",
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "status": "COMPLETED",
  "test_type": "AERODYNAMIC",
  "results": {
    "outcome": "PASS",
    "notes": "Aerodynamic efficiency meets requirements. Lift/drag ratio exceeds minimum threshold by 8.3%."
  },
  "metadata": {
    "created_at": "2023-06-16T09:15:33.654Z",
    "updated_at": "2023-06-18T14:25:12.789Z"
  }
}
```

**Request:**
```
PATCH /api/v1/tests/e5f6g7h8-i9j0-k1l2-m3n4-o5p6q7r8s9t0
```
```json
{
  "status": "COMPLETED",
  "results": {
    "outcome": "PASS",
    "notes": "Structural integrity confirmed. Wing load test completed with 150% of maximum expected load."
  }
}
```

**Response (200 OK):**
```json
{
  "test_id": "e5f6g7h8-i9j0-k1l2-m3n4-o5p6q7r8s9t0",
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "status": "COMPLETED",
  "test_type": "STRUCTURAL",
  "results": {
    "outcome": "PASS",
    "notes": "Structural integrity confirmed. Wing load test completed with 150% of maximum expected load."
  },
  "metadata": {
    "created_at": "2023-06-16T09:20:45.321Z",
    "updated_at": "2023-06-18T16:10:33.456Z"
  }
}
```

#### Step 5: Approve Design

**Request:**
```
PATCH /api/v1/designs/d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1
```
```json
{
  "status": "APPROVED"
}
```

**Response (200 OK):**
```json
{
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "name": "Sky Cruiser 700",
  "status": "APPROVED",
  "version": "1.0.0",
  "description": "Medium-range commercial aircraft with high fuel efficiency",
  "specifications": {
    "wingspan": 35.8,
    "length": 39.5,
    "weight": 42500,
    "engine_count": 2,
    "unit_of_measure_weight": "kg",
    "unit_of_measure_dimensions": "m"
  },
  "metadata": {
    "created_at": "2023-06-15T10:30:45.123Z",
    "updated_at": "2023-06-19T11:45:22.987Z"
  }
}
```

#### Potential Error Scenarios and Rollbacks

**Example 1: Attempting to approve without completing all tests**

If we tried to approve the design before completing the structural test:

**Request:**
```
PATCH /api/v1/designs/d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1
```
```json
{
  "status": "APPROVED"
}
```

**Response (409 Conflict):**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot approve design with tests still in PLANNED status."
}
```

**Example 2: Component weight exceeding design weight**

If components' total weight exceeds the design's specified weight:

**Request:**
```
POST /api/v1/components
```
```json
{
  "design_id": "d8f7a3c1-b5e2-4e7f-9a8b-c6d5e4f3a2b1",
  "name": "Additional Fuel Tank",
  "component_type": "STRUCTURAL",
  "specifications": {
    "material": "Aluminum alloy",
    "weight": 38000,
    "unit_of_measure_weight": "kg"
  }
}
```

**Response (400 Bad Request):**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "COMPONENT_WEIGHT_EXCEEDS_DESIGN_WEIGHT",
      "field": "specifications.weight",
      "message": "Total component weight exceeds the `specifications.weight`. Please verify that component weights associated with the design are correct and update the design specifications accordingly."
    }
  ]
}
```

**Example 3: Modifying component after design approval**

Attempting to modify a component after the design has been approved:

**Request:**
```
PATCH /api/v1/components/a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6
```
```json
{
  "specifications": {
    "material": "Updated carbon fiber composite",
    "weight": 5100
  }
}
```

**Response (409 Conflict):**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "IMMUTABLE_COMPONENT",
      "field": "component_id",
      "message": "Design with ID {designId} is in APPROVED status. Cannot modify component fields."
    }
  ]
}
```