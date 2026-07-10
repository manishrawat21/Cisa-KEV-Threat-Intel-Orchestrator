# CISA KEV Threat Intel Orchestrator

Zero-touch pipeline that turns newly weaponized CVEs from the CISA Known Exploited Vulnerabilities (KEV) catalog into production-ready Sigma detection rules, automatically, every week.

## The problem

Manually turning a new KEV entry into a usable detection is slow: check the advisory, research the attack pattern, write a Sigma rule, test it, brief the team. That's roughly 4-6 hours per CVE. Ten new entries in a week is a full week of work, and most SOCs don't have a spare analyst-week lying around every Monday.

## What this does

Every Monday at 8:00 AM, the pipeline:

1. **Pulls the live CISA KEV catalog** and isolates newly added, weaponized CVEs since the last run.
2. **Generates a Sigma detection rule per CVE** using Google Gemini, constrained by a structured prompt that forces the model to:
   - target the exact product/vendor named in the CVE (not a generic placeholder rule)
   - use the correct Sysmon EventIDs for the technique (process creation, network, image load, LSASS, registry)
   - include 3-5 condition filters to control false positives
   - map to a relevant MITRE ATT&CK technique
   - output clean YAML only, ready to drop into Splunk, Elastic, or Wazuh
3. **Logs every processed CVE to a Google Sheet** a running, timestamped audit trail of when a vulnerability was disclosed vs. when a detection rule existed for it. Built with SOC 2 / ISO 27001 proactive-vulnerability-management evidence requirements in mind, not just as a log.
4. **Emails an analyst-ready weekly briefing** with the CVE list, severity, and draft detection logic.

## Architecture

```
CISA KEV Catalog (API)
        │
        ▼
  n8n Schedule Trigger (Mon 8:00 AM)
        │
        ▼
  Filter: new + weaponized CVEs
        │
        ▼
  Gemini (Sigma rule generation, one CVE per call)
        │
        ├──▶ Google Sheets (audit trail)
        │
        └──▶ Email digest (analyst briefing)
```


## Stack

n8n  · CISA KEV API · Google Gemini · Google Sheets · Gmail/SMTP

## Status and next steps

This is a working weekly pipeline, not a finished product. Currently on the roadmap:

- Auto-deploying generated rules directly to the security team's SIEM instead of email-only delivery
- Automated testing of generated rules against sample logs before they reach an analyst
- Direct integration with the incident response system

## Repo contents

```
/workflow           n8n workflow export (importable JSON)
/prompts             Gemini prompt templates used in the Sigma generation node
/sample-output       One real weekly briefing (sanitized)
README.md
```

## Author

Manish Rawat | Detection Engineer & SOC Analyst (Independent Research)
[LinkedIn](https://linkedin.com/in/manishrawat21) · [GitHub](https://github.com/manishrawat21) · [Medium](https://medium.com/@manishrawat21)
