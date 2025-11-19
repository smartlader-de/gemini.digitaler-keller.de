# Database Architecture PRD - Digital Basement Energy Management Platform

## Overview

This document serves as a comprehensive Product Requirements Document (PRD) for the database architecture, focusing on the core entity relationships between **device_id**, **ou_id**, **api_id**, and **user_id** and how they interact within the Digital Basement platform.

**Database Stack:**
- PostgreSQL (via Supabase)
- Row Level Security (RLS) for access control
- UUID primary keys for all entities
- Hierarchical organizational structure
- Integration-based device management

---

## Core Entity Identifiers

### 1. user_id

**Definition:** Unique identifier for platform users

**Source Tables:**
- `auth.users(id)` - Supabase authentication users
- `public.profiles(id)` - Extended user profile data

**Characteristics:**
- Type: `UUID`
- Primary key in `profiles` table
- References `auth.users(id)` via foreign key
- Used throughout the system for:
  - Authentication and authorization
  - Audit logging
  - Ownership tracking
  - Permission checks

**Key Relationships:**
- One-to-many with `organizational_unit_members` (user can belong to multiple OUs)
- One-to-many with `audit_log` (tracks user actions)
- One-to-many with `integrations` (created_by, updated_by)
- One-to-many with `device_readings` (created_by)

**Access Patterns:**
- Users authenticate via `auth.users`
- Profile data stored in `profiles`
- User permissions checked via `organizational_unit_members` and `has_org_unit_access()` function
- Superadmin users have global access via `profiles.role = 'superadmin'`

---

### 2. ou_id (Organizational Unit ID)

**Definition:** Unique identifier for organizational units in the hierarchical structure

**Source Table:**
- `public.organizational_units(id)`

**Characteristics:**
- Type: `UUID`
- Primary key in `organizational_units` table
- Supports hierarchical structure via `parent_id` self-reference
- Flexible unit types via `unit_type` field
- Metadata storage via `metadata JSONB` field

**Key Relationships:**
- Self-referential: `parent_id` → `organizational_units(id)` (hierarchical structure)
- One-to-many with `organizational_unit_members` (OU has many members)
- One-to-many with `integrations` (integration belongs to OU)
- One-to-many with `devices` (device belongs to OU)
- One-to-many with `integration_audit_log` (audit entries scoped to OU)

**Hierarchical Structure:**
```
Root OU (parent_id = NULL)
  └── Child OU (parent_id = root_ou_id)
      └── Grandchild OU (parent_id = child_ou_id)
```

**Access Control:**
- Users access OUs via `organizational_unit_members` table
- Roles: `unit_admin`, `unit_user`, `unit_guest`
- Permission inheritance via `inherit_to_children` flag
- Access checked via `has_org_unit_access(user_id, ou_id, required_role)` function

**Use Cases:**
- Multi-tenant organization structure
- Hierarchical permission inheritance
- Device onboarding (unassigned devices belong to OU)
- Integration management (integrations can belong to any OU)

---

### 3. api_id (Integration ID)

**Definition:** Unique identifier for external API integrations

**Source Table:**
- `public.integrations(id)`

**Characteristics:**
- Type: `UUID`
- Primary key in `integrations` table
- Links to vendor via `vendor_id` → `integration_vendors(id)`
- Can belong to any organizational unit via `organizational_unit_id` (or NULL for global integrations)
- Status tracking: `draft`, `active`, `disabled`, `error`, `pending`
- Soft deletion via `deleted_at` timestamp

**Key Relationships:**
- Many-to-one with `integration_vendors` (integration belongs to vendor)
- Many-to-one with `organizational_units` (integration can belong to any OU)
- One-to-many with `integration_credentials` (integration has credentials)
- One-to-many with `integration_events` (integration generates events)
- One-to-many with `integration_devices` (integration manages devices)
- One-to-many with `integration_device_configs` (integration has device configs)
- One-to-many with `devices` (device can reference integration)

