# CrisisSync - Use Case Specifications (Group G12)

Text copy of the specifications stored in `G12_Assignment1.vpp`.
In Visual Paradigm they are entered in Use Case Details: preconditions and postconditions in the *Details* tab, the numbered main flow in the *Flow of Events* tab, and the extensions in its *Extension* pane.

## Actors

| ID | Actor | Kind |
|---|---|---|
| AC01 | Registered User | Human, abstract |
| AC02 | System Administrator | Human |
| AC03 | Logistics Officer | Human |
| AC04 | Dispatcher | Human |
| AC05 | Field Commander | Human |
| AC06 | Field Crew Member | Human |
| AC07 | National Clearing & Settlement System | Non-human (right side) |
| AC08 | System Clock | Non-human (right side) |

## Extension points and conditions

| Extension | Base use case | Extension point | Condition |
|---|---|---|---|
| UC-16a Handle Reservation Conflict | UC-16 Reserve Resource Bundle | Availability check | A requested resource is unavailable. |
| UC-18a Retry With an Alternative Bundle | UC-18 Auto-Allocate Resource Bundle | Candidate conflict | A competing allocation took a candidate and fewer than two alternatives have been attempted. |
| UC-20a Cancel Dependent Tasks | UC-20 Cancel Task | Cancellation scope | At least one direct or indirect dependent is non-final. |
| UC-22a Handle Settlement Rejection | UC-22 Close Event | Settlement outcome | The external system explicitly rejected settlement. |

## Use case index

| ID | Use case | Actors | Relationships |
|---|---|---|---|
| UC-01 | Log In | Registered User | - |
| UC-02 | Manage User Accounts and Permissions | System Administrator | - |
| UC-03 | Load Initial Data | System Administrator | - |
| UC-04 | View Event History | System Administrator | - |
| UC-05 | View Management Reports | System Administrator | - |
| UC-06 | Generate Monthly Reports | System Clock | UC-07 Generate Resource Utilization Report; UC-08 Generate SLA Report |
| UC-07 | Generate Resource Utilization Report | System Administrator | included by UC-06 Generate Monthly Reports |
| UC-08 | Generate SLA Report | System Administrator | included by UC-06 Generate Monthly Reports |
| UC-10 | Update Equipment Maintenance Status | Logistics Officer | - |
| UC-11 | Transfer Equipment Between Stations | Logistics Officer | - |
| UC-12 | Update Crew Availability | Logistics Officer | - |
| UC-13 | Open Event | Dispatcher | - |
| UC-14 | Create Task | Dispatcher | - |
| UC-15 | Edit Task Dependencies | Dispatcher | - |
| UC-16 | Reserve Resource Bundle | Dispatcher | UC-17 Validate Resource Reservation; extended by UC-16a Handle Reservation Conflict |
| UC-16a | Handle Reservation Conflict | - | extends UC-16 Reserve Resource Bundle |
| UC-17 | Validate Resource Reservation | - | included by UC-16 Reserve Resource Bundle; included by UC-18 Auto-Allocate Resource Bundle; included by UC-25 Assign Replacement Equipment |
| UC-18 | Auto-Allocate Resource Bundle | Dispatcher | UC-17 Validate Resource Reservation; extended by UC-18a Retry With an Alternative Bundle |
| UC-18a | Retry With an Alternative Bundle | - | extends UC-18 Auto-Allocate Resource Bundle |
| UC-19 | Monitor Event Status | Dispatcher, Field Commander | - |
| UC-20 | Cancel Task | Dispatcher | extended by UC-20a Cancel Dependent Tasks |
| UC-20a | Cancel Dependent Tasks | - | extends UC-20 Cancel Task |
| UC-21 | Cancel Event | Dispatcher | - |
| UC-22 | Close Event | Dispatcher | UC-23 Calculate Operating Costs; UC-24 Request Settlement; extended by UC-22a Handle Settlement Rejection |
| UC-22a | Handle Settlement Rejection | - | extends UC-22 Close Event |
| UC-23 | Calculate Operating Costs | - | included by UC-22 Close Event |
| UC-24 | Request Settlement | National Clearing & Settlement System | included by UC-22 Close Event |
| UC-25 | Assign Replacement Equipment | Dispatcher | UC-17 Validate Resource Reservation |
| UC-26 | Report Task Completion | Field Commander | UC-27 Release Resources and Promote Dependent Tasks |
| UC-27 | Release Resources and Promote Dependent Tasks | - | included by UC-26 Report Task Completion |
| UC-28 | View Assigned Task Details | Field Crew Member | - |
| UC-29 | Report Shift Start and Readiness | Field Crew Member | - |
| UC-30 | Report Equipment Malfunction | Field Commander, Field Crew Member | - |

