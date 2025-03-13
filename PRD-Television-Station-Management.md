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
- Complex aspects: Manage program scheduling, staff assignments, and advertiser placements. The platform supports the creation of new program genres and the management of existing ones. The platofmr also support ratings and viewership analysis.

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
      "slot_duration": "integer",
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
| `schedule.repeat_indicator` | boolean | No | Indicates if program is a rerun | Mutable | false | Flag indicating whether the broadcast is a first-run or repeat showing, important for programming strategy and audience metrics |
| `ratings.rating_value` | decimal | No | Decimal value, can be fractional with Range 0.0 to 10.0 | Mutable | None | Numerical score representing the program's viewership performance, used for success measurement and comparative analysis |
| `ratings.viewers_count` | integer | No | Number of viewers, integer | Mutable | None | Estimated total audience size for the program broadcast, critical for advertising value assessment and content decisions |
| `ratings.analysis_complete` | boolean | No | If advanced analysis conducted | Mutable | false | Flag indicating whether sophisticated analytical methods have been applied to derive deeper insights from the rating data |
| `staff_assignments` | array | No | Array of staff assignment objects | Mutable | [] | Collection of staff members assigned to the program, establishing the relationship between personnel and content |
| `staff_assignments[].staff_id` | uuid | Yes | Must reference an existing staff | Mutable | None | Reference to a staff member working on the program, enabling accountability and resource allocation tracking. Relationship: Staff |
| `advertising_slots` | array | No | Array of advertising placement objects | Mutable | [] | Collection of advertising allocations for the program, connecting advertisers to the content and managing revenue generation |
| `advertising_slots[].advertiser_id` | uuid | Yes | Must reference existing advertiser | Mutable | None | Reference to the company purchasing advertising time during the program broadcast. Relationship: Advertiser |
| `advertising_slots[].slot_duration` | integer | Yes | Duration in seconds | Mutable | None | Length of time allocated for the advertisement, used for scheduling and pricing calculations |
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
    "target_time_slot": "string",
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
| `advertisement_details.target_time_slot` | string | No | Preferred broadcast time | Mutable | None | Preferred broadcast time for advertisements, used to align with target audience viewing patterns and maximize effectiveness |
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

#### 4.1.2 Schedule Model
- `repeat_indicator` must be set to true when the same program is scheduled within a 7-day period
- `start_time` and `end_time` must be chronologically after the `start_time` on the same `date`.
- If the `status` is `LIVE`, then the `start_time` and `end_time` fields become immutable.

#### 4.1.3 Advertiser Model
- `contract_end_date` is required when `advertisement_details.pricing_tier` is provided.

#### 4.1.4 Rating Model
- Rating records with `viewers_count` exceeding 1 million require additional verification.

#### 4.1.5 Staff Model


### 4.2 State Machine Transitions

#### 4.2.1 Schedule Model
- `PLANNED` can transition to `LIVE`
- `PLANNED` can transition to `CANCELLED`
- `LIVE` can transition to `COMPLETED`
- Once a schedule is `COMPLETED` or `CANCELLED`, no further transitions are allowed

#### 4.2.2 Program Model (Implicit Lifecycle)
- `Active` (deleted_at is null) can transition to `Archived` (deleted_at is set)
- `Archived` can transition to `Active` (when deleted_at is cleared)

#### 4.2.3 Advertiser Model (Implicit Lifecycle)
- `Active` (deleted_at is null) can transition to `Archived` (deleted_at is set)
- `Archived` can transition to `Active` only when all contract details are reviewed and revalidated

#### 4.2.4 Rating Model (Implicit Lifecycle)
- `Active` (deleted_at is null) can transition to `Archived` (deleted_at is set)
- `Archived` ratings cannot be reactivated for data integrity purposes

#### 4.2.5 Staff Model (Implicit Lifecycle)
- `Active` (deleted_at is null) can transition to `Archived` (deleted_at is set)
- `Archived` can transition to `Active` when a staff member returns from extended leave

### 4.3 Cross-Resource Validation Rules