**Related Tables:**
- `integration_credentials` - Stores API keys/secrets (encrypted references)
- `integration_events` - Event log for integration activities
- `integration_devices` - Maps vendor devices to platform devices
- `integration_datapoints` - Maps vendor datapoints to device fields
- `integration_device_configs` - Device-specific configuration

**Access Control:**
- Users can view integrations in their accessible OUs
- Unit admins can create/update/delete integrations
- Superadmins have full access
- RLS policies check OU access via `has_org_unit_access()`

**Integration Flow:**
1. Create integration → links to OU and vendor
2. Configure credentials → stored in `integration_credentials`
3. Discover devices → via vendor API
4. Map vendor devices to platform devices → via `integration_devices`
5. Sync data → via `integration_datapoints` mapping

---

### 4. device_id

**Definition:** Unique identifier for physical devices

**Source Table:**
- `public.devices(id)`

**Characteristics:**
- Type: `UUID`
- Primary key in `devices` table
- Belongs to organizational unit via `organizational_unit_id`
- Integration tracking via `integration_id` and `integration_vendor_identifier`
- Supports hierarchical device structure (parent-child devices)

**Key Relationships:**
- Many-to-one with `organizational_units` (device belongs to OU)
- Many-to-one with `integrations` (device linked to integration)
- Self-referential: `parent_device_id` → `devices(id)` (hierarchical devices)
- One-to-many with `device_readings` (device has readings)
- One-to-many with `integration_devices` (device mapped from integration)

**Assignment States:**
1. **Assigned to OU:** `organizational_unit_id IS NOT NULL` (normal state)
2. **Orphaned:** `organizational_unit_id IS NULL` (should not occur in practice)

**Integration Mapping:**
- `integration_id` - Links to integration that manages this device
- `integration_vendor_identifier` - Vendor-specific device ID (e.g., Lobaro device ID)
- `integration_devices.vendor_device_identifier` - Alternative vendor ID storage
- `integration_device_configs.vendor_device_id` - Device configuration mapping

**Access Control:**
- Users access devices via device's OU
- Unit admins can manage devices in their OUs
- RLS policies check device OU access

---

## Entity Relationship Diagram

```
┌─────────────────┐
│    user_id      │
│  (profiles)     │
└────────┬────────┘
         │
         │ (many-to-many)
         │
         ▼
┌─────────────────────────────┐
│ organizational_unit_members  │
│  - user_id                  │
│  - organizational_unit_id   │
│  - role                     │
│  - inherit_to_children     │
└────────────┬────────────────┘
             │
             │ (many-to-one)
             │
             ▼
┌─────────────────────────────┐
│  ou_id                      │
│  (organizational_units)     │
│  - id                       │
│  - parent_id                │
│  - unit_type                │
└─────┬───────────┬───────────┘
      │           │
      │ (one-to-many)  (one-to-many)
      │           │
      ▼           ▼
┌──────────────────┐  ┌──────────────────┐
│   device_id       │  │   api_id         │
│   (devices)       │  │ (integrations)    │
│   - id            │  │   - id            │
│   - organizational_unit_id │ │   - organizational_unit_id │
│   - integration_id │ │   - vendor_id     │
│   - parent_device_id │ └──────────────────┘
└──────────────────┘
```

---

## Interaction Patterns

### Pattern 1: User → OU → Device Access

**Flow:**
1. User authenticates → `user_id` from `auth.users`
2. User belongs to OU → `organizational_unit_members` links `user_id` to `ou_id`
3. Device belongs to OU → `devices.organizational_unit_id = ou_id`
4. Access check → `has_org_unit_access(user_id, ou_id, 'unit_guest')`

**Use Case:** User viewing devices in their organizational unit

**SQL Example:**
```sql
SELECT d.*
FROM devices d
WHERE d.organizational_unit_id = ANY(
  SELECT ou.id 
  FROM organizational_units ou
  WHERE has_org_unit_access(auth.uid(), ou.id, 'unit_guest')
);
```

---

### Pattern 2: User → OU → Integration → Device

**Flow:**
1. User belongs to OU → `organizational_unit_members`
2. Integration belongs to OU → `integrations.organizational_unit_id = ou_id`
3. Integration manages devices → `integration_devices` links `api_id` to `device_id`
4. Access check → User must have OU access, integration must belong to OU

