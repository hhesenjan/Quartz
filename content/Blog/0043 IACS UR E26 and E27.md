---
title: "IACS UR E26 & E27 Regulations (Part 1)"
created: 2026-05-25
modified: 2026-05-25
tags: ["Compliance", "OT-Security"]
draft: false    
---

## Information

Operational Technology (OT) systems are primarily focused on operability and reliability rather than cybersecurity. The OT systems are usually in a isolated environment with limited access to external connectivity because the systems are running older version of operating systems and libraries leaving them vulnerable for cyber threats.

## Understanding OT Systems

OT systems are developed using systems such as  [SCADA](https://en.wikipedia.org/wiki/SCADA), [DCS](https://www.yokogawa.com/no/library/resources/white-papers/what-are-the-roles-of-dcs-and-scada-in-digital-transformation/), and [PLC](https://en.wikipedia.org/wiki/Pilot-controlled_lighting). These are commonly used to build paper manufacturing, power generations, oil and gas processing, telecommunications and much more... All critical systems are running in a old operating systems and libraries which makes them interesting for threat actors.

## Brownfield Challenge

The primary challenges with OT environment is brownfield challenge which is about integrating new OT systems into the legacy environment as the modern systems supports higher bandwidth, ultra-low latency, and connectivity.

## IACS UR E26 & E27 Regulations

The IACS UR E26 & E27 regulations where implemented by International Association of Classification Societies (IACS) to enforce best cybersecurity practices for cargo ships and vendors whom builds OT systems. Here is a overview of the primary difference between E26 and E27

* **IACS UR E26 (Shipowner Responsibility)** - Securing network environment inside the cargo ship by implementing segmentations, incident response process, and recovery process. Additionally, implementing monitoring on all assets inside of the cargo ship.
* **IACS UR27 (OT Vendor Responsibility)** - Securing individual based systems and hardening the systems to improve the cybersecurity resilience.

The E26 and E27 applies to cargo ship that were contracted to be built after before 1 July 2024. So the primary difference between E26 and E27 regulations are whom are responsible.

## Deep Dive Into IACS UR E26

<image style="display:flex; width: 60%; margin: auto;" src="0043 IACS UR E26 and E27 (Part 1) 01.png"/>

The IACS UR E26 uses the Identify, Protect, Detect, Respond, and Recover pillars to improve the cybersecurity posture for the IT systems and OT systems inside of the cargo ships. Here is a overview of the pillars and the requirement for the shipowners:

- **Identify -** Primarily about getting a visibility over the assets such as endpoints, servers, and OT systems inside the cargo ship. It's also about understanding the operating system and software that are running on these different systems. The responsibility of the shipowners is to have a asset inventory which consists of all endpoints, operating system, software, and OT systems.

- **Protect -** Implementing segmentation to systems that are connected to ships network. This improves the cybersecurity resilience since when an endpoint is compromised in the crew network it cannot access OT systems which are business critical.  The responsibility of the shipowner is to implement strict network restriction to prevent collateral damage.

- **Detect -** Implementing monitoring solution for systems such as servers, workstations, and OT systems to identify anomalies. This can assists with detecting threats before an incident occurs with a critical system. The responsibility of the shipowner is to ensure there is a monitoring solution for endpoints, servers, and OT systems.

- **Respond** - Minimizing the impact of a cybersecurity incident by implementing processes that employees can follow. As an example when a cybersecurity incident occurs the employee should have a clear understanding about whom to contact and where the data can be accessed to investigate the incident. The shipowner is responsible for implementing the monitoring solutions and a response plan incase of a cybersecurity incident.

- **Recovery** - The primary purpose of recovery is to ensure the system can be restored to the original operation state after disruption or compromise. A recovery plan can help crew members with responding and recovering quickly to reduce the impact of the cybersecurity incident.

Following these five pillars can contribute to increase the cyber resilience and reduce the fatal concequences that could occur in situation where the cargo ship systems is compromised by threat actors. 

## Network Segmentation

Implementing strict network segmentation is incredibly important since OT systems should have minimal connection to IT systems and external environment. If an system or an endpoint is compromised that could allow the threat actor to compromise a OT system which could lead to fatal concequences. Therefore to be compliant with IACS E26 a strict network segmentation is a requirement.

<image style="display:flex; width: 60%; margin: auto;" src="0043 IACS UR E26 and E27 (Part 1) 02.png"/>
<p style="text-align:center; margin-top:0px; opacity: 0.5;font-size:14px;">Network Segmentation 1</p>

**Network Segmentation 1** - Implementing a network segmentation where there are no connection between the IT systems and OT systems. This option reduces the risk significantly as there are no connection with OT systems and IT systems.

<image style="display:flex; width: 60%; margin: auto;" src="0043 IACS UR E26 and E27 (Part 1) 03.png"/>
<p style="text-align:center; margin-top:0px; opacity: 0.5;font-size:14px;">Network Segmentation 2</p>

**Network Segmentation 2** - Implementing different LANs for OT Systems, Business, and Crews with two firewalls. The purpose of the following configurations is to ensure that the critical systems such as alarm and monitoring system, engine machinery control, and generator engine control are secure.

<image style="display:flex; width: 60%; margin: auto;" src="0043 IACS UR E26 and E27 (Part 1) 04.png"/>
<p style="text-align:center; margin-top:0px; opacity: 0.5;font-size:14px;">Network Segmentation 3</p>

**Network Segmentation 3** -  Implementing different LANs for OT Systems, Business, and Crews with two firewalls. Additionally, implementing DMZ (Demilitarized Zone) for data servers which belongs to ECDIS and VDR where the DMZ is threated as a untrusted network.

<image style="display:flex; width: 60%; margin: auto;" src="0043 IACS UR E26 and E27 (Part 1) 05.png"/>
<p style="text-align:center; margin-top:0px; opacity: 0.5;font-size:14px;">Network Segmentation 4</p>

**Network Segmentation 4** -  Implementing different LANs for OT Systems, Business, and Crews with two firewalls. Additionally, implementing DMZ (Demilitarized Zone) for data servers which belongs to ECDIS and VDR. The DMZ is threated as a trusted network. 

Currently, these are the four network segmentations that are recommended by IACS to better secure OT systems and IT systems. The primary goal with IACS E26 and E27 is to improve the cyber resilience for OT systems where both the shipowners and vendors follows best cybersecurity practices to secure critical systems.

## Conclusion

The primry purpose of IACS UR E26 is to ensure that shipowners implements strict access controls, network segmentations, and has a clear overview of all organizational assets inside of the cargo ship. The IACS UR E27 on the other hand is to ensure that vendors builds secure and patchable systems. 

I believe IACS UE E26 and E27 is a great step going forward since it ensures that new cargo ship are secure from cyber threats especially when it comes to OT systems as an incident could have fatal concequences.