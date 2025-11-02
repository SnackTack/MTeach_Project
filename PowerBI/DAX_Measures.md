DAX measures for the Power BI dashboard.


Total Incidents = COUNT(Incidents_new[IncidentID])

Open Incidents = 
CALCULATE(
    COUNT(Incidents_new[IncidentID]),
    Incidents_new[Status] = "New" || Incidents_new[Status] = "In Progress"
)

Breached Incidents = 
CALCULATE(
    COUNT(Incidents_new[IncidentID]),
    Incidents_new[SLA_Status] = "Breached"
)

SLA Met % = 
VAR MetCount = 
    CALCULATE(
        COUNT(Incidents_new[IncidentID]),
        Incidents_new[SLA_Status] = "Met"
    )
VAR TotalCompleted =
    CALCULATE(
        COUNT(Incidents_new[IncidentID]),
        Incidents_new[SLA_Status] = "Met" || Incidents_new[SLA_Status] = "Breached"
    )
RETURN
DIVIDE(MetCount, TotalCompleted, 0)
