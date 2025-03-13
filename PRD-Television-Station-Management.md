# Television Station Management - Product Requirements Document

## 1. Introduction

### 1.1 Purpose

This document outlines the requirements for the Television Station Management API, which provides a comprehensive set of endpoints for managing television stations, including program scheduling, advertiser management, and rating analysis.

### 1.2 Scope

The APIs are designed to support the operations of a television station, including program scheduling, advertiser management, and rating analysis. It is intended for use by television stations and external partners associated with the station.

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
- Resources: programs, staff, advertisers
- Complex aspects: Manage program scheduling, staff assignments, and advertiser placements. The platform supports the creation of new programs and the management of existing ones. The platform also support ratings and viewership analysis.

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

### 3.1 Program Model
```json
{
  "program_id": "uuid",
  "title": "string",
  "genre": "DRAMA|COMEDY|NEWS|SPORTS|DOCUMENTARY|REALITY|CHILDREN|MUSIC|LIFESTYLE|OTHER",
  "description": "string",
  "duration_minutes": "decimal",
  "production_cost": "decimal",
  "schedule": {
  "date": "date",
  "start_time": "time",
  "end_time": "time",
  "status": "PLANNED|LIVE|COMPLETED|CANCELLED",
    "repeat_indicator": "boolean"
  },
  "ratings": {
    "rating_value": "decimal",
    "viewers_count": "integer"
  },
  "staff_assignments": [
    {
      "staff_id": "uuid"
    }
  ],
  "advertising_slots": [
    {
      "advertiser_id": "uuid",
      "slot_duration (minutes)": "integer",
      "price_paid": "decimal"
    }
  ],
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
| `program_id` | uuid | Yes | Unique identifier for the program | Never | None | Primary identifier for the program that remains consistent throughout its lifecycle, used in all API operations and cross-references from other resources |
| `title` | string | Yes | Descriptive title | Mutable | None | Official title of the television program that will appear in schedules, promotional materials, and viewer guides |
| `genre` | enum | Yes | One of: DRAMA, COMEDY, NEWS, SPORTS, DOCUMENTARY, REALITY, CHILDREN, MUSIC, LIFESTYLE, OTHER | Mutable | None | Classification of program content used for content organization, search filtering, and audience targeting |
| `description` | string | No | Program description up to 5000 characters | Mutable | None | Detailed synopsis of the program content, including plot summary, themes, and key elements for viewer information and promotional purposes |
| `duration_minutes` | decimal | Yes | Positive decimal representing duration | Mutable | None | Length of the program in minutes, used for scheduling, time slot allocation, and broadcast planning |
| `production_cost` | decimal | No | Positive decimal representing cost | Mutable | None | Financial investment required to produce the program, used for budget management, ROI analysis, and financial reporting |
| `schedule.date` | date | Yes | Valid date for broadcast | Mutable | None | Calendar date when the program is scheduled to air, used for planning and viewer information |
| `schedule.start_time` | time | Yes | Start time in HH:MM:SS format | Mutable | None | Precise time when the program broadcast begins, critical for scheduling accuracy and viewer guidance |
| `schedule.end_time` | time | Yes | End time in HH:MM:SS format | Mutable | None | Precise time when the program broadcast concludes, used to determine broadcast duration and subsequent program scheduling |
| `schedule.status` | enum | Yes | One of: PLANNED, LIVE, COMPLETED, CANCELLED | Mutable | PLANNED | Current state of the scheduled broadcast that determines its visibility, modifiability, and operational handling |
| `schedule.repeat_indicator` | boolean | No | Indicates if program is a rerun | Mutable | false | Flag indicating whether the broadcast is a first-run or repeat showing, important understanding the program's history |
| `ratings.rating_value` | decimal | No | Decimal value, can be fractional with Range 0.0 to 10.0 | Mutable | None | Numerical score representing the program's viewership performance, used for success measurement and comparative analysis |
| `ratings.viewers_count` | integer | No | Number of viewers, integer | Mutable | None | Estimated total audience size for the program broadcast, critical for advertising value assessment and content decisions |
| `staff_assignments` | array | No | Array of staff assignment objects | Mutable | [] | Collection of staff members assigned to the program, establishing the relationship between personnel and content |
| `staff_assignments[].staff_id` | uuid | Yes | Must reference an existing staff | Mutable | None | Reference to a staff member working on the program, enabling accountability and resource allocation tracking. Relationship: Staff |
| `advertising_slots` | array | No | Array of advertising placement objects | Mutable | [] | Collection of advertising allocations for the program, connecting advertisers to the content and managing revenue generation |
| `advertising_slots[].advertiser_id` | uuid | Yes | Must reference existing advertiser | Mutable | None | Reference to the company purchasing advertising time during the program broadcast. Relationship: Advertiser |
| `advertising_slots[].slot_duration (minutes)` | integer | Yes | Duration in minutes | Mutable | None | Length of time allocated for the advertisement, used for scheduling and pricing calculations |
| `advertising_slots[].price_paid` | decimal | Yes | Positive amount | Mutable | None | Financial compensation received for the advertising placement, important for revenue tracking and profitability analysis |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Never | Current time | Precise moment when the program was first created in the system, used for audit trails and chronological ordering |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Mutable | Current time | Timestamp of the most recent modification to any program field, used for change tracking and synchronization |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Mutable | None | When populated, indicates the program has been soft-deleted and should be excluded from active operations while retained for archival purposes |

### 3.2 Staff Model
```json
{
  "staff_id": "uuid",
  "first_name": "string",
  "last_name": "string",
  "role": "PRODUCER|DIRECTOR|EDITOR|ANCHOR|MANAGER",
  "email": "string",
  "phone": "string",
  "program_history": [
    {
      "program_id": "uuid"
    }
  ],
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
| `staff_id` | uuid | Yes | Unique identifier for staff | Never | None | Primary identifier for the staff member that remains consistent throughout their employment, used in all API operations and cross-references |
| `first_name` | string | Yes | Staff's first name, characters only | Mutable | None | Given name of the staff member, used for identification, communications, and personalization |
| `last_name` | string | Yes | Staff's last name, characters only | Mutable | None | Family name of the staff member, used for identification, communications, and formal documentation |
| `role` | enum | Yes | One of: PRODUCER, DIRECTOR, EDITOR, ANCHOR, MANAGER | Mutable | None | Job title or functional position within the organization, determining access rights and reporting structure |
| `email` | string | Yes | Valid professional email | Mutable | None | Official work email address for the staff member, used for communications, system access, and notifications |
| `phone` | string | No | Contact number in E.164 format | Mutable | None | Work contact number for the staff member, used for direct communications and emergency contact purposes |
| `program_history` | array | No | Array of program contribution objects | Mutable | [] | Historical record of programs the staff member has worked on, tracking experience and specialization over time.|
| `program_history[].program_id` | uuid | Yes | Must reference existing program | Mutable | None | Reference to a program the staff member contributed to, connecting personnel to content production history. Relationship: Program |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Never | Current time | Precise moment when the staff record was first created in the system, used for audit trails and employment tracking |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Mutable | Current time | Timestamp of the most recent modification to any staff field, used for change tracking and personnel management |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Mutable | None | When populated, indicates the staff record has been soft-deleted and should be excluded from active operations while retained for historical purposes |

### 3.3 Advertiser Model
```json
{
  "advertiser_id": "uuid",
  "name": "string",
  "email": "string",
  "phone": "string",
  "contract_end_date": "date",
  "advertisement_details": {
    "target_time_slot": "time",
    "pricing_tier": "PREMIUM|STANDARD|ECONOMY|PROMOTIONAL",
    "budget": "decimal"
  },
  "program_placements": [
    {
      "program_id": "uuid",
      "slot_count": "integer",
      "total_value": "decimal"
    }
  ],
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
| `advertiser_id` | uuid | Yes | Unique identifier for each advertiser | Never | None | Primary identifier for the advertiser that remains consistent throughout the relationship, used in all API operations and cross-references |
| `name` | string | Yes | Full name of advertiser | Mutable | None | Official name of the advertising entity, used for identification, billing, and relationship management |
| `email` | string | Yes | Valid email format | Mutable | None | Primary contact email address for the advertiser, used for communications, account notifications, and document delivery |
| `phone` | string | Yes | Contact phone in E.164 format | Mutable | None | Primary contact phone number for the advertiser, used for urgent communications and verification purposes |
| `contract_end_date` | date | No | End date of advertising contract | Mutable | None | Expiration date of the current advertising agreement, critical for renewal planning and contract management |
| `advertisement_details.target_time_slot` | time | No | Preferred broadcast time with HH:MM:SS format | Mutable | None | Preferred broadcast time for advertisements, used to align with target audience viewing patterns and maximize effectiveness |
| `advertisement_details.pricing_tier` | enum | No | One of: PREMIUM, STANDARD, ECONOMY, PROMOTIONAL | Mutable | STANDARD | Qualitative classification of the advertising rate structure, determining budget allocation and service level expectations |
| `advertisement_details.budget` | decimal | No | Total campaign budget | Mutable | None | Maximum financial allocation for the advertising campaign, used for planning, resource allocation, and financial forecasting |
| `program_placements` | array | No | Array of program placement objects | Mutable | [] | Collection of programs where advertisements have been or will be placed, linking advertisers to specific content.|
| `program_placements[].program_id` | uuid | Yes | Must reference existing program | Mutable | None | Reference to a program where the advertiser's content will be shown, establishing the relationship between advertiser and content. Relationship: Program |
| `program_placements[].slot_count` | integer | Yes | Number of ad slots purchased | Mutable | None | Quantity of advertising opportunities allocated within the referenced program, used for capacity planning and inventory management |
| `program_placements[].total_value` | decimal | Yes | Financial value of placement | Mutable | None | Cumulative monetary value of all advertising slots within the program, used for revenue forecasting and financial reporting |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Never | Current time | Precise moment when the advertiser record was first created in the system, used for audit trails and chronological ordering |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Mutable | Current time | Timestamp of the most recent modification to any advertiser field, used for change tracking and relationship management |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone | Mutable | None | When populated, indicates the advertiser has been soft-deleted and should be excluded from active operations while retained for archival purposes |

## 4. Business Rules

### 4.1 Conditional Field Requirements

#### 4.1.1 Program Model
- `title` must be unique across all active programs to prevent scheduling confusion
- `duration_minutes` must be compatible with standard broadcast time slots (typically multiples of 15 or 30 minutes)
- `schedule.repeat_indicator` must be set to true when the same program is scheduled within a 7-day period
- If `schedule.status` is `LIVE`, then `schedule.start_time` and `schedule.end_time` fields become immutable

#### 4.1.2 Staff Model
- `email` must be unique across all staff members
- `email` domain must match the organization's approved domain list

#### 4.1.3 Advertiser Model
- `email` must be unique across all advertisers
- When `advertisement_details.pricing_tier` is "PREMIUM", minimum `advertisement_details.budget` must be 50,000
- For each advertiser, total `program_placements` value cannot exceed `advertisement_details.budget`

### 4.2 State Machine Transitions

#### 4.2.1 Program Schedule Status
- `PLANNED` can transition to `LIVE`
- `PLANNED` can transition to `CANCELLED`
- `LIVE` can transition to `COMPLETED`
- Once a schedule is `COMPLETED` or `CANCELLED`, no further transitions are allowed

### 4.3 Cross-Resource Validation Rules

#### 4.3.1 Program-Staff Validation
- Staff with `role` "PRODUCER" must be assigned to a Program before it can transition to `PLANNED` status
- A Program with `genre` "NEWS" must have at least one Staff with `role` "ANCHOR" assigned
- A Staff member cannot be assigned to more than one Program with overlapping broadcast times

#### 4.3.2 Program-Advertiser Validation
- Advertiser's `advertisement_details.target_time_slot` must be compatible with Program's `schedule.start_time` and `schedule.end_time`
- When Program `schedule.status` changes to `CANCELLED`, all associated Advertisers must be notified and their advertising slots must be cancelled.

### 4.4 Multi-Step Operations

#### 4.4.1 Program Broadcast Process
1. Create Program with data including title, genre, description, and duration
2. Assign required Staff members based on Program type and requirements
3. Schedule the Program by setting date, start_time, and end_time
4. Allocate advertising slots based on Program duration
5. Approve final Program schedule and resource allocations
6. Transition schedule status to LIVE at broadcast time
7. Process financial transactions for advertising slots

**Rollback Scenarios:**
- Program time slot conflicts: If a program is scheduled to air during an already scheduled program, the new program will be scheduled for a different time slot.
- Staff unavailability: Reassign roles and reschedule program

### 4.5 Business Logic Triggers

#### 4.5.1 Status Change Actions
- When Program schedule.status changes to COMPLETED:
  * Rating data is automatically updated.
  * Staff program_history is automatically updated
  * Advertising revenue is automatically calculated and recorded
  * Program metadata.updated_at is automatically updated
  
- When Staff role changes:
  * All current Program assignments are reviewed for compatibility

#### 4.5.2 Automatic Calculations
- Program broadcast duration is verified against schedule.start_time and schedule.end_time
- Advertiser campaign remaining budget is recalculated after each advertisement placement
- Program profitability is calculated by comparing production_cost against advertising revenue

#### 4.5.3 Cascading Actions
- When a Program is archived:
  * All future broadcasts are automatically cancelled
  * Associated staff are notified and assignments cleared
  * Advertising slots are made unavailable
  * Program content is moved to long-term storage
  
- When an Advertiser's contract_end_date passes:
  * All future ad placements are cancelled

### 4.6 Deletion Behavior

#### 4.6.1 Program Model

**Prevention Criteria:**
- Cannot delete a Program with schedule.status of LIVE
- Cannot delete a Program with active advertising commitments

**Cascade Effects:**
- Staff assignments are removed from the Program
- Advertising slots are cancelled and advertisers notified
- Ratings data is preserved for historical analysis

**Effects When Deletion Succeeds:**
- Program deleted_at timestamp is set to the current UTC time
- Program ID is preserved to maintain referential integrity in historical records and data is moved to archival storage based on retention policies

#### 4.6.2 Staff Model

**Prevention Criteria:**
- Cannot delete a Staff member assigned to any Program with schedule.status LIVE

**Cascade Effects:**
- All future Program assignments are reassigned or flagged for reassignment

**Effects When Deletion Succeeds:**
- Staff deleted_at timestamp is set to the current UTC time
- Staff record is excluded from active assignment pools
- Staff ID is preserved to maintain referential integrity in historical records and data is moved to archival storage based on retention policies

#### 4.6.3 Advertiser Model

**Prevention Criteria:**
- Cannot delete an Advertiser with current date before contract_end_date
- Cannot delete an Advertiser with scheduled advertisements in Programs with status LIVE
- Cannot delete an Advertiser with outstanding financial transactions

**Cascade Effects:**
- All future advertising placements are cancelled
- Contract information is archived according to financial retention policies
- Advertising content is archived according to content retention policies

**Effects When Deletion Succeeds:**
- Advertiser deleted_at timestamp is set to the current UTC time
- Advertiser ID is preserved to maintain referential integrity in historical records and data is moved to archival storage based on retention policies

### 4.7 Deletion Audit Requirements

**Audit Trail:**

- All deletion attempts (successful or prevented) must be logged with:
  - Timestamp of deletion attempt (ISO 8601 format)
  - User or system identifier that initiated the deletion
  - Resource ID and type

**Retention Period:**

- Deletion audit logs must be retained for a minimum of 7 years
- Deletion of critical infrastructure components must be retained for 15 years
- Audit logs cannot be deleted or modified once created

## 5. Television Station Management API Endpoints

### 5.1 Program Management Endpoints

#### 5.1.1 Create Program
**URL:** POST /api/v1/programs

**Request Body Schema:**
```json
{
  "title": "string",
  "genre": "DRAMA|COMEDY|NEWS|SPORTS|DOCUMENTARY|REALITY|CHILDREN|MUSIC|LIFESTYLE|OTHER",
  "description": "string",
  "duration_minutes": "decimal",
  "production_cost": "decimal",
  "schedule": {
    "date": "YYYY-MM-DD",
    "start_time": "HH:MM:SS",
    "end_time": "HH:MM:SS",
    "status": "PLANNED",
    "repeat_indicator": false
  }
}
```

**Success Response (201 Created):**
```json
{
  "program_id": "uuid",
  "title": "string",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Error Responses:**

- **400 Bad Request (Validation Errors):**
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_TITLE",
      "field": "title",
      "message": "Title is required and must be unique among active programs."
    },
    {
      "id": "INVALID_DURATION",
      "field": "duration_minutes",
      "message": "Duration must be a positive decimal aligned to standard time slots."
    },
    {
      "id": "INVALID_GENRE",
      "field": "genre",
      "message": "Genre must be a valid television genre from DRAMA, COMEDY, NEWS, SPORTS, DOCUMENTARY, REALITY, CHILDREN, MUSIC, LIFESTYLE, or OTHER."
    }
  ]
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "A program with the same title already exists among active programs."
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "errors": [
    {
      "id": "INVALID_SCHEDULE_TIMING",
      "field": "schedule",
      "message": "Schedule start_time and end_time must be valid and align with slots."
    },
    {
      "id": "MISSING_ANCHOR_FOR_NEWS",
      "field": "staff_assignments",
      "message": "A NEWS program must have at least one ANCHOR assigned."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.1.2 Get Program
**URL:** GET /api/v1/programs/{programId}

**Success Response (200 OK):**
```json
{
  "program_id": "uuid",
  "title": "string",
  "genre": "DRAMA",
  "description": "string",
  "duration_minutes": 60,
  "production_cost": 75000,
  "schedule": {
    "date": "YYYY-MM-DD",
    "start_time": "HH:MM:SS",
    "end_time": "HH:MM:SS",
    "status": "PLANNED",
    "repeat_indicator": false
  },
  "ratings": {
    "rating_value": 8.5,
    "viewers_count": 15000
  },
  "staff_assignments": [
    { "staff_id": "uuid" }
  ],
  "advertising_slots": [
    {
      "advertiser_id": "uuid",
      "slot_duration (minutes)": 15,
      "price_paid": 2000
    }
  ],
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Program not found."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.1.3 Update Program
**URL:** PATCH /api/v1/programs/{programId}

**Request Body Schema:**
```json
{
  "title": "string",
  "description": "string",
  "duration_minutes": "decimal",
  "production_cost": "decimal",
  "schedule": {
    "date": "YYYY-MM-DD",
    "start_time": "HH:MM:SS",
    "end_time": "HH:MM:SS",
    "status": "LIVE|CANCELLED|COMPLETED",
    "repeat_indicator": true
  }
}
```

**Success Response (200 OK):**
```json
{
  "program_id": "uuid",
  "title": "string",
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
      "id": "INVALID_STATUS_TRANSITION",
      "field": "schedule.status",
      "message": "Invalid status transition."
    },
    {
      "id": "INVALID_GENRE",
      "field": "genre",
      "message": "Genre must be a valid television genre from DRAMA, COMEDY, NEWS, SPORTS, DOCUMENTARY, REALITY, CHILDREN, MUSIC, LIFESTYLE, or OTHER."
    }
  ]
}
```
- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Program not found."
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.1.4 Delete Program
**URL:** DELETE /api/v1/programs/{programId}

**Success Response:**
- **204 No Content**

**Possible Error Responses:**

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot delete a program with LIVE status or active advertising commitments."
}
```
- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Program not found."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.1.5 Bulk Delete Programs
**URL:** DELETE /api/v1/programs/bulk

