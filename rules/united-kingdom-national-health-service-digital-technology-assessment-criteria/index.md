# UK NHS Digital Technology Assessment Criteria (DTAC) checklist

Converted from the DTAC Form v2.0 (24 February 2026). From 6 April 2026, manufacturers must provide this form instead of v1.0 when health and care organisations request it.

- Sections A and B are non-assessed context.
- Sections C1–C4 are assessed. The product must meet them to pass.
- Section D is scored for comparison only, not pass/fail.
- The manufacturer of the digital health technology (DHT) completes the form. If the seller is not the manufacturer, they work together.
- Answer for the specific product and version under assessment, not for the organisation in general.

## A. Company information (non-assessed)

| Code | Question | Options |
|---|---|---|
| A1 | Name of your company. | Free text |
| A2 | Name of your product. | Free text |
| A3 | Version number of the product this form corresponds to. | Free text |
| A4 | Type of product. | Standalone Software Application or Mobile Application \| Wearable \| Software as a Service (SaaS) \| Other (describe) |
| A5 | Name and job title of the key contact. | Free text |
| A6 | Key contact's email address. | Free text |
| A7 | Key contact's phone number. | Free text |
| A8 | Registered address of your company. | Free text |
| A9 | Country where your organisation is registered. | Free text |
| A10 | Companies House number, charity number, or other organisational reference. | Free text |
| A11 | Date of your last CQC assessment, if you must register with the CQC. | Date \| Not applicable |
| A12 | Latest CQC report, if applicable. | Provided \| Not applicable |

## B. Value proposition (non-assessed)

| Code | Question | Options |
|---|---|---|
| B1 | Intended use of the product. | Patient Care or Support \| Diagnostics \| Clinical Support \| Workforce Support or Management \| Other |
| B2 | Clear, high-level description of what the product does and how it is used. | Free text |
| B3 | Intended users and the intended or proven benefits, and how the benefits were validated. Include any evaluation or clinical trial information. | Free text |
| B4 | Data flow between the product and the Health IT system, and a user journey map where applicable. May be included in the DPIA, in which case attach it separately. | Provided \| Not available |

## C1. Clinical safety (assessed)

Provide responses and documentation for the specific version being assessed. If the product ships with hardware, the DCB0129 documentation covers the whole Health IT system, hardware included. If you consider C1 not applicable, submit your rationale.