**Use Case:** User viewing devices imported via integration

**SQL Example:**
```sql
SELECT d.*, i.label as integration_label
FROM devices d
JOIN integration_devices id ON id.device_id = d.id
JOIN integrations i ON i.id = id.integration_id
WHERE i.organizational_unit_id = ANY(
  SELECT ou.id 
  FROM organizational_units ou
  WHERE has_org_unit_access(auth.uid(), ou.id, 'unit_guest')
);
```

---

### Pattern 3: Integration → Device Discovery → Assignment

**Flow:**
1. Integration created → `integrations` record with `api_id` and `ou_id`
2. Devices discovered → Vendor API returns device list
3. Devices created → `devices` with `organizational_unit_id = ou_id`
4. Devices mapped → `integration_device_configs` stores vendor device IDs
5. Devices assigned to OU → `devices.organizational_unit_id` set from integration

**Use Case:** Onboarding new devices from external integration

**SQL Example:**
```sql
-- Create unassigned device from integration
INSERT INTO devices (
  label,
  organizational_unit_id,
  integration_id,
  integration_vendor_identifier,
  sector,
  resource,
  unit,
  reading_mode
)
VALUES (
  'Device Label',
  (SELECT organizational_unit_id FROM integrations WHERE id = :api_id),
  :api_id,
  :vendor_device_id,
  'electricity',
  'power',
  'W',
  'counter'
);

-- Link via integration_device_configs
INSERT INTO integration_device_configs (
  integration_id,
  vendor_device_id,
  config_data
)
VALUES (
  :api_id,
  :vendor_device_id,
  :config_jsonb
);
```

---

### Pattern 4: Hierarchical Permission Inheritance

**Flow:**
1. User assigned to parent OU → `organizational_unit_members` with `inherit_to_children = true`
2. Child OU inherits permissions → `has_org_unit_access()` checks parent chain
3. User accesses child OU resources → Permission granted via inheritance

**Use Case:** Admin at organization level accessing all child unit devices

**SQL Example:**
```sql
-- Check if user has access (includes inheritance)
SELECT has_org_unit_access(
  auth.uid(),
  :child_ou_id,
  'unit_admin'
);

-- Function internally checks:
-- 1. Direct membership in child OU
-- 2. Parent OU membership with inherit_to_children = true
-- 3. Recursive parent chain traversal
-- 4. Superadmin fallback
```

---

### Pattern 5: Device Reading with User Context

**Flow:**
1. User reads device → `device_id` from `devices`
2. Reading created → `device_readings` with `device_id` and `created_by = user_id`
3. Access check → User must have access to device's OU
4. Audit log → `audit_log` records action with `user_id`

**Use Case:** User manually entering device reading

**SQL Example:**
```sql
-- Insert reading with access check
INSERT INTO device_readings (
  device_id,
  reading_date,
  computed_consumption,
  created_by
)
SELECT 
  :device_id,
  :reading_date,
  :consumption,
  auth.uid()
WHERE EXISTS (
  SELECT 1 FROM devices d
  WHERE d.id = :device_id
  AND has_org_unit_access(auth.uid(), d.organizational_unit_id, 'unit_user')
);
```

---

## Access Control Matrix

### User Roles and Permissions

| Role | OU Access | Device View | Device Manage | Integration View | Integration Manage |
|------|-----------|-------------|---------------|------------------|-------------------|
| `superadmin` | All OUs | All devices | All devices | All integrations | All integrations |
| `unit_admin` | Assigned OUs + children* | OU devices | OU devices | OU integrations | OU integrations |
| `unit_user` | Assigned OUs + children* | OU devices | Read-only | OU integrations | Read-only |
| `unit_guest` | Assigned OUs + children* | OU devices | Read-only | OU integrations | None |

*If `inherit_to_children = true` in `organizational_unit_members`

### Entity Access Rules

