Trigger
Type: When an item is created or modified

List: Incidents_new

Trigger Condition (Loop Prevention):

@not(equals(triggerOutputs()?['body/Editor/Email'], '2021wb86686@wilp.bits-pilani.ac.in'))
Logic
Condition 1 (Initial Prediction): Checks if RootCauseCategory is blank.

If yes: Calls the Custom Connector using Title and Description, then Update item to set RootCauseCategory and RootCauseCategorySource = "DescriptionBased".

Condition 2 (Final Prediction): Checks if ResolutionNotes is not blank AND RootCauseCategorySource is not "ResolutionBased".

If yes: Calls the Custom Connector using Title, Description, and ResolutionNotes. Then Update item to set RootCauseCategory and RootCauseCategorySource = "ResolutionBased".
