# Film Production Design - Product Requirements Document

## 1. Introduction

### 1.1 Purpose
This document defines the API requirements for a comprehensive film production design system that enables coordinated management of sets, props, and locations throughout the lifecycle.

### 1.2 Scope
The API covers set design approval workflows, prop management with status tracking, and location booking processes, focusing on the critical relationships between these resources.

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
- Resources: sets, locations, props
- Complex aspects: Solve complex aspects including design approval workflows across sets and props, and integrated location management.

### 2.2 Base URL Configuration
- **Exact Base URL**: `/api/v1`
- **Versioning Strategy**: Explicit version in URL to support future API evolution
- **Global URL Pattern**:
  - Parent Resources: `/{resource-type}`
  - Child/Nested Resources: `/{parent-resource}/{parent-id}/{child-resource}`

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

### 3.1 Set Model
```json
{
  "set_id": "uuid",
  "name": "string",
  "description": "string",
  "status": "PLANNED|APPROVED|COMPLETED",
  "location_id": "uuid",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### Field Specifications

| Field | Type | Required | Constraints | Mutability | Default | Description |
|-------|------|----------|-------------|------------|---------|--------------|
| `set_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the set that remains consistent throughout its lifecycle, used in all API operations and cross-references from props and other resources |
| `name` | string | Yes | Length: 1-100 | Mutable | None | Official name of the set used in production documentation and planning, appears in schedules, budgets, and production reports |
| `description` | string | No | Length: 1-1000 | Mutable | Null | Detailed description of the set's appearance, purpose, and requirements, used for design reference and production planning meetings |
| `status` | Enum | Yes | `PLANNED`, `APPROVED`, `COMPLETED` | Mutable | `PLANNED` | Current state of the set in the production workflow, determines available actions, resource allocation, and visibility in scheduling |
| `location_id` | uuid | Yes | Must reference existing location | Mutable | None | Reference to the physical location where the set will be built or installed, critical for logistics planning and location management. Relationship: Location |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the set record was first created, used for audit trails and chronological ordering in production timelines |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any set field, used for tracking changes and synchronization with production schedules |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | Null | When populated, indicates the set has been soft-deleted and should be excluded from active production planning while maintaining historical references |

### 3.2 Location Model
```json
{
  "location_id": "uuid",
  "name": "string",
  "address": {
    "line1": "string",
    "city": "string",
    "state": "string",
    "country": "string"
  },
  "availability": "AVAILABLE|BOOKED|UNAVAILABLE",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### Field Specifications

| Field | Type | Required | Constraints | Mutability | Default | Description |
|-------|------|----------|-------------|------------|---------|--------------|
| `location_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the location that remains consistent throughout its lifecycle, used in all API operations and cross-references from sets |
| `name` | string | Yes | Length: 1-100 | Mutable | None | Descriptive name for the location used in production planning, schedules, and location management workflows |
| `address.line1` | string | Yes | Length: 1-200 | Mutable | None | Primary address line for the physical location, used for navigation, logistics planning, and legal documentation |
| `address.city` | string | Yes | Length: 2-100 | Mutable | None | City where the location is situated, essential for geographic organization, permit applications, and regional planning |
| `address.state` | string | Yes | Length: 2-100 | Mutable | None | State or province where the location is situated, required for legal documentation and jurisdictional considerations |
| `address.country` | string | Yes | Length: 2-100 | Mutable | None | Country where the location is situated, important for international productions and cross-border logistics planning |
| `availability` | Enum | Yes | `AVAILABLE`, `BOOKED`, `UNAVAILABLE` | Mutable | `AVAILABLE` | Current availability status of the location for production use, critical for scheduling and resource allocation across production teams |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the location record was first created, used for audit trails and chronological ordering of location acquisition |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any location field, used for tracking changes and synchronization with production schedules |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | Null | When populated, indicates the location has been soft-deleted and should be excluded from active production planning while maintaining historical references |

### 3.3 Prop Model
```json
{
  "prop_id": "uuid",
  "set_id": "uuid",
  "name": "string",
  "type": "FURNITURE|ELECTRONICS|DECOR|MISC",
  "approval_status": "PENDING|APPROVED|REJECTED",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp"
  }
}
```

#### Field Specifications