**Request Body Schema:**
```json
{
  "program_ids": ["uuid", "uuid"]
}
```
**Response Codes:**:  
- 200 OK – Returns results of bulk deletion  
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "PROGRAM_NOT_FOUND"
    },
    {
      "id": "uuid",
      "reason": "PROGRAM_IN_LIVE_STATUS"
    }
  ],
  "deleted_count": 2,
  "failed_count": 2
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Program IDs are required to delete programs"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

### 5.2 Staff Management Endpoints

#### 5.2.1 Create Staff
**URL:** POST /api/v1/staff

**Request Body Schema:**
```json
{
  "first_name": "string",
  "last_name": "string",
  "email": "string",
  "phone": "string",
  "role": "PRODUCER|DIRECTOR|EDITOR|ANCHOR|MANAGER"
}
```

**Success Response (201 Created):**
```json
{
  "staff_id": "uuid",
  "email": "string",
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
      "id": "MISSING_FIRST_NAME",
      "field": "first_name",
      "message": "First name is required."
    },
    {
      "id": "MISSING_EMAIL",
      "field": "email",
      "message": "Email is required and must be unique."
    },
    {
      "id": "INVALID_PHONE_FORMAT",
      "field": "phone",
      "message": "Phone must be in E.164 format."
    },
    {
      "id": "INVALID_ROLE",
      "field": "role",
      "message": "Role must be a valid staff role from PRODUCER, DIRECTOR, EDITOR, ANCHOR, or MANAGER."
    }
  ]
}
```
- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Staff with this email already exists."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.2.2 Get Staff
**URL:** GET /api/v1/staff/{staffId}

