Trigger
Type: Recurrence

Interval: 1

Frequency: Minute

Get items Action
Filter Query (OData):

(SLA_Status eq 'On Track' or SLA_Status eq null or SLA_Status eq 'Near Breach') and (Incident_Status ne 'Met' and Incident_Status ne 'Breached')
Top Count:

100
Condition Logic (Inside Apply to each)
Condition 1: Met

Purpose: Checks if the incident has been resolved before the SLA.

Expression:

and(
  or(
    equals(toLower(item()?['Status']?['Value']), 'resolved'),
    equals(toLower(item()?['Status']?['Value']), 'closed')
  ),
  not(empty(item()?['Issue_End_Date'])),
  greater(ticks(item()?['Issue_End_Date']), ticks(utcNow()))
)
Condition 2: Near Breach (in the "If no" branch of Condition 1)

Purpose: Checks if the SLA is within the 1-hour warning window.

Expression:

and(
  or(
    equals(toLower(item()?['SLA_Status']?['Value']), 'on track'),
    empty(item()?['SLA_Status'])
  ),
  not(empty(item()?['Issue_End_Date'])),
  lessOrEquals(ticks(item()?['Issue_End_Date']), ticks(addHours(utcNow(),1))),
  greater(ticks(item()?['Issue_End_Date']), ticks(utcNow())),
  not(or(equals(toLower(item()?['Status']?['Value']), 'resolved'), equals(toLower(item()?['Status']?['Value']), 'closed')))
)
Actions (If yes): Update item (set SLA_Status to "Near Breach"), Send an email (V2) (Gmail alert).

Condition 3: Breached (in the "If no" branch of Condition 2)

Purpose: Checks if the SLA deadline has passed.

Expression:

and(
  not(empty(item()?['Issue_End_Date'])),
  less(ticks(item()?['Issue_End_Date']), ticks(utcNow())),
  not(equals(toLower(item()?['SLA_Status']?['Value']), 'breached')),
  not(equals(toLower(item()?['SLA_Status']?['Value']), 'met')),
  not(or(equals(toLower(item()?['Status']?['Value']), 'resolved'), equals(toLower(item()?['Status']?['Value']), 'closed')))
)
Actions (If yes): Update item (set SLA_Status to "Breached").
