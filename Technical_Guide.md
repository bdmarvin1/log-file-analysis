# Technical Guide: Chargebee Log Analysis Engine

This guide provides a deep dive into the technical architecture and execution logic of the `Chargebee_Log_Analysis.ipynb` notebook. It is designed to help you understand not just *what* the code does, but *how* it does it at every step.

---

## 1. High-Level Execution Flow

The notebook follows a linear pipeline, where the output of one phase becomes the input for the next.

1.  **Environment Setup**: Connects to Google services (Drive and Sheets).
2.  **Ingestion (The "Filter & Chunk" Strategy)**: Reads massive CSV files without crashing the browser or the server.
3.  **Preprocessing**: Cleans the raw data and performs security/validity checks (IP Verification).
4.  **Analysis**: Segments data into specific SEO-focused modules.
5.  **Visualization**: Converts tabular data into insights.
6.  **Reporting**: Exports findings to an external stakeholder-friendly format (Google Sheets).

---

## 2. Deep Dive: Phase-by-Phase Logic

### Phase 2: Data Ingestion
**Goal:** Load one year of data while staying within Google Colab's RAM limits (usually 12GB).

*   **`chunk_size=50000`**: Instead of loading 1,000,000 rows at once, we load 50,000 at a time. This is like reading a book page-by-page rather than trying to memorize the whole book in one second.
*   **`usecols`**: By specifying only the 8 columns we need (out of 100+), we reduce memory usage by over 90%.
*   **`gc.collect()`**: This manually triggers "Garbage Collection." It tells Python: "I'm done with the temporary chunk of data; please delete it from RAM immediately."

**Key Variable:**
*   `googlebot_data`: A list that holds only the rows where the `useragent` contains "Googlebot".

---

### Phase 3: Data Cleaning & IP Verification
**Goal:** Ensure the data is accurate and the "Googlebot" is actually Google.

#### IP Verification (Double-Reverse DNS)
Spammers often fake their User Agent to look like Googlebot. We verify them using this logic:
1.  **Reverse DNS**: Take the IP (e.g., `66.249.66.1`) and ask the internet for its name. If it's real, it should end in `.googlebot.com`.
2.  **Forward DNS**: Take that name and ask for its IP. If the IP matches the original one, the bot is verified.

**Key Functions:**
*   `socket.gethostbyaddr(ip)`: Finds the hostname.
*   `socket.gethostbyname(host)`: Finds the IP of a hostname.

#### URL Normalization
*   `full_url = uri_path + uri_query`: We combine the path (the page) and the query (parameters like `?id=123`) to see exactly what Google saw.

---

### Phase 4: Core Analysis Modules

#### Module B: Status Code Health
We look for **304 Not Modified** responses.
*   **Why?** A 304 means Googlebot asked "Has this file changed?" and your server said "No." This saves "Crawl Budget" because Google doesn't have to download the file again.
*   **Logic:** If JS/CSS files are mostly `200 OK` instead of `304`, you are wasting bandwidth and Google's time.

#### Module C: Spider Trap Detection
**Definition**: A "Spider Trap" is a part of your site that generates infinite unique URLs (usually via filters or search parameters).
*   **Variable:** `unique_variations`.
*   **Logic**: We group by `uri_path` (the base page) and count how many unique `full_url` values exist for it. If `/products` has 1,000 variations, Googlebot might get stuck there forever.

#### Module D: Rendering Budget
*   **`bytes_sent`**: We sum this up per file type.
*   **Insight**: If "Javascript" uses 80% of your bandwidth and "HTML" only 20%, your site is "Heavy." This makes it harder for Google to render your pages, leading to the "Crawled - Currently Not Indexed" status.

---

### Phase 6: Google Sheets Export
**Goal**: Persistence. Colab files are temporary; Google Sheets are permanent.

*   **`gspread`**: A library that lets Python speak to Google Sheets.
*   **`timestamp`**: We create new tabs with names like `Spider Traps 10/24 14:30`. This allows you to run the analysis multiple times and compare results over time without losing old data.

---

## 3. Variable Dictionary (The "Cheat Sheet")

| Variable Name | Purpose |
| :--- | :--- |
| `df` | The main "DataFrame" (like a giant Excel table in memory). |
| `chunk` | A small slice of the CSV file being processed. |
| `use_cols` | The whitelist of columns to keep to save RAM. |
| `unique_ips` | A list of every unique IP address that claimed to be Google. |
| `ip_map` | A dictionary where the key is the IP and the value is True/False (Verified). |
| `top_traps` | A filtered list of URLs that have more than 50 variations. |

---

## 4. Step-by-Step Execution Map

1.  **START**
2.  **Mount Drive**: Grant permission to read files.
3.  **Loop through Files**: Find every `.csv` in `Chargebee_Logs`.
4.  **Chunked Read**: Read 50k lines -> Filter for Googlebot -> Save to list -> Clear RAM.
5.  **Concatenate**: Combine all small lists into one big `df_raw`.
6.  **Verify**: Check IPs against Google's registry.
7.  **Calculate**: Count status codes, sum bytes, detect traps.
8.  **Plot**: Draw charts for visual confirmation.
9.  **Auth**: Log in to Google Sheets.
10. **Write**: Send the `top_traps` and `status_summary` to the spreadsheet.
11. **END**
