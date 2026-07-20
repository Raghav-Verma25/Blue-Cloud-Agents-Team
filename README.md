Project Overview
River is a virtual admissions assistant built on Salesforce Agentforce for Riverstone University. It engages prospective students 24/7 through the institutional website, guiding them from initial inquiry through application creation, while grounding every response in real Salesforce data and official Knowledge articles.
River is designed to address a structural problem in higher-education admissions: prospective students expect instant, personalized guidance, but admissions counselors are finite, and routine queries consume time better spent on complex cases. River automates the routine and escalates the exceptional, giving every prospective student a tailored experience while freeing counselors to focus on high-touch cases.

### The Student Journey River Enables
A prospective student — for example, Sarah Patel, a marketing manager pivoting into
data-driven roles — can complete an end-to-end journey in a single conversation:
1. Get greeted with real-time program names retrieved from Salesforce
2. Get identified by email or captured as a new Person Account
3. Get a recommendation matched to her background (Masters in Computer Science →
Data Science Specialization)
4. Learn about policies with cited Knowledge articles (deadlines, GMAT waivers, tuition)
5. Start an application with real record creation in Salesforce
6. Get escalated to a human counselor when a query is complex (international credentials, deadline extensions)
All in under 5 minutes, in a single continuous chat.

Key Features
### 🤖 Six-Subagent Modular Architecture
River is composed of four custom subagents plus two built-in ones, each owning a specific
conversational scope. The Agent Router directs messages semantically to the right subagent.
| Subagent | Scope |
|---|---|
| Welcome and Visitor Identification | Greets visitors, identifies returning students, creates new Person Accounts |
| Program Course and Scholarship | Recommends programs and tracks, retrieves fees, matches scholarships, creates applications |
| Admission Policies and Enrollment Process | Answers policy and process questions from grounded Knowledge articles |
| Escalation | Creates Case records for complex or exception-based queries |
| Agent Router (built-in) | Salesforce native topic routing |

