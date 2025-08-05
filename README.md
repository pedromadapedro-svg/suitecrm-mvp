# SuiteCRM MVP for Aesthetic Clinic CRM

This repository provides a minimal viable product (MVP) CRM using SuiteCRM and MariaDB, configured using Docker Compose and deployed to [Render](https://render.com) via a one-click `render.yaml` blueprint. It is designed for an aesthetic clinic and includes custom modules, fields, workflows, and dashboards using SuiteCRM Studio.

## Architecture

- **Docker Compose**: Defines two services:
  - `mariadb` uses the Bitnami MariaDB image and persists data in a local volume.
  - `suitecrm` uses the Bitnami SuiteCRM image, connects to the database, and exposes the web UI on port `8080`.
- **Render**: The `render.yaml` blueprint describes:
  - A private MariaDB database service with an attached 10 GB disk and generated credentials.
  - A SuiteCRM web service that automatically pulls the SuiteCRM image from Docker Hub, sets up environment variables from the database service, and exposes port `8080` publicly via HTTPS. Administrator credentials are generated at deploy time.
  - Persistent disks for both services to retain database and uploaded files.

## Running Locally

Prerequisites:
- Docker and Docker Compose installed (e.g., Docker Desktop on macOS).

To run the stack locally:

```bash
git clone https://github.com/pedromadapedro-svg/suitecrm-mvp.git
cd suitecrm-mvp
cp .env.example .env   # Optionally override environment variables in `.env`
docker-compose up -d
```

SuiteCRM will be available at `http://localhost:8080`. The default credentials are set via environment variables in `docker-compose.yml` (admin / admin123). For production, override the values in a `.env` file.

## Deploying on Render

1. Create an account at [Render.com](https://render.com) if you don't have one.
2. Click **New Web Service**, then **Import from GitHub** and select this repository.
3. Render will detect the `render.yaml` blueprint and prompt you to deploy the services:
   - It will create a private MariaDB service (Starter plan) with generated credentials.
   - It will create a web service running SuiteCRM. The environment variables will be populated automatically from the database service.
4. Once deployed, Render will provide a public HTTPS URL for SuiteCRM. Use the generated admin username and password shown in the Render dashboard to log in for the first time.

## Configuring SuiteCRM Studio

After logging into SuiteCRM as an administrator, perform the following steps to customise the CRM for the clinic:

### Contacts Module (Pacientes)

1. Go to **Admin → Studio → Contacts → Fields** and click **Add Field**. Create the following custom fields:
   - `telefone_c` (Varchar, label: Telefone)
   - `email_c` (Varchar, label: Email)
   - `tratamentos_realizados_c` (TextArea, label: Tratamentos Realizados)
   - `tratamentos_propostos_c` (TextArea, label: Tratamentos Propostos)
2. Click **Relationships → Add Relationship**. Add a one-to-many relationship to a new module `Procedimentos`.
3. Click **Layouts → List View** and **Layouts → Record View**. Drag the custom fields into appropriate panels. Add a subpanel showing the related `Procedimentos`. Add a calculated field or relationship widget labelled **Último Procedimento** that displays the latest related procedure.

### Procedimentos Module

1. In **Studio** click **Create Module**, choose **Module Builder**, and create a new module called **Procedimentos** with a one-to-many relationship to Contacts.
2. Add the following fields:
   - `tipo_procedimento_c` (Dropdown, label: Tipo de Procedimento) with options: Toxina, Laser, Ultrassom Microfocado.
   - `data_procedimento_c` (DateTime, label: Data do Procedimento).

### Leads Module

1. Navigate to **Studio → Leads → Fields** and add:
   - `origem_lead_c` (Dropdown, label: Origem do Lead) with options: Site, WhatsApp, Facebook Ads, Indicação.
   - `tratamentos_propostos_c` (TextArea, label: Tratamentos Propostos).
   - `observacoes_c` (TextArea, label: Observações).
   - `assunto_interesse_c` (Varchar, label: Assunto de Interesse).
   - `status_conversao_c` (Dropdown, label: Status de Conversão) with options: Novo, Em Qualificação, Agendado.
2. Create a Workflow in **Admin → Workflow**:
   - Trigger: When **Status de Conversão** changes to **Agendado**.
   - Action: Convert the Lead to a Contact, copying custom fields to the new contact record.

### Opportunities Pipeline

1. Navigate to **Admin → Dropdown Editor** and modify the `sales_stage_dom` list to include the stages:
   - Novo, Qualificado, Agendado, Em Tratamento, Concluído.
2. Save the changes. These stages will now appear in the Opportunities module pipeline.

### Automated Workflows and Tasks

Use the **Workflow** module to schedule the following tasks:

- **Follow‑up on Proposed Treatments**: Weekly create a task for the assigned user when a contact has *Tratamentos Propostos* filled but no matching *Tratamentos Realizados*.
- **Recall Toxina**: Monthly task for patients whose last **Procedimento** is `Toxina` and occurred 5 months ago.
- **Recall Laser**: Monthly task for patients with `Laser` procedures older than 6 months.
- **Recall Ultrassom Microfocado**: Monthly task for patients with `Ultrassom Microfocado` procedures older than 6 months.
- **Reactivation of Inactive Patients**: Monthly task for contacts without any **Procedimentos** in the last 12 months.

### Reports and Dashboard

Create the following reports in **Reports** module and add them to a new dashboard:

- **Pending Proposed Treatments**: List contacts with non-empty `Tratamentos Propostos` and no matching `Tratamentos Realizados`.
- **Recall Filters**: Create separate reports for each recall type above to identify patients due for Toxina, Laser or Ultrassom recall.
- **Lead Conversion Rate**: Report showing number of leads converted to contacts over time.

After creating the reports, go to **Home → Dashboard → Add Dashlet** and add chart or list dashlets for these reports to monitor them at a glance.

## Access Credentials and Usage

- **Local deployment**: The `docker-compose.yml` file sets default admin credentials as:
  - User: `admin`
  - Password: `admin123`
- **Render deployment**: Render will generate strong passwords for both the database and SuiteCRM admin user. Find these credentials in the Render dashboard under the Environment section. Use them to log in at the deployed URL.

## Notes

- For production use, set strong passwords for the database and admin user. Never enable `ALLOW_EMPTY_PASSWORD` in production.
- Customize dropdown lists and field labels as needed to match your clinic's terminology.
