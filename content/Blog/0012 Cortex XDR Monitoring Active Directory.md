---
title: "Cortex XDR Monitoring Active Directory Events"
created: 2025-09-18
modified: 2025-09-18
tags: ["CORTEX", "XQL"]
draft: false
---

## Introduction

I decided to practise security incidents that could occur when a company administrator changes the `userPrincipalName` of someone important in the organization such as CEO, CFO, and etc. I believe these practises are important since it allows us to understand where our weakpoints are as cybersecurity professionals. The goal with these articles is to see the way I think about security incidents in the future as I gain more experience.

## Simulation

So an IT Administrator in Husenjan LLC went rough and changed the CFO's `userPrincipalName` at Friday night. This happened on the IT administrator account while they were on vacation. What are the stpes that we should take?

## Mitigation

Once it was confirmed that the account was malicious the IT administrator changed the `userPrincipalName` of the CFO these are the actions we should perform. 

1. Disable the CFO's account in Active Directory.
2. Monitor all sign-in events that occurred on Microsoft Entra and Active Directory.
3. Reaching out to people and vendors.

After completing all these actions we should continue forward and start collecting logs of the evidence.

## Windows Events

With a EDR system all the event logs from endpoints and servers will be logged which enables us to find all events that occurred in the last 30 to 90 days. We can use Event IDs to review all the successful and failed login attempts.

* [Event ID 4624 - An account was successfully logged on](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4624)
* [Event ID 4625 - A failed login attempt occurred](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4625)
* [Event ID 4625 - A failed login attempt occurred](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4625)
* [Event ID 4725 - An attempt was made to reset an accounts password](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4724)
* [Event ID 4726 - A user account was deleted](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4726)
* [Event ID 4738 - A user account was updated](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4738)
* [Event ID 4743 - A computer account was deleted](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventid=4743)

A great website for finding more Event IDs is [UltimateWindowsSecurity](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/default.aspx?i=j) where all the events are described in details which I highly recommend checking out.

## Ingvestigation

Instead of being dependent on the third party to respond where every second to minute matters. Its crucial to be familiar with the tools that the organization uses so we can quickly respond when a security incident occurs.

1. Go to **Incident Response -> Investigation -> Query Builder**.   
    ![[0012 Cortex-XDR-Monitoring-Active-Directory-Events-01.png]]

2. Inside **Query Builder** select **Event Log**.
    ![[0012 Cortex-XDR-Monitoring-Active-Directory-Events-02.png]]

3. In **Event_ID** enter the number **4738** and select the date the incident occurred on the account.
    ![[0012 Cortex-XDR-Monitoring-Active-Directory-Events-03.png]]

4. Cortex XDR will display all the endpoints and servers the Event ID 4738 occurred.
    ![[0012 Cortex-XDR-Monitoring-Active-Directory-Events-04.png]]

5. We can further investigate the event by opening it on Cortex XDR.
    ![[0012 Cortex-XDR-Monitoring-Active-Directory-Events-05.png]]

Once we have found the user who changed the `userPrincipalName` of the account we can further investigate the accounts that made these changes by going through events such as 4624 and 4625 to see if someone was performing a brute-force attack on the account. We can also take extra steps by going through Microsoft Entra ID logs such as risky users, risky logins, and login events to ensure that these accounts are secure.

## Remediation

The remediation steps that has been taken to prvent this incident is by implementing correlation rules which consistently monitors important accounts in the organization.

```SQL title="Correlation Rules"
dataset = xdr_data 
| filter event_type = ENUM.EVENT_LOG  
| filter action_evtlog_event_id  in (4724, 4738) and action_evtlog_data_fields->TargetUserName in ("jdoe", "edoe")
| alter actioned_by = action_evtlog_data_fields->SubjectUserName
| fields action_evtlog_event_id, actioned_by, action_evtlog_message, action_evtlog_description, action_evtlog_data_fields
```

All the XQL Query does is it fetches all event logs and searches for event ids such as 4724 and 4738. If these actions has been performed on accounts such as `jdoe` and `edoe` it will create a alert.

## Lesson Learnt

*We should remove domain admins from all IT admin.* Jokes aside its important to monitor highly privileged and managements accounts in the organization since these are commonly targeted by threat actors and insider threats. Here are some things we can takeaway:

- Always perform your own investigations. 
- Always go through the logs since its the truth of source.
- Understand the tools in your organization to perform investigations.

A EDR system is great for detecting external threats. However, once a threat actor has compromised a IT administrator account these are things which EDR system cannot detect unless there are custom detection rules made for it. 