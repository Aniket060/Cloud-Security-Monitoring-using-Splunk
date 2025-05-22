# Cloud-Security-Monitoring-using-Splunk

## 1. Introduction

This project presents a cloud security posture monitoring (CSPM) solution by integrating **AWS Security Hub** with **Splunk**. The main goal is to **classify, monitor, and track AWS cloud misconfigurations** based on severity levels and resource types. The project also includes **real-time dashboards** to visualize the current state of the cloud infrastructure’s security posture.

---

## 2. Project Overview

- Installed and configured the **AWS Add-on for Splunk** to ingest logs from AWS Security Hub.  
- Created **data inputs** in Splunk to enable real-time log ingestion.  
- Wrote custom **SPL (Search Processing Language)** queries to extract, classify, and categorize misconfiguration findings.  
- Designed **KV Store-backed lookups** to maintain a stateful record (e.g., *Active / Closed*) of misconfigurations.  
- Built **dynamic Splunk dashboards** categorized by *Severity Levels* and *Resource Types*.  

---

## 3. Architecture Diagram
![Untitled Diagram drawio](https://github.com/user-attachments/assets/1b66c95f-736c-493b-87e2-26553c654110)

---

## 4. Setup and Configuration

Prerequisite :
1. Splunk Enterprise Installation on any platform. The Splunk Server should be up and running along with access to console page. 

### A. Configuring AWS Security Hub

AWS Security Hub consolidates findings from various AWS security services such as **AWS Config**, **Amazon GuardDuty**, and others. It provides a centralized view for:

- **Generating insights** into potential security issues  
- **Classifying findings** based on severity  
- Assigning **status tags** like *Active* or *Resolved*  
- Enabling or disabling **ingestion from specific AWS services**  

> Enable the Security Hub service — it may take a few minutes to generate initial findings.  
> 
> Insert screenshots of:  
> - Findings View  
> - Additional Information Tabs  
> - Service Ingestion Controls  

---

### B. Installing AWS Add-on for Splunk

Splunk is a central **SIEM (Security Information and Event Management)** platform for:

- Log ingestion  
- Monitoring and visualizing security events  
- Querying and alerting  

Steps:

1. Go to Apps>Find more Apps and install **Splunk Add-on for AWS**
   ![image](https://github.com/user-attachments/assets/df47d7cd-d6f4-4891-a150-1e7188210c14)

2. Configure **CloudWatch Logs** as the ingestion method:
    - Go to Cloudwatch > Logs > Log Groups and create a new Log Group
      ![image](https://github.com/user-attachments/assets/deb807f8-3c24-49df-b711-ec0ad962b6b4)
    - Use **EventBridge rules** to send Security Hub findings to CloudWatch.
      ![image](https://github.com/user-attachments/assets/cc2bc251-7edd-4cd3-8430-2b297ed28caa)
      Define Event Pattern
      ![image](https://github.com/user-attachments/assets/609afb51-e606-47d4-9f20-65e097d029aa)
      Create Event
      ![image](https://github.com/user-attachments/assets/84e44a95-93f6-4a75-b852-3866a7453f32)

    - In Splunk AWS Add-on, Create New Input > Custom Data Type > CloudWatch Logs  
      Provide all the necessary details
      ![image](https://github.com/user-attachments/assets/f7a2881a-2f37-4349-8b85-4889de1bf196)

    - After finishing, the logs will start getting ingested from Security Hub to Splunk. Verify the same by using Search along with the right search index. 
      ![444390271-7144a218-269c-47fa-9444-cb47a4e62972](https://github.com/user-attachments/assets/a9559a3b-9808-40ff-8d14-c9e6ff62e515)


---

### C. Extracting and Preparing Security Events

#### i. KV Store Lookups

To classify events, use **KV Store-backed lookups**:

- Go to Settings > Knowledge > Lookups > Lookup definitions > New Lookup Definition
- Create lookup tables for each severity level 
- Set field names and permissions appropriately

![image](https://github.com/user-attachments/assets/95386b27-cb32-43a3-9971-9988fdb81d6a)
![image](https://github.com/user-attachments/assets/e85a0598-8e1f-4919-bd5e-e1fbb752ee60)
Configure appropriate permissions. For project testing purpose, following permissions were configured. However, it is recommended to give only necessary access. 
![image](https://github.com/user-attachments/assets/da0d3822-1034-4f47-9d8b-3263b536f44d)
![image](https://github.com/user-attachments/assets/dcfd939e-aa96-4845-a412-8a47187a5d90)

- Now in order to finish the setup of KV Store Lookups, locate to /opt/splunk/etc/apps/<your_splunk_app_name>/local, and edit the collections.conf file by just mentioning the Lookups names in square brackets. 
![image](https://github.com/user-attachments/assets/bc5e57e8-fa45-49f7-bc4c-2aa0f89be4e5)

#### ii. Creating Scheduled Reports

To populate the lookups:
1. Go to Search & Reporting tab, enter SPL query to extract necessary data to populate Lookups. Below is an example of a SPL Query to extract data about unencrypted volumes and feed it into the Lookup. You can customize the query based on your requirements.
   
index="security_hub_2"
| spath detail.findings{}.ProductFields.RelatedAWSResources:0/name output=config_rule
| search config_rule IN (
    "securityhub-encrypted-volumes-3504b60a"
)
| spath detail.findings{}.Workflow.Status output=workflowstatus
| spath detail.findings{}.RecordState output=recordstate
| spath detail.findings{}.Resources{}.Id output=resourceId |spath detail.findings{}.Resources{}.Type
output=resourceType
| eval status=case(
    workflowstatus="NEW" AND recordstate="ACTIVE", "active",
    workflowstatus="RESOLVED" AND (recordstate="ACTIVE" OR recordstate="ARCHIVED"), "closed",
    workflowstatus="NEW" AND recordstate="ARCHIVED", "closed",
    true(), "unknown"
)
| search status="active" OR status="closed"
| eval reason=case(
    config_rule="securityhub-encrypted-volumes-3504b60a", "unencrypted-ebs"
)
| eval _time=now()
| sort 0 -_time
| dedup resourceId
| table resourceId,resourceType,status, reason,_time
| outputlookup HighVuln

2. Run the query and check the output in Statistics tab. The KV Store Lookups are now updated with the data. 
3. Save the search as a **report**, then go to Reports and **schedule** it (e.g., hourly/daily). 

---

### D. Building Dashboards

Dashboards help **visualize the security posture** and simplify monitoring.

#### i. Creating the Dashboard

- Go to **Dashboard**  
- Give it a name and choose layout type
  ![image](https://github.com/user-attachments/assets/272d91c3-a45d-4764-bab3-81b17157d8f0)

- Select Add Chart and choose the type of visualisations (Line, Bar, Pie, etc).
- In the configuration tab on the right side, select Set up Primary Data Source > Create Search , enter appropriate SPL query based on which attribute is to be visualised. In this project, the status count of misconfigurations of each severity is to be visualised. Hence, the following query was used,
| inputlookup HighVuln
| stats count by status
Select Apply and Close and then click on Save. Now you shall be able to view the visualisation. 
Do the same for each severity category or any other metric of your choice. 

![Cloud Security Posture_2025-05-22 at 11 46 25+0530_Splunk](https://github.com/user-attachments/assets/c67e1e12-4029-4672-8bff-533030ca0d6c)

---

## E. Conclusion

This project demonstrates the integration of **AWS Security Hub** with **Splunk** to monitor and visualize cloud misconfigurations. Through SPL-based classification, KV Store state tracking, and visual dashboards, it provides a comprehensive view of the security posture, aiding informed decision-making and proactive remediation.

---

## 6. Future Scope

1. **Automated Remediation**:  
   Integrate with **AWS Lambda** to auto-resolve specific misconfigurations.

2. **Hybrid Cloud Support**:  
   Extend support to **Azure Security Center** for multi-cloud visibility.

---

