Conejos County Commissioner's Command Center
Overview
The Conejos County Commissioner's Command Center is a data-driven, interactive web-based report designed to support the Conejos County Board of Commissioners and Antonito Town Council in addressing critical governance priorities. Built as a standalone HTML page, it consolidates key issues, actionable steps, and deadlines for the August 7, 2025, public meeting, ensuring compliance with Colorado Sunshine Laws for transparency. The report serves the 7,461 residents of Conejos County, including Antonito’s 647 residents, by providing clear, accessible insights into:

Water Crisis: Rio Grande flows at 78%, Conejos River at 80%, with a 5% well curtailment threatening the $340M agricultural economy.
Cannabis Revenue: Antonito’s $295,000 annual cannabis tax revenue to fund infrastructure and small businesses.
Constituent Requests: High volume of concerns (e.g., potholes, housing), exacerbated by the July 2025 courthouse fire.
Regional Coordination: Collaboration with SLVCCA, CCI, and federal agencies on water, broadband, and drought relief.

Key Features

Interactive Mind Map: A pseudo-3D, force-directed visualization using D3.js, allowing users to explore priorities (e.g., “Water Crisis,” “Cannabis Revenue Strategies”) with click/tap-to-expand nodes, tooltips, and a slide-in info panel. Users can rotate the map via drag/touch for a dynamic experience.
Mobile-Friendly Design: Responsive layout with media queries (768px/480px breakpoints), touch controls, and a centered mind map for accessibility on phones, tablets, and desktops in rural Conejos County.
Data Visualizations: Chart.js bar chart comparing Antonito’s economic impact (cannabis, tourism) to county agriculture, with data sourced from SLVRDG and county records.
Feedback Form: Collects input from commissioners and council members, with plans for backend integration via n8n.
Geospatial Placeholder: Future integration of a Leaflet.js map for water monitoring stations and infrastructure needs.
Compliance: Adheres to Colorado Sunshine Laws, with verified data from Colorado Division of Water Resources (DWR), SLVRDG, and county records.

Purpose
This system empowers commissioners like Delfino to make informed decisions by presenting consolidated, actionable data in an intuitive interface. It supports public engagement, aligns with grant deadlines (e.g., USDA $500M drought relief by September 15, 2025), and facilitates regional coordination (e.g., SLVCCA meeting on August 20, 2025). The agentic workflow via n8n automates data updates and feedback processing, ensuring the report remains current and responsive to community needs.
Agentic System Workflow with n8n
The report is designed to integrate with a locally hosted, secure n8n instance to create an agentic system that automates data retrieval, processing, and feedback handling. n8n, an open-source workflow automation tool, enables dynamic updates to the report without manual intervention, enhancing efficiency for county staff.
n8n Workflow Overview
The n8n workflow runs on a local server (e.g., Raspberry Pi, local PC, or dedicated server within Conejos County’s network) to ensure data security and compliance with local governance policies. The workflow includes:

Data Retrieval:

Sources: Pulls real-time data from APIs like DWR (https://dwr.state.co.us/Rest/GET/api/v2/surfacewater/surfacewaterstations/), SLVRDG, and Conejos County records (conejoscounty.colorado.gov).
Tasks: Fetches Rio Grande/Conejos River flow levels, constituent request volumes, and economic metrics (e.g., $295,000 cannabis revenue).
Schedule: Daily/weekly cron jobs to update the report’s JSON data (e.g., mind map nodes, Chart.js datasets).


Data Processing:

Validates and formats API responses (e.g., convert DWR flow data to percentages).
Updates the HTML report’s JSON structure, ensuring nodes like “Rio Grande at 78%” reflect the latest data.
Generates alerts for urgent deadlines (e.g., DOLA broadband grants due August 31, 2025).


Feedback Processing:

Captures submissions from the report’s feedback form (requires a backend endpoint, e.g., /submit-feedback).
Categorizes feedback (e.g., “Water Crisis,” “Cannabis Revenue”) and routes to relevant stakeholders (e.g., commissioners, SLVCCA).
Stores feedback securely in a local database (e.g., SQLite) for analysis and follow-up.


Output:

Updates the HTML report file (Conejos County Commissioner's Command Center.html) with new data.
Sends email notifications to commissioners (e.g., via SMTP) for critical updates or feedback summaries.
Logs actions for transparency, per Colorado Sunshine Laws.



Security Measures

Local Hosting: n8n runs on a secure, air-gapped server within Conejos County’s network to protect sensitive data (e.g., constituent requests, financial metrics).
Authentication: Configures n8n with user authentication (username/password) and HTTPS (via self-signed certificates or Let’s Encrypt).
Data Encryption: Uses TLS for API calls and encrypts stored feedback data.
Access Control: Restricts n8n access to authorized county staff via IP whitelisting or VPN.
Audit Logging: Tracks all workflow executions and data updates for compliance.

n8n Setup Instructions
To set up the n8n workflow locally:

Install n8n:

On a local server (Ubuntu, Raspberry Pi, or Windows/Mac):npm install -g n8n


Start n8n:n8n start --tunnel


Access the n8n interface at http://localhost:5678 (configure HTTPS for production).


Create Workflow:

In the n8n UI, create a new workflow with the following nodes:
Cron Node: Triggers daily/weekly updates (e.g., 0 0 * * * for midnight updates).
HTTP Request Node: Fetches data from DWR, SLVRDG, and county APIs.
Function Node: Processes and formats data (e.g., update JSON for mind map nodes).
Write File Node: Updates the HTML report’s JSON data section.
Webhook Node: Handles feedback form submissions (set endpoint to /submit-feedback).
SQLite Node: Stores feedback in a local database.
Email Node: Sends notifications to commissioners (configure SMTP server).


Example workflow JSON (import into n8n):{
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [{"cronExpression": "0 0 * * *"}]
        }
      },
      "name": "Daily Update",
      "type": "n8n-nodes-base.cron",
      "typeVersion": 1,
      "position": [240, 300]
    },
    {
      "parameters": {
        "url": "https://dwr.state.co.us/Rest/GET/api/v2/surfacewater/surfacewaterstations/",
        "options": {}
      },
      "name": "Fetch DWR Data",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 1,
      "position": [460, 300]
    },
    {
      "parameters": {
        "jsCode": "return { data: $node['Fetch DWR Data'].json.map(item => ({ name: item.stationName, flow: item.flowPercent })) }"
      },
      "name": "Process Data",
      "type": "n8n-nodes-base.function",
      "typeVersion": 1,
      "position": [680, 300]
    },
    {
      "parameters": {
        "filePath": "./Conejos County Commissioner's Command Center.html",
        "dataPropertyName": "data"
      },
      "name": "Update HTML",
      "type": "n8n-nodes-base.writeBinaryFile",
      "typeVersion": 1,
      "position": [900, 300]
    }
  ],
  "connections": {
    "Daily Update": {
      "main": [[{ "node": "Fetch DWR Data", "type": "main", "index": 0 }]]
    },
    "Fetch DWR Data": {
      "main": [[{ "node": "Process Data", "type": "main", "index": 0 }]]
    },
    "Process Data": {
      "main": [[{ "node": "Update HTML", "type": "main", "index": 0 }]]
    }
  }
}




