# SAlesInsightsUsingAgentforce
# 🤖 AI-Assisted Opportunity Management (Agentforce for Sales)

> **Goal:** Automate opportunity insights, next-best actions, and email drafts for sales reps using Salesforce + AI (Agentforce style).

---

## 🚀 Overview

This project demonstrates how to integrate **Agentforce / AI-powered assistance** into Salesforce to help sales teams:

- Get instant opportunity summaries.  
- Receive actionable next-step recommendations.  
- Auto-generate personalized email drafts.  
- Continuously score and prioritize deals.  

Built entirely on **Salesforce Platform**, leveraging **Apex, Flows, Named Credentials**, and **LLM API integrations**.

---

## 🧭 Architecture Flow

User → Lightning UI / Agentforce
→ Salesforce (Opportunity, Account, Contact, Activities)
→ Apex / Platform Event → LLM API (Named Credential)
→ JSON Response → Flow → Update Salesforce fields, Tasks, Email drafts
→ Analytics & Monitoring


---

## 🎯 Business Objectives

- Reduce manual CRM data entry.  
- Improve forecast accuracy and pipeline visibility.  
- Boost sales productivity with AI-generated insights.  
- Increase opportunity win-rates via timely engagement.

---

## 🗂️ Data Model

**Core fields (Opportunity):**
- `Name`, `StageName`, `Amount`, `CloseDate`, `Probability`  
- `Account.Name`, `Industry`, `Owner.Name`, `LastActivityDate`  
- `AI_Health_Score__c` (Number)  
- `AI_Summary__c` (Long Text)  
- `AI_Next_Action__c` (Text)  
- `AI_Confidence__c` (Percent)

**Interaction Log Object:** `AI_Interaction__c`
| Field | Description |
|-------|--------------|
| `Opportunity__c` | Related Opportunity |
| `Prompt__c` | Full text sent to LLM |
| `Response__c` | Full JSON response |
| `User__c` | Triggered by |
| `Timestamp__c` | Execution time |
| `ActionTaken__c` | System or user action |

---

## 🧠 AI Prompt Template

**System Prompt**


You are a sales assistant. Given Opportunity details, return a strict JSON:
{ summary, health_score, risk_reasons[], recommended_next_action, email_subject, email_body, confidence }

Summarize Opportunity:
Name: {{Name}}
Account: {{Account.Name}}
Stage: {{StageName}}
Amount: {{Amount}}
CloseDate: {{CloseDate}}
LastActivityDate: {{LastActivityDate}}
RecentNotes: {{Recent_Notes}}
Return valid JSON.


**Example Response**
```json
{
  "summary": "ACME opportunity: mid-funnel, procurement engaged.",
  "health_score": 72,
  "risk_reasons": ["Pending PO", "Legal review open"],
  "recommended_next_action": "Call procurement to confirm timeline.",
  "email_subject": "Quick sync re: ACME procurement timeline",
  "email_body": "Hi <Name>, thanks for the last call. Can we schedule a quick check-in to confirm next steps?",
  "confidence": 0.86
}



**Example Response**
```json
{
  "summary": "ACME opportunity: mid-funnel, procurement engaged.",
  "health_score": 72,
  "risk_reasons": ["Pending PO", "Legal review open"],
  "recommended_next_action": "Call procurement to confirm timeline.",
  "email_subject": "Quick sync re: ACME procurement timeline",
  "email_body": "Hi <Name>, thanks for the last call. Can we schedule a quick check-in to confirm next steps?",
  "confidence": 0.86
}


public with sharing class AIService {
    @AuraEnabled
    public static Map<String,Object> analyzeOpportunity(Id oppId) {
        Opportunity opp = [
            SELECT Id, Name, StageName, Amount, CloseDate, LastActivityDate, Account.Name
            FROM Opportunity WHERE Id = :oppId LIMIT 1
        ];
        
        String prompt = buildPrompt(opp);
        HttpRequest req = new HttpRequest();
        req.setEndpoint('callout:LLM_Named_Cred/v1/generate');
        req.setMethod('POST');
        req.setHeader('Content-Type','application/json');
        req.setBody(JSON.serialize(new Map<String,Object>{
            'prompt' => prompt, 'max_tokens' => 600
        }));
        
        Http http = new Http();
        HttpResponse res = http.send(req);
        Map<String,Object> output = (Map<String,Object>) JSON.deserializeUntyped(res.getBody());
        return output;
    }
}


🌀 Flow Integration

Add Apex Action → AIService.analyzeOpportunity.

Map the output JSON fields to Opportunity updates.

Create:

Task → Recommended next action.

EmailMessage → Draft email body/subject.

AI_Interaction__c → Log prompt/response.
| Mode             | Trigger                        | Use Case                               |
| ---------------- | ------------------------------ | -------------------------------------- |
| **On-Demand**    | Quick Action                   | Rep clicks "Summarize Opportunity"     |
| **Event-Driven** | Stage change / Activity update | Fresh analysis on key events           |
| **Batch**        | Nightly job                    | Update all open deals for dashboarding |


🔐 Security Best Practices

Use Named Credentials (never store API keys in code).

Restrict Apex/Flow access via Permission Sets.

Redact PII in prompts.

Log all interactions for audit and compliance.

✅ Testing & Monitoring

Unit test with HttpCalloutMock.

Sandbox test flows before production.

Handle edge cases (missing data, malformed JSON).

Create dashboards:

Avg AI_Health_Score__c

AI task completion rate

Conversion % of AI-recommended deals

📈 Success Metrics

↓ Manual updates per rep.

↑ Forecast accuracy.

↑ Win rate for AI-scored top opportunities.

↑ Rep satisfaction and CRM adoption.

🧪 Example User Flow

Rep opens Opportunity.

Clicks “AI: Summarize Opportunity.”

LLM analyzes and returns JSON insights.

Opportunity gets updated → Task & email draft auto-created.

Rep reviews → sends email → moves deal forward.

🧰 Deployment Checklist

 Fields & objects created.

 Named Credential configured.

 Apex callout class + test class deployed.

 Flow wired to Apex output.

 Quick Action added to Opportunity page.

 Pilot rollout & feedback loop.

 Metrics dashboard built.