## UC-01 Log In

**Actors:** Registered User

**Preconditions:** The client can reach the server.

**Postconditions - success:** A session opens with the permissions of the authenticated user.

**Postconditions - failure:** No authenticated session is opened for this attempt.

**Main flow**

1. The user enters a username and password.
2. The server verifies the credentials.
3. The system opens the role-specific interface.

**Extensions**

- **2a. Invalid credentials:** Show a generic login error. Return to step 1.
- **2b. Server unavailable:** Show the connection error and end the attempt.

## UC-02 Manage User Accounts and Permissions

**Actors:** System Administrator

**Preconditions:** An administrator is signed in.

**Postconditions - success:** The approved account or permission changes are stored.

**Postconditions - failure:** The requested changes are not applied.

**Main flow**

1. The administrator selects an account to manage or starts a new account.
2. The system displays the account details and available roles.
3. The administrator submits the account and permission changes.
4. The server verifies the administrator's authority, validates the changes and saves them.

**Extensions**

- **4a. Invalid account data or role:** Identify the invalid field. Return to step 3.

## UC-03 Load Initial Data

**Actors:** System Administrator

**Preconditions:** An administrator is signed in and has an initial data file in a supported format.

**Postconditions - success:** The initial users, personnel and equipment are stored.

**Postconditions - failure:** The failed import leaves no partial data from this attempt.

**Main flow**

1. The administrator opens the import utility and selects a data file.
2. The system validates the records, identifiers, equipment types and personnel roles.
3. The system commits the import and displays a record count.

**Extensions**

- **2a. Invalid data or duplicate identifier:** Identify the offending records and reject the import. Return to step 1.
- **3a. Database write fails:** Roll back the import and report the failure. End the attempt.

## UC-04 View Event History

**Actors:** System Administrator

**Preconditions:** An administrator is signed in.

**Postconditions - success:** The administrator sees the selected historical records in read-only form.

**Postconditions - failure:** No history entry is modified or disclosed to an unauthorized requester.

**Main flow**

1. The administrator opens event history and selects date, type or status filters.
2. The system lists matching open, closed and canceled events.
3. The administrator selects an event.
4. The system displays its tasks, resource assignments, settlement approvals and timestamped action timeline, including the acting user.

**Extensions**

- **2a. No events match:** Show an empty result and allow the filters to be changed. Return to step 1.

## UC-05 View Management Reports

**Actors:** System Administrator

**Preconditions:** An administrator is signed in.

**Postconditions - success:** The selected stored report is displayed for the chosen month.

**Postconditions - failure:** No unavailable or unauthorized report is displayed.

**Main flow**

1. The administrator opens the reports view.
2. The system lists available months and report types.
3. The administrator selects a stored report.
4. The server verifies access and displays the report.

**Extensions**

- **4a. Report unavailable:** Show its availability status and return to step 2.
- **4b. Session invalid or role unauthorized:** Reject the request without returning report data. End the attempt.

## UC-06 Generate Monthly Reports

**Actors:** System Clock

**Preconditions:** A calendar month has ended according to the server clock.

**Postconditions - success:** A utilization report and an SLA report are available for the completed month.

**Postconditions - failure:** A missing report can be identified and generated on recovery.

**Main flow**

1. The system clock signals the monthly boundary.
2. The system identifies the completed month.
3. The system generates the utilization report for that month (UC-07).
4. The system generates the SLA report for that month (UC-08).
5. The system makes each completed report available to the administrator.

**Extensions**

- **1a. Server was down at the boundary:** On startup, compare elapsed months since service began with stored month/type keys. Generate each missing report once, from step 3 or 4 as applicable.
- **3a. Utilization generation interrupted:** Retain any already completed report. Retry only missing or unfinished work on recovery. End this run.
- **4a. SLA generation interrupted:** Retain any already completed report. Retry only missing or unfinished work on recovery. End this run.

