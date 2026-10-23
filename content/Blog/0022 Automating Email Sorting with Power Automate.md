---
title: "Automating Email Sorting with Power Automate"
created: 2025-11-10
modified: 2025-11-10
tags: ["PA"]
draft: false
---

## Introduction

We are currently implementing a new system in the organization and the employees using these systems made a request to implement automatic email sorting for shared mailbox where all emails with PDFs are moved to a different folder than emails without PDF documents. Unfortunately, this is not possible with default Outlook rules becuase it doesn't allow us to check of specific file type.

However, after resarching for a bit I found out that Power Automate allows us to sort emails using file type so I purchased a Power Automate license and made the following integration for shared mailbox.

## Automating with Power Automate

1. Go to **Power Automate -> Connectors -> Office 365 Outlook**.
    
    ![[0022 Automating-Email-Sorting-with-Power-Automate-01.png]]

2. Click on **When I receive new email arrives**.

    ![[0022 Automating-Email-Sorting-with-Power-Automate-02.png]]

3. Setup connector on a account.

    ![[0022 Automating-Email-Sorting-with-Power-Automate-03.png]]

4. Initialize `hasPDF` variable with value `false`.

    ![[0022 Automating-Email-Sorting-with-Power-Automate-04.png]]

5. Create **For Each** and include **Attachment** from Outlook connector.

    ![[0022 Automating-Email-Sorting-with-Power-Automate-05.png]]

6. Inside **Condition** use the parameters **Has Attachments** and **Attachment Type** with the following configuration.

    ![[0022 Automating-Email-Sorting-with-Power-Automate-06.png]]

7. Use **Set variable** to set `hasPDF` variable to `true`.

    ![[0022 Automating-Email-Sorting-with-Power-Automate-07.png]]

8. Create another **Condition** to check if `hasPDF` is `true`.

    ![[0022 Automating-Email-Sorting-with-Power-Automate-08.png]]

9. Create **Move email (v2)** object to move email to different folder using **Message Id** from Outlook connector.

    ![[0022 Automating-Email-Sorting-with-Power-Automate-09.png]]

The **when I receive new email arrives** feature in Power Automate allows us to do the following on the account connected to Outlook connector.

* When an email comes into mailbox a boolean variable is created with name `hasPDF` and the value is `false`. 
* The while loop will enumerate through all the attachment and when PDF document is found the `hasPDF` variable is set to `true`. 
* The last condition checks if `hasPDF` variable is `true` and then moves the email to `Processed by Power Automate` folder. 

With this implementation each time a email comes into the mailbox it will automatically move the emails with PDF attached to them into a different folder than emails without.

## Conclusion

Power Automate is low-code platform designed to automate manual tasks with employees performs manually. In this situation it came in good use because all emails with PDF attchments will be moved to different folder than the emails without PDF documents. This implementation allows employees to focus on more important things than spending time shorting emails.