| Field | Type | Required | Constraints | Mutability | Default | Description |
|-------|------|----------|-------------|------------|---------|--------------|
| `prop_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the prop that remains consistent throughout its lifecycle|
| `set_id` | uuid | Yes | Must reference existing set | Mutable | None | Reference to the set where the prop will be used, establishing the relationship between props and their intended sets. Relationship: Set |
| `name` | string | Yes | Length: 1-100 | Mutable | None | Descriptive name of the prop for identification and tracking, appears in inventory lists, production documents, and set dressing plans |
| `type` | Enum | Yes | [`FURNITURE`, `ELECTRONICS`, `DECOR`, `MISC`] | Mutable | `MISC` | Classification of the prop for inventory organization, search filtering, and determining handling procedures and storage requirements |
| `approval_status` | Enum | Yes | [`PENDING`, `APPROVED`, `REJECTED`] | Mutable | `PENDING` | Approval state of the prop for use in production, controls whether the prop can be used in filming and appears in final set designs |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the prop record was first created, used for audit trails and chronological ordering in inventory management |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any prop field, used for tracking changes and synchronization with production timelines |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | Null | When populated, indicates the prop has been soft-deleted and should be excluded from active inventory while maintaining historical references |

## 4. Complex Business Rules

### 4.1 Conditional Field Requirements

#### Set Model
- `description` field becomes required when `status` transitions to "APPROVED"
- A Set cannot be directly created with `status` "COMPLETED" (must follow proper state transitions)

#### Location Model
- A Location cannot directly transition from "UNAVAILABLE" to "BOOKED" (must go through "AVAILABLE" first)

#### Prop Model
- A Prop with `approval_status` "APPROVED" cannot have its `type` modified

### 4.2 State Machine Transitions

#### 4.2.1 Set Status Transitions
- From `PLANNED` to `APPROVED`
- From `APPROVED` to `COMPLETED`
- Once `COMPLETED`, the status cannot be changed.

#### 4.2.2 Location Availability Transitions
- From `AVAILABLE` to `BOOKED`
- From `BOOKED` to `AVAILABLE`
- From `BOOKED` or `AVAILABLE` to `UNAVAILABLE`
- From `UNAVAILABLE` to `AVAILABLE`

#### 4.2.3 Prop Approval Status Transitions
- From `PENDING` to `APPROVED`
- From `PENDING` to `REJECTED`
- Once `APPROVED`, the status cannot be changed.
- Once `REJECTED`, the status cannot be changed.

### 4.3 Cross-Resource Validation Rules

#### 4.3.1 Set and Location Relationships
- A `Set` can only reference a `Location` with availability status `AVAILABLE` or `BOOKED`
- When a `Set` references a `Location`, that `Location`'s availability should transition to `BOOKED`
- A `Location`'s availability cannot be set to `AVAILABLE` if it is referenced by any other `Set`

#### 4.3.2 Set and Prop Relationships
- A `Prop` must reference a `Set` that exists and is not in `COMPLETED` status because a `prop` has to become available for reuse when the `set` is completed.
- A `Prop` with `approval_status` `REJECTED` cannot be associated with a `Set` in `COMPLETED` status because a rejected `prop` cannot be used in a completed `set`.
- A `Prop` `type` cannot be modified if its referenced `Set` has status "COMPLETED"


### 4.4 Multi-Step Operations

#### 4.4.1 Set Approval Process
1. **Initial Creation**
   - Create `Set` record with status `PLANNED`
   - Define basic attributes (name, description)

2. **Approval Request**
   - Verify locations with status `AVAILABLE`. Book the location by changing the availability to `BOOKED`.
   - Assign Props with approval_status `PENDING`.
   - Once the Prop is approved and the location is also in `BOOKED` status, change `status` of the Set to `APPROVED`. In this step, the description of the Set is not empty.

3. **Completion**
   - Change status to `COMPLETED`.

**Rollback Scenarios:**
- If the location is `UNAVAILABLE` due to some reason after it was booked, the Set should be changed to `PLANNED` status and new location should be assigned.

### 4.5 Deletion Behavior

#### 4.5.1 Set Model Deletion Rules

**Prevention Criteria:**
- Cannot delete a Set that has associated Props with approval_status "APPROVED"
- Cannot delete a Set if its referenced Location has availability "BOOKED".

**Cascade Effects:**
- When a Set is marked for deletion (metadata.deleted_at populated) and the Set is in `PLANNED` status:
  - All associated Props should remove `set_id` reference
  - If the Location was "BOOKED" exclusively for this Set, change the availability of the Location to `AVAILABLE`.

**Effects When Deletion Succeeds:**
- The Set record is maintained with metadata.deleted_at populated

#### 4.5.2 Location Model Deletion Rules

**Prevention Criteria:**
- Cannot delete a Location that is referenced by any Set with status "APPROVED" or "PLANNED"

**Cascade Effects:**
- When a Location is marked for deletion (metadata.deleted_at populated):
  - All Sets in "PLANNED" status referencing this Location must be reassigned to a new Location

**Effects When Deletion Succeeds:**
- The Location record is maintained with metadata.deleted_at populated

#### 4.5.3 Prop Model Deletion Rules

**Prevention Criteria:**
- Cannot delete a Prop with approval_status "APPROVED"
- Cannot delete a Prop referenced by a Set with status "COMPLETED"

**Effects When Deletion Succeeds:**
- The Prop record is maintained with metadata.deleted_at populated

## 5. Comprehensive API Endpoints

### 5.1 Set Model API Endpoints

#### 5.1.1 Create Set
**URL:** POST /api/v1/sets

**Request Body Schema:**
```json
{
  "name": "string",
  "description": "string",
  "status": "PLANNED",
  "location_id": "uuid"
}
```

**Success Response (201 Created):**
```json
{
  "set_id": "uuid",
  "name": "string",
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
      "id": "MISSING_NAME",
      "field": "name",
      "message": "Name is required for set creation"
    }
  ]
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Referenced location is not available for booking."
}
```
- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "Status cannot be set to APPROVED or COMPLETED while creating a set."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.1.2 Get Set
**URL:** GET /api/v1/sets/{setId}

**Success Response (200 OK):**
```json
{
  "set_id": "uuid",
  "name": "string",
  "description": "string",
  "status": "PLANNED|APPROVED|COMPLETED",
  "location": {
    "id": "uuid",
    "availability": "AVAILABLE|BOOKED|UNAVAILABLE"
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
  "message": "Set not found based on the provided set_id"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.1.3 Update Set
**URL:** PATCH /api/v1/sets/{setId}

**Request Body Schema:**
```json
{
  "name": "string",
  "description": "string",
  "status": "PLANNED|APPROVED|COMPLETED",
  "location_id": "uuid"
}
```

**Success Response (200 OK):**
```json
{
  "set_id": "uuid",
  "metadata": {
    "updated_at": "timestamp"
  }
}
```

**Error Responses:**

- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Name must be between 1 and 100 characters."
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "location_id provided is already booked by another set"
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_LOCATION",
      "field": "location_id",
      "message": "Location reference is required for set creation"
    },
    {
      "id": "MISSING_DESCRIPTION",
      "field": "description",
      "message": "Description is required when transitioning to APPROVED status"
    },
    {
      "id": "INVALID_STATE_TRANSITION",
      "field": "status",
      "message": "Cannot transition from PLANNED to COMPLETED without APPROVED status"
    }
  ]
}
```

#### 5.1.4 Delete Set
**URL:** DELETE /api/v1/sets/{setId}

**Success Response:**
- 204 No Content – Set successfully deleted (soft delete)

**Possible Error Responses:**
- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Set not found based on the provided set_id"
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Set cannot be deleted due to props in APPROVED status or location in BOOKED status"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.1.5 Bulk Delete Sets
**URL:** DELETE /api/v1/sets

**Request Body Schema:**
```json
{
  "set_ids": ["uuid", "uuid"]
}
```

**Response:**
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "SET_HAS_APPROVED_PROPS"
    }
  ]
}
```