**Success Response (200 OK):**
```json
{
  "staff_id": "uuid",
  "first_name": "string",
  "last_name": "string",
  "email": "string",
  "phone": "string",
  "role": "EDITOR",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff not found."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.2.3 Update Staff
**URL:** PATCH /api/v1/staff/{staffId}

**Request Body Schema:**
```json
{
  "first_name": "string",
  "last_name": "string",
  "email": "string",
  "phone": "string",
  "role": "PRODUCER|DIRECTOR|EDITOR|ANCHOR|MANAGER"
}
```

**Success Response (200 OK):**
```json
{
  "staff_id": "uuid",
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
      "id": "INVALID_PHONE_FORMAT",
      "field": "phone",
      "message": "Phone must be in E.164 format."
    },
    {
      "id": "INVALID_ROLE",
      "field": "role",
      "message": "Role must be a valid staff role from PRODUCER, DIRECTOR, EDITOR, ANCHOR, or MANAGER."
    }
  ]
}
```
- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff not found."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.2.4 Delete Staff
**URL:** DELETE /api/v1/staff/{staffId}

**Success Response:**
- **204 No Content**

**Possible Error Responses:**

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot delete staff member assigned to upcoming programs."
}
```
- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff not found."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```
#### 5.2.5 Bulk Delete Staff
**URL:** DELETE /api/v1/staff/bulk

