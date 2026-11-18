# CrisisSync - Requirements, Traceability Matrix and Acceptance Tests

**Group:** G12

| Name | College Email |
|---|---|
| Or Elimelech | Or.Elimelech@e.braude.ac.il |
| Eyal Hirsch | Eyal.Hirsch@e.braude.ac.il |
| Liron Skverchak | Liron.Skverchak@e.braude.ac.il |

---

## 1. Scope

This document defines the requirements and acceptance tests for two CrisisSync operations:

1. **Reserving a resource bundle for a task, including reporting task completion.** Use cases: UC-16 Reserve Resource Bundle, UC-17 Validate Resource Reservation, UC-16a Handle Reservation Conflict, UC-18 Auto-Allocate Resource Bundle, UC-18a Retry With an Alternative Bundle, UC-25 Assign Replacement Resource, UC-26 Report Task Completion, UC-27 Release Resources and Promote Dependent Tasks, UC-12 Update Crew Availability, UC-29 Report Shift Start and Readiness, UC-30 Report Equipment Malfunction.
2. **Generating the SLA (time-compliance and service-level) report.** Use cases: UC-06 Generate Monthly Reports, UC-08 Generate SLA Report, UC-05 View Management Reports.

Use case IDs match the Visual Paradigm model G12_Assignment1.vpp. Manual and automatic allocation follow the same resource availability and all-or-nothing reservation rules. A resource can leave an Active task through UC-30 Report Equipment Malfunction or UC-12 Update Crew Availability. Its replacement is reserved in UC-25, and the task stays Active rather than starting a new task (FR-RES-13, FR-RES-14).

This version applies the course clarifications published on 15 November 2026 (Assignment 1 forum). Where a clarification covers a topic, it replaces the earlier team assumption.

## 2. Terms and Working Assumptions

### 2.1 Terms used in this document

- **Resource:** an individual crew member or equipment item. A resource can be reserved for at most one task at a time. A reserved Field Commander can still use the system.
- **Medic:** the medical crew role in the story's fixed list of personnel roles.
- **Current assignment:** a resource reservation that is still in effect. Earlier assignments remain in the event history for auditing and cost calculations.
- **Relevant stations:** connected Dispatchers and Field Commanders authorized to view the event, and crew members assigned to an affected task. Crew members receive their permitted task details, not unrestricted access to the event.
- **Task states:** Blocked, Ready, Active, Completed and Canceled. Completed and Canceled are final states. A dependent task becomes Ready only when all its prerequisites are Completed.
- **Event states:** Open, In Settlement, Closed and Canceled. An event In Settlement is not Open: it cannot be canceled, closed again or given new tasks. It becomes Closed only after settlement is approved.
- **Missing resource:** a required role or equipment slot of an Active task whose resource was released, for example after an equipment malfunction or when the Logistics Officer marks a reserved crew member unavailable. The task stays Active while a slot is missing.
- **Event SLA time:** the elapsed time from event opening until the last Completed transition of its tasks. Canceled tasks are ignored, and time spent waiting for financial settlement is not added to this duration.

### 2.2 Working assumptions

The story does not specify every reporting rule or numeric quality target. The following choices make the requirements testable; they are team assumptions, not additional facts stated in the story.

| ID | Working assumption |
|---|---|
| A-SLA | **Course clarification (15 Nov 2026).** The report includes only events that were finally Closed (settlement approved) and that have at least one Completed task. Each event belongs to the month of its final closure. Its SLA time ends at its last Completed task; Canceled tasks are ignored. Canceled events, and events that are still Open or In Settlement, are excluded. |
| A-TIME | Use the Asia/Jerusalem calendar. Start the monthly report process at 00:00 on the first day of the following month. Calculate durations from actual elapsed time, so clock changes do not change the result. |
| A-QUALITY | The response times, load levels, TLS version, audit timestamp precision, contrast ratio and action counts below are measurable targets selected by the team. Performance checks use the course LAN environment with connected clients and a running server, except when a test explicitly introduces a failure. |
| A-DESIGN | The following behaviors are team design decisions that make the story's rules complete and testable; the story does not state them explicitly: auditing rejected and unauthenticated requests (FR-RES-10), preventing two concurrent bundles from activating the same task (FR-RES-02), showing "No data" for an event type without events (FR-RPT-05), regenerating missed monthly reports at startup (FR-RPT-06), the event count and duration format in the report (FR-RPT-08), a manual SLA request not starting the utilization report (AT-RPT-02), and recovery after a server crash or a lost response (NFR-REL-03). |
| A-UI | For the common manual reservation flow, the screen suggests a complete matching bundle. The Dispatcher can change the selection before confirming. The three-action target applies when the suggested bundle is valid. A manual report request selects one report type and one completed month. |

When a task ends, equipment already reported as faulty must not become Available automatically. Only the Logistics Officer can return that equipment to Available. Completion releases resources that are still assigned to the task and preserves the earlier assignment history.

---

## 3. Functional Requirements (FR)

### 3.1 Operation 1 - Resource bundle reservation and task completion

