# Broadcast News Room - Product Requirements Document

# 1. Introduction

## 1.1 Purpose

This document outlines the requirements for the Broadcast News Room API, which provides a comprehensive set of endpoints for managing broadcast news, including story creation, reporter assignment, equipment allocation, and broadcast scheduling.

## 1.2 Scope

The API's are designed to support the operations of a broadcast news room, including story creation, reporter assignment, equipment allocation, and broadcast scheduling. It is intended for use by broadcast news organizations and authorized external partners.

## 2. Global API Specifications

### 2.1 Core Requirements:
- HTTP only (no HTTPS)
- No authentication required
- No pagination
- No concurrency handling
- No rate-limiting
- No performance requirements
- Resource IDs should be UUIDs
- Handle complex aspects like story assignment, equipment allocation, and broadcast planning appropriately
- When a resource ID is invalid or not found, always return 404 (no validation of UUID format)
- Resources: stories, staff, equipment, feeds, broadcasts
- Complex aspects: Manage story assignment, equipment allocation, and broadcast planning. The platform should also support the creation of new equipment types and the management of existing ones. The platform should also support the creation of new story categories and the management of existing ones.

### 2.2 Base URL Configuration
- **Exact Base URL**: `/api/v1`
- **Versioning Strategy**: Explicit version in URL to support future API evolution
- **Global URL Pattern**:
  - Parent Resources: `/{resource-type}` (for resources like stories, staff, equipment, feeds, broadcasts)
  - Child/Nested Resources: `/{parent-resource}/{parent-id}/{child-resource}`

### 2.3 Timestamp Management
- **Timestamp Type**: Server-managed
- **Timestamp Fields**:  
  - `created_at`: Immutable creation timestamp
  - `updated_at`: Dynamically updated on any resource modification
- **Format**: ISO 8601 Extended Format (e.g., "2024-02-03T15:30:45.123Z")
- **Time Zone**: Always stored in UTC
- **Precision**: Millisecond-level accuracy

### 2.4 Error Response Specification
**Detailed Error Response Schema**:
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

|