Secure the Instance:

Set up authentication:export N8N_BASIC_AUTH_ACTIVE=true
export N8N_BASIC_AUTH_USER=admin
export N8N_BASIC_AUTH_PASSWORD=securepassword


Enable HTTPS (e.g., via Let’s Encrypt or self-signed certificate).
Configure firewall rules to restrict access (e.g., ufw allow from 192.168.1.0/24 to any port 5678).


Test the Workflow:

Run the workflow manually in n8n to verify data retrieval and HTML updates.
Test the feedback form webhook by submitting a test form (requires a local server to host the HTML).


Deploy:

Host the HTML report on a local server (e.g., Apache/Nginx on http://localhost:8080).
Schedule the n8n workflow to run daily/weekly.
Monitor logs for errors (~/.n8n/logs).



Prerequisites

Node.js: v14 or higher for n8n.
Local Server: Ubuntu, Raspberry Pi, or similar with network access.
APIs: Access to DWR, SLVRDG, and county APIs (request credentials from respective agencies).
Browser: Chrome, Firefox, Safari, or Edge for viewing the HTML report.
Storage: SQLite or similar for feedback storage.

Installation and Usage

Clone the Repository:
git clone <repository-url>
cd conejos-county-command-center


Open the Report:

Open Conejos County Commissioner's Command Center.html in a browser (e.g., file://path/to/file.html) for local testing.
For production, host on a local web server:sudo apt install nginx
sudo cp "Conejos County Commissioner's Command Center.html" /var/www/html/index.html
sudo systemctl start nginx

Access at http://localhost.


Interact with the Mind Map:

Click/tap nodes (e.g., “Water Crisis”) to view details in the info panel.
Drag or pinch to rotate/zoom the pseudo-3D mind map.
On mobile, tap the info panel to close it.


Submit Feedback:

Use the feedback form to suggest improvements (requires n8n webhook setup for processing).



Development

Mind Map: Built with D3.js v7, using a force-directed layout with pseudo-3D scaling (scale(1 - d.z / 200)) and CSS transforms (perspective: 1000px).
Charts: Chart.js for economic impact visualizations.
Styling: Responsive CSS with Segoe UI font, gradients, and shadows, using colors #4a90e2, #5a7fc7, #2c5aa0, #1a365d.
Future Enhancements:
Integrate live DWR API data for real-time flow updates.
Implement Leaflet.js for geospatial visualization of water stations and infrastructure.
Add rotation control buttons for the mind map.
Transition to a true 3D mind map with Three.js (note performance trade-offs for mobile).



Contributing
Contributions are welcome to enhance the report’s functionality or n8n workflow. Please:

Fork the repository.
Create a feature branch (git checkout -b feature/new-feature).
Commit changes (git commit -m "Add new feature").
Push to the branch (git push origin feature/new-feature).
Open a pull request, ensuring compliance with Colorado Sunshine Laws.

Contact
Developer:  James Harwood, Sentinel AI Systems jameskharwood2@gmail.com,  www.jameskeithharwood.com
Conejos County Commissioners: slvcca@conejoscounty.org, (719) 274-5100
SLVDRG: info@slvdrg.org, (719) 589-6099
DWR: craig.cotten@state.co.us, (719) 589-6683
USDA Colorado: co.usda@usda.gov, (720) 544-2876

License
GNU GENERAL PUBLIC LICENSE Version 3, 29 June 2007 See the LICENSE file for details.

Acknowledgments

Data sourced from Colorado Division of Water Resources, SLVRDG, and Conejos County records.
Beta Demo Built for the August 7, 2025, Conejos County public meeting to show capability to empower data-driven governance.