| ID | Requirement |
|---|---|
| FR-RES-01 | The system shall accept a new bundle only from a Dispatcher for a *Ready* task whose prerequisites are all *Completed*, including a task with no prerequisites. |
| FR-RES-02 | The system shall reserve a complete bundle in one SQL transaction. It shall lock records in one fixed order (the event record, then the task, then the requested resources in ascending ID order) before checking availability and updating them. It shall also prevent two requests from activating the same task with different bundles. If the reservation fails, the system shall roll back all changes made by that request and preserve other committed changes. The SQL database shall be the source of truth. |
| FR-RES-03 | The system shall reject a bundle that does not match the task's required roles, equipment types and quantities, including exactly one Field Commander. |
| FR-RES-04 | The system shall reject the reservation of an equipment item whose status is *Faulty/Under Maintenance* and of a crew member whose status is *Unavailable*. |
| FR-RES-05 | The system shall reject the reservation of any resource, including a Field Commander, that is already *Reserved* for another task. Reservation shall not prevent the Field Commander from using the system. |
| FR-RES-06 | Upon a successful reservation, the system shall set every resource of the bundle to *Reserved* for the task, set the task state to *Active*, and push the change to all relevant stations. |
| FR-RES-07 | If a reservation fails, the system shall explain the reason and refresh the Dispatcher's screen from the current database state. If resource availability caused the failure, the message shall identify the affected resource and its current status. |
| FR-RES-08 | The system shall accept a task-completion report only from the Field Commander assigned to that task, and only while the task is *Active*. |
| FR-RES-09 | When a completion report is accepted, the system shall set the task to *Completed*, release its currently assigned resources to *Available*, and set each *Blocked* dependent task to *Ready* only if all its prerequisites are *Completed*. These changes shall be saved in one SQL transaction. Previously faulty equipment shall remain faulty, and assignment history shall be preserved. |
| FR-RES-10 | The system shall record successful and rejected reservation and completion requests in the Audit Log, including the action, result, acting user ID and timestamp to millisecond precision. Successful records shall identify the affected task and resource state changes. Requests without a valid session shall be recorded as unauthenticated. |
| FR-RES-11 | The system shall allow the Dispatcher to request automatic allocation using a selected criterion. If a competing request takes a selected resource, the system shall replace only the taken resources with the next best available resources by the same criterion, keep the rest of the selection, and retry the whole bundle in a new all-or-nothing transaction. It shall make up to two alternative attempts (three attempts in total), without a new Dispatcher request and without blocking service to other users. Allocation shall fail if a taken resource has no available alternative, or if the second alternative attempt also loses to a competing request. In both cases the task shall remain *Ready* with no partial reservation, and the message to the Dispatcher shall state which of the two cases occurred. Every attempt shall follow the same reservation rules as manual allocation. |
| FR-RES-12 | The system shall reject a completion report while the task has a missing resource, unless the Dispatcher has assigned a replacement (UC-25) or has confirmed that the missing resource is no longer needed to complete the task. The rejection shall identify the missing role or equipment type. A missing Field Commander cannot be waived: a replacement commander is required and receives the task's commander permissions. |
| FR-RES-13 | When a crew member or the Field Commander assigned to an *Active* task reports a malfunction of an item reserved for it, or the Logistics Officer marks a reserved crew member *Unavailable*, the system shall set the resource's new status, end its assignment with an end time, keep its assigned hours for cost and utilization calculations, keep the task *Active*, and alert the Dispatcher to assign a replacement. The system shall reject a reserved crew member's own report of being *Unavailable*. |
| FR-RES-14 | The system shall reserve a replacement, chosen manually or by automatic selection, only for a missing slot of an *Active* task and only with an available resource of the missing role or equipment type. It shall apply the lock order and availability rules of FR-RES-02, FR-RES-04 and FR-RES-05, keep the task *Active*, and reject the attempt without changes if the slot or the candidate changed. A replacement Field Commander shall receive the task's commander permissions, and the released commander shall lose them. |


### 3.2 Operation 2 - SLA report

| ID | Requirement |
|---|---|
| FR-RPT-01 | The system shall start automatic generation of the previous month's SLA report at 00:00 on the first day of each month, according to the server clock and A-TIME. |
| FR-RPT-02 | The system shall calculate the average Event SLA time separately for each event type. It shall measure each included event from its opening time to its last Completed task, ignoring Canceled tasks and without adding financial settlement time. |
| FR-RPT-03 | Under A-SLA, the system shall include only events finally Closed during the selected month that have at least one Completed task, and shall assign each event to the month of its final closure. It shall exclude Canceled events and events that are Open or In Settlement. |
| FR-RPT-04 | The system shall allow the System Administrator to generate the SLA report on demand for any selected completed past month. |
| FR-RPT-05 | The system shall display "No data" for an event type with no included events in the report period, instead of an average of 0. |
| FR-RPT-06 | At server startup, the system shall check for missing or unfinished monthly SLA reports since the service began. It shall generate those reports and keep one complete stored SLA report per month, without duplicate results. |
| FR-RPT-07 | The server shall allow only authenticated System Administrators to view stored SLA reports or request manual generation. The internal monthly job shall be allowed to generate reports independently. |
| FR-RPT-08 | The system shall display all five event types from the story, the number of included events for each type, the average SLA time, the report month and the generation timestamp. It shall display durations as hours and minutes, rounded to the nearest minute, with hours allowed to exceed 24. |