### 📚 Knowledge Grounding via Data Library
Every deadline, test requirement, and process step is cited from Riverstone's official Knowledge base, leaving little to no scope for hallucination. This is achieved through:
- Data Categories — 5 Riverstone-authored articles tagged with `Riverstone University Specific
- Data Library — `Riverstone Library` filters retrieval to those tagged articles only
- Answer Questions with Knowledge standard action — invokes semantic retrieval and returns cited excerpts

### 🎯 Personalized Two-Level Recommendations
River recommends both a parent Learning Program (e.g., Masters in Computer Science)
AND a specific Learning Program Plan / Track (e.g., Data Science Specialization), tying the
recommendation to the visitor's stated background. Tuition, application fee, and minimum GPA
come from the Plan record.

### 💬 Structured Discovery Flow (State Machine)
The program discovery experience follows a strict listing-then-selection pattern:
1. List programs → wait for selection
2. Ask "tracks or scholarships?" → wait for choice
3. List items in the chosen branch → wait for selection
4. Present full details for the selected item
This prevents information overload and gives the visitor an explicit choice at every step.

### 📝 Real-Time Application Creation
When a visitor confirms intent to apply, River creates an **Individual Application** record in real
time — linked to the Person Account, Learning Program, and Learning Program Plan. The Application ID is returned to the visitor for reference.

### 🤝 Empathetic Case Escalation
For complex queries (international credential evaluation, deadline extensions, accessibility accommodations), River creates a Case record with full conversation context. The response acknowledges the situation empathetically before sharing the Case ID.

### 🧠 Conversation Memory
River maintains context across turns. If a visitor mentions six years of experience early in the chat, River references that context later when checking scholarship or test-waiver eligibility.

Setup & Installation
### Prerequisites
Before deploying River, ensure your Salesforce org meets these requirements:
- **Salesforce Edition:** Enterprise, Performance, or Unlimited with Education Cloud
- **Person Account:** Enabled in your org
- **Agentforce:** Enabled with the appropriate user licenses
- **Education Cloud:** Standard Education Cloud objects available (`LearningProgram`,
`LearningProgramPlan`, etc.)
- **Knowledge:** Lightning Knowledge enabled
- **Experience Cloud:** A site available to embed the chat widget
- **Messaging for In-App and Web:** Enabled for embedded messaging
- **Permission Set Licenses:**
- `Education Cloud AI Agent User`
- Any relevant Agentforce Service Agent licenses
  
### Step 1: Set Up the Data Model
1. **Load Learning Programs and Plans**
- Create at least 2 top-level `LearningProgram` records (e.g., Masters in Computer Science,Masters in Business Administration)
- For each program, create multiple `LearningProgramPlan` records (specialization tracks) with populated Tuition Fee, Application Fee, and Minimum GPA
- Ensure `IsActive = TRUE` and `IsTopLevelProgram = TRUE` on parent programs

2. **Load Scholarships**
- Create `Scholarship__c` records with either Award Amount (flat) or Award Percentage (of tuition)
- Link each scholarship to a parent Learning Program
3. **Verify Person Account setup**
- Confirm Person Account is enabled
- Note the active Person Account record type (needed for account creation flow)
- 
### Step 2: Set Up Knowledge & Data Library
1. **Create a Data Category Group**
- Setup → Data Categories → New Category Group: `Riverstone Knowledge`
- Add child category: `Riverstone University Specific`
2. **Assign the Category Group to Knowledge**
- Setup → Data Category Assignments → assign `Riverstone Knowledge` to Knowledge object
3. **Create the 5 Riverstone Knowledge Articles**
- Admission Policies — Masters in Computer Science
- Application Requirements — Masters in Computer Science
- Admission Policies — Masters in Business Administration
- Application Requirements — Masters in Business Administration
- Enrollment Process at Riverstone University
For each article: tag with `Riverstone University Specific` category and set PublishStatus to `Online`.
4. **Create the Data Library**
- Setup → Agentforce Data Library → New Data Library
- Name: `Riverstone Library`
- Data Type: Knowledge
- Identifying Fields: Title, Details
- Knowledge Settings → Filter by Data Categories → check `Riverstone University Specific`
- Wait for indexing to complete (status should show `Ready`)
### Step 3: Deploy the Flows
Deploy the following autolaunched Flows from this repository (see `/flows` directory):
- `Get_Active_Learning_Programs` — Retrieves active programs for dynamic greeting
- `Search_Person_Account_By_Email` — Looks up existing student by email
- `ATF_Create_Person_Account` — Creates new Person Account with correct record type
- `Get_All_Scholarship_And_Course` — Retrieves programs and scholarships based on user choice
- `ATF_Detail_Of_Program_Plan` — Retrieves specialization tracks for a selected program
- `ATF_Get_Scholarship_And_Course_Details` — Retrieves internal course details
- `ATF_Scholarship_Match_Creation` — Creates Scholarship Match records for eligibility
- `Getting_Scholarship_Detail` — Retrieves details of a specific scholarship
- `ATF_Creation_Of_Student_Application` — Creates Individual Application records
- `ATF_Check_Student_Application` — Checks status of existing applications
- `ATF_Creation_Of_Case` — Creates Case records for escalation
Activate each flow after deployment.
### Step 4: Configure Permissions
Assign the following to the Agent User (typically `EinsteinServiceAgent User`):

**Permission Set License Assignments:**
- Education Cloud AI Agent User
**Permission Set Assignments:**
- Agentforce Service Agent User
- Agentforce Service Agent Object Access
- Agentforce Service Agent Secure Base
- Education Cloud AI Agent Access
- EDU Agent Access
- A custom permission set granting Read on `LearningProgram`, `LearningProgramPlan`, `Scholarship__c`, and CRUD on `Account`, `Individual Application`, `Case`
  
### Step 5: Deploy the Agent
1. **Create the Agent**
- Setup → Agentforce Studio → New Agent
- Name: `River`
- Assign to the Agent User with Education Cloud AI Agent User license
2. **Configure the Agent Definition**
- See `/agent-config/agent-description.md` for the agent-level description
3. **Create Subagents**
- Deploy the 4 custom subagents with their descriptions and instructions from
`/agent-config/subagents/`
- Delete the auto-provisioned subagents that aren't needed (Account Management, General
FAQ, Case Management)
4. **Wire the Actions**
- For each subagent, add the corresponding flow-based actions
- Configure action-level Description, Input Descriptions, and Output Descriptions.
5. **Connect the Data Library**
- Agent Builder → River → Data → Add Data Library → select `Riverstone Library`
- Verify Feature Assignments count updates from 0 to 1
6. **Wire the Standard Knowledge Action**
- Add `Answer Questions with Knowledge`to the Admission Policies and Enrollment Process subagent
  
### Step 6: Configure the Channel
1. **Create Messaging Channel**
- Setup → Messaging Settings → New Channel
- Type: Messaging for In-App and Web
- Name: `Riverstone Web Chat`
- Route to the River agent
2. **Create Embedded Service Deployment**
- Setup → Embedded Service Deployments → New
- Name: `Riverstone Site Chat`
- Attach the Riverstone Web Chat messaging channel
3. **Embed on Experience Cloud Site**
- Add the Embedded Messaging component to your Experience Cloud site pages
- Publish the site
- 
### Step 7: Activate and Test
1. **Activate River** (top right of Agent Builder)
2. **Open the Experience Cloud site in an incognito window**
3. **Test each phase** using the storyline scenarios shown in the PPT.

Usage Instructions
### For Prospective Students
1. Visit the Riverstone University Experience Cloud site
2. Click the chat widget to start a conversation with River
3. Provide first name, last name, and email when prompted
4. Ask about programs, tracks, fees, scholarships, admission policies, or the enrollment process
5. When ready, ask River to start the application process
6. Receive the Application ID and portal link for document upload
### For Administrators
**Adding new programs:**
- Create the `LearningProgram` and `LearningProgramPlan` records in Salesforce with
populated descriptions
- River will automatically surface them in the next conversation — no agent changes needed
**Adding new Knowledge articles:**
- Create the article, tag with `Riverstone University Specific` data category, and Publish
- Wait for the Data Library to re-index (usually 1-5 minutes)
- River will surface the article content when relevant questions arise
**Adjusting agent behavior:**

- Update subagent Descriptions to change routing behavior
- Update subagent Reasoning Instructions to change conversational behavior
- Commit a new agent version and reactivate

Prerequisites & Important Notes
### Salesforce Prerequisites
- Salesforce org with Education Cloud, Person Account, Lightning Knowledge, Experience Cloud, and Messaging for In-App and Web all enabled
- Agentforce license and Agent User with appropriate permission set licenses
- At minimum 2 active Learning Programs, multiple Learning Program Plans, and 3+ Scholarships with realistic data
- 5 Knowledge articles authored and tagged with the correct Data Category

### Known Limitations
- **Guest users cannot upload documents** — Salesforce restricts document upload for unauthenticated site visitors. River creates the application record and directs students to the authenticated portal for document upload.
- **Einstein Agent User license restrictions** — The default Agent User license doesn't support
the Education Cloud - Limited Access PSL. Use `Education Cloud AI Agent User` PSL instead.
# Salesforce DX Project

Salesforce DX is a development approach that brings source-driven development, team collaboration, and continuous integration to the Salesforce Platform. Instead of working directly in an org through a web browser, you work with metadata as source files in a local DX project, track changes in version control, and deploy through automated processes.

This project template gets you started with the tools and structure you need to build Salesforce applications using source control, scratch orgs, and the Salesforce CLI.


## Project Structure

Your DX project follows this structure:

- **`force-app/main/default/`** - Your metadata source files live in this default package directory. You can configure additional package directories in the `sfdx-project.json` file.
- **`config/`** - Scratch org definitions and project settings
- **`scripts/`** - Automation scripts for common tasks
- **`sfdx-project.json`** - Project manifest that defines package directories, namespace, API version, and other project-level settings

## Get Started

Ready to start developing? The [Get Started with Salesforce DX](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_get_started_dx.htm) guide walks you through your first project, from creating a scratch org to creating a simple Apex class or LWC to deploying your code to a sandbox.

## Common Salesforce CLI Commands

Here are common CLI commands that you'll use the most:

- `sf org login web`: Authorize an org
- `sf org open`: Open your org in a browser
- `sf org create scratch`: Create a scratch org
- `sf project deploy start`: Deploy metadata to your org
- `sf project retrieve start`: Retrieve metadata from your org
- `sf template generate <artifact>`: Scaffold new components, such as Apex classes and triggers, LWC components, Lightning apps, and more
- `sf apex <command>`: Run Apex tests, run anonymous Apex blocks, and view logs
- `sf data <command>`: Work with test data
- `sf alias <command>`: Manage org aliases
- `sf config <command>`: Configure CLI settings

## Use Agentforce Vibes to Build Lightning Apps

Transform your ideas into custom Lightning apps that extend CRM workflows directly in Lightning Experience. Through natural conversations with Agentforce Vibes, implement custom objects and fields, complex business logic, and dynamic UI components. See [Build a Lightning App Using Agentforce Vibes](https://developer.salesforce.com/docs/platform/einstein-for-devs/guide/lexapp-overview.html).

## Additional Resources

- [Agentforce Vibes Developer Guide](https://developer.salesforce.com/docs/platform/einstein-for-devs/guide/einstein-overview.html)
- [Salesforce CLI Installation Guide](https://developer.salesforce.com/docs/atlas.en-us.sfdx_setup.meta/sfdx_setup/sfdx_setup_intro.htm)
- [Salesforce DX Developer Guide](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/)
- [Salesforce CLI Command Reference](https://developer.salesforce.com/docs/atlas.en-us.sfdx_cli_reference.meta/sfdx_cli_reference/)
- [Salesforce CLI Plugin Development Guide](https://developer.salesforce.com/docs/platform/salesforce-cli-plugin/guide/conceptual-overview.html)
- [Salesforce VS Code Extensions Documentation](https://developer.salesforce.com/tools/vscode/)

=======
# Blue-Agent-Team
>>>>>>> dcba68e2635f2b80aa4c1c85bf6314f7da8bb105