**Possible Error Responses:**
- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "set_ids are required to delete sets"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

### 5.2 Location Model API Endpoints

#### 5.2.1 Create Location
**URL:** POST /api/v1/locations

**Request Body Schema:**
```json
{
  "name": "string",
  "address": {
    "line1": "string",
    "city": "string",
    "state": "string",
    "country": "string"
  },
  "availability": "AVAILABLE|BOOKED|UNAVAILABLE"
}
```

**Success Response (201 Created):**
```json
{
  "location_id": "uuid",
  "name": "string",
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
      "id": "MISSING_NAME",
      "field": "name",
      "message": "Name is required for location creation"
    },
    {
      "id": "MISSING_ADDRESS",
      "field": "address",
      "message": "Address information is required"
    },
    {
      "id": "INVALID_AVAILABILITY",
      "field": "availability",
      "message": "Availability must be one of: AVAILABLE, BOOKED, UNAVAILABLE"
    }
  ]
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "Partial address provided, please provide all address fields"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.2.2 Get Location
**URL:** GET /api/v1/locations/{locationId}

**Success Response (200 OK):**
```json
{
  "location_id": "uuid",
  "name": "string",
  "address": {
    "line1": "string",
    "city": "string",
    "state": "string",
    "country": "string"
  },
  "availability": "AVAILABLE|BOOKED|UNAVAILABLE",
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
  "message": "Location not found based on the provided location_id"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.2.3 Update Location
**URL:** PATCH /api/v1/locations/{locationId}

**Request Body Schema:**
```json
{
  "name": "string",
  "address": {
    "line1": "string",
    "city": "string",
    "state": "string",
    "country": "string"
  },
  "availability": "AVAILABLE|BOOKED|UNAVAILABLE"
}
```

**Success Response (200 OK):**
```json
{
  "location_id": "uuid",
  "metadata": {
    "updated_at": "timestamp"
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
      "id": "INVALID_AVAILABILITY",
      "field": "availability",
      "message": "Availability must be one of: AVAILABLE, BOOKED, UNAVAILABLE"
    },
    {
      "id": "INVALID_NAME",
      "field": "name",
      "message": "Name must be between 1 and 100 characters."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Location not found based on the provided location_id"
}
```

- **409 Conflict:**
```json
{
  "error_id": "LOCATION_IN_USE",
  "message": "Cannot change availability of location referenced by any other set"
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Cannot transition availability directly from UNAVAILABLE to BOOKED"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.2.4 Delete Location
**URL:** DELETE /api/v1/locations/{locationId}

**Success Response:** 
- 204 No Content – Location successfully deleted (soft delete)

**Possible Error Responses:**
- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Location not found based on the provided location_id"
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_IN_USE",
  "message": "Location cannot be deleted as it is referenced by sets in APPROVED or PLANNED status"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.2.5 Bulk Delete Locations
**URL:** DELETE /api/v1/locations

**Request Body Schema:**
```json
{
  "location_ids": ["uuid", "uuid"]
}
```

**Response:**
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "LOCATION_REFERENCED_BY_SETS"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

**Possible Error Responses:**
- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "location_ids are required to delete locations"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

### 5.3 Prop Model API Endpoints

#### 5.3.1 Create Prop
**URL:** POST /api/v1/props

**Request Body Schema:**
```json
{
  "set_id": "uuid",
  "name": "string",
  "type": "FURNITURE|ELECTRONICS|DECOR|MISC",
  "approval_status": "PENDING"
}
```

**Success Response (201 Created):**
```json
{
  "prop_id": "uuid",
  "name": "string",
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
      "id": "MISSING_NAME",
      "field": "name",
      "message": "Name is required for prop creation"
    },
    {
      "id": "INVALID_TYPE",
      "field": "type",
      "message": "Type must be one of: FURNITURE, ELECTRONICS, DECOR, MISC"
    },
    {
      "id": "INVALID_NAME",
      "field": "name",
      "message": "Name must be between 1 and 100 characters."
    }

  ]
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot assign prop to a set with status COMPLETED"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.3.2 Get Prop
**URL:** GET /api/v1/props/{propId}