---

## 4. Non-Functional Requirements (NFR)

| ID | Category | Requirement (measurable threshold) |
|---|---|---|
| NFR-PERF-01 | Performance | With **20 concurrent Dispatcher clients**, the system shall display a reservation result within **2 seconds for at least 95%** of requests in the reference workload in AT-RES-11. All requests in that workload and the lock-timeout test shall receive a result within **7 seconds** of submission. |
| NFR-PERF-02 | Performance | The system shall push a task-state or resource-state change to all relevant stations within **1 second** of the transaction commit, using server push without client polling. |
| NFR-PERF-03 | Performance | The system shall generate and display an SLA report for up to **10,000 eligible events** within **10 seconds** of the Administrator's request. |
| NFR-SEC-01 | Security | The server shall enforce role-based authorization on **100%** of reservation, completion-report and report requests. It shall reject unauthorized requests with an error and record them in the Audit Log within **1 second**. |
| NFR-SEC-02 | Security | All client-server messages for the two operations shall use **TLS 1.2 or higher**, with **AES-128-GCM or AES-256-GCM** encryption. |
| NFR-REL-01 | Reliability | For **100 concurrent valid requests** for different Ready tasks that share one available resource and otherwise use separate available resources, exactly **1** reservation shall succeed and **0** double reservations shall exist. No task completes or releases the shared resource during the test. |
| NFR-REL-02 | Reliability | If a reservation spends a total of **5 seconds** waiting for database locks without obtaining all required locks, the system shall abort the transaction. It shall release that transaction's locks within **1 second** of the abort and leave **0** reservations created by the failed request. |
| NFR-REL-03 | Reliability | After a server crash during reservation or completion, **0** partial reservations or partial releases shall remain after restart. Changes not committed before the crash shall be undone. Committed changes shall remain saved, even if the client did not receive a response. |
| NFR-REL-04 | Reliability | After a client reconnects following a network interruption, the client shall resynchronize its event status screen with the server state within **3 seconds**. |
| NFR-USA-01 | Usability | A Dispatcher shall be able to complete a manual bundle reservation from the event status screen in **at most 3 user actions** when the preselected bundle is valid (select task → *Reserve Bundle* → confirm pre-selected resources). |
| NFR-USA-02 | Usability | All **5 task states** shall have distinct colors and text labels. Each state label shall have a text-to-background contrast ratio of at least **4.5:1**. |
| NFR-USA-03 | Usability | The System Administrator shall be able to open the SLA report of any stored month in **at most 2 user actions** from the main screen. |

---

## 5. Requirements Traceability Matrix (RTM)