**Request Body Schema:**
```json
{
  "staff_ids": ["uuid", "uuid"]
}
```
**Success Response:**
- **200 OK**
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "STAFF_NOT_FOUND"
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
  "message": "Staff IDs are required to delete staff"
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

### 5.3 Advertiser Management Endpoints

#### 5.3.1 Create Advertiser
**URL:** POST /api/v1/advertisers

**Request Body Schema:**
```json
{
  "name": "string",
  "email": "string",
  "phone": "string",
  "contract_end_date": "YYYY-MM-DD",
  "advertisement_details": {
    "target_time_slot": "HH:MM:SS",
    "pricing_tier": "PREMIUM|STANDARD|ECONOMY|PROMOTIONAL",
    "budget": "decimal"
  }
}
```

**Success Response (201 Created):**
```json
{
  "advertiser_id": "uuid",
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
      "message": "Advertiser name is required."
    },
    {
      "id": "INVALID_EMAIL_FORMAT",
      "field": "email",
      "message": "Email must be a valid email address."
    },
    {
      "id": "INVALID_PHONE_FORMAT",
      "field": "phone",
      "message": "Phone must be in E.164 format."
    }
  ]
}
```

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Advertiser with this email already exists."
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "errors": [
    {
      "id": "INSUFFICIENT_BUDGET_FOR_PREMIUM",
      "field": "advertisement_details.budget",
      "message": "Minimum budget for PREMIUM tier must be 50,000."
    },
    {
      "id": "INVALID_TIME_SLOT",
      "field": "advertisement_details.target_time_slot",
      "message": "Target time slot is invalid."
    }
  ]
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.3.2 Get Advertiser
**URL:** GET /api/v1/advertisers/{advertiserId}