## UC-07 Generate Resource Utilization Report

**Actors:** System Administrator

**Preconditions:** The monthly job supplies a completed month or a signed-in administrator requests this report for a past month.

**Postconditions - success:** The report shows reserved time as a percentage of the month, grouped by equipment type and personnel role.

**Postconditions - failure:** No incomplete result is published.

**Main flow**

1. The administrator selects a past month and this report, or the monthly job supplies the month.
2. The server validates the month and the requester's authority.
3. The system clips reservation intervals to the month and calculates utilization relative to time in that month.
4. The system produces a graphical breakdown by equipment type and personnel role and stores the result for viewing.

**Extensions**

- **2a. Current or future month selected manually:** Ask the administrator to select a completed month. Return to step 1.
- **2b. Unauthorized requester:** Deny the request. End the attempt.

## UC-08 Generate SLA Report

**Actors:** System Administrator

**Preconditions:** The monthly job supplies a completed month or a signed-in administrator requests this report for a past month.

**Postconditions - success:** An SLA result for the month is stored with counts and average elapsed times by event type.

**Postconditions - failure:** No partial report replaces a completed result.

**Main flow**

1. The administrator selects a past month and the SLA report, or the monthly job supplies the month.
2. The server validates the month and requester.
3. The system selects eligible events under reporting policy A-SLA.
4. For each eligible event, the system subtracts the opening time from the last task's final timestamp.
5. The system averages the durations separately for each event type and records the count.
6. The system stores the report with its month and generation time and makes it available to the administrator.

**Extensions**

- **2a. Invalid month or unauthorized request:** Explain an invalid month to the administrator or deny unauthorized access. End the attempt.
- **5a. An event type has no eligible events:** Display a count of zero and No data for its average. Continue at step 6.
- **6a. Generation interrupted:** Keep completed data intact. Scheduled work is reconciled at startup (UC-06). End this run.

## UC-10 Update Equipment Maintenance Status

**Actors:** Logistics Officer

**Preconditions:** A logistics officer is signed in.

**Postconditions - success:** The item's maintenance status is updated, audited and pushed to relevant stations.

**Postconditions - failure:** No conflicting transition or reservation change is committed.

**Main flow**

1. The officer selects an equipment item and its maintenance status.
2. The server validates the current item status and the requested transition.
3. The system records the change, including a repaired item's return to Available, and pushes the update.

**Extensions**

- **2a. A live reservation would be overwritten:** Show the current assignment. An in-task breakdown is reported through UC-30. End the attempt.

## UC-11 Transfer Equipment Between Stations

**Actors:** Logistics Officer

**Preconditions:** A logistics officer is signed in.

**Postconditions - success:** The equipment location is updated and the change is audited and pushed.

**Postconditions - failure:** The failed request does not change the recorded location.

**Main flow**

1. The officer selects an item and a destination station.
2. The server verifies the item and destination.
3. The system records the new station and publishes the update.

**Extensions**

- **2a. Unknown item or destination:** Show the error and return to step 1.

## UC-12 Update Crew Availability

**Actors:** Logistics Officer

**Preconditions:** A logistics officer is signed in.

**Postconditions - success:** The exceptional availability change is recorded and published.

**Postconditions - failure:** The request does not overwrite another task's reservation.

**Main flow**

1. The officer selects a crew member and enters an availability change, for example sickness or leave.
2. The server checks the current state against any live assignment.
3. The system records the availability and publishes the update.

**Extensions**

- **2a. The change conflicts with a current reservation:** Preserve the live assignment and show the conflict. End the attempt.

## UC-13 Open Event

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in.

**Postconditions - success:** An open event exists with a server-recorded opening time and a live status view.

**Postconditions - failure:** The failed request creates no event.

**Main flow**

1. The dispatcher enters an event name, location and one of the five event types.
2. The server validates the fields.
3. The system records the opening time, audits the opening and displays the live event.

**Extensions**

- **2a. Missing field or unknown type:** Identify the field and return to step 1.

## UC-14 Create Task

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in and the event is open.

**Postconditions - success:** The task belongs to the event, with a required bundle and an initial state of Blocked or Ready.

**Postconditions - failure:** No invalid task, prerequisite edge or bundle is created.

**Main flow**