| Req ID | Requirement (short) | Use Case(s) | Acceptance Test ID(s) |
|---|---|---|---|
| FR-RES-01 | Reserve only for *Ready* task | UC-16 Reserve Resource Bundle | AT-RES-01, AT-RES-02, AT-RES-15 |
| FR-RES-02 | All-or-nothing transaction, full rollback | UC-16, UC-17 Validate Resource Reservation | AT-RES-03, AT-RES-04, AT-RES-15 |
| FR-RES-03 | Complete matching bundle with exactly one commander | UC-16, UC-17 | AT-RES-05, AT-RES-16 |
| FR-RES-04 | No faulty equipment / unavailable crew | UC-17 | AT-RES-04 |
| FR-RES-05 | No double reservation (incl. Field Commander) | UC-17 | AT-RES-03, AT-RES-06 |
| FR-RES-06 | Success → *Active*, *Reserved*, push | UC-16 | AT-RES-01 |
| FR-RES-07 | Failure message and screen refresh | UC-16, UC-16a Handle Reservation Conflict | AT-RES-02, AT-RES-03, AT-RES-04, AT-RES-07 |
| FR-RES-08 | Assigned commander and Active task required | UC-26 Report Task Completion | AT-RES-08, AT-RES-09, AT-RES-17 |
| FR-RES-09 | Completion → release + promote dependents | UC-26, UC-27 Release Resources and Promote Dependent Tasks | AT-RES-08, AT-RES-18 |
| FR-RES-10 | Audit Log for reservation and completion | UC-16, UC-26 | AT-RES-01, AT-RES-03, AT-RES-08, AT-RES-09, AT-RES-17, AT-RES-19 |
| FR-RES-11 | Automatic allocation; replace only taken resources, up to two alternatives | UC-18 Auto-Allocate Resource Bundle, UC-18a Retry With an Alternative Bundle, UC-17 | AT-RES-23, AT-RES-24 |
| FR-RES-12 | No completion while a resource is missing, unless replaced or waived | UC-26, UC-25 Assign Replacement Resource | AT-RES-25 |
| FR-RES-13 | Mid-task release keeps the task *Active*; no self-release while reserved | UC-30 Report Equipment Malfunction, UC-12 Update Crew Availability, UC-29 Report Shift Start and Readiness | AT-RES-26, AT-RES-27 |
| FR-RES-14 | Replacement only for a missing slot; commander permissions move | UC-25, UC-17 | AT-RES-26, AT-RES-27 |
| FR-RPT-01 | Automatic month-end generation | UC-06 Generate Monthly Reports | AT-RPT-01 |
| FR-RPT-02 | Average SLA time per event type, to last Completed task | UC-08 Generate SLA Report | AT-RPT-01, AT-RPT-09 |
| FR-RPT-03 | Closed events with a Completed task, by closure month (A-SLA) | UC-08 | AT-RPT-04, AT-RPT-09 |
| FR-RPT-04 | On-demand report for past month | UC-08 Generate SLA Report | AT-RPT-02 |
| FR-RPT-05 | "No data" instead of 0 | UC-08 | AT-RPT-03 |
| FR-RPT-06 | Recover absent or unfinished monthly report | UC-06 | AT-RPT-05 |
| FR-RPT-07 | Admin-only human access; scheduled job permitted | UC-05 View Management Reports, UC-06, UC-08 | AT-RPT-01, AT-RPT-06 |
| FR-RPT-08 | Report content and format | UC-08 | AT-RPT-01, AT-RPT-09 |
| NFR-PERF-01 | Reservation ≤ 2 s (95%) / ≤ 7 s (100%) | UC-16 | AT-RES-07, AT-RES-11 |
| NFR-PERF-02 | Push ≤ 1 s | UC-16, UC-26, UC-19 Monitor Event Status | AT-RES-01, AT-RES-08, AT-RES-22, AT-RES-26 |
| NFR-PERF-03 | SLA report ≤ 10 s for 10,000 events | UC-08 | AT-RPT-07 |
| NFR-SEC-01 | Server-side role authorization (100%) | UC-16, UC-26, UC-05, UC-08 | AT-RES-01, AT-RES-08, AT-RES-09, AT-RES-19, AT-RPT-02, AT-RPT-06 |
| NFR-SEC-02 | TLS 1.2+ for both operations | UC-16, UC-26, UC-05, UC-08 | AT-RES-13 |
| NFR-REL-01 | 0 double reservations under 100 concurrent requests | UC-17 | AT-RES-11 |
| NFR-REL-02 | Lock timeout 5 s → abort, 0 new reservations | UC-16, UC-17 | AT-RES-07 |
| NFR-REL-03 | Crash → no partial bundle | UC-16, UC-26 | AT-RES-10, AT-RES-20, AT-RES-21 |
| NFR-REL-04 | Reconnect resync ≤ 3 s | UC-19, UC-26 | AT-RES-12 |
| NFR-USA-01 | Reservation in ≤ 3 user actions | UC-16 | AT-RES-14 |
| NFR-USA-02 | State color + label, contrast ≥ 4.5:1 | UC-19 | AT-RES-14 |
| NFR-USA-03 | SLA report in ≤ 2 user actions | UC-05 | AT-RPT-08 |

---

## 6. Acceptance Tests (BDD - Given / When / Then)

Each scenario uses the same ID as the RTM. These are acceptance specifications, not records of executed tests. Each scenario starts with independent test data. Unless a scenario states otherwise, selected bundles are complete, all unrelated resources are Available, and no background action changes the tested data.

### 6.1 Operation 1 - Resource reservation and task completion

