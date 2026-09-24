# NFL AI Matchup Viewer & Predictor 🏈🤖

An automated Python pipeline and interactive web dashboard that aggregates real-time NFL data, generates tactical game scripts using a locally hosted Large Language Model (LLM), and publishes a predictive matchup hub. 

This project operates as a headless data pipeline on a Proxmox server, securely querying a local Mac Mini running Ollama, before automatically deploying a static HTML frontend to GitHub Pages.

## 🚀 Key Features

* **AI-Driven Game Scripts:** Feeds depth charts, injuries, team stats, and weather data into a local `llama3` model via structured JSON prompting to generate score predictions, confidence ratings, and tactical breakdowns.
* **Live NFL Data Ingestion:** Hooks into the ESPN API to pull live schedules, betting markets (Spread, Moneyline, Over/Under), ESPN FPI, full depth charts, and injury reports.
* **Hyper-Local Weather Engine:** Bypasses unreliable standard API weather feeds by mapping stadium GPS coordinates to the Open-Meteo API, delivering exact hourly forecasts matching the UTC kickoff time. 
* **Dynamic UI & Color Correction:** Calculates the Euclidean distance between team hex codes to resolve color clashes (e.g., Red vs. Red). It automatically swaps to alternate colors and uses a luminance algorithm to prevent white text on light backgrounds.
* **Interactive Frontend:** A responsive, zero-dependency HTML/JS dashboard featuring:
  * **All Games View:** Stacked matchup cards with visual slider bars for Offense, Defense, Special Teams, and Team Health.
  * **Quick Picks Cheat Sheet:** A compact, sortable table showing AI projected winners, implied scores, market lines, and upset alerts.
  * **Deep Dive View:** Granular breakdown of offensive and defensive depth charts categorized by phase and tactical groupings (Trenches, Aerial Weapons, etc.).

## 🏗️ Architecture & Tech Stack

1. **Backend / Data Pipeline:** Python 3 (Requests, Subprocess, JSON, Shutil)
2. **AI Inference:** Ollama (Llama 3) hosted locally, heavily constrained with system prompts.
3. **External APIs:**
   * ESPN v2 Sports API (Scoreboard, Matchups, Rosters)
   * Open-Meteo API (Hourly high-precision weather)
4. **Deployment:** GitHub Pages (Static hosting) with automated Git commits driven directly by the Python script.
5. **Hosting Infrastructure:** Proxmox LXC Container -> Local Mac Mini (AI) -> GitHub (Public Frontend).

## ⚙️ How the Pipeline Works

Whenever `generate_ai_matchups.py` is triggered (either manually, via cron, or through the web dashboard's `/api/refresh` Flask endpoint), the following sequence occurs:

1. **HTML Backup:** The previous week's `index.html` is stamped and archived into a local `/backups/` directory.
2. **Data Aggregation:** The script loops through the active NFL week, combining live betting odds with roster mapping and fetching the weather forecast based on stadium roof-type and coordinates.
3. **LLM Evaluation:** The aggregated data is injected into `prompt_template.txt`. Ollama analyzes trench leverage, coverage matchups, and injury impact scores, returning a strictly formatted JSON response.
4. **Probability Math:** Implied points are mathematically derived from Vegas Spreads + O/U, and NFELO win probabilities are calculated alongside ESPN's FPI.
5. **Static Generation:** A single, monolithic `index.html` file is written, baking all JSON data directly into the JavaScript `GAMES_DATA` constant for instant, client-side rendering.
6. **Automated Git Push:** The script executes a silent `git commit` and `git push`, instantly updating the public-facing GitHub Pages dashboard.

## 🛡️ Security & Privacy

This repository serves strictly as the deployment target for the frontend dashboard. 
* No internal IP addresses, local server names, or LLM network paths are exposed in the source code.
* The frontend relies purely on statically injected JSON; it makes zero external requests back to the host network.
* The "Refresh" functionality is isolated to the local network environment via a protected Flask endpoint.
