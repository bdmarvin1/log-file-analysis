# Chargebee Log Analysis Engine

A specialized tool for analyzing Googlebot crawl behavior, identifying spider traps, and diagnosing "Crawled - Currently Not Indexed" issues using server logs.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/bdmarvin1/log-file-analysis/blob/main/Chargebee_Log_Analysis.ipynb)

## Prerequisites
1. **Log Files:** Your server logs must be in **CSV** format.
2. **Google Drive:** Create a folder in your Google Drive and upload your CSV log files there.

## Setup Instructions
1. Click the **Open in Colab** button above.
2. Locate the **Constants** section in the first code cell (Phase 1).
3. Update these two variables:
   - `LOGS_PATH`: The path to your Google Drive folder (e.g., `'/content/drive/MyDrive/My_Logs/'`).
   - `OUTPUT_SHEET_NAME`: The name you want for the final Google Sheet report.

## How to Run
1. In the top menu, go to **Runtime** > **Run all**.
2. **Permissions:** You will see pop-up windows asking for permission to access your Google Drive and Google Sheets. Click **Allow** or **Connect** for each one.
3. **Wait for completion:** The notebook will process the logs and create a new Google Sheet in your account with the final analysis.