```gherkin
Feature: Reserve resources for a task and report its completion

  Background:
    Given event "EV-100" is an Open building-collapse event in Haifa
    And Dispatcher "D1" is logged in
    And task "T-1" belongs to "EV-100" and requires this bundle
      | role or equipment type   | quantity |
      | Field Commander          | 1        |
      | Heavy Equipment Operator | 1        |
      | Bulldozer                | 1        |
      | Medic                    | 1        |
    And the matching resources are "FC-7", "OP-3", "BD-2" and "MD-5"

  @AT-RES-01
  Scenario: AT-RES-01 - Reserve a complete available bundle
    Given task "T-1" is Ready and all four resources are Available
    When "D1" confirms the bundle for "T-1"
    Then all four resources are Reserved for "T-1"
    And "T-1" becomes Active
    And all relevant connected stations receive the permitted updates within 1 second of commit without polling
    And the Audit Log records the reservation, user "D1", the task and resource changes, and a timestamp to millisecond precision

  @AT-RES-02
  Scenario Outline: AT-RES-02 - Reject a new reservation when the task is not Ready
    Given "T-1" is in state "<state>"
    When "D1" requests a new bundle for "T-1"
    Then the system rejects the request and explains that the task is not Ready
    And the task and its existing assignments remain unchanged
    And no new resources are reserved
    And the screen shows the current database state
    Examples:
      | state     |
      | Blocked   |
      | Active    |
      | Completed |
      | Canceled  |

  @AT-RES-03
  Scenario: AT-RES-03 - Two Dispatchers try to reserve the same bulldozer
    Given "D1" and "D2" are logged in at different stations
    And their Ready tasks require separate available bundles except for bulldozer "BD-2"
    And "BD-2" is Available
    And neither task releases its resources during the test
    When both Dispatchers request their bundles at the same time
    Then database tracing shows that each request locks its event record, then its task, then its resource records in ascending ID order, before availability checks and updates
    And each reservation attempt uses one SQL transaction
    And exactly one reservation succeeds
    And "BD-2" remains Reserved for the successful task
    And the other task remains Ready with no resources reserved by its failed request
    And the failed request leaves all other committed data unchanged
    And the unsuccessful Dispatcher sees that "BD-2" is Reserved for another task
    And that screen refreshes to the current resource state
    And the Audit Log records the rejected request, Dispatcher ID and a timestamp to millisecond precision

  @AT-RES-04
  Scenario Outline: AT-RES-04 - Reject an unavailable resource without a partial reservation
    Given "T-1" is Ready and the requested bundle matches its requirements
    And resource "<id>" has status "<status>"
    And the other selected resources are Available
    When "D1" requests the complete bundle
    Then the request is rejected and "T-1" remains Ready
    And none of the resources becomes Reserved for "T-1"
    And the screen identifies "<id>", explains its status and refreshes the selected resources
    Examples:
      | id   | status                   |
      | BD-2 | Faulty/Under Maintenance |
      | MD-5 | Unavailable              |

  @AT-RES-05
  Scenario Outline: AT-RES-05 - Require exactly one Field Commander
    Given "T-1" is Ready
    And the selected operator, bulldozer and medic are Available
    When "D1" submits the bundle with <count> Field Commanders
    Then the system explains that exactly one Field Commander is required
    And "T-1" remains Ready and no resources are reserved
    Examples:
      | count |
      | 0     |
      | 2     |

  @AT-RES-06
  Scenario: AT-RES-06 - Prevent double reservation of a Field Commander
    Given "T-1" is Ready
    And "FC-7" is already Reserved for another task
    When "D1" requests a bundle containing "FC-7" for "T-1"
    Then the entire reservation is rejected
    And the system identifies "FC-7" as already Reserved
    And "FC-7" remains assigned to the original task
    And "FC-7" can still log in and view that task

  @AT-RES-07
  Scenario: AT-RES-07 - Recover from a database lock timeout
    Given "T-1" is Ready and all selected resources are Available
    And another transaction holds the database lock on "BD-2"
    When "D1" requests the bundle and the lock remains held for 5 seconds
    Then the reservation transaction is aborted
    And all locks held by that transaction are released within 1 second of the abort
    And no resource is reserved by the failed request
    And the screen refreshes to the current database state
    And "D1" sees a timeout message within 7 seconds of submission
    When the other transaction releases its lock and "D1" retries
    Then the bundle is reserved successfully and "T-1" becomes Active

  @AT-RES-08
  Scenario: AT-RES-08 - Complete a task and enable its dependent task
    Given "T-1" is Active with "FC-7", "OP-3", "BD-2" and "MD-5" Reserved for it
    And "T-3" is Blocked and its only prerequisite is "T-1"
    And the assigned Field Commander "FC-7" is logged in
    When "FC-7" reports that "T-1" is complete
    Then "T-1" becomes Completed
    And all four resources become Available
    And "T-3" becomes Ready
    And all relevant connected stations receive the permitted updates within 1 second of commit without polling
    And the Audit Log records the completion, user "FC-7", the task and resource changes, and a timestamp to millisecond precision

  @AT-RES-09
  Scenario: AT-RES-09 - Reject completion by a different Field Commander
    Given "T-1" is Active and its assigned Field Commander is "FC-7"
    And another Field Commander "FC-8" is logged in
    When "FC-8" reports that "T-1" is complete
    Then the server rejects the request as unauthorized within 1 second
    And "T-1" remains Active and its resources remain Reserved
    And the Audit Log records the rejected completion, user "FC-8" and a timestamp to millisecond precision within 1 second

  @AT-RES-10
  Scenario: AT-RES-10 - Recover from a crash before reservation is committed
    Given "T-1" is Ready and all four selected resources are Available
    When the server crashes after updating two resources but before committing the reservation
    And the server restarts
    Then all four resources are Available and "T-1" is Ready
    When "D1" reconnects and requests the bundle again
    Then the complete reservation succeeds

  @AT-RES-11
  Scenario: AT-RES-11 - Meet response times and prevent double reservation under load
    Given 20 Dispatcher clients each have 50 Ready tasks with separate complete available bundles
    When all clients start together and each sends its next request after receiving the previous result
    Then at least 950 of the 1000 results are displayed within 2 seconds of submission
    And all 1000 results are displayed within 7 seconds of submission
    Given 100 other Ready tasks have valid available bundles sharing only bulldozer "BD-2"
    And no task completes or releases "BD-2" during this test
    When 100 reservation requests for these tasks arrive at the same time
    Then exactly one request succeeds
    And the database contains zero resources reserved for more than one task

  @AT-RES-12
  Scenario: AT-RES-12 - Refresh the event screen after reconnection
    Given "D1" is viewing "EV-100" and the station loses its network connection
    And "FC-7" completes "T-1" from another connected station during the interruption
    When "D1" reconnects
    Then the event screen shows "T-1" as Completed within 3 seconds
    And the screen shows the current resource and dependent-task states

  @AT-RES-13
  Scenario: AT-RES-13 - Encrypt reservation, completion and report traffic
    Given network traffic is being captured for the Dispatcher, assigned Commander and Administrator
    When they reserve a bundle, complete the task, generate an SLA report and open it
    Then all messages for these operations use TLS 1.2 or higher
    And the connections use AES-128-GCM or AES-256-GCM encryption
    And task IDs, resource IDs and report data are not readable in the captured network traffic

  @AT-RES-14
  Scenario: AT-RES-14 - Reserve a suggested bundle in three actions
    Given "D1" is on the event status screen
    And the screen contains tasks in all five task states
    And "T-1" is Ready with a valid suggested bundle
    When "D1" selects "T-1", chooses "Reserve Bundle" and confirms the selection
    Then the reservation succeeds using no more than 3 user actions
    And each of the five task states has a distinct color and a text label
    And each state label has a text-to-background contrast ratio of at least 4.5:1

  @AT-RES-15
  Scenario: AT-RES-15 - Prevent two bundles from activating the same task
    Given two Dispatchers see the same Ready task
    And each selects a different complete available bundle with no shared resources
    When both submit their reservations at the same time
    Then exactly one complete bundle is reserved for the task
    And the task becomes Active
    And the failed request reserves no resources and does not undo the successful reservation

  @AT-RES-16
  Scenario Outline: AT-RES-16 - Match all required resource types and quantities
    Given "T-1" is Ready and all selected resources are Available
    When "D1" submits a bundle with "<error>"
    Then the system identifies the mismatch and rejects the entire bundle
    And "T-1" remains Ready and no resources are reserved
    Examples:
      | error                                                 |
      | a generator instead of the required bulldozer         |
      | no medic                                              |
      | two medics instead of the required one                |
      | a firefighter instead of the heavy equipment operator |

  @AT-RES-17
  Scenario Outline: AT-RES-17 - Allow completion only for an Active task
    Given "FC-7" is logged in
    And "T-1" has state "<state>"
    And "FC-7" is the assigned or previously assigned commander if the task has an assignment
    When "FC-7" requests completion of "T-1"
    Then the request is rejected
    And the task state and existing assignments remain unchanged
    And the Audit Log records the rejected completion, user "FC-7" and a timestamp to millisecond precision
    Examples:
      | state     |
      | Blocked   |
      | Ready     |
      | Completed |
      | Canceled  |

  @AT-RES-18
  Scenario: AT-RES-18 - Preserve faulty equipment status and check every prerequisite
    Given "T-1" is Active under "FC-7"
    And bulldozer "BD-2" failed earlier and was replaced by "BD-3"
    And "BD-2" is Faulty/Under Maintenance and "BD-3" is currently Reserved for "T-1"
    And Blocked task "T-3" depends only on "T-1"
    And Blocked task "T-4" depends on "T-1" and another unfinished task
    When "FC-7" completes "T-1"
    Then "BD-3" and the other currently assigned resources become Available
    And "BD-2" remains Faulty/Under Maintenance and its assignment history is retained
    And "T-3" becomes Ready while "T-4" remains Blocked

  @AT-RES-19
  Scenario Outline: AT-RES-19 - Enforce permissions at the server
    Given the requester is "<caller>"
    When the requester directly submits a "<operation>" request to the server
    Then the server denies the request within 1 second without changing task or resource data
    And it records the rejection within 1 second with a timestamp to millisecond precision
    And the record includes the user ID if authenticated, or marks the request as unauthenticated
    Examples:
      | caller                    | operation   |
      | a Field Crew Member       | reservation |
      | a Field Crew Member       | completion  |
      | a System Administrator    | reservation |
      | a System Administrator    | completion  |
      | a Logistics Officer       | reservation |
      | a Logistics Officer       | completion  |
      | a Field Commander         | reservation |
      | a Dispatcher              | completion  |
      | an unauthenticated client | reservation |
      | an unauthenticated client | completion  |

  @AT-RES-20
  Scenario: AT-RES-20 - Undo an interrupted completion
    Given "T-1" is Active and has a complete reserved bundle
    And "T-3" is Blocked and depends only on "T-1"
    When the server crashes after releasing one resource but before committing completion
    And the server restarts
    Then "T-1" is still Active and its original resources remain Reserved
    And "T-3" remains Blocked
    When the assigned Commander reconnects and reports completion again
    Then "T-1" becomes Completed, its resources become Available and "T-3" becomes Ready

  @AT-RES-21
  Scenario Outline: AT-RES-21 - Keep a committed result after the response is lost
    Given a valid "<operation>" has been committed to the database
    And the server crashes before the client receives the response
    When the server restarts and the client reconnects
    Then the task and resources show the committed result
    When the client repeats the original request
    Then the saved result is not undone and no duplicate assignment or state change is created
    Examples:
      | operation   |
      | reservation |
      | completion  |

  @AT-RES-22
  Scenario: AT-RES-22 - Send live updates only to authorized users
    Given a Dispatcher, the assigned Commander and assigned crew are connected
    And an unrelated crew member is also connected
    When the assigned Commander completes the task
    Then the Dispatcher and assigned Commander receive the permitted event updates within 1 second of commit
    And the assigned crew receive their permitted task updates within 1 second of commit
    And no client needs to poll for these updates
    And the unrelated crew member receives no data about that task or event

  @AT-RES-23
  Scenario: AT-RES-23 - Allocate a complete bundle automatically
    Given "T-1" is Ready and a complete matching bundle is Available
    And the Dispatcher selects the nearest-resources criterion
    When "D1" requests automatic allocation for "T-1"
    Then the system selects a matching bundle using that criterion
    And it reserves the complete bundle in one SQL transaction
    And "T-1" becomes Active with exactly one assigned Field Commander

  @AT-RES-24
  Scenario Outline: AT-RES-24 - Replace only the taken resource after a conflict
    Given "T-1" is Ready and automatic allocation has selected "FC-7", "OP-3", "BD-2" and "MD-5"
    And another Dispatcher reserves "BD-2" before this allocation commits
    And <situation>
    When the system handles the conflict
    Then no partial reservation from a failed attempt remains
    And each new attempt keeps "FC-7", "OP-3" and "MD-5" and replaces only the taken bulldozer
    And each attempt reserves the whole bundle in one all-or-nothing transaction
    And the system makes no more than two alternative attempts, without a new Dispatcher request
    And another Dispatcher can reserve an unrelated available bundle while this request is handled
    And the outcome is: <outcome>
    Examples:
      | situation                                                   | outcome                                                                                     |
      | bulldozer "BD-3" is Available                               | "T-1" is Active with "BD-3" replacing "BD-2"                                                |
      | "BD-3" is also taken before commit, and "BD-4" is Available | the second alternative attempt reserves "BD-4" and "T-1" is Active                          |
      | no other bulldozer is Available                             | "T-1" remains Ready and the Dispatcher is told that no alternative bulldozer is available   |
      | "BD-3" and then "BD-4" are both taken by competing requests | "T-1" remains Ready and the Dispatcher is told that allocation failed because of contention |

  @AT-RES-25
  Scenario Outline: AT-RES-25 - Block completion while a required resource is missing
    Given "T-1" is Active under "FC-7"
    And bulldozer "BD-2" was reported Faulty and released from "T-1", which stays Active
    And <dispatcher action>
    When "FC-7" reports that "T-1" is complete
    Then <result>
    Examples:
      | dispatcher action                                               | result                                                                               |
      | no replacement is assigned and the bulldozer was not waived     | the report is rejected, "T-1" remains Active and the missing bulldozer is identified |
      | the Dispatcher assigned replacement bulldozer "BD-3"            | "T-1" becomes Completed and "BD-3" becomes Available                                 |
      | the Dispatcher confirmed that the bulldozer is no longer needed | "T-1" becomes Completed and "BD-2" remains Faulty/Under Maintenance                  |

  @AT-RES-26
  Scenario Outline: AT-RES-26 - Replace a resource released from an Active task
    Given "T-1" is Active with "FC-7", "OP-3", "BD-2" and "MD-5" Reserved for it
    And bulldozer "BD-3" and Field Commander "FC-9" are Available
    When <release>
    Then "<released>" has status "<status>" and is no longer assigned to "T-1"
    And its assignment record has an end time and its assigned hours are kept for cost and utilization calculations
    And "T-1" remains Active and its other resources remain Reserved for it
    And "D1" receives an alert within 1 second of commit to assign a replacement <slot>
    When "D1" assigns "<replacement>" to the missing slot
    Then "<replacement>" is Reserved for "T-1" and "T-1" remains Active
    And <commander result>
    Examples:
      | release                                                            | released | status                   | slot            | replacement | commander result                                                         |
      | crew member "MD-5" reports that bulldozer "BD-2" has malfunctioned | BD-2     | Faulty/Under Maintenance | bulldozer       | BD-3        | "FC-7" remains the commander of "T-1"                                    |
      | the Logistics Officer marks "FC-7" Unavailable because of sickness | FC-7     | Unavailable              | Field Commander | FC-9        | "FC-9" can report completion of "T-1" and "FC-7" can no longer report it |

  @AT-RES-27
  Scenario Outline: AT-RES-27 - Reject an invalid replacement or a self-reported unavailability
    Given "T-1" is Active and bulldozer "BD-2" was released from it after a reported malfunction
    And "FC-7", "OP-3" and "MD-5" remain Reserved for "T-1"
    When <request>
    Then the request is rejected and the reason states that <reason>
    And "T-1" remains Active with its existing assignments unchanged
    And no resource is newly Reserved for "T-1"
    Examples:
      | request                                                                           | reason                                                                 |
      | "D1" selects generator "GN-1" as the replacement                                  | a bulldozer is required                                                |
      | "D1" selects medic "MD-6" although no medic slot is missing                       | no medic slot is missing                                               |
      | another Dispatcher reserves "BD-3" just before "D1" commits it as the replacement | "BD-3" is already Reserved for another task                            |
      | "MD-5" reports that they are Unavailable                                          | only the Logistics Officer can change a reserved member's availability |

```