#### 4.3.1 Program-Schedule Validation
- Schedule `date`, `start_time`, and `end_time` must accommodate the Program's `duration_minutes`
- Program `genre` must be appropriate for the scheduled time slot according to regulatory guidelines
- When a Program is archived, any associated Schedules must be either `COMPLETED` or `CANCELLED`
- A Program cannot be scheduled more than 3 times in a 24-hour period unless `repeat_indicator` is true

#### 4.3.2 Advertiser-Schedule Validation
- Advertiser `advertisement_details.target_time_slot` must be compatible with available Schedule slots
- Ad placements from competing Advertisers in the same `industry` must maintain minimum separation
- When Advertiser status changes to `Archived`, all pending ad placements must be automatically reassigned
- Advertisers with `contract_end_date` in the past cannot have new ad placements scheduled

#### 4.3.3 Schedule-Rating Validation
- Rating `date` must match a Schedule `date` for the associated Program
- Rating records cannot be created for Schedules in `PLANNED` or `CANCELLED` status
- A Schedule can have multiple Rating records to represent different measurement methodologies
- If the Schedule is `CANCELLED`, any associated Rating data must be flagged as invalid

#### 4.3.4 Staff-Program Validation
- Staff with production roles must be authorized to create and modify Programs
- Staff with scheduling roles must be authorized to create and modify Schedules
- A Staff member cannot be assigned to more than 5 simultaneous productions
- Staff role must be appropriate for the assigned responsibilities

### 4.4 Multi-Step Operations

#### 4.4.1 Program Broadcast Process
1. Create Program with complete metadata
2. Schedule the Program for broadcast
3. Transition Schedule to `LIVE` at broadcast time
4. Transition Schedule to `COMPLETED` after broadcast ends
5. Collect and associate Rating data

Rollback Scenarios:
- Technical difficulties: Cancel broadcast and reschedule
- Content issues: Replace with alternative programming
- Schedule conflict: Adjust time slot or move to different date

#### 4.4.2 Advertiser Campaign Management
1. Create Advertiser profile with contact information
2. Define contract terms including `contract_end_date`
3. Set `advertisement_details.target_time_slot` preferences
4. Allocate time slots in Schedule
5. Associate advertising content with allocated slots
6. Review campaign performance via Rating data

Rollback Scenarios:
- Contract dispute: Suspend campaign and renegotiate terms
- Content rejection: Request revised content from advertiser
- Schedule changes: Reallocate to alternative time slots

#### 4.4.3 Schedule Optimization Process
1. Create initial Schedule based on programming requirements
2. Analyze historical Rating data for optimal time slots
3. Adjust Schedule to maximize viewer engagement
4. Integrate Advertiser requirements and preferences
5. Finalize optimized Schedule

Rollback Scenarios:
- Unexpected conflicts: Revert to previous schedule version
- Regulatory issues: Adjust to comply with broadcast standards
- Advertiser objections: Re-optimize with additional constraints

### 4.5 Business Logic Triggers

#### 4.5.1 Status Change Actions
- When Schedule status changes to `COMPLETED`, Rating data collection is automatically initiated
- When Program metadata is updated, all associated `PLANNED` Schedules are flagged for review
- When Advertiser is archived, all pending ad placements are automatically flagged for reassignment
- When Staff role changes, all assigned responsibilities are automatically reviewed for compatibility

#### 4.5.2 Automatic Updates
- Moving Schedule to `COMPLETED` automatically updates `metadata.updated_at`
- Creating a new Program automatically sets `metadata.created_at` to current timestamp
- Updating any Advertiser field automatically updates `metadata.updated_at`
- Setting any resource's `deleted_at` field automatically excludes it from standard API responses

#### 4.5.3 Cascading Actions
- When a Program is archived, all future Schedules are flagged for review or cancellation
- When an Advertiser's `contract_end_date` passes, all ad placements are automatically ended
- When a Staff member is archived, all assigned responsibilities are automatically reassigned
- When a Schedule is cancelled, all associated ad placements are automatically rescheduled

### 4.6 Deletion Behavior

#### 4.6.1 Program Model