1. The dispatcher enters a name, estimated duration, prerequisite tasks and required resource bundle.
2. The system checks the event, task data and exactly one required Field Commander.
3. The system validates same-event prerequisites and verifies that the graph remains acyclic.
4. The server creates the task as Blocked if a prerequisite is still open, or Ready if every prerequisite is Completed, including when there are none.
5. The system audits the new state and pushes the updated task tree.

**Extensions**

- **2a. Invalid bundle or task data:** Identify the missing or invalid value. Return to step 1.
- **3a. Cycle, foreign event or canceled prerequisite:** Reject the selection if it creates a cycle, refers to a task from another event, or includes a canceled prerequisite. Return to step 1.

## UC-15 Edit Task Dependencies

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in and selects an existing task in an open event.

**Postconditions - success:** The event remains a DAG. A Blocked task becomes Ready when all its predecessors are Completed.

**Postconditions - failure:** The rejected edit leaves the committed graph intact.

**Main flow**

1. The dispatcher opens the task's prerequisites.
2. The system checks that the task is Blocked.
3. The dispatcher submits the revised prerequisite set.
4. Under a concurrency guard, the server rechecks the task state, same-event membership, candidate states and the absence of cycles.
5. The server saves the graph, promotes the task if eligible, audits state changes and pushes the update.

**Extensions**

- **2a. Task is Ready, Active, Completed or Canceled:** Explain that its prerequisites are locked. End the attempt.
- **4a. The proposed edge creates a cycle:** Reject the edit and identify the conflicting dependency. Return to step 3.
- **4b. The state changed since editing began:** Reject the edit, refresh the task and end the attempt.
- **4c. Foreign or canceled prerequisite:** Reject the selection if a prerequisite belongs to another event or is already Canceled. Return to step 3.

## UC-16 Reserve Resource Bundle

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in and the task was shown as Ready.

**Postconditions - success:** The complete bundle is reserved for the task, the task is Active and relevant stations receive the committed update.

**Postconditions - failure:** This request creates no reservation. The screen shows the current committed state, including a concurrent winner.

**Main flow**

1. The dispatcher selects a Ready task and opens manual reservation.
2. The system shows the roles, equipment types and available matching candidates.
3. The dispatcher confirms a complete selection.
4. The server validates the reservation under SQL locks (UC-17).
5. The server commits every reservation and the Ready-to-Active task transition together, records the action and pushes the change.

**Extensions**

- **4a. A requested resource is no longer available:** Apply UC-16a at Availability check. Show the current state and return to step 2.
- **4b. Wrong task state, incomplete bundle, wrong type or commander count:** Reject the attempt and explain why. Return to step 1.
- **4c. Lock deadline exceeded:** Roll back this attempt, release its locks and refresh the screen. End the attempt.
- **5a. Commit outcome unknown to the client:** On reconnect read committed task and assignments before another attempt. End the attempt.

## UC-16a Handle Reservation Conflict

**Preconditions:** A manual attempt finds one or more unavailable resources at UC-16 step 4.

**Postconditions - success:** The losing attempt is rolled back. Existing reservations remain owned by their winning tasks.

**Postconditions - failure:** If the client disconnects, it must resynchronize before displaying a new selection.

**Main flow**

1. The server rolls back all writes made by the losing attempt.
2. The system refreshes the current resources and identifies the unavailable items and reasons to the dispatcher.
3. The dispatcher may revise the selection in UC-16.

**Extensions**

- **2a. Station disconnected:** On reconnect reload the current state before returning to UC-16 step 2.

## UC-17 Validate Resource Reservation

**Preconditions:** The caller is processing a manual bundle, automatic bundle or single replacement within a reservation transaction.

**Postconditions - success:** The caller holds the required locks and may commit the validated reservation.

**Postconditions - failure:** The caller rolls back this attempt without altering another task's resources.

**Main flow**

1. The caller requests a reservation check inside its transaction.
2. The server locks the task and requested resource rows and rereads their committed state.
3. For a new bundle the server verifies Ready, all predecessors Completed, exact bundle composition and one Field Commander. For a replacement it verifies Active and one unfilled equipment slot of the requested type.
4. The server verifies availability and exclusive ownership of every candidate and returns the result without an independent commit.

**Extensions**