**Success Response (200 OK):**
```json
{
  "prop_id": "uuid",
  "set_id": "uuid",
  "name": "string",
  "type": "FURNITURE|ELECTRONICS|DECOR|MISC",
  "approval_status": "PENDING|APPROVED|REJECTED",
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
  "message": "Prop not found based on the provided prop_id"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.3.3 Update Prop
**URL:** PATCH /api/v1/props/{propId}

**Request Body Schema:**
```json
{
  "set_id": "uuid",
  "name": "string",
  "type": "FURNITURE|ELECTRONICS|DECOR|MISC",
  "approval_status": "PENDING|APPROVED|REJECTED"
}
```

**Success Response (200 OK):**
```json
{
  "prop_id": "uuid",
  "metadata": {
    "updated_at": "timestamp"
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
      "id": "INVALID_TYPE",
      "field": "type",
      "message": "Type must be one of: FURNITURE, ELECTRONICS, DECOR, MISC"
    },
    {
      "id": "INVALID_NAME",
      "field": "name",
      "message": "Name must be between 1 and 100 characters."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Prop not found based on the provided prop_id"
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot reassign prop to a completed set"
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "prop type cannot be modified since the set is in completed status."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.3.4 Delete Prop
**URL:** DELETE /api/v1/props/{propId}

**Success Response:**
- 204 No Content – Prop successfully deleted (soft delete)

**Possible Error Responses:**
- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Prop not found based on the provided prop_id"
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "PROP_APPROVED_CANT_DELETE",
      "field": "approval_status",
      "message": "Cannot delete prop with approval_status APPROVED"
    },
    {
      "id": "PROP_SET_COMPLETED_CANT_DELETE",
      "field": "set_id",
      "message": "Cannot delete a Prop referenced by a Set with status COMPLETED"
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.3.5 Bulk Delete Props
**URL:** DELETE /api/v1/props

**Request Body Schema:**
```json
{
  "prop_ids": ["uuid", "uuid"]
}
```

**Response:**
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "PROP_APPROVAL_STATUS_APPROVED"
    },
    {
      "id": "uuid",
      "reason": "PROP_SET_COMPLETED"
    }
  ],
  "deleted_count": 2,
  "failed_count": 2
}
```

**Possible Error Responses:**
- **400 Bad Request:**
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "prop_ids are required to delete props"
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

## 6. Film Production Design Workflow Examples

### 6.1 Set Approval Process Workflow

#### 6.1.1 Initial Creation Phase

**Step 1: Create a New Set with PLANNED Status**

**Request:**
```http
POST /api/v1/sets
Content-Type: application/json