**Success Response (200 OK):**
```json
{
  "advertiser_id": "uuid",
  "name": "string",
  "email": "string",
  "phone": "string",
  "contract_end_date": "YYYY-MM-DD",
  "advertisement_details": {
    "target_time_slot": "HH:MM:SS",
    "pricing_tier": "STANDARD",
    "budget": 30000
  },
  "program_placements": [
    {
  "program_id": "uuid",
      "slot_count": 2,
      "total_value": 4000
    }
  ],
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Error:**

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Advertiser not found."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.3.3 Update Advertiser
**URL:** PATCH /api/v1/advertisers/{advertiserId}

**Request Body Schema:**
```json
{
  "name": "string",
  "email": "string",
  "phone": "string",
  "contract_end_date": "YYYY-MM-DD",
  "advertisement_details": {
    "target_time_slot": "HH:MM:SS",
    "pricing_tier": "PREMIUM|STANDARD|ECONOMY|PROMOTIONAL",
    "budget": "decimal"
  }
}
```

**Success Response (200 OK):**
```json
{
  "advertiser_id": "uuid",
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
      "id": "INVALID_EMAIL_FORMAT",
      "field": "email",
      "message": "Email must be a valid email address."
    },
    {
      "id": "INVALID_PHONE_FORMAT",
      "field": "phone",
      "message": "Phone must be in E.164 format."
    },
    {
      "id": "INVALID_PRICING_TIER",
      "field": "advertisement_details.pricing_tier",
      "message": "Pricing tier must be PREMIUM, STANDARD, ECONOMY, or PROMOTIONAL."
    },
    {
      "id": "BUDGET_BELOW_MINIMUM",
      "field": "advertisement_details.budget",
      "message": "Budget for PREMIUM tier must be at least 50,000."
    }
  ]
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Advertiser not found."
}
```

- **422 Unprocessable Entity:**
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "errors": [
    {
      "id": "INVALID_TIME_SLOT",
      "field": "advertisement_details.target_time_slot",
      "message": "Target time slot is invalid."
    },
    {
      "id": "BUDGET_EXCEEDED",
      "field": "advertisement_details.budget",
      "message": "Total program placements value exceeds the specified budget."
    }
  ]
}
```

- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

#### 5.3.4 Delete Advertiser
**URL:** DELETE /api/v1/advertisers/{advertiserId}

**Success Response:**
- **204 No Content**

**Possible Error Responses:**

- **409 Conflict:**
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot delete advertiser with active contracts or scheduled advertisements."
}
```

- **404 Not Found:**
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Advertiser not found."
}
```
- **500 Internal Server Error:**
```json
{
  "error_id": "INTERNAL_SERVER_ERROR",
  "message": "An unexpected error occurred while processing the request."
}
```

## 6. Television Station Management Workflow Examples

### 6.1 Program Scheduling and Broadcast Workflow

**Steps:**
1. Create a New Program
2. Assign Required Staff
3. Allocate Advertising Slots
4. Transition Program to LIVE Status
5. Complete the Broadcast

#### Step 1: Create a New Program

**Request:**
```json
POST /api/v1/programs
Content-Type: application/json

{
  "title": "Evening News",
  "genre": "NEWS",
  "description": "Daily evening news program covering local and national events",
  "duration_minutes": 60,
  "production_cost": 25000,
  "schedule": {
    "date": "2024-06-15",
    "start_time": "19:00:00",
    "end_time": "20:00:00",
    "status": "PLANNED",
    "repeat_indicator": false
  }
}
```