- **2a. Lock deadline exceeded:** Abort the caller's attempt and release its locks. End this check.
- **3a. Task or selection changed:** Return the failed validation and current state to the caller. End this check.
- **4a. Unavailable resource:** Identify the conflicting resources for the caller's rollback or retry. End this check.

## UC-18 Auto-Allocate Resource Bundle

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in and the task was shown as Ready.

**Postconditions - success:** A complete bundle is reserved according to the selected criterion, and the task becomes Active.

**Postconditions - failure:** The unsuccessful request leaves no partial reservation.

**Main flow**

1. The dispatcher selects a Ready task, chooses an allocation criterion and requests automatic allocation.
2. The system ranks matching resources and selects a complete candidate bundle, for example by distance to the event.
3. The server validates the candidate bundle inside a transaction (UC-17).
4. The server commits the bundle and task activation, audits the change and pushes it to relevant stations.

**Extensions**

- **2a. No complete available bundle exists:** Report the missing requirement and refresh the current state. End the request.
- **3a. Another allocation won a selected resource:** Apply UC-18a at Candidate conflict to try the next best available bundle automatically, with up to two alternative attempts.
- **3b. Task no longer Ready:** Roll back this attempt, refresh the current state and end the request.

## UC-18a Retry With an Alternative Bundle

**Preconditions:** A competing allocation took a candidate and fewer than two alternative attempts have been made.

**Postconditions - success:** A complete alternative bundle is reserved, and the task becomes Active.

**Postconditions - failure:** No partial reservation remains. If no complete alternative bundle is available, the Dispatcher is notified. No more than two alternative attempts are made.

**Main flow**

1. The server rolls back the losing candidate and releases its locks.
2. The system selects the next best available complete bundle by the same criterion without asking the dispatcher to resubmit.
3. The server repeats UC-18 steps 3 and 4 using the next alternative, without blocking unrelated requests.

**Extensions**

- **2a. No complete alternative is available:** Report that no alternative is available and end the request.
- **3a. Second alternative also loses a race:** Roll back this attempt, leave the task Ready with no partial reservation, and notify the dispatcher that the allocation can be repeated. End the request.

## UC-19 Monitor Event Status

**Actors:** Dispatcher, Field Commander

**Preconditions:** A dispatcher or field commander is signed in.

**Postconditions - success:** The user sees the current status of events within their authority.

**Postconditions - failure:** An unauthorized requester receives no event data.

**Main flow**

1. The user requests the list of open events.
2. The server returns all open events to a dispatcher and only events with a task under the commander's responsibility to a commander.
3. The user selects an event.
4. The system displays name, type, location, time since opening, completed/total tasks and the dependency tree. For each task it shows state, commander, reserved resources and elapsed versus estimated duration.
5. The server pushes committed task and resource changes to authorized stations without client polling.

**Extensions**

- **4a. An Active task exceeds its estimate:** Highlight that task while displaying its elapsed time. Continue at step 5.
- **5a. The station reconnects:** Reload an authorized current snapshot before resuming updates. Continue at step 5.

## UC-20 Cancel Task

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in and selects a task shown in a non-final state.

**Postconditions - success:** The target and all non-final descendants covered by the confirmation are canceled and their live resources are released.

**Postconditions - failure:** No cancellation by this request is committed if confirmation is declined or obsolete.

**Main flow**

1. The dispatcher requests cancellation of a task.
2. The server checks its current state and finds its distinct non-final direct and indirect dependents.
3. The dispatcher confirms cancellation of the target.
4. The server rechecks the scope, cancels the target and releases its live assignments, with audit and push.

**Extensions**

- **2a. Cancel Dependent Tasks:** At Cancellation scope, UC-20a replaces steps 3 and 4. End UC-20 with the result of that single confirmation.
- **2b. Target already final:** Show its current state and end the request.
- **3a. Dispatcher declines:** Preserve the committed state and end the request.
- **4a. The affected set changed:** Recompute the affected set at step 2 before requesting new confirmation.

## UC-20a Cancel Dependent Tasks

**Preconditions:** UC-20 found non-final tasks directly or indirectly dependent on the target.

**Postconditions - success:** The confirmed descendants are canceled together with the target.

**Postconditions - failure:** Declining the cascade cancels neither the target nor its descendants.

**Main flow**

