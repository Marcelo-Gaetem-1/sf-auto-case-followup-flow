# Salesforce – 72h Case Auto-Followup Flow

This project implements an **automated follow-up and email notification** for Service Cloud cases that have been open for at least 72 hours.

When the criteria are met, the flow:

1. Logs the action in a custom object (`Case_Followup_Log_72h__c`).
2. Stamps the current Date/Time.
3. Sends an email to the customer using an Email Alert (based on an Email Template).
4. Updates the Case (optional fields such as status or internal notes).

This is a small but realistic example of how to use **Record-Triggered Flows** to automate customer-service processes in Salesforce.

---

## Business Scenario

Support wants to:

- Detect cases that have been open for **72 hours**.
- **Notify the customer** proactively by email.
- Keep an **internal log** of every automatic follow-up action.
- Avoid hardcoded IDs and keep the solution maintainable.

---

## Architecture

**Main components:**

- **Object:** `Case`  
- **Custom Object:** `Case_Followup_Log_72h__c`  
  - `Case__c` (Lookup to Case)  
  - `Action__c` (Text) – e.g. “Case auto-closed after 72h”  
  - `Date__c` (Date/Time) – stores the current date-time using `Now()`  
  - `Email_Sent__c` (Checkbox, optional)  
  - `Error_Message__c` (Text, optional)

- **Email Template:**  
  - API Name: `<Your_Email_Template_API_Name>`  
  - Target: Contact (customer)  

- **Email Alert:**  
  - Type: Send email using the template above  
  - Recipient: **Case Contact**  

- **Flow:** Record-Triggered Flow on `Case`
  - Entry conditions:  
    - Case is open (Status filter)  
    - Case has been open for at least 72 hours (formula / condition)  
    - Case has a Contact with email
  - Actions in order:
    1. Create `Case_Followup_Log_72h__c` record.
    2. Set `Action__c` text and `Date__c = Now()`.
    3. Invoke Email Alert to send email to the customer.
    4. (Optional) Update Case fields (e.g. internal comments, status).

See `/img/flow-overview.png` for a visual overview of the Flow.

---

## How It Works (Flow Logic)

1. **Trigger**  
   - Type: Record-Triggered Flow  
   - Object: `Case`  
   - Runs: After Save  
   - Condition example (pseudo):
     - `Status` = "In Progress"
     - `CreatedDate` is 72h or more in the past
     - `Contact.Email` is not blank

2. **Create Follow-up Log**  
   - Action: *Create Records* on `Case_Followup_Log_72h__c`  
   - Fields:
     - `Case__c` = `$Record.Id`
     - `Action__c` = `"Case auto-closed after 72h"`
     - `Date__c` = `Now()`

3. **Send Email to Customer**  
   - Action: *Email Alert*  
   - Email uses the Contact from the Case as the recipient.  
   - No hardcoded email addresses.

4. **Case Update (optional)**  
   - You can update fields like:
     - `Last_Auto_Close_Email__c`
     - `Status` = "Pending Customer"
     - etc.

---

## Deployment / Reuse

To reuse this idea in another org:

1. Create custom object `Case_Followup_Log_72h__c` with the fields listed above.
2. Create an Email Template and an Email Alert that uses **Case Contact** as recipient.
3. Rebuild the Flow using the same logic, adapting:
   - Status values
   - 72h condition (could be a formula or Scheduled Path)
4. Test with:
   - Case with Contact (and Email)
   - Case that meets the 72h criteria
   - Case without email (to validate error handling)

---

## Possible Enhancements

- Add a **Scheduled Path** to re-try sending emails that failed.
- Add a checkbox on Case: `Auto_Followup_Sent__c`.
- Create a **Report / Dashboard**:
  - Number of auto-followup emails sent
  - Cases touched by the automation
- Add translations and more dynamic email content.

---

## Author

**Marcelo Gaete**  
Salesforce Consultant / Admin / Marketing Cloud & Data Cloud  
