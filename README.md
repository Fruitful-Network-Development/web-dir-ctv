# web-dir-ctv
Website for cuyahoga Tera Vita

## 1. aws-etc repository (system configs)

### Current README context: The README already lays out the multi‑tenant architecture (Nginx reverse proxy, manifest-driven Flask backend, separate client directories) ￼.

### Recommended additions (new “Agent Guidance” section):
	•	Purpose of this repo: This repository contains only system‑level configuration (Nginx virtual hosts, systemd units, deployment scripts and audit logs). It is a sandbox for the server administrator; application code and client websites are stored elsewhere ￼.
	•	No app or frontend code: Agents must not add or edit Flask modules or client website files in this repo. All application logic belongs in flask-app, and all client web assets belong in web-dir-fnd or web-dir-ctv.
	•	How to change configs: Use the provided scripts under scripts/ to stage and deploy changes. Do not modify /etc directly. For example, updating Nginx config should follow the sync‑to‑aws and deploy scripts described in docs ￼.
	•	Audit logs and prompt context: All audit scripts write into docs/ with timestamped files so agents can review outputs without touching /etc ￼.
	•	Working with the manifest & multi‑tenant design:  Agents should never attempt to register blueprints or modules here.  The multi‑tenant server uses host‑based routing and a per‑client msn_<userId>.json manifest to control file access ￼.

---

## 2. flask-app repository (shared backend)

### Current README context: Explains that the Flask app serves multiple client frontends based on each client’s msn_<user>.json manifest ￼.

### Recommended additions (new “Agent Guidance” section):
	•	Single Flask application: This repo contains the only Flask application for all clients. Agents should never create a new app.py or a separate Flask app.  The existing app.py discovers client manifests and routes requests accordingly ￼.
	•	Modules and blueprints: Reusable APIs (e.g. weather, donation box) live under platform/modules/ and are registered via blueprints ￼.  To add or modify a module, implement it under modules/ and import its blueprint in app.py, then register it with app.register_blueprint.  Do not copy modules into the client repositories or system repository.
	•	CORS configuration: If a frontend is served from a different origin (for example when developing locally or hosting the frontend on a separate domain), set FLASK_ENABLE_CORS=1 in the environment.  This flag turns on CORS for the /api/* routes so that browsers can call the endpoints without errors.  Agents should not implement custom CORS logic or duplicate it in each module.
	•	Manifest‑driven security: The data_access.py helper reads the client’s manifest to determine which data files can be exposed ￼.  Agents must update the manifest (msn_<userId>.json) rather than bypass this logic when adding new data files.
	•	Adding new API endpoints: Follow the pattern used in existing modules (e.g. weather.py).  Each endpoint should:
	1.	read necessary parameters from the request,
	2.	validate the client ID or user ID,
	3.	operate within the allowed data whitelist, and
	4.	return JSON.  Register the endpoint via blueprint.
	•	Don’t handle frontend here: This repo serves HTML only via app.py using the manifest’s default_entry.  Any HTML/CSS/JS belongs in the client repos.

---

## 3. web-dir-fnd and web-dir-ctv (client websites)

### Current READMEs: The FND README explains the Mycite profile concept ￼; CTV’s README has only one line.

### Recommended additions for both repos (new “Agent Guidance” section):
	•	Purpose of this repo: Each of these repositories hosts the static frontend for a single client domain (e.g. fruitfulnetworkdevelopment.com or cuyahogaterravita.com).  The contents are served by Nginx from /srv/webapps/clients/<domain>/frontend.
	•	Manifest file: A file named msn_<userId>.json (e.g. msn_32357767435.json) must exist alongside index.html.  This manifest defines site configuration—including the default_entry HTML file and a backend_data whitelist—and is consumed by the Flask backend.  Agents should update the manifest rather than editing backend code when altering these settings.
	•	Universal index.html: The entry file should include two <meta> tags:

```html
<meta name="msn-id" content="<userId>">
<meta name="msn-config" content="msn_<userId>.json">
```
These tags let the Flask backend map the host to the correct manifest

	•	No backend logic here: Do not add Python code or attempt to call the file system; this repository is static.  All API calls should be made to /api/... endpoints exposed by the shared Flask app.
	•	CORS & API access: If your frontend is served from a different origin (e.g. local development or a separate static site host), ask the server admin to set FLASK_ENABLE_CORS=1 on the Flask backend.  Do not implement your own CORS workaround in JavaScript; the backend will handle it.
	•	Adding data files: When adding new JSON/CSV files under data/, update the backend_data list in the manifest so the backend will allow access.  Without whitelisting, requests to /api/data/<file> will be denied.

### Additional notes for FND (web-dir-fnd):
	•	Maintain the Mycite profile schema (Compendium, Oeuvre, Anthology) described in the existing README ￼.  Agents should expand the schema via the manifest rather than altering backend structures.

---

### Additional notes for CTV (web-dir-ctv):
	•	Provide similar guidance as above.  Consider adding a description of the site’s content (e.g. “CSA site for Cuyahoga Terra Vita”) and mention that its msn_<userId>.json manifest defines available crops, seasons, and program details.

---

## Summary
where to run the pull scripts
	•	The pull_fnd.sh and pull_ctv.sh scripts live in GH-etc/scripts/, but they operate on /srv/webapps/clients/<domain> and can be run from any directory.  Running them from ~/GH-etc keeps your workflow consistent; they will navigate to the correct repo based on REPO_PATH and perform a fast-forward pull.