### 6.2 Operation 2 - SLA report

The report tests use the selection policy in A-SLA (course clarification). Unless stated otherwise, all times below are local server times, all tasks of the listed events are final, and each listed event has at least one Completed task.

```gherkin
Feature: Generate and view the monthly SLA report

  Background:
    Given the server time zone is "Asia/Jerusalem"
    And System Administrator "ADM1" is logged in
    And the following events exist
      | event | type          | opened           | last Completed task | finally closed   | event status |
      | EV-1  | Fire          | 2026-10-03 10:00 | 2026-10-03 14:00    | 2026-10-03 15:00 | Closed       |
      | EV-2  | Fire          | 2026-10-10 08:00 | 2026-10-10 10:00    | 2026-10-10 11:00 | Closed       |
      | EV-3  | Road Accident | 2026-10-12 09:00 | 2026-10-12 10:30    | 2026-10-12 11:30 | Closed       |
      | EV-4  | Road Accident | 2026-10-20 12:00 | 2026-10-20 12:20    | -                | Canceled     |
    And no other events are included unless a scenario adds them

  @AT-RPT-01
  Scenario: AT-RPT-01 - Generate the previous month's report automatically
    Given no SLA report exists for October 2026
    When the System Clock reaches 2026-11-01 00:00
    Then the system starts generating the October 2026 SLA report
    And the completed report is stored with these rows
      | event type        | events | average SLA time |
      | Building Collapse | 0      | No data          |
      | Fire              | 2      | 03:00            |
      | Road Accident     | 1      | 01:30            |
      | HazMat Leak       | 0      | No data          |
      | Natural Disaster  | 0      | No data          |
    And the report shows "October 2026" and the generation timestamp
    And the monthly process also generates the resource utilization report

  @AT-RPT-02
  Scenario Outline: AT-RPT-02 - Generate a report only for a completed past month
    Given the current date is 2026-11-07
    When "ADM1" requests the SLA report for "<month>"
    Then the system returns "<result>"
    And the manual SLA request does not start a resource utilization report
    Examples:
      | month         | result                                               |
      | October 2026  | a report showing Fire 03:00 and Road Accident 01:30  |
      | November 2026 | an error asking the user to select a completed month |
      | December 2026 | an error asking the user to select a completed month |

  @AT-RPT-03
  Scenario: AT-RPT-03 - Show No data when a type has no eligible events
    Given no HazMat Leak event qualifies for October under A-SLA
    When the October SLA report is generated
    Then the HazMat Leak row shows zero events and "No data"
    And it does not show an average of "00:00"

  @AT-RPT-04
  Scenario: AT-RPT-04 - Exclude a canceled event under A-SLA
    When the October SLA report is generated
    Then "EV-4" is not included
    And the Road Accident row shows 1 event and an average of "01:30"

  @AT-RPT-05
  Scenario Outline: AT-RPT-05 - Recover a missed or interrupted monthly report
    Given the server stopped before the October SLA report was completed
    And the report state is "<state>"
    When the server restarts on 1 November and checks for missing or unfinished reports
    Then it generates and stores one complete October SLA report
    And a second restart does not create a duplicate report
    Examples:
      | state                                |
      | not started                          |
      | started but no complete result saved |

  @AT-RPT-06
  Scenario Outline: AT-RPT-06 - Restrict report access to the Administrator
    Given the requester is "<caller>"
    When the requester asks to "<action>" the October SLA report
    Then the server denies the request within 1 second
    And it returns no report data and starts no report generation
    And it records the rejection in the Audit Log within 1 second
    Examples:
      | caller                    | action   |
      | a Dispatcher              | view     |
      | a Dispatcher              | generate |
      | a Logistics Officer       | view     |
      | a Logistics Officer       | generate |
      | a Field Commander         | view     |
      | a Field Commander         | generate |
      | a Field Crew Member       | view     |
      | a Field Crew Member       | generate |
      | an unauthenticated client | view     |
      | an unauthenticated client | generate |

  @AT-RPT-07
  Scenario: AT-RPT-07 - Generate a report for a large month within the time limit
    Given the database contains 10000 events eligible for September 2026 under A-SLA
    When "ADM1" requests the September SLA report
    Then the completed report is displayed within 10 seconds of the request

  @AT-RPT-08
  Scenario: AT-RPT-08 - Open a stored report in two actions
    Given "ADM1" is on the Administrator main screen
    And the October SLA report is stored
    When "ADM1" chooses "Reports" and selects "SLA - October 2026"
    Then the report opens using no more than 2 user actions

  @AT-RPT-09
  Scenario: AT-RPT-09 - Count by final closure month and measure to the last Completed task
    Given Natural Disaster event "EV-5" opens on 30 October at 20:00
    And its last Completed task is completed on 31 October at 22:01:20
    And another of its tasks is Canceled on 31 October at 23:30
    And "EV-5" is In Settlement at the end of October and is finally Closed on 1 November at 09:00
    And Natural Disaster event "EV-6" is finally Closed in November and all its tasks are Canceled
    And no other Natural Disaster event is closed in October or November
    When the October SLA report is generated at the start of November
    Then "EV-5" is not included, because it was not finally Closed in October
    And the Natural Disaster row shows zero events and "No data"
    When the November SLA report is generated after November ends
    Then "EV-5" is included in November, the month of its final closure
    And its SLA time ends at its last Completed task, not at the Canceled task or at the closure
    And the Natural Disaster row shows 1 event with an average of "26:01"
    And "EV-6" is excluded because it has no Completed task
```