1. The system determines the distinct affected non-final descendants.
2. The system displays how many tasks will be canceled, identifying the target and its descendants.
3. The dispatcher confirms the full scope once.
4. The server rechecks that scope, cancels all confirmed tasks, releases their live assignments, audits the changes and pushes them.

**Extensions**

- **3a. Dispatcher declines:** End UC-20 without cancelling any task.
- **4a. Affected set changed:** Show the revised affected set at step 2 and obtain a fresh confirmation.

## UC-21 Cancel Event

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in and the event is open.

**Postconditions - success:** The event is Canceled and timestamped. Its non-final tasks are canceled and their live resources released.

**Postconditions - failure:** The failed request makes no partial cancellation.

**Main flow**

1. The dispatcher selects an open event for cancellation.
2. The system identifies all non-final tasks and their live reservations.
3. The dispatcher confirms the event cancellation.
4. The server cancels those tasks, releases their live resources, records the event cancellation time and publishes the changes. Final tasks retain their existing states.

**Extensions**

- **3a. Dispatcher declines:** End the request without changes.
- **4a. Event no longer open:** Refresh its current status and end the request.

## UC-22 Close Event

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in and selects an open event for closure.

**Postconditions - success:** The approval and transaction token are stored, and the event is Closed with its automatic closing time.

**Postconditions - failure:** No unapproved closure is recorded. The dispatcher sees why closure did not complete.

**Main flow**

1. The dispatcher requests event closure.
2. The server verifies that all event tasks are Completed or Canceled.
3. The system calculates operating costs (UC-23).
4. The system requests settlement and receives the outcome (UC-24).
5. On approval with a transaction token, the server stores the approval and token, records the closure time and closes the event.

**Extensions**

- **2a. At least one task is non-final:** Show the outstanding tasks and end the request.
- **4a. The clearing system explicitly rejects settlement:** Apply UC-22a at Settlement outcome and end the request with the event open.
- **4b. No response, or approval lacks a token:** Keep the event open, flag the outcome as unconfirmed and notify the dispatcher. End the request without asserting a decline.

## UC-22a Handle Settlement Rejection

**Preconditions:** The clearing system explicitly rejected settlement at UC-22 step 4.

**Postconditions - success:** The rejection is recorded and the dispatcher is alerted. The event remains open.

**Postconditions - failure:** The request cannot close the event without approval and a token.

**Main flow**

1. The system records the rejection against the event.
2. The system alerts the dispatcher that the event remains open.

**Extensions**

- **2a. The workstation has disconnected:** Keep the recorded outcome available when the dispatcher reconnects. End the handling.

## UC-23 Calculate Operating Costs

**Preconditions:** An event is being closed after its tasks have reached final states.

**Postconditions - success:** A total with personnel hours, equipment engine hours and consumables is available for settlement.

**Postconditions - failure:** No incomplete cost total is sent for settlement.

**Main flow**

1. The closing process requests the event's operating costs.
2. The server retrieves personnel time, engineering and medical equipment engine hours and consumables, including previously replaced equipment.
3. The system calculates the total and returns the cost summary to the closing process.

**Extensions**

- **2a. Required cost data missing:** Identify the missing data to the dispatcher; leave the event open. End closure.

## UC-24 Request Settlement

**Actors:** National Clearing & Settlement System

**Preconditions:** The closing process has a complete operating-cost summary.

**Postconditions - success:** The external service returns an approval and transaction token for the closing process.

**Postconditions - failure:** An explicit rejection or unconfirmed outcome is returned to the closing process.

**Main flow**

1. CrisisSync sends the charge and reimbursement request to the National Clearing and Settlement System.
2. The external system returns its decision.
3. CrisisSync checks the response and passes an approval and token to UC-22.

**Extensions**

- **2a. Explicit rejection:** Return the rejected decision to UC-22 step 4.
- **2b. No response:** Return an unconfirmed outcome to UC-22 step 4.
- **3a. Missing token:** Return an unconfirmed outcome to UC-22 step 4.

## UC-25 Assign Replacement Equipment

**Actors:** Dispatcher

**Preconditions:** A dispatcher is signed in. An equipment breakdown left a matching slot unfilled on an Active task.

**Postconditions - success:** One available replacement of the required type is reserved for the active task.

**Postconditions - failure:** This attempt reserves no replacement. The faulty item remains faulty.