**Response:** 201 Created
```json
{
  "program_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "title": "Evening News",
  "metadata": {
    "created_at": "2024-05-20T14:30:45.123Z"
  }
}
```

#### Step 2: Assign Required Staff

**Request:**
```json
PATCH /api/v1/programs/f47ac10b-58cc-4372-a567-0e02b2c3d479
Content-Type: application/json

{
  "staff_assignments": [
    {"staff_id": "a1b2c3d4-e5f6-4a5b-8c7d-9e0f1a2b3c4d"}, // PRODUCER
    {"staff_id": "b2c3d4e5-f6a5-4b8c-7d9e-0f1a2b3c4d5e"}  // ANCHOR
  ]
}
```

**Response:** 200 OK
```json
{
  "program_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "metadata": {
    "updated_at": "2024-05-20T14:35:22.456Z"
  }
}
```

**Error Scenario - Missing Required Staff Role:**

If attempting to schedule a NEWS program without an ANCHOR:

**Response:** 422 Unprocessable Entity
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "A NEWS program must have at least one ANCHOR assigned."
}
```

#### Step 3: Allocate Advertising Slots

**Request:**
```json
PATCH /api/v1/programs/f47ac10b-58cc-4372-a567-0e02b2c3d479
Content-Type: application/json

{
  "advertising_slots": [
    {
      "advertiser_id": "d4e5f6a5-b8c7-4d9e-0f1a-2b3c4d5e6f7a",
      "slot_duration (minutes)": 2,
      "price_paid": 5000
    },
    {
      "advertiser_id": "e5f6a5b8-c7d9-4e0f-1a2b-3c4d5e6f7a8b",
      "slot_duration (minutes)": 1,
      "price_paid": 2500
    }
  ]
}
```

**Response:** 200 OK
```json
{
  "program_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "metadata": {
    "updated_at": "2024-05-20T14:40:15.789Z"
  }
}
```

**Error Scenario - Advertiser Budget Exceeded:**

**Response:** 422 Unprocessable Entity
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "Advertiser e5f6a5b8-c7d9-4e0f-1a2b-3c4d5e6f7a8b has insufficient budget for this placement."
}
```

#### Step 4: Transition Program to LIVE Status

