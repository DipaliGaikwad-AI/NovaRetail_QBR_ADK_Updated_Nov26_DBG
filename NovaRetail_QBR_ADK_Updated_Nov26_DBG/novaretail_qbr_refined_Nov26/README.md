# NovaRetail Capstone: Automated Executive QBR Deck

This project analyzes NovaRetail's three-sheet business dataset and populates the supplied five-slide PowerPoint template using Python and `python-pptx`.

## Project contents

- `generate_deck.py`: complete analysis and deck-generation workflow
- `novaretail_dataset.xlsx`: supplied source dataset
- `novaretail_template.pptx`: supplied fixed-layout template
- `icons/`: supplied icon assets
- `NovaRetail_QBR_Final.pptx`: generated executive presentation
- `requirements.txt`: Python dependencies

## Setup

```bash
python -m venv .venv
```

Activate the environment:

**Windows PowerShell**

```powershell
.venv\Scripts\Activate.ps1
```

**macOS/Linux**

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

## Generate the presentation

```bash
python generate_deck.py
```

The command creates `NovaRetail_QBR_Final.pptx` in the project folder.

## Execution logic

1. Load `Regional_Performance`, `Customer_Experience`, and `Operational_Risk` with `openpyxl`.
2. Index each record by quarter and region.
3. Calculate latest-quarter revenue, quarter-over-quarter growth, unit-weighted NPS, and region-level trends.
4. Open the supplied PowerPoint template.
5. Locate all writable areas by `shape.name`.
6. Replace KPI, headline, summary, insight, recommendation, and closing-statement text.
7. Remove dashed chart placeholders and instruction labels.
8. Add native PowerPoint line charts at the exact placeholder coordinates and dimensions.
9. Replace icon placeholders with the supplied theme-relevant artwork.
10. Save the completed five-slide presentation.

## Key dataset-driven narrative

- APAC is the strongest growth region and leads customer experience.
- North America remains the largest revenue market.
- Latin America shows declining revenue and worsening customer and operational indicators.
- APAC warehouse utilization and stockouts indicate a capacity constraint that could limit growth.

## Regeneration notes

- Keep the source filenames unchanged, or update the path constants at the top of `generate_deck.py`.
- Keep the template shape names unchanged because the script uses those names to locate content regions.
- If the dataset is refreshed with the same sheets and columns, rerun the script to regenerate the deck.


### Red-flag fixes included
- Packages all supplied PNG assets inside the expected `icons/` directory.
- Adds a backward-compatible icon lookup for older bundles that kept PNGs in the project root.
- Preserves square icon proportions when inserting icons, preventing stretching.
- Raises a clear error when an icon is missing.
- Guards percentage-change calculations against zero or missing baselines.

## Google ADK agentic workflow

This version uses Google Agent Development Kit (ADK) as a required part of presentation generation. `generate_deck.py` loads and calculates workbook metrics deterministically, then calls `adk_pipeline.py` before writing narrative content to PowerPoint.

The ADK pipeline contains two cooperating LLM agents:

1. `metric_analyst` reviews the supplied workbook rows and identifies material performance, customer, and operational patterns.
2. `executive_editor` converts the analysis into a Pydantic-validated `QBRNarrative` containing the executive summary, slide titles, insights, risks, recommendations, and closing statement.

The agents run through ADK `SequentialAgent`, `Runner`, and `InMemorySessionService`. The final structured response is used directly by the PowerPoint generator. Charts and KPI calculations remain deterministic Python logic so the language model cannot alter the source numbers.

### Configure Gemini

Copy `.env.example` to `.env` or export the values in the shell. For Google AI Studio:

```bash
export GOOGLE_API_KEY="your-key"
export ADK_MODEL="gemini-2.5-flash"
python generate_deck.py
```

For Vertex AI, use Application Default Credentials and set `GOOGLE_GENAI_USE_VERTEXAI=TRUE`, `GOOGLE_CLOUD_PROJECT`, and `GOOGLE_CLOUD_LOCATION`.

Generation intentionally stops with a clear configuration error if neither Google AI Studio nor Vertex AI authentication is configured. This prevents silently bypassing the required ADK/LLM workflow.

## Code quality changes

- Added docstrings to every function and to the structured narrative model.
- Added inline comments around ADK session state, agent handoff, event consumption, icon fallback, PowerPoint XML removal, and chart rendering.
- Added Pydantic validation so malformed or incomplete model output cannot be inserted into the deck.
