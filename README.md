# Personal MCP Server for AI Resume Submission

This repository contains a **Model Context Protocol (MCP)** server used to securely deliver my resume to **Puch AI** for application validation and submission. It doubles as a neat personal project demonstrating token-based auth, LLM-ready content pipelines, and cloud deployment.

---

## Features

* **Bearer Token Auth** – Simple provider that validates a single application key.
* **Resume Tool** – Reads a local **PDF/DOCX/TXT** resume, converts it to **Markdown**, and returns raw MD text (no extra formatting).
* **Fetch Tool** – Retrieves webpages and simplifies HTML to Markdown for LLMs; supports pagination of long pages.
* **Cloud Ready** – Designed for one-click deploy on **Render**; works locally with `uvicorn`.

---

## Tech Stack

* **Python 3.11+**
* **fastmcp** (MCP server framework)
* **httpx**, **readabilipy**, **markdownify** (content fetching & cleaning)
* **pypandoc** (document → Markdown conversion)
* **Render** (hosting)

---

## Project Structure

```
.
├── mcp_server.py       # Main server
├── requirements.txt    # Python dependencies
├── Procfile            # Start command for Render
└── README.md           # This file
```

---

## ⚙Configuration

The server expects three values in the source file. For convenience, they are defined as constants near the top of `mcp_server.py`.

* `TOKEN` – Your Puch application key (from `/apply <URL>`)
* `MY_NUMBER` – Phone number in E.164 **without the plus** (e.g., `917988236305`)
* `resume_path` – Absolute path to your resume (PDF/DOCX/TXT)

> **Security tip:** You may move `TOKEN`/`MY_NUMBER` to environment variables instead of hardcoding.

Example (env-based):

```bash
export TOKEN="<your_key>"
export MY_NUMBER="<countrycode><number>"  # e.g. 917988236305
```

Then in `mcp_server.py`, read them via `os.getenv("TOKEN")` and `os.getenv("MY_NUMBER")` with sensible fallbacks.

---

## Quick Start (Local)

1. **Install system Pandoc** (required by `pypandoc`):

   * macOS: `brew install pandoc`
   * Ubuntu/Debian: `sudo apt-get update && sudo apt-get install -y pandoc`
   * Windows (choco): `choco install pandoc`

2. **Install Python dependencies**

   ```bash
   pip install -r requirements.txt
   ```

3. **Edit `mcp_server.py`**

   * Set `TOKEN`, `MY_NUMBER`, and `resume_path`.

4. **Run**

   ```bash
   python mcp_server.py
   ```

   You should see `Uvicorn running on http://0.0.0.0:8085`.

5. **(Optional) Tunnel for public URL**
   If testing locally, you can use ngrok:

   ```bash
   ngrok http 8085
   ```

---

## ☁Deploy on Render (Recommended)

1. Push this repo to **GitHub**.
2. Go to **Render → New → Web Service**.
3. Connect the repository.
4. Configure:

   * **Runtime**: Python 3.11+
   * **Build Command**: `pip install -r requirements.txt`
   * **Start Command**: `python mcp_server.py`
   * **Instance**: Free
5. (Optional) Add **Environment Variables**:

   * `TOKEN` = your application key
   * `MY_NUMBER` = e.g., `917988236305`
   * `RESUME_PATH` = absolute path in the container (if you commit your resume or mount storage)
6. Deploy. Render will output a URL like:

   ```
   https://your-service.onrender.com
   ```

> **Note:** If your resume is private and not in the repo, consider Render **Persistent Disks** or upload during build via private storage. For the application step, committing a temporary PDF to the repo is simplest; remove it after submission.

---

## MCP Tools / Endpoints

This server is MCP-compliant and exposes tools (consumed via `/mcp` by Puch):

### `validate`

* **Description:** Returns the configured phone number.
* **Purpose:** Puch uses it to validate `{TOKEN}` and phone pairing.

### `resume`

* **Description:** Returns the resume **as Markdown string**.
* **Input:** none (reads the file from `resume_path`)
* **Output:** Pure Markdown text (no extra formatting or wrapping).

### `fetch`

* **Description:** Fetch a URL, simplify HTML → Markdown, paginate long outputs.
* **Args:**

  * `url` (string, required)
  * `max_length` (int, default 5000)
  * `start_index` (int, default 0)
  * `raw` (bool, default `False`) – if `True`, returns raw HTML

---

## Connect from Puch

Once the service is public:

```text
/mcp connect https://your-service.onrender.com/mcp <YOUR_TOKEN>
```

---

##  Quick Smoke Test (Without Puch)

You can sanity-check that the service is up (root will 404, that’s expected). For real checks, connect via Puch or hit the MCP route with an authorized client. If you must curl, confirm the service port and health by requesting `/` and expecting `404` rather than connection refused.

---

##  Troubleshooting

* **`404 Not Found` at `/`** – Expected. Use `/mcp` with an MCP client (e.g., Puch).
* **`Failed to read or convert resume`** – Ensure Pandoc is installed and `resume_path` is correct. Try converting locally with `pandoc <file> -t gfm -o /tmp/test.md`.
* **Render build fails** – Check Python version, `requirements.txt`, and that `pandoc` is not required at build time (only at runtime via `pypandoc`).
* **Auth fails in Puch** – Verify you passed the correct token and that `validate` returns the exact phone format `{countrycode}{number}` with no plus sign.

---

##  Security Notes

* The included `SimpleBearerAuthProvider` validates a **single** bearer token. For production, use a proper key management solution.
* Avoid committing your real token or phone number. Prefer **environment variables**.
* Remove your resume from the repository once the application process is complete.

---