| Entity | Access Check | RLS Policy |
|--------|--------------|------------|
| `organizational_units` | `has_org_unit_access(user_id, ou_id, 'unit_guest')` | SELECT: Guest access<br>INSERT/UPDATE/DELETE: Admin access |
| `devices` | Direct OU access | SELECT: Guest access<br>INSERT/UPDATE/DELETE: Admin access |
| `integrations` | OU access | SELECT: Guest access<br>INSERT/UPDATE/DELETE: Admin access |
| `integration_devices` | Inherited from integration | SELECT: Guest access<br>INSERT/UPDATE/DELETE: Admin access |
| `device_readings` | Inherited from device | SELECT: Guest access<br>INSERT: User access<br>UPDATE/DELETE: Admin access |

---

## Data Flow Examples

### Example 1: New User Onboarding

```
1. User signs up → auth.users record created
2. Profile created → profiles record with user_id
3. User invited to OU → organizational_unit_members record
   - user_id: <new_user_uuid>
   - organizational_unit_id: <ou_uuid>
   - role: 'unit_guest'
   - inherit_to_children: true
4. User can now access OU resources
```

### Example 2: Integration Device Import

```
1. Admin creates integration → integrations record
   - id: <api_id>
   - organizational_unit_id: <ou_id>
   - vendor_id: <vendor_uuid>
2. Integration discovers devices → Vendor API call
3. Devices created → devices records
   - id: <device_id>
   - organizational_unit_id: <ou_id>
   - integration_id: <api_id>
   - integration_vendor_identifier: <vendor_device_id>
4. Device configs stored → integration_device_configs
   - integration_id: <api_id>
   - vendor_device_id: <vendor_device_id>
5. Device mapping created → integration_devices
   - integration_id: <api_id>
   - device_id: <device_id>
   - vendor_device_identifier: <vendor_device_id>
```

### Example 3: Hierarchical Access

```
1. Root OU created → organizational_units
   - id: <root_ou_id>
   - parent_id: NULL
2. Child OU created → organizational_units
   - id: <child_ou_id>
   - parent_id: <root_ou_id>
3. User assigned to root → organizational_unit_members
   - user_id: <user_id>
   - organizational_unit_id: <root_ou_id>
   - role: 'unit_admin'
   - inherit_to_children: true
4. User can access child OU → has_org_unit_access() returns true
5. User can manage child OU devices → Admin permissions inherited
```

---

## Key Database Functions

### has_org_unit_access()

**Purpose:** Check if user has required access level to organizational unit

**Signature:**
```sql
has_org_unit_access(
  _user_id UUID,
  _unit_id UUID,
  _required_role org_unit_role DEFAULT 'unit_guest'
) RETURNS BOOLEAN
```

**Logic:**
1. Check direct membership in OU
2. Check inherited access from parent OUs (if `inherit_to_children = true`)
3. Recursive parent chain traversal
4. Fallback to superadmin check

**Usage:**
```sql
-- Check if user can view OU
SELECT has_org_unit_access(auth.uid(), :ou_id, 'unit_guest');

-- Check if user can manage OU
SELECT has_org_unit_access(auth.uid(), :ou_id, 'unit_admin');
```

---

### get_user_org_unit_tree()

**Purpose:** Get all OU IDs user can access (including inherited)

**Signature:**
```sql
get_user_org_unit_tree(_user_id UUID) RETURNS UUID[]
```

**Returns:** Array of OU IDs user can access

**Usage:**
```sql
-- Get all accessible OUs
SELECT get_user_org_unit_tree(auth.uid());

-- Use in WHERE clause
SELECT * FROM devices
WHERE organizational_unit_id = ANY(get_user_org_unit_tree(auth.uid()));
```

---

## Constraints and Business Rules

### 1. Device Assignment Constraint

**Rule:** Device must belong to an organizational unit

**Implementation:**
- `devices.organizational_unit_id` must be NOT NULL (required)
- RLS policies check OU access for device access control

**Validation:**
```sql
-- Valid state:
-- organizational_unit_id IS NOT NULL (assigned to OU)
-- Invalid: organizational_unit_id IS NULL
```

---

### 2. Last Admin Constraint