{
  "error_id": "ERROR_CODE",
  "message": "ERROR_MESSAGE"
}
```

**Error Handling Principles**:
- Always return appropriate HTTP status codes
- Include unique error identifiers for log tracking
- Maintain consistency across all endpoints

## 3. Comprehensive Resource Models

### 3.1 Story Model
```json
{
  "story_id": "uuid",
  "title": "string",
  "content": "string",
  "category": "string",
  "status": "DRAFT|PUBLISHED|ARCHIVED",
  "assignment": {
    "assigned_staff_id": "uuid",
    "priority": "LOW|HIGH",
    "due_date": "date"
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
|-------|------|----------|-------------|------------|---------|--------------|
| `story_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the story that remains consistent throughout its lifecycle, used in all API operations and cross-references from other resources |
| `title` | string | Yes | Min length: 1 | Mutable | None | Headline or primary title of the news story that will appear in broadcasts, feeds, and content management systems |
| `content` | string | Yes | Min length: 1 | Mutable | None | Full narrative text of the news story including all details, quotes, and contextual information needed for broadcast |
| `category` | string | No | Min length: 1 | Mutable | None | Editorial classification (e.g., "Politics", "Sports", "Weather") used for content organization, search filtering, and audience targeting |
| `status` | enum | Yes | Allowed values: [`DRAFT`, `PUBLISHED`, `ARCHIVED`] | Mutable | `DRAFT` | Current workflow state of the story that determines its visibility, editability, and availability for broadcast |
| `assignment.assigned_staff_id` | uuid | Yes | Must reference existing staff member | Mutable | None | Identifier of the staff member responsible for creating, editing, or managing this story, enabling accountability and workflow tracking. Relationship: Staff |
| `assignment.priority` | enum | No | Allowed values: [`LOW`, `HIGH`] | Mutable | `LOW` | Editorial importance level that affects scheduling, resource allocation, and deadline enforcement |
| `assignment.due_date` | date | No | Valid future date with format YYYY-MM-DD | Mutable | None | Calendar deadline for story completion that drives workflow scheduling and resource planning |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the story was first created in the system, used for audit trails and chronological ordering |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any story field, used for change tracking and synchronization |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | null | When populated, indicates the story has been soft-deleted and should be excluded from active operations while retained for archival purposes |


### 3.2 Staff Model
```json
{
  "staff_id": "uuid",
  "first_name": "string",
  "last_name": "string",
  "email": "string",
  "phone": "string",
  "experience_years": "integer",
  "role": "REPORTER|PRODUCER|CAMERA_OPERATOR|EDITOR|OTHER",
  "status": "ACTIVE|INACTIVE",
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
| `staff_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the staff member used in all assignments, equipment allocations, and broadcast planning |
| `first_name` | string | Yes | Min length: 1 | Mutable | None | Staff member's given name used for identification in the system and potentially in broadcast credits |
| `last_name` | string | Yes | Min length: 1 | Mutable | None | Staff member's family name used for identification in the system and potentially in broadcast credits |
| `email` | string | Yes | Valid email format | Mutable | None | Primary electronic contact address for the staff member used for system notifications and communications |
| `phone` | string | No | E.164 format | Mutable | None | Contact number for urgent communications, field coordination, and emergency notifications |
| `experience_years` | integer | No | Non-negative integer | Mutable | 0 | Quantified professional experience that may influence assignment eligibility, equipment authorization, and role assignments |
| `role` | enum | Yes | Allowed values: [`REPORTER`, `PRODUCER`, `CAMERA_OPERATOR`, `EDITOR`, `OTHER`] | Mutable | `REPORTER` | Primary functional position that determines workflow responsibilities, equipment access, and assignment eligibility |
| `status` | enum | Yes | Allowed values: [`ACTIVE`, `INACTIVE`] | Mutable | `ACTIVE` | Current availability state that controls whether the staff member can be assigned to stories or broadcasts |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the staff record was created, used for employment records and system audits |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any staff field, used for personnel record tracking |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | null | When populated, indicates the staff record has been soft-deleted while preserving historical assignments and broadcasts |

### 3.3 Equipment Model
```json
{
  "equipment_id": "uuid",
  "type": "CAMERA|MICROPHONE|LIGHTING|DRONE|OTHER",
  "model": "string",
  "status": "AVAILABLE|IN_USE|MAINTENANCE|RETIRED",
  "last_maintenance_date": "timestamp",
  "allocations": [
    {
      "allocated_to_story_id": "uuid",
      "allocated_to_staff_id": "uuid",
      "allocation_date": "date"
    }
  ],
  "rental_cost": "decimal",
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
| `equipment_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the equipment item used in all allocations, maintenance records, and inventory management |
| `type` | enum | Yes | Allowed values: [`CAMERA`, `MICROPHONE`, `LIGHTING`, `DRONE`, `OTHER`] | Mutable | `OTHER` | Technical classification that determines usage scenarios, staff allocation rules, and maintenance schedules |
| `model` | string | Yes | Min length: 1 | Mutable | None | Manufacturer's model designation that identifies specific capabilities, compatibility, and replacement parts |
| `status` | enum | Yes | Allowed values: [`AVAILABLE`, `IN_USE`, `MAINTENANCE`, `RETIRED`] | Mutable | `AVAILABLE` | Current operational state that controls whether the equipment can be allocated to staff or stories |
| `last_maintenance_date` | timestamp | No | Valid past date with format YYYY-MM-DD | Mutable | null | Most recent service date used to schedule preventive maintenance and track equipment reliability |
| `allocations` | array | No | Array of allocation objects | Mutable | [] | List of allocations for this equipment, enabling resource tracking and historical usage records |
| `allocations[].allocated_to_story_id` | uuid | Yes (in allocation) | Must reference existing story | Mutable | None | Identifier of the story using this equipment, enabling resource tracking and content association. Relationship: Story |
| `allocations[].allocated_to_staff_id` | uuid | Yes (in allocation) | Must reference existing staff member | Mutable | None | Identifier of the staff member responsible for this equipment, establishing accountability and usage tracking. Relationship: Staff |
| `allocations[].allocation_date` | date | Yes (in allocation) | Valid date with format YYYY-MM-DD | Mutable | None | Date when the equipment is allocated, used for resource planning and scheduling |
| `rental_cost` | decimal | No | Non-negative decimal | Mutable | 0.00 | Daily or hourly cost value for budget tracking, client billing, and financial reporting |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Timestamp when the equipment was added to inventory, used for asset lifecycle management |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any equipment field, used for change tracking |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | null | When populated, indicates the equipment has been removed from active inventory while preserving historical usage records |


### 3.4 Feed Model
```json
{
  "feed_id": "uuid",
  "name": "string",
  "description": "string",
  "status": "ACTIVE|INACTIVE",
  "audience_metrics": {
    "average_viewers": "integer",
    "peak_viewers": "integer"
  },
  "configuration": {
    "update_interval": "integer",
    "format": "JSON|XML|RSS"
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
|-------|------|----------|-------------|------------|---------|--------------|
| `feed_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the distribution channel used in broadcast scheduling and content delivery |
| `name` | string | Yes | Min length: 1 | Mutable | None | Human-readable identifier for the feed used in broadcast planning, scheduling interfaces, and reports |
| `description` | string | No | Min length: 1 | Mutable | None | Detailed explanation of the feed's purpose, audience, and technical characteristics for operational reference |
| `status` | enum | Yes | Allowed values: [`ACTIVE`, `INACTIVE`] | Mutable | `ACTIVE` | Current operational state that determines whether broadcasts can be scheduled on this feed |
| `audience_metrics.average_viewers` | integer | No | Non-negative integer | Mutable | 0 | Typical audience size based on historical data, used for advertising, content planning, and performance analysis |
| `audience_metrics.peak_viewers` | integer | No | Non-negative integer | Mutable | 0 | Maximum audience size during high-interest broadcasts, used for capacity planning and performance benchmarking |
| `configuration.update_interval` | integer | No | Positive integer | Mutable | None | Frequency (in seconds) at which the feed refreshes content, critical for technical synchronization and scheduling |
| `configuration.format` | enum | No | Allowed values: [`JSON`, `XML`, `RSS`] | Mutable | `JSON` | Data structure format for digital feeds, determining compatibility with downstream systems and integration requirements |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the feed was established in the system, used for lifecycle management |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any feed field, used for configuration tracking |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | null | When populated, indicates the feed has been decommissioned while preserving historical broadcast records |


### 3.5 Broadcast Model
```json
{
  "broadcast_id": "uuid",
  "title": "string",
  "story_id": "uuid",
  "feed_id": "uuid",
  "scheduled_start": "timestamp",
  "scheduled_end": "timestamp",
  "status": "SCHEDULED|LIVE|COMPLETED|CANCELLED",
  "planning_details": {
    "assigned_staff_ids": [
      "uuid"
    ],
    "location": "string",
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
|-------|------|----------|-------------|------------|---------|--------------|
| `broadcast_id` | uuid | Yes | Immutable unique identifier | Never | None | Primary identifier for the broadcast event used in scheduling, resource allocation, and performance analytics |
| `title` | string | Yes | Min length: 1 | Mutable | None | Public-facing name of the broadcast that appears in program guides, schedules, and viewer interfaces |
| `story_id` | uuid | Yes | Must reference existing story | Never | None | Identifier of the primary news story being broadcast, establishing the content-to-broadcast relationship. Relationship: Story |
| `feed_id` | uuid | Yes | Must reference existing feed | Never | None | Identifier of the distribution channel used for this broadcast, determining audience reach and technical requirements. Relationship: Feed  |
| `scheduled_start` | timestamp | Yes | Must be future moment with ISO 8601 format YYYY-MM-DDThh:mm:ss.sssZ | Mutable | None | Planned commencement time that drives scheduling, resource allocation, and promotional activities |
| `scheduled_end` | timestamp | Yes | Must occur after scheduled_start with ISO 8601 format YYYY-MM-DDThh:mm:ss.sssZ | Mutable | None | Anticipated conclusion time that affects subsequent scheduling, resource release, and broadcast duration metrics |
| `status` | enum | Yes | Allowed values: [`SCHEDULED`, `LIVE`, `COMPLETED`, `CANCELLED`] | Mutable | `SCHEDULED` | Current operational state of the broadcast that controls workflow actions, resource allocation, and system behavior |
| `planning_details.assigned_staff_ids` | array of uuid | Yes | Must reference existing staff | Mutable | [] | List of staff members required for this broadcast, enabling personnel scheduling and resource planning. Relationship: Staff |
| `planning_details.location` | string | No | Min length: 1 | Mutable | None | Physical or virtual setting where the broadcast will originate, critical for logistics and technical setup |
| `planning_details.notes` | string | No | Max length: 2000 | Mutable | None | Production instructions, special requirements, or contextual information for the broadcast team |
| `metadata.created_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, set on creation | Never | Current time | Precise moment when the broadcast was scheduled in the system, used for planning chronology |
| `metadata.updated_at` | timestamp | Yes | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Updates on any change | Mutable | Current time | Timestamp of the most recent modification to any broadcast field, used for change tracking |
| `metadata.deleted_at` | timestamp | No | ISO 8601 format (YYYY-MM-DDThh:mm:ss.sssZ), UTC timezone, Valid date if deleted | Mutable | null | When populated, indicates the broadcast has been removed from schedules while preserving historical records |



## 4. Business Rules

### 4.1 Conditional Field Requirements

#### 4.1.1 Story Model
- `assignment.due_date` is required when `assignment.priority` is `HIGH`
- `content` must be non-empty when `status` is `PUBLISHED` or `ARCHIVED`

#### 4.1.2 Staff Model
- `email` must be unique across all staff members
- `phone` must be provided when `status` is `ACTIVE`

#### 4.1.3 Equipment Model
- A valid `allocation_date` is required for each allocation in the `allocations` array
- For any date with an allocation, the equipment `status` will be `IN_USE` for that entire day
- Cannot have multiple allocations on the same date

#### 4.1.4 Feed Model
- If `peak_viewers` is provided, then `average_viewers` must be less than or equal to `peak_viewers`.

#### 4.1.5 Broadcast Model
- `scheduled_end` must be after `scheduled_start`
- `story_id` must reference a story with status `PUBLISHED` when broadcast status is `LIVE`

### 4.2 State Machine Transitions

#### 4.2.1 Story Model
- `DRAFT` can transition to `PUBLISHED`
- `DRAFT` can transition to `ARCHIVED`
- `PUBLISHED` can transition to `ARCHIVED`
- Once a story is `ARCHIVED`, no further transitions are allowed

#### 4.2.2 Staff Model
- `ACTIVE` can transition to `INACTIVE`
- `INACTIVE` can transition to `ACTIVE`
- Transitions between `ACTIVE` and `INACTIVE` are bidirectional when conditions are met

#### 4.2.3 Equipment Model
- `AVAILABLE` can transition to `IN_USE` (automatically happens when an allocation's date is the current date)
- `IN_USE` can transition to `MAINTENANCE`
- `MAINTENANCE` can transition to `IN_USE`
- `MAINTENANCE` can transition to `RETIRED`
- `IN_USE` can transition to `AVAILABLE` (automatically happens when there are no allocations for the current date)
- Once equipment is `RETIRED`, no further transitions are allowed

#### 4.2.4 Feed Model
- `ACTIVE` can transition to `INACTIVE`
- `INACTIVE` can transition to `ACTIVE`
- Transitions between `ACTIVE` and `INACTIVE` are bidirectional

#### 4.2.5 Broadcast Model
- `SCHEDULED` can transition to `LIVE`
- `LIVE` can transition to `COMPLETED`
- `SCHEDULED` can transition to `CANCELLED`
- `LIVE` can transition to `CANCELLED`
- Once a broadcast is `COMPLETED` or `CANCELLED`, no further transitions are allowed

### 4.3 Cross-Resource Validation Rules

#### 4.3.1 Story-Broadcast Validation
- Broadcast `scheduled_start` must be after Story `assignment.due_date`
- Story `status` must be `PUBLISHED` before associated broadcast can go `LIVE`
- When a Story is archived, any associated Broadcasts must be either `COMPLETED` or `CANCELLED`
- A Story cannot be referenced by more than 10 active Broadcasts simultaneously

#### 4.3.2 Staff-Equipment Allocation Validation
- `allocated_to_staff_id` in Equipment allocations must reference a Staff member with `status` = `ACTIVE`
- Equipment allocation to a Staff member requires the Staff member to have appropriate role for the equipment type, e.g. `REPORTER` for Camera, `PRODUCER` for Microphone, `EDITOR` for Lighting, etc.
- When Staff status changes to `INACTIVE`, all Equipment allocations for that staff member must be automatically removed
- Equipment cannot have overlapping allocations for the same date

#### 4.3.3 Feed-Broadcast Validation
- Broadcast `feed_id` must reference a Feed with `status` = `ACTIVE`
- A Feed cannot be referenced by more than one `LIVE` Broadcast at the same time
- When Feed status changes to `INACTIVE`, any associated `SCHEDULED` Broadcasts must be automatically cancelled
- If the Broadcast is `LIVE`, the Feed must be `ACTIVE`.

#### 4.3.4 Story-Staff Validation
- Story `assignment.assigned_staff_id` must reference a Staff member with `status` = "ACTIVE"
- A Staff member can only be assigned to a maximum of 15 "DRAFT" Stories simultaneously
- Staff role must be appropriate for the Story assignment (e.g., "REPORTER" role for story creation)

#### 4.3.5 Broadcast-Equipment Validation
- If `IN_USE` Equipment moves to `MAINTENANCE` then associated staff will be notified and allocation will be removed.

### 4.4 Multi-Step Operations

#### 4.4.1 Story Publication Process
1. Assign a staff member to the Story
2. Set publication priority based on editorial guidelines
3. Complete content requirements
4. Move status to "PUBLISHED"

Rollback Scenarios:
- Assigned staff unavailable: Reassign to another staff member
- Content requirements not met: Can assign to another staff member or change priority.

#### 4.4.2 Broadcast Execution
1. Assign required staff members to the Broadcast
2. Allocate necessary Equipment to assigned Staff
3. Verify Feed availability
4. Change Broadcast status to `LIVE`
5. Change status to `COMPLETED` post-broadcast review.

Rollback Scenarios:
- Equipment failure: Reschedule broadcast or find replacement equipment
- Staff unavailability: Reassign roles or reschedule broadcast
- Feed issues: Switch to backup feed or reschedule broadcast
- Technical difficulties: Cancel broadcast if issues cannot be resolved

#### 4.4.3 Equipment Maintenance Process
1. Change Equipment status from "AVAILABLE" or "IN_USE" to "MAINTENANCE"
2. De-allocate Equipment from any Staff or Story
3. Record maintenance details and update `last_maintenance_date`
4. Return Equipment to `AVAILABLE` status after maintenance is complete.

Rollback Scenarios:
- Failed maintenance: Keep in "MAINTENANCE" status until resolved
- Irreparable equipment: Change status to "RETIRED"

### 4.5 Business Logic Triggers

#### 4.5.1 Status Change Notifications
- Story status changes (`Story.status`) notify all assigned staff (`assignment.assigned_staff_id`)
- Broadcast status changes (`Broadcast.status`) alert all associated staff (`planning_details.assigned_staff_ids`)
- Equipment status changes to `MAINTENANCE` or `RETIRED` notify technical staff
- Feed status changes to `INACTIVE` notify broadcast planning team

#### 4.5.2 Automatic Updates
- For each equipment, if the current date matches any `allocation_date` in the `allocations` array, the equipment `status` is automatically set to `IN_USE`
- For each equipment, if the current date does not match any `allocation_date` in the `allocations` array and the `status` is `IN_USE`, the status is automatically set to `AVAILABLE`
- Moving Broadcast to `COMPLETED` automatically updates `metadata.updated_at`
- Setting Story `status` to `ARCHIVED` automatically sets `metadata.deleted_at`
- Changing Equipment status to `IN_USE` automatically sets `allocation.allocation_date` to current timestamp
- Changing Staff status to `INACTIVE` automatically clears all assignments and equipment allocations

#### 4.5.3 Cascading Actions
- When a Story is archived, all associated Equipment allocations are automatically released
- When a Broadcast is cancelled, all assigned Equipment is automatically de-allocated
- When a Feed becomes inactive, all scheduled broadcasts using that feed are flagged for review
- When Equipment is set to `RETIRED`, any pending maintenance records are automatically closed

### 4.6 Deletion Behavior

#### 4.6.1 Story Model

**Prevention Criteria:**
- Cannot delete a Story referenced by any Broadcast with status `SCHEDULED` or `LIVE`

**Cascade Effects:**
- All Equipment allocations referencing the Story are automatically de-allocated
- Story content is archived according to retention policies

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Story content is preserved for archival purposes but not accessible through standard API endpoints

#### 4.6.2 Staff Model

**Prevention Criteria:**
- Cannot delete a Staff record if `status` is `ACTIVE`
- Cannot delete a Staff record assigned to any upcoming Broadcast

**Cascade Effects:**
- Equipment allocations are automatically de-allocated
- Story assignments are reassigned or flagged for review

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Staff record is excluded from assignment options

#### 4.6.3 Equipment Model

**Prevention Criteria:**
- Cannot delete Equipment if `status` is `IN_USE`
- Cannot delete Equipment allocated for future dates
- Cannot delete Equipment scheduled for use in upcoming Broadcasts
- Cannot delete Equipment with pending maintenance records

**Cascade Effects:**
- All allocation history is preserved for record-keeping
- Maintenance records are archived

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Equipment is removed from available inventory
- Historical usage data is preserved for reporting

#### 4.6.4 Feed Model

**Prevention Criteria:**
- Cannot delete a Feed if `status` is `ACTIVE`

**Cascade Effects:**
- All scheduled Broadcasts using this Feed are flagged for review
- Historical audience metrics are preserved
- Feed configuration data is archived

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Feed is removed from available broadcast options
- Historical broadcast data is preserved with proper references

#### 4.6.5 Broadcast Model

**Prevention Criteria:**
- Cannot delete a Broadcast if `status` is `LIVE`
- Cannot delete a Broadcast if `status` is `SCHEDULED` and `scheduled_start` is in the future.

**Cascade Effects:**
- Staff assignments are cleared
- Audience metrics are finalized and archived

**Effects When Deletion Succeeds:**
- `deleted_at` timestamp is set to the current UTC time
- Historical broadcast data is preserved for compliance and reporting

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

**Regulatory Compliance:**

- Deletion of certain resources may require regulatory notification
- Critical infrastructure deletions may require additional approval workflow
- Mass deletions (more than 10 resources of the same type within 24 hours) require higher level authorization

## 5. Newsroom API Endpoints

### 5.1 Story Management Endpoints

#### 5.1.1 Create Story  
URL: POST /api/v1/stories

**Request Body Schema:**  
```json
{
  "title": "string",                // Required, Min length: 1
  "content": "string",              // Required, Min length: 1
  "category": "string",             // Optional, Min length: 1
  "status": "DRAFT|PUBLISHED|ARCHIVED",  // Required, Default: DRAFT
  "assignment": {
    "assigned_staff_id": "uuid",    // Required
    "priority": "LOW|HIGH",  // Optional, Default: LOW
    "due_date": "date"              // Conditionally required if priority is "HIGH" (Valid future date)
  }
}
```
**Response Codes:**:  
- 201 Created – Story successfully created  
```json
{
  "story_id": "uuid",
  "title": "string",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_TITLE",
      "field": "title",
      "message": "Title is required"
    },
    {
      "id": "MISSING_CONTENT",
      "field": "content",
      "message": "Content is required"
    },
    {
      "id": "MISSING_ASSIGNMENT",
      "field": "assignment.assigned_staff_id",
      "message": "Staff assignment is required for stories."
    },
    {
      "id": "INVALID_STATUS",
      "field": "status",
      "message": "Status provided is invalid, must be one of the following: DRAFT|PUBLISHED|ARCHIVED"
    }
  ]
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "TITLE_CONFLICT",
      "field": "title",
      "message": "Story already exists for this title"
    },
    {
      "id": "INVALID_STATUS_TRANSITION",
      "field": "status",
      "message": "Invalid status transition"
    }
  ]
}
```

- 422 Unprocessable Entity  
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "errors": [
    {
      "id": "INVALID_ASSIGNMENT_STAFF_ID",
      "field": "assignment.assigned_staff_id",
      "message": "Referenced staff member does not exist or is inactive"
    },
    {
      "id": "INVALID_ASSIGNMENT_DUE_DATE",
      "field": "assignment.due_date",
      "message": "assignment.due_date is required when assignment.priority is HIGH"
    }
  ]
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.1.2 Get Story  
URL: GET /api/v1/stories/{storyId}

**Response Codes:**:  
- 200 OK – Returns story details  
```json
{
  "story_id": "uuid",
  "title": "string",
  "content": "string",
  "category": "string",
  "status": "DRAFT|PUBLISHED|ARCHIVED",
  "assignment": {
    "assigned_staff_id": "uuid",
    "priority": "LOW|HIGH",
    "due_date": "date"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Story not found"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.1.3 Update Story  
URL: PATCH /api/v1/stories/{storyId}

**Request Body Schema:**  
```json
{
  "title": "string",
  "content": "string",
  "category": "string",
  "status": "DRAFT|PUBLISHED|ARCHIVED",
  "assignment": {
    "assigned_staff_id": "uuid",
    "priority": "LOW|HIGH",
    "due_date": "date"
  }
}
```

**Response Codes:**:  
- 200 OK – Story successfully updated  
```json
{
  "story_id": "uuid",
  "title": "string",
  "content": "string",
  "category": "string",
  "status": "DRAFT|PUBLISHED|ARCHIVED",
  "assignment": {
    "assigned_staff_id": "uuid",
    "priority": "LOW|HIGH",
    "due_date": "date"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_STATUS",
      "field": "status",
      "message": "Status provided is invalid, must be one of the following: DRAFT|PUBLISHED|ARCHIVED"
    },
    {
      "id": "INVALID_PRIORITY",
      "field": "assignment.priority",
      "message": "Priority provided is invalid, must be one of the following: LOW|HIGH"
    }
  ]
}
```  
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Story not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot transition from PUBLISHED to DRAFT"
}
```  
- 422 Unprocessable Entity  
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "errors": [
    {
      "id": "INVALID_ASSIGNMENT_STAFF_ID",
      "field": "assignment.assigned_staff_id",
      "message": "Referenced staff member does not exist or is inactive"
    },
    {
      "id": "INVALID_ASSIGNMENT_DUE_DATE",
      "field": "assignment.due_date",
      "message": "assignment.due_date is required when assignment.priority is HIGH"
    }
  ]
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.1.4 Delete Story  
URL: DELETE /api/v1/stories/{storyId}

**Response Codes:**:  
- 204 No Content – Story successfully deleted (soft delete)

**Possible Errors:**
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Story not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Story is referenced by active broadcasts and cannot be deleted"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.1.5 Bulk Delete Stories  
URL: DELETE /api/v1/stories

**Request Body Schema:**  
```json
{
  "story_ids": ["uuid", "uuid", "uuid"] // Required, Array of Story IDs to delete
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
      "reason": "STORY_NOT_ARCHIVED"
    },
    {
      "id": "uuid",
      "reason": "STORY_IN_USE"
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
  "message": "Story IDs are required to delete stories"
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
URL: POST /api/v1/staff

**Request Body Schema:**  
```json
{
  "first_name": "string",         // Required, Min length: 1
  "last_name": "string",          // Required, Min length: 1
  "email": "string",              // Required, Valid email format
  "phone": "string",              // Optional, E.164 format
  "experience_years": "integer",  // Optional, Non-negative integer, Default: 0
  "role": "REPORTER|PRODUCER|CAMERA_OPERATOR|EDITOR|OTHER", // Required, Default: "REPORTER"
  "status": "ACTIVE|INACTIVE"     // Required, Default: ACTIVE
}
```

**Response Codes:**:  
- 201 Created – Staff successfully created  
```json
{
  "staff_id": "uuid",
  "email": "string",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_FIRST_NAME",
      "field": "first_name",
      "message": "First name is required"
    },
    {
      "id": "MISSING_LAST_NAME",
      "field": "last_name",
      "message": "Last name is required"
    },
    {
      "id": "MISSING_EMAIL",
      "field": "email",
      "message": "Email is required"
    },
    {
      "id": "INVALID_PHONE_FORMAT",
      "field": "phone",
      "message": "Phone number must be in E.164 format"
    },
    {
      "id": "INVALID_ROLE",
      "field": "role",
      "message": "Role provided is invalid"
    }
  ]
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Staff with this email already exists"
}
```
- 422 Unprocessable Entity
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "Status is changed to active but phone is not provided"
}
```

- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.2.2 Get Staff  
URL: GET /api/v1/staff/{staffId}

**Response Codes:**:  
- 200 OK – Returns staff details  
```json
{
  "staff_id": "uuid",
  "first_name": "string",
  "last_name": "string",
  "email": "string",
  "phone": "string",
  "experience_years": "integer",
  "role": "REPORTER|PRODUCER|CAMERA_OPERATOR|EDITOR|OTHER",
  "status": "ACTIVE|INACTIVE",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff not found"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.2.3 Update Staff  
URL: PATCH /api/v1/staff/{staffId}

**Request Body Schema:**  
```json
{
  "first_name": "string",         // Optional
  "last_name": "string",          // Optional
  "email": "string",              // Optional
  "phone": "string",              // Conditionally required if status becomes "ACTIVE"
  "experience_years": "integer",  // Optional
  "role": "REPORTER|PRODUCER|CAMERA_OPERATOR|EDITOR|OTHER", // Optional
  "status": "ACTIVE|INACTIVE"     // Optional;
}
```

**Response Codes:**:  
- 200 OK – Staff successfully updated  
```json
{
  "staff_id": "uuid",
  "first_name": "string",
  "last_name": "string",
  "email": "string",
  "phone": "string",
  "experience_years": "integer",
  "role": "REPORTER|PRODUCER|CAMERA_OPERATOR|EDITOR|OTHER",
  "status": "ACTIVE|INACTIVE",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_PHONE_FORMAT",
      "field": "phone",
      "message": "Phone number must be in E.164 format"
    },
    {
      "id": "MISSING_EMAIL",
      "field": "email",
      "message": "Email is required when status is ACTIVE"
    },
    {
      "id": "INVALID_ROLE",
      "field": "role",
      "message": "Role provided is invalid"
    }
  ]
}
```  
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff not found"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.2.4 Delete Staff  
URL: DELETE /api/v1/staff/{staffId}

**Response Codes:**:  
- 204 No Content – Staff successfully deleted (soft delete)

**Possible Errors:** 
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Staff not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Staff is linked to a live broadcast"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.2.5 Bulk Delete Staff  
URL: DELETE /api/v1/staff

**Request Body Schema:**  
```json
{
  "staff_ids": ["uuid", "uuid", "uuid"] // Required, Array of Staff IDs to delete
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
      "reason": "STAFF_ACTIVE_OR_REFERENCED"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

**Possible Errors:** 
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Staff IDs are required to delete staff"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

### 5.3 Equipment Management Endpoints

#### 5.3.1 Create Equipment  
URL: POST /api/v1/equipment

**Request Body Schema:**  
```json
{
  "type": "CAMERA|MICROPHONE|LIGHTING|DRONE|OTHER", 
  "model": "string",                                  
  "status": "AVAILABLE|IN_USE|MAINTENANCE|RETIRED",  
  "last_maintenance_date": "timestamp",               
  "allocations": [                                    
    {
      "allocated_to_story_id": "uuid",                
      "allocated_to_staff_id": "uuid",            
      "allocation_date": "date"                     
    }
  ],
  "rental_cost": "decimal"                          
}
```

**Response Codes:**:  
- 201 Created – Equipment successfully created  
```json
{
  "equipment_id": "uuid",
  "model": "string",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_MODEL",
      "field": "model",
      "message": "Model is required"
    },
    {
      "id": "MISSING_ALLOCATION_STORY_ID",
      "field": "allocations[0].allocated_to_story_id",
      "message": "Story ID is required for each allocation"
    },
    {
      "id": "MISSING_ALLOCATION_STAFF_ID",
      "field": "allocations[0].allocated_to_staff_id",
      "message": "Staff ID is required for each allocation"
    },
    {
      "id": "INVALID_STATUS",
      "field": "status",
      "message": "Status provided is invalid, must be one of the following: AVAILABLE|IN_USE|MAINTENANCE|RETIRED"
    }
  ]
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "DUPLICATE_ALLOCATION_DATE",
      "field": "allocations",
      "message": "Cannot have multiple allocations on the same date"
    },
    {
      "id": "EQUIPMENT_STATUS_CONFLICT",
      "field": "status",
      "message": "Equipment status conflicts with allocations"
    }
  ]
}
```  
- 422 Unprocessable Entity  
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "errors": [
    {
      "id": "INVALID_ALLOCATION_STAFF_ID",
      "field": "allocations[0].allocated_to_staff_id",
      "message": "Referenced staff member does not exist or is inactive"
    },
    {
      "id": "INVALID_ALLOCATION_STORY_ID",
      "field": "allocations[0].allocated_to_story_id",
      "message": "Referenced story does not exist"
    }
  ]
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.3.2 Get Equipment  
URL: GET /api/v1/equipment/{equipmentId}

**Response Codes:**:  
- 200 OK – Returns equipment details  
```json
{
  "equipment_id": "uuid",
  "type": "CAMERA|MICROPHONE|LIGHTING|DRONE|OTHER",
  "model": "string",
  "status": "AVAILABLE|IN_USE|MAINTENANCE|RETIRED",
  "last_maintenance_date": "timestamp",
  "allocations": [
    {
      "allocated_to_story_id": "uuid",
      "allocated_to_staff_id": "uuid",
      "allocation_date": "date"
    }
  ],
  "rental_cost": "decimal",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Equipment not found"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.3.3 Update Equipment  
URL: PATCH /api/v1/equipment/{equipmentId}

**Request Body Schema:**  
```json
{
  "type": "CAMERA|MICROPHONE|LIGHTING|DRONE|OTHER", 
  "model": "string",                                 
  "status": "AVAILABLE|IN_USE|MAINTENANCE|RETIRED",  
  "last_maintenance_date": "timestamp",            
  "allocations": [                               
    {
      "allocated_to_story_id": "uuid",               
      "allocated_to_staff_id": "uuid",         
      "allocation_date": "date"                   
    }
  ],
  "rental_cost": "decimal"                           
}
```

**Response Codes:**:  
- 200 OK – Equipment successfully updated  
```json
{
  "equipment_id": "uuid",
  "type": "CAMERA|MICROPHONE|LIGHTING|DRONE|OTHER",
  "model": "string",
  "status": "AVAILABLE|IN_USE|MAINTENANCE|RETIRED",
  "last_maintenance_date": "timestamp",
  "allocations": [
    {
      "allocated_to_story_id": "uuid",
      "allocated_to_staff_id": "uuid",
      "allocation_date": "date"
    }
  ],
  "rental_cost": "decimal",
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_STATUS",
      "field": "status",
      "message": "Status provided is invalid"
    },
    {
      "id": "INVALID_MAINTENANCE_DATE",
      "field": "last_maintenance_date",
      "message": "Maintenance date must be in the past"
    },
    {
      "id": "MISSING_ALLOCATION_STORY_ID",
      "field": "allocations[0].allocated_to_story_id",
      "message": "Story ID is required for each allocation"
    },
    {
      "id": "MISSING_ALLOCATION_STAFF_ID",
      "field": "allocations[0].allocated_to_staff_id",
      "message": "Staff ID is required for each allocation"
    }
  ]
}
```
- 403 Forbidden
```json
{
  "error_id": "FORBIDDEN",
  "message": "Allocated staff does not have the role to access this equipment"
}
```
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Equipment not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "DUPLICATE_ALLOCATION_DATE",
      "field": "allocations",
      "message": "Cannot have multiple allocations on the same date"
    },
    {
      "id": "EQUIPMENT_STATUS_CONFLICT",
      "field": "status",
      "message": "Cannot change status to RETIRED with existing allocations"
    },
    {
      "id": "INVALID_STATUS_TRANSITION", 
      "field": "status",
      "message": "Invalid status transition, cannot transition to `RETIRED` before doing the maintenance"
    }
  ]
}
```
- 422 Unprocessable Entity  
```json
{
  "error_id": "INVALID_RELATIONSHIP",
  "message": "Referenced staff member does not exist or is inactive"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.3.4 Delete Equipment  
URL: DELETE /api/v1/equipment/{equipmentId}

**Response Codes:**:  
- 204 No Content – Equipment successfully deleted (soft delete)

**Possible Errors:**
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Equipment not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "errors": [
    {
      "id": "EQUIPMENT_IN_USE",
      "field": "status",
      "message": "Equipment is currently in use and cannot be deleted"
    },
    {
      "id": "EQUIPMENT_FUTURE_ALLOCATIONS",
      "field": "allocations",
      "message": "Equipment has future allocations and cannot be deleted"
    },
    {
      "id": "EQUIPMENT_IN_MAINTENANCE",
      "field": "status",
      "message": "Equipment in maintenance and cannot be deleted"
    }
  ]
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.3.5 Bulk Delete Equipment  
URL: DELETE /api/v1/equipment

**Request Body Schema:**  
```json
{
  "equipment_ids": ["uuid", "uuid", "uuid"] // Required, Array of Equipment IDs to delete
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
      "reason": "EQUIPMENT_IN_USE"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Equipment IDs are required to delete equipment"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

### 5.4 Feed Management Endpoints

#### 5.4.1 Create Feed  
URL: POST /api/v1/feeds

**Request Body Schema:**  
```json
{
  "name": "string",               // Required, Min length: 1, Unique
  "description": "string",        // Optional, Min length: 1
  "status": "ACTIVE|INACTIVE",    // Required, Default: ACTIVE
  "audience_metrics": {
    "average_viewers": "integer", // Optional, Non-negative integer
    "peak_viewers": "integer"     // Optional, Non-negative integer
  },
  "configuration": {
    "update_interval": "integer", // Conditionally required when status is "ACTIVE", Positive integer
    "format": "JSON|XML|RSS"      // Optional, Default: JSON
  }
}
```

**Response Codes:**:  
- 201 Created – Feed successfully created  
```json
{
  "feed_id": "uuid",
  "name": "string",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_NAME",
      "field": "name",
      "message": "Name is required"
    },
    {
      "id": "MISSING_UPDATE_INTERVAL",
      "field": "configuration.update_interval",
      "message": "Update interval is required when status is ACTIVE"
    },
    {
      "id": "INVALID_STATUS",
      "field": "status",
      "message": "Status provided is invalid, must be one of the following: ACTIVE|INACTIVE"
    }
  ]
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Feed with this name already exists"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.4.2 Get Feed  
URL: GET /api/v1/feeds/{feedId}