**Prevention Criteria:**
- Cannot delete a Program referenced by any Schedule with status `PLANNED` or `LIVE`
- Cannot delete a Program that has active advertising contracts associated with it
- Cannot delete a Program with significant historical Rating data (over 1 million viewers)

**Cascade Effects:**
- All future Schedules referencing the Program are flagged for review
- Rating data is preserved for historical analysis

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Program is excluded from future scheduling options
- Program metadata is preserved for archival purposes

#### 4.6.2 Schedule Model

**Prevention Criteria:**
- Cannot delete a Schedule if `status` is `LIVE`
- Cannot delete a Schedule if it has active Advertiser placements
- Cannot delete a Schedule within 24 hours of its `start_time` unless status is `CANCELLED`

**Cascade Effects:**
- Associated Rating data is flagged as "Schedule Deleted"
- Advertiser placements are automatically rescheduled

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Schedule is removed from broadcast planning tools
- Historical broadcast data is preserved for reporting

#### 4.6.3 Advertiser Model

**Prevention Criteria:**
- Cannot delete an Advertiser if current date is before `contract_end_date`
- Cannot delete an Advertiser with active ad placements in upcoming Schedules
- Cannot delete an Advertiser with outstanding financial transactions

**Cascade Effects:**
- All ad placements are marked for review
- Contact information is archived according to data retention policies

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Advertiser is excluded from new campaign opportunities
- Historical advertising data is preserved for reporting and analytics

#### 4.6.4 Rating Model

**Prevention Criteria:**
- Cannot delete Rating data that is less than 7 days old
- Cannot delete Rating data used in active analytics or reports
- Cannot delete Rating data for Programs that are still in active rotation

**Cascade Effects:**
- Associated analytical reports are flagged for recalculation
- Historical trends are preserved for long-term analysis

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Rating is excluded from current performance metrics
- Historical data is preserved for trend analysis

#### 4.6.5 Staff Model

**Prevention Criteria:**
- Cannot delete Staff records for individuals with active production assignments
- Cannot delete Staff records referenced in current broadcasts
- Cannot delete Staff records with administrative system access

**Cascade Effects:**
- Production responsibilities are reassigned
- System access is revoked
- Contact information is archived

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Staff member is removed from assignment options
- Employment history is preserved for compliance purposes

## 5. Program, Schedule, Advertiser, Rating, and Staff API Endpoints

─────────────────────────────────────────
5.1 Program Endpoints

─────────────────────────────────────────
5.1.1 Create Program
URL: POST /api/v1/programs

Request Body Schema:
```json
{
  "program_id": "uuid",              // Required, Unique, Immutable
  "title": "string",                 // Required, Descriptive title
  "genre": "string",                 // Required, Categorization of program
  "description": "string",           // Optional, up to 5000 characters
  "duration_minutes": "decimal"      // Required, Positive decimal representing duration
}
```

