App
Property: OnStart

Purpose: Clears old collections from the app template.

Code:

Collect(CalendarLocalizedLabel, {Value:"Calendar"},{Value:"Kalender"},...);
Concurrent(
    If(CountRows(Office365.GetRoomLists().value) <= 1,
        Set(NoRoomsList, true);
        ClearCollect(AllRooms, Office365.GetRooms().value);
        Set(AllRoomsConnector, Concat(AllRooms, Address & ";")),
        Set(NoRoomsList, false);
        ClearCollect(RoomsLists, Office365.GetRoomLists().value)
    ),
    ClearCollect(Durations, {Time: "30 min", Minutes: 30}, {Time: "1 hr", Minutes: 60}, ...),
    ClearCollect(Times, {Text: "12:00 am", Minutes: 0}, {Text: "12:30 am", Minutes: 30}, ...),
    Set(MyCalendar, LookUp(Office365.CalendarGetTables().value, DisplayName = LookUp(CalendarLocalizedLabel,Value=DisplayName).Value).Name);
    Set(MyName, Office365Users.MyProfile());
    Set(MyOrg, Proper(Left(Right(User().Email, Len(User().Email) - Find("@", User().Email)), Find(".", Right(User().Email, Len(User().Email) - Find("@", User().Email)))-1)))
)
Form (e.g., Form1)

Property: OnSuccess

Purpose: Generates the unique IncidentID after submission and resets the form.

Code:

// 1. Update the newly created item with a formatted IncidentID.
Patch(
    Incidents_new,
    Self.LastSubmit,
    {
        IncidentID: "INC" & Text(Self.LastSubmit.ID, "0000000")
    }
);

// 2. Notify the user and reset the form.
Set(varSubmitting, false);
Notify("Incident " & "INC" & Text(Self.LastSubmit.ID, "0000000") & " created successfully!", NotificationType.Success);
NewForm(Self);
ResetForm(Self);
Submit Button (e.g., SubmitButton1)
Property: OnSelect

Purpose: Captures local time, calculates SLA, and submits the form. This is the final working version that manually handles time.

Code:

If(
  !varSubmitting,
  Set(varSubmitting, true);

  // --- 1. Get the current local time from the controls or Now()
  Set(
    varStartLocal,
    If(
      !IsBlank(DateValue3.SelectedDate),
      DateTime(
        Year(DateValue3.SelectedDate),
        Month(DateValue3.SelectedDate),
        Day(DateValue3.SelectedDate),
        Value(HourValue3.Selected.Value),
        Value(MinuteValue3.Selected.Value),
        0
      ),
      Now()
    )
  );

  // --- 2. Calculate the local end time based on priority
  Set(
    varHours,
    Switch(
      DataCardValue20.Selected.Value, // This is your Priority dropdown
      "P1", 4,
      "P2", 8,
      "P3", 12,
      "P4", 24,
      0
    )
  );
  Set(varEndLocal, DateAdd(varStartLocal, varHours, Hours));

  // --- 3. Manually create the IST time to be saved
  Set(varStartToSave, varStartLocal);
  Set(varEndToSave, varEndLocal);

  // --- 4. Submit the form
  SubmitForm(Form1)
)
Property: DisplayMode

Purpose: Disables the button until required fields are filled.

Code:

If(
    !IsBlank(TitleTextInput.Text) && 
    !IsBlank(PriorityDropdownInput.Selected.Value),
    DisplayMode.Edit,
    DisplayMode.Disabled
)
Data Cards
Control: Issue_Start_Time_DataCard

Property: Update

Purpose: Binds the data card to the manually set time variable.

Code:

varStartToSave
Control: Issue_End_Date_DataCard

Property: Update

Purpose: Binds the data card to the manually set time variable.

Code:

varEndToSave
Control: Reporter_DataCard

Property: Update

Purpose: Auto-populates the Person/Group field with the current user's details.

Code:


{
    '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedUser",
    Claims: "i:0#.f|membership|" & User().Email,
    Department: "",
    DisplayName: User().FullName,
    Email: User().Email,
    JobTitle: ""
}
