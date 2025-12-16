```
AlertInfo
| where TimeGenerated > ago(1h)
| where Title == "Email reported by user as malware or phish"
| join kind=inner (
    AlertEvidence
    | where TimeGenerated > ago(1h)
    | where EntityType == "MailMessage"
    | extend af = parse_json(AdditionalFields)
    | extend
        Sender   = tostring(af.Sender),
        Reporter = tostring(af.Recipient)
    | project
        AlertId,
        AlertTime = TimeGenerated,
        NetworkMessageId,
        Sender,
        Reporter,
        EmailSubject
) on AlertId
| join kind=inner (
    EmailEvents
    | where Timestamp > ago(1h)
    | project
        DeliveryTime = Timestamp,
        NetworkMessageId,
        RecipientEmailAddress,
        ReportId
) on NetworkMessageId
| summarize
    Timestamp       = max(DeliveryTime),
    ReportId        = any(ReportId),
    Sender          = any(Sender),
    EmailSubject    = any(EmailSubject),
    Reporter        = make_set(Reporter),
    AllRecipients   = make_set(RecipientEmailAddress),
    TotalRecipients = dcount(RecipientEmailAddress),
    AlertIds        = make_set(AlertId)
  by NetworkMessageId
| project
    Timestamp,
    ReportId,
    NetworkMessageId,
    Sender,
    EmailSubject,
    Reporter,
    AllRecipients,
    TotalRecipients,
    AlertIds
```
