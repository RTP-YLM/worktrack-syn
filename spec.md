# Feature Specification: WorkTrack UPS Engineering Time Logging (Spent-Time First)

**Feature Branch**: `001-worktrack-bau-type-first`  
**Created**: 2026-04-26  
**Status**: Draft  
**Input**: User description: "วิศวกรเข้ามา เลือกโครงการ/ไลน์ผลิต UPS > เลือกกลุ่มงานวิศวกรรม (ออกแบบ/ทดสอบ/ซัพพอร์ตไลน์/กิจกรรมโครงการ) > กรอกเวลาที่ใช้ (spent) > ส่ง API แจ้งเตือน"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Log UPS engineering work with type-first flow (Priority: P1)

As a UPS engineer, I can create a work log by selecting project/production line, engineering work group, activity, and spent time in minutes/hours so I can report work quickly in an operational workflow.

**Why this priority**: This is the core value of the MVP. Without this flow, the system is not usable.

**Independent Test**: A user can submit one complete entry (project + type group + activity + spent + description) and receive a success response with saved data.

**Acceptance Scenarios**:

1. **Given** a logged-in user with accessible UPS projects/production lines, **When** user selects project, type group, activity, work date, spent time, and submits, **Then** system saves one work log successfully.
2. **Given** a selected type group, **When** user opens activity dropdown, **Then** system shows only activities mapped to that type group.
3. **Given** spent time is zero or negative, **When** user submits, **Then** system rejects with a validation error.

---

### User Story 2 - Reliable notification delivery after save (Priority: P2)

As an operations stakeholder, I can trust that each saved entry is sent to the notification API with delivery status tracking.

**Why this priority**: The user explicitly needs API-based notification as part of the business flow.

**Independent Test**: Submit an entry and verify status transitions to `notify_sent`; simulate downstream timeout and verify `notify_failed` with retry metadata.

**Acceptance Scenarios**:

1. **Given** a successfully persisted work log, **When** notification API returns success, **Then** entry status becomes `notify_sent`.
2. **Given** notification API fails, **When** retry policy is applied, **Then** system records `notify_failed`, attempt count, and latest error.
3. **Given** duplicate submit from client retry, **When** the same idempotency key is reused with identical payload, **Then** system does not create a duplicate work log.

---

### User Story 3 - Personal history and correction window (Priority: P3)

As an engineer, I can review my recent logs and correct mistakes within allowed policy.

**Why this priority**: Reduces admin overhead and improves data quality in early rollout.

**Independent Test**: User opens My Logs, filters by date, edits an eligible record, and sees updated value with audit trace.

**Acceptance Scenarios**:

1. **Given** existing user logs, **When** user opens My Logs, **Then** system shows date, project, type group, activity, spent, and notification status.
2. **Given** log is within editable policy window, **When** user updates spent/description, **Then** system saves change and updates audit fields.

---

### Edge Cases