| Code | Question | Options | Notes |
|---|---|---|---|
| C1.1.1 | Does the product or any component qualify as Software or AI as a Medical Device under the UK Medical Devices Regulations 2002? | Yes \| No | Pass: if Yes, a completed [Pre-acquisition questionnaire (PAQ)](https://www.england.nhs.uk/publication/pre-acquisition-questionnaire/) is provided.<br>If No, skip to C1.2. |
| C1.1.2 | Is the product classified as a standalone medical device? | Yes \| No | Read the [DCB0129 and DCB0160 applicability guidance](https://digital.nhs.uk/services/clinical-safety/applicability-of-dcb-0129-and-dcb-0160) first.<br>If Yes, go to C2. If No, go to C1.2. |
| C1.2 | Is the product designed to provide electronic information that influences, supports or manages real-time or near-real-time direct care? | Yes \| No | If Yes, skip to C1.2.2. If No, answer C1.2.1 and then go to C2. |
| C1.2.1 | If No to C1.2, justify why the product is not in scope of DCB0129. | Free Text | Commissioners can challenge the justification. |
| C1.2.2 | Have you carried out clinical risk management activities that comply with DCB0129? | Yes \| No | Pass: the manufacturer confirms compliance. |
| C1.2.3 | Detail your clinical risk management system. | Provided \| Not provided | Pass: a DCB0129-compliant system exists and was followed throughout development, with evidence that risks were identified, evaluated and mitigated across the lifecycle. |
| C1.2.4 | Supply your Clinical Safety Case Report and Hazard Log. | Provided \| Not provided | Include:<br>- scope of the assessment<br>- summary of your clinical risk management approach and activities<br>- summary of the hazard assessment: risks, evaluation, and mitigations considered and implemented<br>- test summary showing functional and non-functional testing<br>- summary of test issues (defects)<br>- hazards that need user or commissioner action to reach acceptable mitigation (for example training or business process change)<br>- hazard log (can be appended)<br>- declaration of the risk scoring scheme you used<br>- residual clinical risks and related operational constraints<br>- hazards transferred to the deploying organisation, with declared risk controls<br>- outstanding test issues with a possible clinical safety impact<br>- hazards you could not mitigate as low as reasonably practicable<br><br>Pass: the report and log comply with DCB0129 and are proportionate to the product's scale and clinical functionality. |
| C1.2.5 | Name, profession and registration details of your Clinical Safety Officer (CSO). | Free Text | The CSO must be a suitably qualified and experienced clinician, hold current registration with an appropriate professional body, know risk management as applied to clinical domains, and have enough responsibility to ensure DCB0129 processes are followed.<br>Pass: a named CSO, which may be outsourced.<br>The v1.0 requirement for NHS DTAC-specific CSO training no longer applies. Suitable training is still strongly recommended. |

## C2. Data protection (assessed)

| Code | Question | Options | Notes |
|---|---|---|---|
| C2.1 | If you have direct or remote access to patient data or NHS systems, confirm your Data Security and Protection Toolkit ([DSPT](https://www.dsptoolkit.nhs.uk/)) status. | Confirmed \| Unable to Confirm \| No access to patient data or national NHS System | Pass: Standards Met or Exceeded for the current year, or the previous year if you have no current return yet. Assessors validate this against the DSPT database. |
| C2.2 | Does the product or service process any personal data or data about deceased individuals, including data processed by a sub-processor? | Yes \| No | If No, skip to C3.<br>You may answer No if you have no role in operating or hosting the product and no means of accessing its data, including remote support access. |
| C2.2.1 | Evidence of current ICO registration. | Provided \| Not provided | Pass: evidence with expiry date, such as a screenshot of the registration number. Assessors validate it against the ICO Register of Fee Payers. |
| C2.2.2 | Data Protection Impact Assessment (DPIA) for the product. | Provided \| Not provided | Must cover:<br>- summary of the product and how it processes data<br>- list of data fields required<br>- how data flows into, within and out of the product<br>- end-user security controls: on/off-boarding and access limits<br>- technical and organisational measures for data in transit and at rest, proportionate to risk<br>- countries where data is stored or flows through<br>- whether the manufacturer's staff can access personal data, with proportionate access and an identified legal basis<br>- who is the controller for each element of processing<br>- retention and disposal arrangements<br>- processors and sub-processors, with confirmation of legally binding agreements<br>- confidentiality, availability and integrity risks and their mitigations<br>- how the product supports data subject rights |
| C2.2.3 | Copy or link to the product's transparency information (privacy notice). | Provided \| Not provided | Pass: transparency materials are available to the buyer to help meet UK GDPR transparency requirements. |
| C2.2.4 | Product terms and conditions on use of user data, end user licence agreement or equivalent. If this does not apply, state why. | Provided \| Not provided \| Not applicable | Pass: the terms are clear and fair on privacy and data use. |
| C2.2.5 | Where does the product, including third-party components, store and process data? | UK only \| Outside of UK | If UK only, skip to C3. |
| C2.2.6 | If outside the UK, name the country and explain how the arrangements comply with current legislation. | Free text | Pass: a statement that the arrangements comply, such as UK adequacy status, an IDTA, or binding corporate rules. Non-adequacy routes need a transfer risk assessment. |

## C3. Technical security (assessed)

| Code | Question | Options | Notes |
|---|---|---|---|
| C3.1 | Attach your Cyber Essentials certificate. | Provided \| Not Provided | Pass: the certificate is valid and in date (12-month validity). Assessors check it against the IASME database. |
| C3.2 | Have you signed the [Cyber Security Charter for Suppliers to the NHS](https://digital.nhs.uk/cyber-and-data-security/guidance-and-assurance/cyber-security-charter-for-suppliers-to-the-nhs)? | Yes \| No | If Yes, skip the rest of C3. If No, answer C3.3 to C3.6. |
| C3.3 | For internet-based or internet-accessible products, provide the summary report of an external penetration test from the last 12 months that covered the OWASP Top 10. | Provided \| Not provided | Pass: a third-party test covering the OWASP Top 10, with no vulnerabilities scoring 7.0 or above on CVSS. |
| C3.4 | Confirm the software was produced in line with the DSIT/NCSC [Software Security Code of Practice](https://www.gov.uk/government/publications/software-security-code-of-practice/software-security-code-of-practice), and that you commit to its principles: secure design and development, secure build environment, secure deployment and maintenance, and communication with customers. | Confirmed / Unable to Confirm | Pass: development aligns to the code and you commit to its principles. |
| C3.5 | Confirm you have a plan to implement multi-factor authentication (MFA) for all account types, preferably through identity federation. | Yes \| No | Pass: a plan is in place. |
| C3.5.1 | If applicable, confirm all supplier accounts with privileged access to the product have MFA, or equivalent MFA at the remote end. | Yes \| No \| Not applicable as supplier has no access to the product | Pass: MFA is enforced on all privileged and remote access connections. It need not be enforced on the product itself. |
| C3.6 | Confirm logging and reporting requirements have been defined. | Yes \| No | To answer Yes, logging such as audit trails of all access must be in place.<br>Pass: requirements are defined. |

## C4. Interoperability (assessed)

| Code | Question | Options | Notes |
|---|---|---|---|
| C4.1 | Does the product expose any APIs or integration channels for other products used in health or social care? | Yes \| No | If No, skip to C4.2. |
| C4.1.1 | List the international or industry interoperability standards your APIs use and explain why they are appropriate. | Free text | Pass: relevant standards are listed and justified. |
| C4.1.2 | Confirm the APIs follow GDS Open API best practice and are openly documented and freely available to third parties. | Confirm \| Cannot confirm | Pass: confirmed, or an approach is set out in C4.1.3. |
| C4.1.3 | If you cannot confirm C4.1.2, set out the basis on which your APIs are documented and made available to third parties. | Free Text | Pass: you set out the basis on which third parties can build integrations. |
| C4.2 | Is the product intended to share or receive data with national or local care or administrative systems where patient identity is relevant? | Yes \| No | If No, skip to C4.3. |
| C4.2.1 | Can the product use the NHS number to identify patient data when exchanging data? | Yes \| No | Pass: Yes, or an alternative approach for data quality.<br>If No, go to C4.2.3. |
| C4.2.2 | Does the product integrate with the NHS Personal Demographics Service, or other local record systems, to establish or validate the NHS number? | Yes \| No | Pass: Yes, or an alternative approach for data quality.<br>If onboarding to PDS is incomplete, describe it in C4.2.3.<br>If Yes, skip to C4.2.4. |
| C4.2.3 | If you answered No to C4.2.1 or C4.2.2, describe how you identify patient records correctly and keep data quality. | Free Text | Pass: you set out how patients are correctly identified and how data accuracy is maintained when integrating with other systems. |
| C4.2.4 | If the product is used directly by patients, do you use NHS login to verify and authenticate users? | Yes \| No \| Not Applicable – product is not used directly by patients. | Pass: Yes, or adequate data protection measures in C4.2.6.<br>If Yes, skip to Section D. If No, go to C4.2.5. |
| C4.2.5 | If public health or adult social care organisations use the product to deliver care services, does it support compliance with the [DAPB3051](https://digital.nhs.uk/data-and-information/information-standards/governance/latest-activity/standards-and-collections/dapb3051-identity-verification-and-authentication-standard-for-digital-health-and-care-services) identity verification and authentication standard? | Yes \| No \| Not Applicable – product is not used by public health or adult social care services | Pass: Yes, if applicable.<br>If Yes, skip to Section D. If Not applicable, complete C4.2.6. |
| C4.2.6 | If you do not use NHS login, describe how you authenticate users and what data protection measures are in place. | Free Text | Pass: the approach protects user privacy. |

## D1. Usability and accessibility (scored, not pass/fail)

| Code | Question | Options | Notes |
|---|---|---|---|
| D1.1 | Explain how the product fits into existing systems or care pathways. For example, provide a user journey or instructions for use. | Provided \| Not provided | |
| D1.2 | Do you test with intended users to validate usability? | Yes \| No | |
| D1.3 | Confirm you have read the [Accessible Information Standard](https://www.england.nhs.uk/about/equality/equality-hub/patient-equalities-programme/equality-frameworks-and-information-standards/accessibleinfo/) and considered it in your design. | Confirm \| Cannot confirm | |
| D1.4 | Is the product a web or mobile application? | Yes \| No | If No, Section D is complete. |
| D1.4.1 | Does it comply with WCAG 2.2 level AA or higher? | Yes \| No, but a plan and timeline for achieving WCAG 2.2 AA or higher is in place \| No \| Not applicable as not a web of mobile application | If you have a plan, answer D1.4.2. Otherwise go to D1.4.3. |
| D1.4.2 | If you have a plan, give the timescale for achieving WCAG 2.2 AA. | Free Text | |
| D1.4.3 | Link to your published accessibility statement. | Free text | |
| D1.5 | Average service availability over the past 12 months, as a percentage to two decimal places. | Free text | |

## Supporting documentation

Label each document with your company name, the question number and the date of submission.

| Code | Document |
|---|---|
| A12 | CQC report |
| B4 | User journeys and data flows |
| C1.1.1 | Pre-acquisition questionnaire (PAQ) form |
| C1.2.3, C1.2.4 | Clinical risk management system and Clinical Safety Case Report |
| C1.2.4 | Hazard Log |
| C2.2.1 | ICO registration |
| C2.2.2 | Data Protection Impact Assessment (DPIA) |
| C2.2.3 | Transparency information (privacy notice) |
| C2.2.4 | Product terms and conditions on use of user data, end user licence agreement or equivalent |
| C3.1 | Cyber Essentials certificate |
| C3.3 | External penetration test summary report |
| D1.1 | User journeys, or how the product fits into a user pathway |