**Main flow**

1. The dispatcher opens the malfunction alert and selects a matching replacement.
2. The server validates the single replacement under SQL locks (UC-17).
3. The system reserves the replacement in the existing equipment slot, audits it and pushes the change.

**Extensions**

- **1a. No matching item is available:** Show the shortage to the dispatcher and keep the slot unfilled. End the attempt.
- **2a. A competing request filled the slot or took the candidate:** Reject this attempt and show the current state. Return to step 1 if a replacement is still needed.
- **2b. Task is no longer Active:** Refresh the task; do not reserve a replacement. End the attempt.

## UC-26 Report Task Completion

**Actors:** Field Commander

**Preconditions:** A field commander is signed in and selects an assigned task shown as Active.

**Postconditions - success:** The task is Completed, its live reservations are released and newly eligible dependent tasks are Ready.

**Postconditions - failure:** The request adds no partial changes and preserves the latest committed state.

**Main flow**

1. The commander selects an assigned task and reports completion.
2. The server verifies under lock that the caller is its commander and the task is Active.
3. The server marks the task Completed and applies UC-27 in the same transaction.
4. The server commits and audits the transitions and pushes the authorized changes.

**Extensions**

- **2a. Caller is not the assigned commander:** Reject the report, disclose no unauthorized details and end the request.
- **2b. Task no longer Active, including concurrent cancellation:** Reject the report and show the permitted current state. End the request.
- **3a. Transaction interrupted before commit:** Roll back the task, assignment and dependent changes together. End the request.
- **4a. Acknowledgement lost after commit:** On reconnect display the committed completion, without reapplying it. End the request.

## UC-27 Release Resources and Promote Dependent Tasks

**Preconditions:** The authorized completion transaction has marked the task Completed and is still open.

**Postconditions - success:** Live resource assignments are released and every newly eligible direct dependent is Ready when the caller commits.

**Postconditions - failure:** The caller rolls back the entire completion transaction if a step fails.

**Main flow**

1. The server releases current assignments, including the commander, while preserving historical assignments and previously marked equipment faults.
2. The server checks each Blocked direct dependent against all its predecessors.
3. The server promotes each dependent whose predecessors are all Completed and returns control to UC-26 for commit.

**Extensions**

- **2a. Another predecessor is not Completed:** Leave that dependent Blocked and continue checking other dependents.
- **3a. Database update fails:** Abort the entire completion transaction and return failure to UC-26.

## UC-28 View Assigned Task Details

**Actors:** Field Crew Member

**Preconditions:** A field crew member is signed in.

**Postconditions - success:** The user sees details of a task to which they are assigned.

**Postconditions - failure:** No information about another user's assignment is disclosed.

**Main flow**

1. The crew member requests assigned tasks and selects one.
2. The server verifies the current assignment.
3. The system displays task details, state and relevant assigned resources.

**Extensions**

- **2a. No current assignment or assignment changed:** Show the permitted current list and end the request.

## UC-29 Report Shift Start and Readiness

**Actors:** Field Crew Member

**Preconditions:** A field crew member is signed in.

**Postconditions - success:** The member's shift start and reported readiness are recorded.

**Postconditions - failure:** No live assignment is overwritten by the readiness report.

**Main flow**

1. The crew member reports shift start and selects available or unavailable.
2. The server checks the current resource state and preserves any live reservation.
3. The system records the report and publishes the updated authorized resource state.

**Extensions**

- **2a. Member is currently reserved:** Record the readiness report without making the person available for a second task. End the request.

## UC-30 Report Equipment Malfunction

**Actors:** Field Commander, Field Crew Member

**Preconditions:** The signed-in field crew member or commander is assigned to the affected Active task.

**Postconditions - success:** The item is marked Faulty immediately, its live assignment ends, and the dispatcher is notified that a replacement is needed.

**Postconditions - failure:** A rejected report does not alter inventory or assignments.

**Main flow**

1. The reporter selects the affected item and reports its malfunction.
2. The server verifies the reporter's assignment, task state and item assignment.
3. The system marks the item Faulty, ends its live assignment, retains its usage history, audits the change and pushes the alert to the dispatcher.

**Extensions**

- **2a. Item no longer assigned or reporter unauthorized:** Reject the request and return only authorized current information. End the request.
