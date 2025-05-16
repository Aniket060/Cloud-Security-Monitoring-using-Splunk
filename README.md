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

> *(Insert diagram here showing AWS → EventBridge → CloudWatch → Splunk → Dashboards. Optionally, queries and KV Store connections can go on a separate page.)*

---

## 4. Setup and Configuration

Prerequisite :
1. Splunk Enterprise Installation on any platform. The Splunk Server should be up and running along with access to console page. 
   
---

### B. Configuring AWS Security Hub

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

### C. Installing AWS Add-on for Splunk

Splunk is a central **SIEM (Security Information and Event Management)** platform for:

- Log ingestion  
- Monitoring and visualizing security events  
- Querying and alerting  

Steps:

1. Go to Apps>Find more Apps and install **Splunk Add-on for AWS**
   ![image](https://github.com/user-attachments/assets/df47d7cd-d6f4-4891-a150-1e7188210c14)

3. Configure **CloudWatch Logs** as the ingestion method:
    - Go to Cloudwatch > Logs > Log Groups and create a new Log Group
    - ![image](https://github.com/user-attachments/assets/deb807f8-3c24-49df-b711-ec0ad962b6b4)
    - Use **EventBridge rules** to send Security Hub findings to CloudWatch
    - ![image](https://github.com/user-attachments/assets/cc2bc251-7edd-4cd3-8430-2b297ed28caa)
![image](https://github.com/user-attachments/assets/609afb51-e606-47d4-9f20-65e097d029aa)
![image](https://github.com/user-attachments/assets/84e44a95-93f6-4a75-b852-3866a7453f32)

    - In Splunk AWS Add-on, Create New Input > Custom Data Type > CloudWatch Logs  
    - Provide all the necessary details
      ![image](https://github.com/user-attachments/assets/f7a2881a-2f37-4349-8b85-4889de1bf196)

> After finishing, the logs will start getting ingested from Security Hub to Splunk. Verify the same by using Search along with the right search index. 
![image](https://github.com/user-attachments/assets/7144a218-269c-47fa-9444-cb47a4e62972)

---

### D. Extracting and Preparing Security Events

#### i. KV Store Lookups

To classify events, use **KV Store-backed lookups**:

- Go to `Settings > Knowledge > Lookups`  
- Create lookup tables for each severity level:  
  - `low_severity_lookup`  
  - `medium_severity_lookup`  
  - `high_severity_lookup`  
  - `critical_severity_lookup`  
- Set field names and permissions appropriately

> Insert screenshots for lookup creation and permissions setup.

#### ii. Creating Scheduled Reports

To populate the lookups:

1. Write SPL queries to extract misconfiguration data  
2. View result statistics and confirm expected structure  
3. Save the search as a **report**, then **schedule** it (e.g., hourly/daily)

> Add example SPL queries and screenshots of the process.

---

### E. Building Dashboards

Dashboards help **visualize the security posture** and simplify monitoring.

#### i. Creating the Dashboard

- Go to **Dashboard**  
- Give it a name and choose layout type  
- Add panels using preferred chart types (e.g., *Pie*, *Bar*)

#### ii. Configuring Dashboard Panels

- Select a panel, choose *Create Search*  
- Use SPL queries to fetch data from KV Store lookups  
- Save the panel and repeat for:
  - *High*, *Medium*, *Low*, and *Critical* Severity
  - *Resource Types*

> Include example queries and screenshots of the final dashboards.

---

## 5. Conclusion

This project demonstrates the integration of **AWS Security Hub** with **Splunk** to monitor and visualize cloud misconfigurations. Through SPL-based classification, KV Store state tracking, and visual dashboards, it provides a comprehensive view of the security posture, aiding informed decision-making and proactive remediation.

---

## 6. Future Scope

1. **Automated Remediation**:  
   Integrate with **AWS Lambda** to auto-resolve specific misconfigurations.

2. **Hybrid Cloud Support**:  
   Extend support to **Azure Security Center** for multi-cloud visibility.

---