{
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "name": "Downtown Café Set",
  "status": "PLANNED",
  "description": "Modern café interior with seating for 20"
}
```

**Response (201 Created):**
```json
{
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "name": "Downtown Café Set",
  "metadata": {
    "created_at": "2024-05-15T09:30:00.000Z"
  }
}
```

#### 6.1.2 Approval Request Phase

**Step 2: Verify and Book Location**

First, we need to find an available location:

**Request:**
```http
GET /api/v1/locations/l7d8e9f0-1a2b-3c4d-5e6f-7g8h9i0j1k2l
```

**Response (200 OK):**
```json
{
  "location_id": "l7d8e9f0-1a2b-3c4d-5e6f-7g8h9i0j1k2l",
  "name": "Studio B",
  "address": {
    "line1": "123 Production Way",
    "city": "Hollywood",
    "state": "California",
    "country": "USA"
  },
  "availability": "AVAILABLE",
  "metadata": {
    "created_at": "2024-04-01T10:00:00.000Z",
    "updated_at": "2024-04-01T10:00:00.000Z",
    "deleted_at": null
  }
}
```

Now, assign this location to our set, which will automatically book it:

**Request:**
```http
PATCH /api/v1/sets/s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
Content-Type: application/json

{
  "location_id": "l7d8e9f0-1a2b-3c4d-5e6f-7g8h9i0j1k2l"
}
```

**Response (200 OK):**
```json
{
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "metadata": {
    "updated_at": "2024-05-15T10:15:00.000Z"
  }
}
```

Verify that the location is now booked:

**Request:**
```http
GET /api/v1/locations/l7d8e9f0-1a2b-3c4d-5e6f-7g8h9i0j1k2l
```

**Response (200 OK):**
```json
{
  "location_id": "l7d8e9f0-1a2b-3c4d-5e6f-7g8h9i0j1k2l",
  "name": "Studio B",
  "address": {
    "line1": "123 Production Way",
    "city": "Hollywood",
    "state": "California",
    "country": "USA"
  },
  "availability": "BOOKED",
  "metadata": {
    "created_at": "2024-04-01T10:00:00.000Z",
    "updated_at": "2024-05-15T10:15:00.000Z",
    "deleted_at": null
  }
}
```

**Step 3: Assign Props with PENDING Status**

**Request:**
```http
POST /api/v1/props
Content-Type: application/json

{
  "prop_id": "p3f4g5h6-7i8j-9k0l-1m2n-3o4p5q6r7s8t",
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "name": "Vintage Espresso Machine",
  "type": "DECOR",
  "approval_status": "PENDING"
}
```

**Response (201 Created):**
```json
{
  "prop_id": "p3f4g5h6-7i8j-9k0l-1m2n-3o4p5q6r7s8t",
  "name": "Vintage Espresso Machine",
  "metadata": {
    "created_at": "2024-05-15T11:00:00.000Z"
  }
}
```

Add another prop:

**Request:**
```http
POST /api/v1/props
Content-Type: application/json