- Duplicate submit due to network retry/tap-spam from mobile.
- Payload tampering: activity does not belong to selected type group.
- Project becomes inactive while form is open.
- Notification endpoint timeout/intermittent 5xx.
- Client timezone differs from server timezone.
- Extremely large spent value (e.g., >24h/day) must be validated by policy.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST require user to select a UPS project or production line before submission.
- **FR-002**: System MUST support engineering work groups: `Design Engineering`, `Verification & Testing`, `Production Line Support`, `Project Activities`.
- **FR-003**: System MUST provide dependent activity selection based on selected type group.
- **FR-004**: System MUST use spent-time input as primary mode (`spent_minutes`), not mandatory start/end period.
- **FR-005**: System MUST validate `spent_minutes > 0` and enforce configurable max per entry.
- **FR-006**: System MUST persist work log before calling notification API.
- **FR-007**: System MUST call notification API after persistence and track delivery state (`notify_sent` / `notify_failed`).
- **FR-008**: System MUST support idempotency key on create-worklog endpoint to prevent duplicates.
- **FR-009**: System MUST store retry metadata for failed notifications (attempt_count, last_error, last_attempt_at).
- **FR-010**: Users MUST be able to view their own historical work logs.
- **FR-011**: System MUST capture audit fields (`created_at/by`, `updated_at/by`) for create/edit actions.
- **FR-012**: Admin MUST manage master data for type groups and activities (active/inactive).
- **FR-013**: System SHOULD keep optional `started_at` nullable field for future Jira/worklog interoperability.
- **FR-014**: System MUST provide a month calendar view (Sun–Sat) with 5–6 week rows and include overflow days from adjacent months.
- **FR-015**: System MUST support selected-day state; selecting a day updates sidebar date and daily total hours immediately.
- **FR-016**: System MUST show a `+ New Work Log` primary action in sidebar and prefill selected date when opening create flow.
- **FR-017**: System MUST render at least two visual event categories in calendar cells: work/project events and leave events.
- **FR-018**: System MUST display daily total hours per day cell and in sidebar for selected day.
- **FR-019**: System SHOULD render holiday/non-working labels in day cells when holiday data exists.
- **FR-020**: System MUST provide a `New Work Log` / `Edit Work Log` modal with 4 tabs: `UPS Project / Production Line`, `Factory Engineering Work`, `Testing & Certification`, `Leave`.
- **FR-021**: Modal MUST support staged multi-entry flow: user adds entries to `Work Log List` first, then commits all entries with final `Save`.
- **FR-022**: `Save to Work Log List` MUST validate required fields for active tab before staging.
- **FR-023**: For `UPS Project / Production Line` tab, system MUST require project/line selection and stage selection before staging.
- **FR-024**: For `Factory Engineering Work` tab, system MUST support selecting work type and (if configured) plant/site or business unit before staging.
- **FR-025**: System MUST support `StartDate`, `EndDate`, and `Time Spent (per day)` input for applicable tabs.
- **FR-026**: System MUST enforce date rule `EndDate >= StartDate`.
- **FR-027**: System MUST provide quick spent-time chips: `15m`, `30m`, `1h`, `2h`, `3h`, `4h`, where clicking a chip sets spent-time value.
- **FR-028**: System MUST calculate entry total duration from `Time Spent (per day)` and selected date range by configured policy.
- **FR-029**: Work Log List MUST show at least: entry title/type, date range, and total duration per item.
- **FR-030**: System MUST allow `Add new` to create another draft entry without losing existing staged items.
- **FR-031**: System MUST support deleting a staged item from edit context.
- **FR-032**: Final `Save` MUST persist all staged items in one batch operation with duplicate-submit protection (loading/disabled state + idempotency-safe backend).
- **FR-033**: If final save fails, system MUST preserve staged items and show retry-capable error feedback.

### Key Entities *(include if feature involves data)*

- **Project**: UPS program / production line master (`project_id`, `project_code`, `project_name`, `line_or_site`, `active_flag`).
- **TypeGroup**: Top-level work classification (`type_group_id`, `name`, `active_flag`).
- **ActivityType**: Activity options mapped to a type group (`activity_type_id`, `type_group_id`, `name`, `active_flag`).
- **WorkLog**: Main submitted entry (`worklog_id`, `user_id`, `project_id`, `type_group_id`, `activity_type_id`, `work_date`, `spent_minutes`, `description`, `status`, `idempotency_key`, optional `started_at`, audit fields).
- **NotificationEvent**: Outbound notification attempts (`notification_event_id`, `worklog_id`, `endpoint`, `request_payload`, `response_code`, `response_body`, `attempt_no`, `sent_at`, `success_flag`).
- **WorkLogDraftItem**: Client-side staged item in modal before final save (`draft_id`, `entry_type`, `title`, `project_or_type_ref`, `business_ref`, `start_date`, `end_date`, `spent_per_day_minutes`, `total_minutes`, `description`, `is_dirty`).
- **WorkLogBatchSave**: Final commit request wrapper for staged items (`batch_idempotency_key`, `items[]`, `submitted_by`, `submitted_at`).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: Median time to create one valid work log is <= 30 seconds.
- **SC-002**: 100% of invalid spent-time submissions are rejected before persistence.
- **SC-003**: Duplicate work log creation rate from repeated submit is 0 when idempotency key is provided.
- **SC-004**: Notification delivery reaches >= 99% within configured retry window.
- **SC-005**: At least 90% of pilot users can submit a log successfully without training beyond a one-page guide.
- **SC-006**: Users can stage multiple entries in one modal session and complete final save with <= 2 clicks after data entry (`Save to Work Log List` then `Save`).
- **SC-007**: Final batch save failure recovery retains 100% of staged entries for retry (no client-side data loss).

## Assumptions

- Existing organization auth and user identity are already available.
- MVP phase excludes approval workflow.
- Primary use case is UPS engineering and production-support effort tracking, not payroll/billing.
- Notification API contract is available and reachable from backend.

## Out of Scope

- Multi-level approval chains.
- Payroll, invoicing, and cost-rate calculations.
- Advanced analytics dashboards beyond basic history/status.
- Enterprise SSO enhancements beyond current auth baseline.