**Response Codes:**:  
- 200 OK – Returns feed details  
```json
{
  "feed_id": "uuid",
  "name": "string",
  "description": "string",
  "status": "ACTIVE|INACTIVE",
  "audience_metrics": {
    "average_viewers": "integer",
    "peak_viewers": "integer"
  },
  "configuration": {
    "update_interval": "integer",
    "format": "JSON|XML|RSS"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Feed not found"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.4.3 Update Feed  
URL: PATCH /api/v1/feeds/{feedId}

**Request Body Schema:**  
```json
{
  "name": "string",               // Optional
  "description": "string",        // Optional
  "status": "ACTIVE|INACTIVE",    // Optional
  "audience_metrics": {
    "average_viewers": "integer", // Optional
    "peak_viewers": "integer"     // Optional
  },
  "configuration": {              
    "update_interval": "integer", // Conditionally required when status becomes "ACTIVE"
    "format": "JSON|XML|RSS"      // Optional
  }
}
```

**Response Codes:**:  
- 200 OK – Feed successfully updated  
```json
{
  "feed_id": "uuid",
  "name": "string",
  "description": "string",
  "status": "ACTIVE|INACTIVE",
  "audience_metrics": {
    "average_viewers": "integer",
    "peak_viewers": "integer"
  },
  "configuration": {
    "update_interval": "integer",
    "format": "JSON|XML|RSS"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_UPDATE_INTERVAL",
      "field": "configuration.update_interval",
      "message": "Update interval is required when status is ACTIVE"
    },
    {
      "id": "INVALID_STATUS",
      "field": "status",
      "message": "Status provided is invalid, must be one of the following: ACTIVE|INACTIVE"
    }
  ]
}
```  
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Feed not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Feed with this name already exists"
}
```
- 422 Unprocessable Entity  
```json
{
  "error_id": "UNPROCESSABLE_ENTITY",
  "message": "Average viewers must be less than or equal to peak viewers"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.4.4 Delete Feed  
URL: DELETE /api/v1/feeds/{feedId}

**Response Codes:**:  
- 204 No Content – Feed successfully deleted (soft delete)

**Possible Errors:**
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Feed not found"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.4.5 Bulk Delete Feeds  
URL: DELETE /api/v1/feeds

**Request Body Schema:**  
```json
{
  "feed_ids": ["uuid", "uuid", "uuid"] // Required, Array of Feed IDs to delete
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
      "reason": "FEED_ACTIVE_OR_REFERENCED"
    }
  ],
  "deleted_count": 2,
  "failed_count": 1
}
```

**Possible Errors:** 
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "message": "Feed IDs are required to delete feeds"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

### 5.5 Broadcast Management Endpoints

#### 5.5.1 Create Broadcast  
URL: POST /api/v1/broadcasts

**Request Body Schema:**  
```json
{
  "title": "string",                      // Required, Min length: 1
  "story_id": "uuid",                     // Required, Must reference an existing story (Story.status must be "PUBLISHED" to later go LIVE)
  "feed_id": "uuid",                      // Required, Must reference an active feed
  "scheduled_start": "timestamp",         // Required, Must be a future moment and after associated Story.metadata.created_at
  "scheduled_end": "timestamp",           // Required, Must occur after scheduled_start
  "status": "SCHEDULED|LIVE|COMPLETED|CANCELLED",  // Required, Default: SCHEDULED
  "planning_details": {
    "assigned_staff_ids": ["uuid"],       // Required
    "location": "string",                 // Optional, Min length: 1
    "notes": "string"                     // Optional, Max length: 2000
  }
}
```

**Response Codes:**:  
- 201 Created – Broadcast successfully created  
```json
{
  "broadcast_id": "uuid",
  "title": "string",
  "metadata": {
    "created_at": "timestamp"
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "MISSING_TITLE",
      "field": "title",
      "message": "Title is required"
    },
    {
      "id": "MISSING_ASSIGNED_STAFF_IDS",
      "field": "planning_details.assigned_staff_ids",
      "message": "Staff assignments are required for broadcasts"
    },
    {
      "id": "INVALID_STATUS",
      "field": "status",
      "message": "Status provided is invalid, must be one of the following: SCHEDULED|LIVE|COMPLETED|CANCELLED"
    }
  ]
}
```  
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Referenced story or feed not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Broadcast scheduling conflict or invalid state transition"
}
```  
- 422 Unprocessable Entity  
```json
{
  "error_id": "INVALID_RELATIONSHIP",
  "errors": [
    {
      "id": "INACTIVE_FEED",
      "field": "feed_id",
      "message": "Referenced feed is inactive"
    },
    {
      "id": "UNPUBLISHED_STORY",
      "field": "story_id",
      "message": "Story must be PUBLISHED for broadcast to go LIVE"
    }
  ]
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.5.2 Get Broadcast  
URL: GET /api/v1/broadcasts/{broadcastId}

**Response Codes:**:  
- 200 OK – Returns broadcast details  
```json
{
  "broadcast_id": "uuid",
  "title": "string",
  "story_id": "uuid",
  "feed_id": "uuid",
  "scheduled_start": "timestamp",
  "scheduled_end": "timestamp",
  "status": "SCHEDULED|LIVE|COMPLETED|CANCELLED",
  "planning_details": {
    "assigned_staff_ids": ["uuid"],
    "location": "string",
    "notes": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Broadcast not found"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.5.3 Update Broadcast  
URL: PATCH /api/v1/broadcasts/{broadcastId}

**Request Body Schema:**  
```json
{
  "title": "string",                    
  "scheduled_start": "timestamp",        
  "scheduled_end": "timestamp",     
  "status": "SCHEDULED|LIVE|COMPLETED|CANCELLED",
  "planning_details": {
    "assigned_staff_ids": ["uuid"],      
    "location": "string",              
    "notes": "string"              
  }
}
```

**Response Codes:**:  
- 200 OK – Broadcast successfully updated  
```json
{
  "broadcast_id": "uuid",
  "title": "string",
  "story_id": "uuid",
  "feed_id": "uuid",
  "scheduled_start": "timestamp",
  "scheduled_end": "timestamp",
  "status": "SCHEDULED|LIVE|COMPLETED|CANCELLED",
  "planning_details": {
    "assigned_staff_ids": ["uuid"],
    "location": "string",
    "notes": "string"
  },
  "metadata": {
    "created_at": "timestamp",
    "updated_at": "timestamp",
    "deleted_at": null
  }
}
```

**Possible Errors:**
- 400 Bad Request  
```json
{
  "error_id": "VALIDATION_ERROR",
  "errors": [
    {
      "id": "INVALID_STATUS",
      "field": "status",
      "message": "Status provided is invalid"
    },
    {
      "id": "INVALID_SCHEDULED_START",
      "field": "scheduled_start",
      "message": "Scheduled start must be in the future"
    }
  ]
}
```  
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Broadcast not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Cannot transition from COMPLETED to any other status"
}
```  
- 422 Unprocessable Entity  
```json
{
  "error_id": "INVALID_RELATIONSHIP",
  "message": "Referenced staff members do not exist or are inactive"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.5.4 Delete Broadcast  
URL: DELETE /api/v1/broadcasts/{broadcastId}

**Response Codes:**:  
- 204 No Content – Broadcast successfully deleted (soft delete)

**Possible Errors:**  
- 404 Not Found  
```json
{
  "error_id": "RESOURCE_NOT_FOUND",
  "message": "Broadcast not found"
}
```  
- 409 Conflict  
```json
{
  "error_id": "RESOURCE_CONFLICT",
  "message": "Broadcast deletion prevented if status is LIVE or associated resources are active"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

#### 5.5.5 Bulk Delete Broadcasts  
URL: DELETE /api/v1/broadcasts

**Request Body Schema:**  
```json
{
  "broadcast_ids": ["uuid", "uuid", "uuid"] // Required, Array of Broadcast IDs to delete
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
      "reason": "BROADCAST_LIVE"
    },
    {
      "id": "uuid",
      "reason": "BROADCAST_SCHEDULED_FUTURE"
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
  "message": "Broadcast IDs are required to delete broadcasts"
}
```  
- 500 Internal Server Error  
```json
{
  "error_id": "SERVER_ERROR",
  "message": "An unexpected error occurred"
}
```

## 6. Broadcast News Room Workflow Examples

### 6.1 Story Creation and Broadcasting Workflow

**Steps:**
1. Create a Story (Draft)
2. Update Story Content and Publish
3. Schedule a Broadcast for the Story
4. Allocate Equipment for the Broadcast
5. Start the Broadcast (Go Live)
6. Complete the Broadcast

#### Step 1: Create a Story (Draft)

**Request:**
```http
POST /api/v1/stories
Content-Type: application/json

{
  "title": "Breaking: New Climate Policy Announced",
  "content": "The government has announced a new climate policy that aims to reduce carbon emissions by 30% by 2030.",
  "category": "Politics",
  "status": "DRAFT",
  "assignment": {
    "assigned_staff_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "priority": "HIGH",
    "due_date": "2023-06-15"
  }
}
```

**Response:**
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "story_id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Breaking: New Climate Policy Announced",
  "metadata": {
    "created_at": "2023-06-10T14:30:45.123Z"
  }
}
```

#### Step 2: Update Story Content and Publish

**Request:**
```http
PATCH /api/v1/stories/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "content": "The government has announced a new climate policy that aims to reduce carbon emissions by 30% by 2030. The policy includes incentives for renewable energy and carbon taxes for heavy polluters.",
  "status": "PUBLISHED"
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "story_id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Breaking: New Climate Policy Announced",
  "content": "The government has announced a new climate policy that aims to reduce carbon emissions by 30% by 2030. The policy includes incentives for renewable energy and carbon taxes for heavy polluters.",
  "category": "Politics",
  "status": "PUBLISHED",
  "assignment": {
    "assigned_staff_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
    "priority": "HIGH",
    "due_date": "2023-06-15"
  },
  "metadata": {
    "created_at": "2023-06-10T14:30:45.123Z",
    "updated_at": "2023-06-12T09:15:22.456Z",
    "deleted_at": null
  }
}
```

#### Step 3: Schedule a Broadcast for the Story

**Request:**
```http
POST /api/v1/broadcasts
Content-Type: application/json

{
  "title": "Evening News: Climate Policy Special",
  "story_id": "550e8400-e29b-41d4-a716-446655440000",
  "feed_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "scheduled_start": "2023-06-15T19:00:00.000Z",
  "scheduled_end": "2023-06-15T19:30:00.000Z",
  "status": "SCHEDULED",
  "planning_details": {
    "assigned_staff_ids": [
      "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "a1b2c3d4-e5f6-4a5b-8c7d-9e0f1a2b3c4d"
    ],
    "location": "Studio A",
    "notes": "Prepare graphics showing emission reduction targets"
  }
}
```

**Response:**
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "broadcast_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "title": "Evening News: Climate Policy Special",
  "metadata": {
    "created_at": "2023-06-13T10:45:33.789Z"
  }
}
```

#### Step 4: Allocate Equipment for the Broadcast

**Request:**
```http
POST /api/v1/equipment/3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t/allocate
Content-Type: application/json

{
  "allocated_to_staff_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
  "allocated_to_story_id": "550e8400-e29b-41d4-a716-446655440000",
  "allocation_date": "2023-06-15"
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "equipment_id": "3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t",
  "status": "AVAILABLE",
  "allocations": [
    {
      "allocated_to_staff_id": "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "allocated_to_story_id": "550e8400-e29b-41d4-a716-446655440000",
      "allocation_date": "2023-06-15"
    }
  ],
  "metadata": {
    "updated_at": "2023-06-14T08:30:15.456Z"
  }
}
```

#### Step 5: Start the Broadcast (Go Live)

**Request:**
```http
PATCH /api/v1/broadcasts/6ba7b810-9dad-11d1-80b4-00c04fd430c8
Content-Type: application/json

{
  "status": "LIVE"
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "broadcast_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "title": "Evening News: Climate Policy Special",
  "story_id": "550e8400-e29b-41d4-a716-446655440000",
  "feed_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "scheduled_start": "2023-06-15T19:00:00.000Z",
  "scheduled_end": "2023-06-15T19:30:00.000Z",
  "status": "LIVE",
  "planning_details": {
    "assigned_staff_ids": [
      "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "a1b2c3d4-e5f6-4a5b-8c7d-9e0f1a2b3c4d"
    ],
    "location": "Studio A",
    "notes": "Prepare graphics showing emission reduction targets"
  },
  "metadata": {
    "created_at": "2023-06-13T10:45:33.789Z",
    "updated_at": "2023-06-15T19:00:05.123Z",
    "deleted_at": null
  }
}
```

#### Step 6: Complete the Broadcast

**Request:**
```http
PATCH /api/v1/broadcasts/6ba7b810-9dad-11d1-80b4-00c04fd430c8
Content-Type: application/json

{
  "status": "COMPLETED"
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "broadcast_id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
  "title": "Evening News: Climate Policy Special",
  "story_id": "550e8400-e29b-41d4-a716-446655440000",
  "feed_id": "7c9e6679-7425-40de-944b-e07fc1f90ae7",
  "scheduled_start": "2023-06-15T19:00:00.000Z",
  "scheduled_end": "2023-06-15T19:30:00.000Z",
  "status": "COMPLETED",
  "planning_details": {
    "assigned_staff_ids": [
      "f47ac10b-58cc-4372-a567-0e02b2c3d479",
      "a1b2c3d4-e5f6-4a5b-8c7d-9e0f1a2b3c4d"
    ],
    "location": "Studio A",
    "notes": "Prepare graphics showing emission reduction targets"
  },
  "metadata": {
    "created_at": "2023-06-13T10:45:33.789Z",
    "updated_at": "2023-06-15T19:32:10.456Z",
    "deleted_at": null
  }
}
```

### 6.2 Equipment Maintenance Workflow

**Steps:**
1. Change Equipment Status to Maintenance
2. Update Maintenance Information
3. Return Equipment to Available Status

#### Step 1: Change Equipment Status to Maintenance

**Request:**
```http
PATCH /api/v1/equipment/3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t
Content-Type: application/json

{
  "status": "MAINTENANCE"
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "equipment_id": "3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t",
  "type": "CAMERA",
  "model": "Sony PXW-Z190",
  "status": "MAINTENANCE",
  "last_maintenance_date": null,
  "allocations": [],
  "rental_cost": 150.00,
  "metadata": {
    "created_at": "2023-05-01T09:00:00.000Z",
    "updated_at": "2023-06-20T11:15:22.456Z",
    "deleted_at": null
  }
}
```

#### Step 2: Update Maintenance Information

**Request:**
```http
PATCH /api/v1/equipment/3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t
Content-Type: application/json

{
  "last_maintenance_date": "2023-06-20T11:30:00.000Z"
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "equipment_id": "3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t",
  "type": "CAMERA",
  "model": "Sony PXW-Z190",
  "status": "MAINTENANCE",
  "last_maintenance_date": "2023-06-20T11:30:00.000Z",
  "allocations": [],
  "rental_cost": 150.00,
  "metadata": {
    "created_at": "2023-05-01T09:00:00.000Z",
    "updated_at": "2023-06-20T11:35:10.789Z",
    "deleted_at": null
  }
}
```

#### Step 3: Return Equipment to Available Status

**Request:**
```http
PATCH /api/v1/equipment/3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t
Content-Type: application/json

{
  "status": "AVAILABLE"
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "equipment_id": "3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t",
  "type": "CAMERA",
  "model": "Sony PXW-Z190",
  "status": "AVAILABLE",
  "last_maintenance_date": "2023-06-20T11:30:00.000Z",
  "allocations": [],
  "rental_cost": 150.00,
  "metadata": {
    "created_at": "2023-05-01T09:00:00.000Z",
    "updated_at": "2023-06-20T14:45:33.123Z",
    "deleted_at": null
  }
}
```

### 6.3 Staff Assignment and Equipment Allocation Workflow

**Steps:**
1. Create a New Staff Member
2. Assign Staff to a Story
3. Allocate Equipment to Staff
4. View Staff Assignments

#### Step 1: Create a New Staff Member

**Request:**
```http
POST /api/v1/staff
Content-Type: application/json

{
  "first_name": "Jane",
  "last_name": "Smith",
  "email": "jane.smith@newsroom.com",
  "phone": "+12025550179",
  "experience_years": 5,
  "role": "REPORTER",
  "status": "ACTIVE"
}
```

**Response:**
```http
HTTP/1.1 201 Created
Content-Type: application/json

{
  "staff_id": "b5f8c1d2-e3f4-5a6b-7c8d-9e0f1a2b3c4d",
  "email": "jane.smith@newsroom.com",
  "metadata": {
    "created_at": "2023-06-22T09:30:45.123Z"
  }
}
```

#### Step 2: Assign Staff to a Story

**Request:**
```http
PATCH /api/v1/stories/550e8400-e29b-41d4-a716-446655440000
Content-Type: application/json

{
  "assignment": {
    "assigned_staff_id": "b5f8c1d2-e3f4-5a6b-7c8d-9e0f1a2b3c4d",
    "priority": "LOW"
  }
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "story_id": "550e8400-e29b-41d4-a716-446655440000",
  "title": "Breaking: New Climate Policy Announced",
  "content": "The government has announced a new climate policy...",
  "category": "Politics",
  "status": "DRAFT",
  "assignment": {
    "assigned_staff_id": "b5f8c1d2-e3f4-5a6b-7c8d-9e0f1a2b3c4d",
    "priority": "LOW",
    "due_date": null
  },
  "metadata": {
    "created_at": "2023-06-22T10:15:22.456Z",
    "updated_at": "2023-06-22T10:20:33.789Z",
    "deleted_at": null
  }
}
```

#### Step 3: Allocate Equipment to Staff

**Request:**
```http
POST /api/v1/equipment/3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t/allocate
Content-Type: application/json

{
  "allocated_to_staff_id": "b5f8c1d2-e3f4-5a6b-7c8d-9e0f1a2b3c4d",
  "allocated_to_story_id": "550e8400-e29b-41d4-a716-446655440000",
  "allocation_date": "2023-06-22"
}
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "equipment_id": "3e4f5g6h-7i8j-9k0l-1m2n-3o4p5q6r7s8t",
  "status": "AVAILABLE",
  "allocations": [
    {
      "allocated_to_staff_id": "b5f8c1d2-e3f4-5a6b-7c8d-9e0f1a2b3c4d",
      "allocated_to_story_id": "550e8400-e29b-41d4-a716-446655440000",
      "allocation_date": "2023-06-22"
    }
  ],
  "metadata": {
    "updated_at": "2023-06-22T11:05:45.123Z"
  }
}
```

#### Step 4: View Staff Assignments

**Request:**
```http
GET /api/v1/staff/b5f8c1d2-e3f4-5a6b-7c8d-9e0f1a2b3c4d/assignments
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "staff_id": "b5f8c1d2-e3f4-5a6b-7c8d-9e0f1a2b3c4d",
  "assignments": [
    {
      "story_id": "550e8400-e29b-41d4-a716-446655440000",
      "title": "Breaking: New Climate Policy Announced",
      "status": "DRAFT",
      "priority": "LOW",
      "due_date": null
    }
  ],
  "total_count": 1
}
```