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
- Complex aspects: Manage program scheduling, staff assignments, and advertiser placements. The platform supports the creation of new program genres and the management of existing ones. The platform also support ratings and viewership analysis.

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
| `ratings.analysis_complete` | boolean | No | If advanced analysis conducted | Mutable | false | Flag indicating whether sophisticated analytical methods have been applied to derive deeper insights from the rating data |
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