Response Codes:
201 Created: Program successfully created
```json
{
  "program_id": "uuid",
  "title": "string",
  "created_at": "timestamp"
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 409 Conflict
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Program already exists or conflicts with constraints"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.1.2 Get Program
URL: GET /api/v1/programs/{programId}

Response Codes:
200 OK: Returns program details
```json
{
  "program_id": "uuid",
  "title": "string",
  "genre": "string",
  "description": "string",
  "duration_minutes": "decimal",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "The requested program does not exist"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.1.3 Update Program
URL: PATCH /api/v1/programs/{programId}

Request Body Schema:
```json
{
  "title": "string",                 // Optional, cannot update program_id or metadata.created_at
  "genre": "string",                 // Optional
  "description": "string",           // Optional
  "duration_minutes": "decimal"      // Optional
}
```

Response Codes:
200 OK: Program successfully updated
```json
{
  "program_id": "uuid",
  "title": "string",
  "genre": "string",
  "description": "string",
  "duration_minutes": "decimal",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Program not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.1.4 Delete Program
URL: DELETE /api/v1/programs/{programId}

Response Codes:
204 No Content: Program successfully deleted (soft delete)

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "DELETE_PREVENTED",
  "message": "Program deletion prevented due to associated LIVE or COMPLETED schedules"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Program not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.1.5 Bulk Delete Programs
URL: DELETE /api/v1/programs

Request Body Schema:
```json
{
  "program_ids": ["uuid", "uuid", "uuid"]
}
```

Response Codes:
200 OK: Returns results of bulk deletion operation
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "PROGRAM_HAS_ACTIVE_SCHEDULES"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 403 Forbidden
```json
{
  "error_id": "INSUFFICIENT_PERMISSIONS",
  "message": "Bulk deletion is not permitted"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.2 Schedule Endpoints

─────────────────────────────────────────
5.2.1 Create Schedule
URL: POST /api/v1/schedules

Request Body Schema:
```json
{
  "schedule_id": "uuid",             // Required, Unique, Immutable
  "program_id": "uuid",              // Required, Must reference an existing program
  "date": "date",                    // Required, Valid date
  "start_time": "time",              // Required, Format HH:MM:SS
  "end_time": "time",                // Required, Format HH:MM:SS
  "status": "PLANNED|LIVE|COMPLETED|CANCELLED",  // Required, Default: PLANNED
}
```

Response Codes:
201 Created: Schedule successfully created
```json
{
  "schedule_id": "uuid",
  "program_id": "uuid",
  "created_at": "timestamp"
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 409 Conflict
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Schedule conflicts with existing records or violates constraints"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Referenced program not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.2.2 Get Schedule
URL: GET /api/v1/schedules/{scheduleId}

Response Codes:
200 OK: Returns schedule details
```json
{
  "schedule_id": "uuid",
  "program_id": "uuid",
  "date": "date",
  "start_time": "time",
  "end_time": "time",
  "status": "PLANNED|LIVE|COMPLETED|CANCELLED",
  "complex_aspects": {
    "schedule_optimization": "boolean"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Schedule not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.2.3 Update Schedule
URL: PATCH /api/v1/schedules/{scheduleId}

Request Body Schema:
```json
{
  "date": "date",                    // Optional
  "start_time": "time",              // Optional
  "end_time": "time",                // Optional
  "status": "PLANNED|LIVE|COMPLETED|CANCELLED",  // Optional (Note: Transition rules apply: PLANNED → LIVE → COMPLETED or CANCELLED)
  "complex_aspects": {
    "schedule_optimization": "boolean"         // Optional
  }
}
```

Response Codes:
200 OK: Schedule successfully updated
```json
{
  "schedule_id": "uuid",
  "program_id": "uuid",
  "date": "date",
  "start_time": "time",
  "end_time": "time",
  "status": "PLANNED|LIVE|COMPLETED|CANCELLED",
  "complex_aspects": {
    "schedule_optimization": "boolean"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed or state transition is invalid"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Schedule not found"
}
```
• 409 Conflict
```json
{
  "error_id": "INVALID_STATE_TRANSITION",
  "message": "Cannot transition schedule to the requested state"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.2.4 Delete Schedule
URL: DELETE /api/v1/schedules/{scheduleId}

Response Codes:
204 No Content: Schedule successfully deleted (soft delete)

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "DELETE_PREVENTED",
  "message": "Schedule deletion prevented in LIVE or COMPLETED state"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Schedule not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.2.5 Bulk Delete Schedules
URL: DELETE /api/v1/schedules

Request Body Schema:
```json
{
  "schedule_ids": ["uuid", "uuid", "uuid"]
}
```

Response Codes:
200 OK: Returns results of bulk deletion operation
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "SCHEDULE_IN_NON_DELETABLE_STATE"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 403 Forbidden
```json
{
  "error_id": "INSUFFICIENT_PERMISSIONS",
  "message": "Bulk deletion not permitted"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.3 Advertiser Endpoints

─────────────────────────────────────────
5.3.1 Create Advertiser
URL: POST /api/v1/advertisers

Request Body Schema:
```json
{
  "advertiser_id": "uuid",          // Required, Unique, Immutable
  "name": "string",                 // Required, Full name of advertiser
  "email": "string",                // Required, Valid email format
  "phone": "string",                // Required, E.164 format
  "contract_end_date": "date",     // Required, End date of advertising contract
  "advertisement_details": {
    "target_time_slot": "string",   // Required, target time slot for ad placement
    "pricing_tier": "PREMIUM|STANDARD|ECONOMY|PROMOTIONAL",  // Required, must be one of the specified values
    "budget": "decimal"              // Required, must be positive if provided
  }
}
```

Response Codes:
201 Created: Advertiser successfully created
```json
{
  "advertiser_id": "uuid",
  "name": "string",
  "created_at": "timestamp"
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 409 Conflict
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Advertiser already exists or violates constraints"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.3.2 Get Advertiser
URL: GET /api/v1/advertisers/{advertiserId}

Response Codes:
200 OK: Returns advertiser details
```json
{
  "advertiser_id": "uuid",
  "name": "string",
  "email": "string",
  "phone": "string",
  "contract_end_date": "date",
  "advertisement_details": {
    "target_time_slot": "string",
    "pricing_tier": "PREMIUM|STANDARD|ECONOMY|PROMOTIONAL",
    "budget": "decimal"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Advertiser not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.3.3 Update Advertiser
URL: PATCH /api/v1/advertisers/{advertiserId}

Request Body Schema:
```json
{
  "name": "string",                 // Optional
  "email": "string",                // Optional
  "phone": "string",                // Optional
  "contract_end_date": "date",       // Optional
  "advertisement_details": {
    "target_time_slot": "string",   // Optional, should align with valid schedule time slots
    "pricing_tier": "PREMIUM|STANDARD|ECONOMY|PROMOTIONAL",  // Optional
    "budget": "decimal"              // Optional
  }
}
```

Response Codes:
200 OK: Advertiser successfully updated
```json
{
  "advertiser_id": "uuid",
  "name": "string",
  "email": "string",
  "phone": "string",
  "contract_end_date": "date",
  "advertisement_details": {
    "target_time_slot": "string",
    "pricing_tier": "PREMIUM|STANDARD|ECONOMY|PROMOTIONAL",
    "budget": "decimal"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Advertiser not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.3.4 Delete Advertiser
URL: DELETE /api/v1/advertisers/{advertiserId}

Response Codes:
204 No Content: Advertiser successfully deleted (soft delete)

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "DELETE_PREVENTED",
  "message": "Advertiser deletion prevented due to upcoming scheduled ad placements"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Advertiser not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.3.5 Bulk Delete Advertisers
URL: DELETE /api/v1/advertisers

Request Body Schema:
```json
{
  "advertiser_ids": ["uuid", "uuid", "uuid"]
}
```

Response Codes:
200 OK: Returns results of bulk deletion operation
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "UPCOMING_AD_PLACEMENT"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 403 Forbidden
```json
{
  "error_id": "INSUFFICIENT_PERMISSIONS",
  "message": "Bulk deletion not permitted"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.4 Rating Endpoints

─────────────────────────────────────────
5.4.1 Create Rating
URL: POST /api/v1/ratings

Request Body Schema:
```json
{
  "rating_id": "uuid",             // Required, Unique, Immutable
  "program_id": "uuid",            // Required, Must reference an existing program
  "date": "date",                  // Required, Date of rating
  "rating_value": "decimal",         // Required, Can be fractional
  "viewers_count": "integer",      // Required, Number of viewers
  "analysis_complete": "boolean"  // Optional, Default: false
}
```

Response Codes:
201 Created: Rating successfully created
```json
{
  "rating_id": "uuid",
  "program_id": "uuid",
  "created_at": "timestamp"
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 409 Conflict
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Rating conflicts with existing records"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Referenced program not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.4.2 Get Rating
URL: GET /api/v1/ratings/{ratingId}

Response Codes:
200 OK: Returns rating details
```json
{
  "rating_id": "uuid",
  "program_id": "uuid",
  "date": "date",
  "rating_value": "decimal",
  "viewers_count": "integer",
  "analysis_complete": "boolean",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" // optional (archived)
  }
}
```

Possible Errors:
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Rating not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.4.3 Update Rating
URL: PATCH /api/v1/ratings/{ratingId}

Request Body Schema:
```json
{
  "date": "date",                   // Optional
  "rating_value": "decimal",          // Optional
  "viewers_count": "integer",          // Optional
  "analysis_complete": "boolean"       // Optional
}
```

Response Codes:
200 OK: Rating successfully updated
```json
{
  "rating_id": "uuid",
  "program_id": "uuid",
  "date": "date",
  "rating_value": "decimal",
  "viewers_count": "integer",
  "analysis_complete": "boolean",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Update validation failed"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Rating not found"
}
```
• 409 Conflict
```json
{
  "error_id": "INVALID_STATE_TRANSITION",
  "message": "Rating cannot be updated in its archived state"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.4.4 Delete Rating
URL: DELETE /api/v1/ratings/{ratingId}

Response Codes:
204 No Content: Rating successfully archived (soft delete; actual deletion is prevented)

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "DELETE_PREVENTED",
  "message": "Ratings cannot be permanently deleted; use archive instead"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Rating not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.4.5 Bulk Delete Ratings
URL: DELETE /api/v1/ratings

Request Body Schema:
```json
{
  "rating_ids": ["uuid", "uuid", "uuid"]
}
```

Response Codes:
200 OK: Returns results of bulk deletion (archival) operation
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "RATING_ALREADY_ARCHIVED"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 403 Forbidden
```json
{
  "error_id": "INSUFFICIENT_PERMISSIONS",
  "message": "Bulk archival is not permitted"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.5 Staff Endpoints

─────────────────────────────────────────
5.5.1 Create Staff
URL: POST /api/v1/staff

Request Body Schema:
```json
{
  "staff_id": "uuid",              // Required, Unique, Immutable
  "first_name": "string",          // Required, Characters only
  "last_name": "string",           // Required, Characters only
  "role": "PRODUCER|DIRECTOR|EDITOR|ANCHOR|MANAGER",  // Required, Position or role assigned
  "email": "string",               // Required, Valid professional email
  "phone": "string",               // Optional, E.164 format
  "program_history": [
    {
      "program_id": "uuid"
    }
  ]
}
```

Response Codes:
201 Created: Staff successfully created
```json
{
  "staff_id": "uuid",
  "first_name": "string",
  "created_at": "timestamp"
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 409 Conflict
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Staff member already exists or violates constraints"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.5.2 Get Staff
URL: GET /api/v1/staff/{staffId}

Response Codes:
200 OK: Returns staff details
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
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff member not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.5.3 Update Staff
URL: PATCH /api/v1/staff/{staffId}

Request Body Schema:
```json
{
  "first_name": "string",          // Optional
  "last_name": "string",           // Optional
  "role": "PRODUCER|DIRECTOR|EDITOR|ANCHOR|MANAGER",  // Optional
  "email": "string",               // Optional
  "phone": "string",               // Optional
  "program_history": [
    {
      "program_id": "uuid"
    }
  ]
}
```

Response Codes:
200 OK: Staff successfully updated
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
    "deleted_at": "timestamp" // optional
  }
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff member not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.5.4 Delete Staff
URL: DELETE /api/v1/staff/{staffId}

Response Codes:
204 No Content: Staff successfully deleted (soft delete)

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "DELETE_PREVENTED",
  "message": "Staff deletion is prevented for archived records"
}
```
• 404 Not Found
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff member not found"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

─────────────────────────────────────────
5.5.5 Bulk Delete Staff
URL: DELETE /api/v1/staff

Request Body Schema:
```json
{
  "staff_ids": ["uuid", "uuid", "uuid"]
}
```

Response Codes:
200 OK: Returns results of bulk deletion operation
```json
{
  "successful_deletions": ["uuid", "uuid"],
  "failed_deletions": [
    {
      "id": "uuid",
      "reason": "STAFF_ARCHIVED_OR_IN_USE"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

Possible Errors:
• 400 Bad Request
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Request validation failed"
}
```
• 403 Forbidden
```json
{
  "error_id": "INSUFFICIENT_PERMISSIONS",
  "message": "Bulk deletion not permitted"
}
```
• 500 Internal Server Error
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```