**Request:**
```json
PATCH /api/v1/programs/f47ac10b-58cc-4372-a567-0e02b2c3d479
Content-Type: application/json

{
  "schedule": {
    "status": "LIVE"
  }
}
```

**Response:** 200 OK
```json
{
  "program_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "metadata": {
    "updated_at": "2024-06-15T19:00:05.123Z"
  }
}
```

**Error Scenario - Invalid Status Transition:**

If attempting to transition directly from PLANNED to COMPLETED:

**Response:** 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Cannot transition directly from PLANNED to COMPLETED. Program must be LIVE first."
}
```

#### Step 5: Complete the Broadcast

**Request:**
```json
PATCH /api/v1/programs/f47ac10b-58cc-4372-a567-0e02b2c3d479
Content-Type: application/json

{
  "schedule": {
    "status": "COMPLETED"
  },
  "ratings": {
    "rating_value": 8.5,
    "viewers_count": 250000
  }
}
```

**Response:** 200 OK
```json
{
  "program_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "metadata": {
    "updated_at": "2024-06-15T20:00:15.456Z"
  }
}
```

### 6.2 Advertiser Management

**Steps:**
1. Create Advertiser Profile
2. Allocate Advertising Slots to Programs

#### Step 1: Create Advertiser Profile

**Request:**
```json
POST /api/v1/advertisers
Content-Type: application/json

{
  "name": "Global Electronics",
  "email": "advertising@globalelectronics.com",
  "phone": "+12025550179",
  "contract_end_date": "2024-12-31",
  "advertisement_details": {
    "target_time_slot": "19:30:00",
    "pricing_tier": "PREMIUM",
    "budget": 75000
  }
}
```

**Response:** 201 Created
```json
{
  "advertiser_id": "d4e5f6a5-b8c7-4d9e-0f1a-2b3c4d5e6f7a",
  "name": "Global Electronics",
  "metadata": {
    "created_at": "2024-05-15T10:15:30.123Z"
  }
}
```

#### Step 2: Allocate Advertising Slots to Programs

**Request:**
```
PATCH /api/v1/programs/f47ac10b-58cc-4372-a567-0e02b2c3d479
Content-Type: application/json

{
  "advertising_slots": [
    {
      "advertiser_id": "d4e5f6a5-b8c7-4d9e-0f1a-2b3c4d5e6f7a",
      "slot_duration (minutes)": 2,
      "price_paid": 5000
    }
  ]
}
```

**Response:** 200 OK
```json
{
  "program_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "metadata": {
    "updated_at": "2024-05-15T10:20:45.789Z"
  }
}
```

### 6.3 Staff Assignment and Workflow Management

**Steps:**
1. Create Staff Record
2. Assign Staff to Program

#### Step 1: Create Staff Record

**Request:**
```json
POST /api/v1/staff
Content-Type: application/json

{
  "first_name": "Jane",
  "last_name": "Smith",
  "email": "jane.smith@tvstation.com",
  "phone": "+12025550143",
  "role": "PRODUCER"
}
```

**Response:** 201 Created
```json
{
  "staff_id": "a1b2c3d4-e5f6-4a5b-8c7d-9e0f1a2b3c4d",
  "email": "jane.smith@tvstation.com",
  "metadata": {
    "created_at": "2024-05-10T09:30:15.123Z"
  }
}
```

**Error Scenario - Email Domain Validation:**

**Response:** 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_EMAIL_DOMAIN",
      "field": "email",
      "message": "Email domain must match the organization's approved domain list."
    }
  ]
}
```

#### Step 2: Assign Staff to Program

**Request:**
```json
PATCH /api/v1/programs/f47ac10b-58cc-4372-a567-0e02b2c3d479
Content-Type: application/json

{
  "staff_assignments": [
    {
      "staff_id": "a1b2c3d4-e5f6-4a5b-8c7d-9e0f1a2b3c4d"
    }
  ]
}
```

**Response:** 200 OK
```json
{
  "program_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "metadata": {
    "updated_at": "2024-05-10T09:35:45.789Z"
  }
}
```

**Error Scenario - Overlapping Assignments:**

**Response:** 409 Conflict
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "OVERLAPPING_ASSIGNMENT",
      "field": "staff_id",
      "message": "Staff member is already assigned to another program with overlapping broadcast time."
    }
  ]
}
```
