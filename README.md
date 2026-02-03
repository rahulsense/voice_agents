# Voice Agents Setup Guide

## Overview

This guide provides comprehensive step-by-step instructions for creating, configuring, and deploying voice agents for candidate screening and reactivation purposes. Voice agents can be deployed for post-application screening, pre-application screening, or reactivation scenarios.

This document encompasses configuration in Retell AI, function setup, webhook & FWB configuration,and import to Lowcoder for testing and deployment to agency environments.

## Prerequisites

- Access to Retell AI platform
- Access to Lowcoder (requires AWS VPN connection)
- Agency details and configuration information
- Admin portal access for entity configuration

Note: All agencies are being onboarded to v2 voice going forward.

## Step 1: Select and Configure Voice Agent Template

### 1.1 Determine Agent Type

Based on agency requirements, identify the agent category and question format:

**Post-Application Agents** (calls placed after candidate applies)
- Dynamic->Static: [`v2 post application model bot - [Dynamic - Static]`](https://dashboard.retellai.com/agents/agent_b44756f8b7a69429beab7d5aff)
- Static->Dynamic: [`v2 post application model bot - [Static - Dynamic]`](https://dashboard.retellai.com/agents/agent_54763558070e75671eb8d0aa89)
- Static Only: [`v2 post application model bot - [Static]`](https://dashboard.retellai.com/agents/agent_67735ebc78564cce9e21fe29c0)
- Dynamic Only: [`v2 post application model bot - [Dynamic]`](https://dashboard.retellai.com/agents/agent_71376d015f044d7a8d8647389f)

**Pre-Application Agents** (calls placed to gather information before application)
- Dynamic->Static: [`V2 pre application model bot - [Dynamic - Static]`](https://dashboard.retellai.com/agents/agent_dcba1371617da052b26c7856ef)
- Static->Dynamic: [`V2 pre application model bot - [Static - Dynamic]`](https://dashboard.retellai.com/agents/agent_713ca5dfb6905c6827fc22a54c)
- Static Only: [`V2 pre application model bot - [Static]`](https://dashboard.retellai.com/agents/agent_d9436f90019c2d935fdd0a8b5d)
- Dynamic Only: [`V2 pre application model bot - [Dynamic]`](https://dashboard.retellai.com/agents/agent_8eb186fabaec429923af4006e8)

**Reactivation Agents**
- Reactivation: [`v2 reactivation agent model bot`](https://dashboard.retellai.com/agents/agent_4205fc04c1e119b63a10fc3f3b)

### 1.2 Duplicate and Rename the Voice Bot

Select the appropriate template and duplicate it. Rename according to agency and use case specifications.

## Step 2: Configure Main Prompt

Access the main prompt and update the following placeholder values identified by `<< >>` markers:

- **Voice bot name**: Update to the agent name (typically "Grace" or agency-specified name)
- **Agency name**: Replace with the specific agency identifier
- **Common Candidate Questions**: Under "### Answering Common Candidate Questions ###", update all questions and answers according to the Asana form specifications

Only these three elements require modification in the main prompt.

## Step 3: Configure Functions

### 3.1 Update Agency ID in Function Payloads

For each function except `end_call_global`:

1. Click the edit icon for the function
2. Locate the JSON payload
3. Update the `enum` value to match the agency ID being configured
4. Repeat for all applicable functions

### 3.2 Configure send_field_writeback_global Function

For the `send_field_writeback_global` function specifically:

1. Click edit to open the JSON payload
2. Update the `recipient_entity_type` enum value

**To determine the correct recipient_entity_type enum value:**

1. Open the admin portal
2. Navigate to the agency using the dropdown in the top right corner
3. Use the search feature (top left) to find "Writeback OWB Field Mapping"
4. for the`sense_entity_type is candidate`
5. The corresponding `entity_type` value becomes your `recipient_entity_type` enum value

## Step 4: Configure Voicemail Settings

If voicemail functionality is required:

1. Navigate to Call Settings
2. Under Voicemail Response, select "Leave a message if reaching voicemail"
3. Configure the voicemail message text according to Asana form specifications

## Step 5: Configure Webhook

1. Locate the Webhook URL configuration
2. Update the agency ID to match the agency for which the voice bot is being configured

## Step 6: Customize Questions and Responses

### 6.1 Introduction State

- Edit the introductory message the agent will deliver when initiating calls
- Update the agent name to match configuration
- Follow Asana form specifications for messaging

### 6.2 Dynamic Questions State (if applicable)

- Only change dynamic_questions_count wherever its mentioned in the prompt based on Asana form specifications 
### 6.3 Static Questions State (if applicable)

- Edit the question text
- Only change static_questions_count wherever its mentioned in the prompt based on Asana form specifications
- Do not modify the underlying structure or variable assignments
- Update all questions according to Asana form specifications

### 6.4 End Call State

- Edit the closing statement the agent will deliver when ending the call
- Align messaging with Asana form specifications

## Step 7: Configure Field Writebacks (FWB)

### 7.1 Gather Requirements

Request the following information from the client:

- `ats_field_name`: The specific ATS field identifier
- Question association: Which question the field writeback corresponds to

### 7.2 Validate Field Existence

1. Navigate to "Writeback OWB Field Mapping" in the admin portal
2. Use the search field (top right) to query the provided `ats_field_name`
3. Verify the field exists in the `ats_field_name` column
4. If not found, request the correct field name from the client

### 7.3 Determine Field Data Type

For the validated field, note the following attributes:

- `sense_data_type`: Determines the data type which also determines the format of the field writeback in the prompt
- `possible_values`: Lists acceptable values for the field

### 7.4 Implement Field Writebacks by Data Type

#### STRING Data Type

Use this format for string-based fields:

```
Once you get the answer for [question], store the value in <local_variable> 
and immediately and reliably call:

send_field_writeback_global with variable 
field_writeback: "{\"ats_field_name\": \"<local_variable>\"}"
```

Example:

```
Question: "What city do you currently live in?"

Once you get the answer for the city, store the value in <City> 
and immediately and reliably call:

send_field_writeback_global with variable 
field_writeback: "{\"city\": \"<City>\"}"
```

#### DOUBLE Data Type

Use this format for numeric fields requiring decimal precision:

```
Once you get the answer: Extract the numeric value.
Convert it to a double value. Always include a decimal point.

Examples: 25 → 25.0, "30/hr" → 30.0, "45.50" → 45.50

Store the value in <local_variable> as a DOUBLE.
Immediately call:

send_field_writeback_global with:
field_writeback: "{\"ats_field_name\": <local_variable>}"
```

Example:

```
Question: "What is your target salary range?"

Once you get the answer, extract the pay rate and convert to double format.
Store the value in <pay_rate>.
Immediately call:

send_field_writeback_global with:
field_writeback: "{\"preferredsalary\": <pay_rate>}"
```

#### LIST Data Type

Use this format for fields accepting multiple values:

```
Once the candidate answers, determine the value by matching against 
the provided mapping and store the matched value in <local_variable>.

Immediately and reliably call:

send_field_writeback_global with variable 
field_writeback: "{\"ats_field_name\": [\"<local_variable>\"]}"
```

Example:

```
Question: "What shifts are you available to work?"

Candidate responses map to: ['First Shift', 'Second Shift', 'Third Shift', 'Weekends', 'Part Time']

Once the candidate answers, match their response to the correct value.
Store the matched value in <preferred_shift>.
Immediately call:

send_field_writeback_global with variable 
field_writeback: "{\"customText5\": [\"<preferred_shift>\"]}"
```

### 7.5 Handle Complex Mappings with possible_values

If the field contains `possible_values`:

1. Review the number of possible values
2. If more than 1000 values: escalate to voice team for knowledge base implementation (beyond 1000 LLM tends to hallucinate)
3. If less than 1000 values: proceed with mapping

**To create value mappings:**

1. Copy the possible values list
2. Input to an LLM (Claude, Gemini Pro, or equivalent with large output tokens)
3. Use the following prompt:

```
Please return a single dictionary (key–value mapping). 
Every key must have a corresponding value, and no entries should be missing. 
Output only the dictionary in valid syntax—no extra text, comments, or formatting.
```

4. Use the returned dictionary in your prompt

Example implementation:

```
Question: "What language do you prefer?"

Language mapping: {"English": "Eng", "Spanish": "Span"}

Once you get the answer for preferred language, analyze it and 
map it to the values present in the dictionary above.

Store the mapped value in <preferred_language>.
Immediately call:

send_field_writeback_global with variable 
field_writeback: "{\"preferred_language\": \"<preferred_language>\"}"

For example, if the user says "English", <preferred_language> should map to "Eng".
```

To test out FWB please follow this [doc](https://github.com/rahulsense/testing_out_WBs)

## Step 8: Import Voice Agent to Agency via Lowcoder

### 8.1 Connect to VPN

Connect to the AWS VPN before proceeding. VPN access is required to reach internal services.

### 8.2 Access Lowcoder

Navigate to Lowcoder via the Internal Hub: https://internalhub.prod.sensehq.co/apps/6753f53d273afe4a77cd1012/view

### 8.3 Complete Import Configuration

Fill out the import form with the following values:

| Field | Value |
|-------|-------|
| Environment | `prod` |
| Current Agency | `multientity (bullhorn)` |
| Retell Dashboard Bots | Select your newly created bot |
| Entity | `bh_job_submission` |
| Phone Number | Select default phone number |

### 8.4 Execute Import

Click the Import button to complete the import process. The voice agent is now available for testing in the multientity environment.

## Testing and Validation

After importing to Lowcoder:

1. Verify the agent appears in the voice flows section
2. Test all function calls, particularly field writebacks
3. Confirm webhook integration with the agency
4. Validate question and answer flows
5. Test voicemail functionality if applicable

## Deployment to Agency

Once testing in multientity is successful:

1. Re-run the import process with the target agency selected in the "Current Agency" field
2. Perform agency-specific testing
3. Confirm all FWB operations write to correct ATS fields
4. Validate agency-specific configurations

Note: if you are importing to multientity and testing it there, the agency_id should be of multientity for all the functionality to work

## Troubleshooting

- **Field writeback failures**: Verify the `ats_field_name` exists in OWB Field Mapping
- **Function call errors**: Ensure agency ID is correctly updated in all function payloads
- **Webhook issues**: Confirm webhook URL includes correct agency ID
- **Data type mismatches**: Verify field data type matches the writeback format (STRING, DOUBLE, LIST)

## Support and Escalation

For issues beyond standard configuration, escalate to the voice team.
