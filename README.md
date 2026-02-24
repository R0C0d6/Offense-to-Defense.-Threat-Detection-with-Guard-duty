
# Offense-to-Defense: Threat Detection with AWS GuardDuty (DVF-Cybersecurity)

This project demonstrates an end-to-end adversary emulation in AWS, deploying a vulnerable application, simulating attacks, detecting them with AWS native security services, and applying remediations. It follows the full cycle:

Attack → Detection → Validation → Remediation

The lab environment leverages OWASP Juice Shop (a deliberately vulnerable web app) deployed via CloudFormation for reproducibility and cleanup.



## Project Objectives

Safely simulate credential theft and misuse in a controlled AWS environment
Validate detection and monitoring using **GuardDuty, Config, and Security Hub**
Identify and record control gaps (e.g., reliance on IMDSv1)
Apply and verify remediations through iterative lab runs
Build repeatable infrastructure for offensive and defensive security testing



## Lab Setup

1. CloudFormation Deployment
   Deployed a stack containing OWASP Juice Shop and supporting resources.
   Recorded key outputs:
   `JuiceShopURL` (application endpoint)
   `TheSecureBucket` (S3 bucket for sensitive objects).

2. Accessing Juice Shop
   Navigated to the Juice Shop URL.
   Dismissed the welcome dialog and cookie consent notice.
   Switched the UI language to Japanese for testing purposes.

3. Authentication Bypass
   Simulated an admin login bypass.
   Confirmed successful sign-in as administrator.



## Credential Exposure

From the admin profile page, executed a server-side injection that exposed system details.

Retrieved a public JSON file (`aws-creds.json`) containing temporary AWS credentials:
`AccessKeyId`
`SecretAccessKey`
`SessionToken`

These credentials allowed attacker-style access to AWS resources until expiration.



## Retrieving Sensitive Files
Using the compromised credentials:

1. Exported them as environment variables.

2. Set the `AWS_SECURE_BUCKET` variable to the bucket name from CloudFormation outputs.

3. Listed bucket contents:
   ```bash
   aws s3 ls s3://$AWS_SECURE_BUCKET/
   ```

4. Downloaded a sensitive file:
   ```bash
   aws s3 cp s3://$AWS_SECURE_BUCKET/secret-information.txt .
   ```

5. Verified the exfiltrated contents locally.
   

## Threat Detection with GuardDuty
Enabled Amazon GuardDuty to monitor VPC Flow Logs and CloudTrail events.
Findings confirmed:
Use of stolen temporary credentials from an unexpected location.
Suspicious activity flagged as anomalous.
GuardDuty provided evidence of unauthorized S3 access linked to compromised credentials.



## Misconfiguration Detection with Config & Security Hub

1. AWS Config
   Enabled with default settings for baseline resource tracking.

2. AWS Security Hub
   Activated with the AWS Foundational Security Best Practices v1.0.0 standard.
   Generated a compliance score and surfaced multiple failed controls.

3. Key Finding
   EC2.8: EC2 instances should enforce Instance Metadata Service Version 2 (IMDSv2).
   This was directly related to the credential theft scenario, highlighting IMDSv1 as an exploitable weakness.



## Key Learnings

Repeatable Infrastructure: CloudFormation ensured reproducibility, making it possible to re-run attacks and detections consistently.
Credential Risk: Temporary credentials and instance metadata remain high-value attack vectors without proper hardening.
Detection-in-Depth: GuardDuty provided actionable signals; Config and Security Hub revealed deeper systemic misconfigurations.
Hands-on Value: End-to-end exercises reveal risks not captured in static checklists.



## Immediate Remediations

Enforce IMDSv2 and disable IMDSv1.
Remove sensitive files from public app directories.
Apply least-privilege policies to roles and temporary credentials.
Harden S3 bucket policies and enable object-level logging.
Integrate GuardDuty findings into automated incident response playbooks.



## Next Steps

Package CloudFormation + runbook for reproducible training environments.
Build a GuardDuty → Security Hub → SIEM pipeline for automated triage.
Re-run the lab post-remediation to validate fixes.
Expand with additional adversary emulation scenarios.



## Cleanup

Empty S3 Bucket: Remove contents of the lab bucket via the S3 console.
Delete Stack: Tear down the CloudFormation stack to avoid incurring costs.



## Project Status

This lab successfully demonstrated:

Credential theft simulation
Detection with GuardDuty
Compliance checks with Security Hub & Config
Identification of IMDS-related misconfigurations
A full attack → detect → remediate cycle in AWS


📜 License
This project is licensed under the MIT License.
See the LICENSE file for more information.