**Rule:** Cannot remove last admin from OU

**Implementation:**
- Database trigger prevents deletion/update of last `unit_admin`
- Applies to both `organizational_unit_members` and `profiles` (superadmin)

**Trigger:**
```sql
CREATE TRIGGER prevent_last_unit_admin_removal
BEFORE DELETE OR UPDATE ON organizational_unit_members
FOR EACH ROW
EXECUTE FUNCTION prevent_last_admin_removal();
```

---

### 3. Integration OU Assignment

**Rule:** Integration can belong to any OU (or NULL for global)

**Implementation:**
- `integrations.organizational_unit_id` can be NULL (global integration accessible to all)
- If set, integration belongs to that OU and is accessible to OU members
- Integration can be assigned to any OU without restrictions
- Devices imported via integration inherit OU from integration

---

### 4. Hierarchical Cycle Prevention

**Rule:** Cannot create circular references in OU hierarchy

**Implementation:**
- Database trigger prevents `parent_id` cycles
- Function `prevent_organizational_unit_cycle()` checks before insert/update

---

### 5. Unique Device Mapping

**Rule:** Device can only be mapped once per integration

**Implementation:**
- `integration_devices` has `UNIQUE(integration_id, device_id)`
- `integration_device_configs` has `UNIQUE(integration_id, vendor_device_id)`

---

## Migration and Data Integrity

### Device Migration Path

**Device Assignment:**
1. Device created with `organizational_unit_id` from integration or manual assignment
2. Device belongs to OU and is accessible to OU members

**Device Reassignment:**
1. Device has `organizational_unit_id`
2. User changes OU assignment
3. `organizational_unit_id` updated to new OU
4. Access control updated based on new OU

**Integration Removal:**
1. Integration deleted → `integrations.deleted_at` set
2. Devices keep `integration_id` (soft reference)
3. `integration_devices` records remain (historical)
4. New readings stop syncing

---

## Performance Considerations

### Indexes

**Critical Indexes:**
- `idx_org_unit_members_user` on `organizational_unit_members(user_id)`
- `idx_org_unit_members_unit` on `organizational_unit_members(organizational_unit_id)`
- `idx_devices_organizational_unit_id` on `devices(organizational_unit_id)`
- `idx_integrations_org_unit_id` on `integrations(organizational_unit_id)`
- `idx_integration_devices_integration_id` on `integration_devices(integration_id)`
- `idx_integration_devices_device_id` on `integration_devices(device_id)`

### Query Optimization

**Common Patterns:**
1. Use `get_user_org_unit_tree()` for bulk access checks
2. Filter by OU before joining with devices
3. Use indexes on foreign keys
4. Materialize OU tree for frequent access (if needed)

---

## Security Considerations

### Row Level Security (RLS)

**All tables have RLS enabled:**
- `organizational_units` - OU access checks
- `devices` - Direct OU access
- `integrations` - OU access checks
- `integration_devices` - Inherited from integration
- `organizational_unit_members` - Self or admin access

**Security Functions:**
- All access checks use `SECURITY DEFINER` functions
- Functions have `SET search_path = public` to prevent injection
- User context via `auth.uid()` in RLS policies

---

## Summary

This PRD documents the core database architecture focusing on four key identifiers:

1. **user_id** - Platform users and authentication
2. **ou_id** - Hierarchical organizational structure
3. **api_id** - External API integrations
4. **device_id** - Physical devices

**Key Relationships:**
- Users belong to OUs via `organizational_unit_members`
- Integrations belong to OUs via `integrations.organizational_unit_id`
- Devices belong to OUs via `devices.organizational_unit_id`
- Devices can be managed by integrations via `integration_devices`

**Access Control:**
- Hierarchical permission inheritance
- Role-based access (admin, user, guest)
- RLS policies enforce access at database level
- Security functions provide consistent access checks

**Use Cases:**
- Multi-tenant organization management
- Device onboarding and assignment
- Integration-based device discovery
- Hierarchical permission inheritance
- Audit logging and compliance

This architecture supports flexible organizational structures while maintaining strict access control and data isolation.
