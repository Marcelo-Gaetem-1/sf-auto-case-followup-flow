# Design Notes – 72h Case Auto-Followup

## Goal

Proactively contact customers with open cases older than 72 hours and log the action in a separate object for audit and reporting.

## Key Decisions

- **Record-Triggered Flow** instead of Apex to keep it fully declarative.
- Separate **log object** (`Case_Followup_Log_72h__c`) to avoid overloading the Case with technical fields.
- Use **Email Alert** + Template instead of `Send Email` action, so admins can edit content without touching the Flow.

## Flow Highlights

- **Entry criteria** ensure:
  - Case is still open.
  - Contact and email exist.
  - 72h threshold is met.

- **No hardcoded emails**:
  - Email Alert sends the message directly to Case Contact.

- **Date/Time field**:
  - `Date__c` populated with `Now()` to keep exact timestamp of the action.

## Testing Scenarios

1. Case with Contact + Email + 72h passed → log created + email sent.
2. Case with Contact but no Email → log optional, email not sent (or error tracked).
3. Case closed before 72h → flow does not fire.
4. Multiple updates on the same case → use conditions to avoid duplicate logs (if needed).
