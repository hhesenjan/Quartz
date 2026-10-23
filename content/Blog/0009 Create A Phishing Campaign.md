---
title: "Creating A Phishing Campaign"
created: 2024-12-25
modified: 2024-12-25
tags: ["M365", "MDE"]
draft: false
---

## Introduction

We were going to hire external consultants to do a phishing campaign for us since the Board Members requested it. However, due to the short time frame the external consultants wouldn't be able to deliver therefore I decided to take on the responsibility to create a phishing campaign within 48 hours. I decided to apply all the knowledge I gained from the [OSCP certification](/posts/OSCP-Review) as it taught me a-lot about creating phishing campaign.

## Phishing Email

The most important part of creating a phishing campaign is understanding our target and creating a real-istic phishing email as that will help us with finding users who need more cyber security awareness training. In an phishing email we should try to include the following points:

- **Impersonate Domain:** If our domain is contoso.com we should preferrably create a phishing campaign with consto.com or contoso.co as that will create some authencity for the phishing email.
- **Impersonate Departement/User:** We should try to send email as HR@contoso.co or CEO@contoso.co since that will create some urgency for the user to read the email.
- **Spelling Mistakes:** We should include spelling mistakes on our email to help users with spotting that the email is a phishing email. 
- **Creating urgency:** It's important to create urgency on the subject and body of the phishing email as that will increase the likelyhood of the user skim reading through the phishing email.

We should preferrably try to include all these factors to our phishing email as that will help us with finding users who needs cyber security awareness training. Here's an example of a phishing email that has all the-se factors:

```text
From: "HR@contso.co"
Subject: "Salary increase for 2024"
Body:

Hi [Firstname],

We are happy to inform you that you have rercieved a salary incrase for 2024.

Please urgenlty will out the following form to get your salary increase:

[Malicious Payload]

Kind regards,
HR
```

The email is well crafted and includes spelling mistakes such as **rercieved** and **incrase** to help the users spot that it's a phishing email. It's important to include urgency and spelling mistakes as it will help us with tea-ching our users what to look after in a phishing email.

## Microsoft 365 Phishing Campaign

I'll be using Microsoft Attack Simulation feature to create our phishing campaign and the method that I'll be using is credential harvest as that will allow us to see which users clicked on the link and logged in with their credentials. It will also allow us to assign users who failed the phishing campaign training.

First we'll need to go to the [Microsoft 365 Security Portal](https://security.microsoft.com/) then we'll then need to scroll down to **Email & Collaboration** and select **Attack Simulation Training**.

![[0009 Creating-A-Phishing-Campaign-01.png]]

We'll then need to go to **Simulation** and select **Launch a Simulation** to create our phishing campaign.

![[0009 Creating-A-Phishing-Campaign-02.png]]

The attack simulation allows you to choose many methods for your phishing campaign. However, I recom-mend going with **Credential Harvest** as that is the most common method malicious actors uses for their phishing attacks. 

![[0009 Creating-A-Phishing-Campaign-03.png]]

We can now choose **Tenant Payloads** and **Create a payload** to create our own custom phishing email.

![[0009 Creating-A-Phishing-Campaign-04.png]]

Inside **Configure Payload** we can start creating our own custom phishing email using **Code** section. The language used for programming the custom email is HTML and I recommend using a IDE such as Visual Studio Code to create your custom phishing email as it comes with autocompletion and code hightlights.

![[0009 Creating-A-Phishing-Campaign-05.png]]

You can also customize the login page which will be displayed to the users when they click on the link by selecting the payload and going to **Login Page** section. 

![[0009 Creating-A-Phishing-Campaign-06.png]]

Once we are happy with our phishing email and the login page we can now select the users who will be sent the phishing email from us.

![[0009 Creating-A-Phishing-Campaign-07.png]]

The attack simulation also allows us to assign training to users who failed the phishing campaign by click-ing on the link or logging with their credentials. 

![[0009 Creating-A-Phishing-Campaign-08.png]]

Once we have choosen the traning method, we can now create our own custom landing page which will show up after the user logins to the phishing website. I highly recommend writing the message in a empat-hic way and explain how they can protect themselves and the organization from the phishing attack and ways to spot it.

![[0009 Creating-A-Phishing-Campaign-09.png]]

We can now configure our **User Notification Settings** to send positive reinforcement for reporting the phishing email and training reminders. The positive reinforcement will be sent to the user if they report the phishing email through Outlook report feature.

![[0009 Creating-A-Phishing-Campaign-10.png]]

Now we can choose when the phishing campaign will be sent out to the users. We can choose to send it out right after we submit it or choose a specific time. 

![[0009 Creating-A-Phishing-Campaign-11.png]]

Now once we click on submit it will automatically send out the phishing email to the employees that we selected to be a part of the phishing campaign.

![[0009 Creating-A-Phishing-Campaign-12.png]]

Congratulation! You have now successfully launched a phishing campaign using the attack simulation and once the phishing campaign ends you can get analysis about the amount of users who clicked on the phishing link and logged in with their credentials and the amount of employees that finished their training after failing the phishing campaign.

## Conclusion

In a phishing campaign it's important to include domain impersonation, user/departement impersonation, spelling mistakes, and create urgency as these factors will help us with finding out which users in our organization needs cyber security training. We should never shame our users for clicking on the link and logging in with their credentials instead we should teach and train them up to understand the way malicious actors can attack them and the organization.