{
  "prop_id": "p9t8s7r6-5q4p-3o2n-1m0l-9k8j7i6h5g4f",
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "name": "Barista Counter",
  "type": "FURNITURE",
  "approval_status": "PENDING"
}
```

**Response (201 Created):**
```json
{
  "prop_id": "p9t8s7r6-5q4p-3o2n-1m0l-9k8j7i6h5g4f",
  "name": "Barista Counter",
  "metadata": {
    "created_at": "2024-05-15T11:05:00.000Z"
  }
}
```

**Step 4: Approve the Props**

**Request:**
```http
PATCH /api/v1/props/p3f4g5h6-7i8j-9k0l-1m2n-3o4p5q6r7s8t
Content-Type: application/json

{
  "approval_status": "APPROVED"
}
```

**Response (200 OK):**
```json
{
  "prop_id": "p3f4g5h6-7i8j-9k0l-1m2n-3o4p5q6r7s8t",
  "metadata": {
    "updated_at": "2024-05-15T13:30:00.000Z"
  }
}
```

**Request:**
```http
PATCH /api/v1/props/p9t8s7r6-5q4p-3o2n-1m0l-9k8j7i6h5g4f
Content-Type: application/json

{
  "approval_status": "APPROVED"
}
```

**Response (200 OK):**
```json
{
  "prop_id": "p9t8s7r6-5q4p-3o2n-1m0l-9k8j7i6h5g4f",
  "metadata": {
    "updated_at": "2024-05-15T13:35:00.000Z"
  }
}
```

**Step 5: Approve the Set**

Now that we have the location booked and props approved, we can approve the set:

**Request:**
```http
PATCH /api/v1/sets/s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
Content-Type: application/json

{
  "status": "APPROVED",
  "description": "Modern café interior with seating for 20, featuring a vintage espresso machine and full barista counter. All props are approved and the location is booked."
}
```

**Response (200 OK):**
```json
{
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "metadata": {
    "updated_at": "2024-05-15T14:00:00.000Z"
  }
}
```

**Error Scenario - Missing Description:**

If we had attempted to approve the set without providing a comprehensive description:

**Request:**
```http
PATCH /api/v1/sets/s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
Content-Type: application/json

{
  "status": "APPROVED"
}
```

**Response (422 Unprocessable Entity):**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_DESCRIPTION",
      "field": "description",
      "message": "Description is required when transitioning to APPROVED status"
    }
  ]
}
```

#### 6.1.3 Completion Phase

**Step 6: Complete the Set**

After the set has been used for production, we mark it as completed:

**Request:**
```http
PATCH /api/v1/sets/s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
Content-Type: application/json

{
  "status": "COMPLETED"
}
```

**Response (200 OK):**
```json
{
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "metadata": {
    "updated_at": "2024-06-30T17:00:00.000Z"
  }
}
```

**Error Scenario - Invalid State Transition:**

If we had attempted to transition directly from PLANNED to COMPLETED:

**Request:**
```http
PATCH /api/v1/sets/s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
Content-Type: application/json

{
  "status": "COMPLETED"
}
```

**Response (422 Unprocessable Entity):**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_STATE_TRANSITION",
      "field": "status",
      "message": "Cannot transition from PLANNED to COMPLETED without APPROVED status"
    }
  ]
}
```

### 6.2 Rollback Scenario

In case the location becomes unavailable after it was booked, we need to rollback and reassign:

**Step 1: Location Becomes Unavailable**

**Request:**
```http
PATCH /api/v1/locations/l7d8e9f0-1a2b-3c4d-5e6f-7g8h9i0j1k2l
Content-Type: application/json

{
  "availability": "UNAVAILABLE"
}
```

**Response (200 OK):**
```json
{
  "location_id": "l7d8e9f0-1a2b-3c4d-5e6f-7g8h9i0j1k2l",
  "metadata": {
    "updated_at": "2024-05-20T09:00:00.000Z"
  }
}
```

**Step 2: Set Status Reverts to PLANNED**

**Request:**
```http
PATCH /api/v1/sets/s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
Content-Type: application/json

{
  "status": "PLANNED"
}
```

**Response (200 OK):**
```json
{
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "metadata": {
    "updated_at": "2024-05-20T09:15:00.000Z"
  }
}
```

**Step 3: Assign New Location**

**Request:**
```http
PATCH /api/v1/sets/s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d
Content-Type: application/json

{
  "location_id": "l2k3j4h5-6g7f-8e9d-0c1b-2a3b4c5d6e7f"
}
```

**Response (200 OK):**
```json
{
  "set_id": "s1b2c3d4-e5f6-7a8b-9c0d-1e2f3a4b5c6d",
  "metadata": {
    "updated_at": "2024-05-20T09:30:00.000Z"
  }
}
```