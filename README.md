# casa-agent_viewable
Casa Agent is a multi-agent tool that evaluates Roman real-estate listings against a buyer's house type preferences, budget, sunlight exposure, neighbourhood livability and commute. It parses listings, checks dealbreakers, and generates a structured report to quickly assess whether a property is worth pursuing. Walkthrough: [https://us06web.zoom.us/clips/share/b2ucu4zJRCGT2EU9DtriNA](https://www.youtube.com/watch?v=yn_uenRDJ6Q)

---

# 🏠 Casa Agent

**A multi-agent system that evaluates real-estate listings against a buyer's lifestyle, budget, and preferences.**

Casa Agent analyses a property listing (from any Italian property portal — Idealista, Immobiliare.it, Casa.it, and others) and produces a structured decision report covering:

* preference alignment
* financial viability
* sunlight exposure
* neighbourhood livability
* commute accessibility

The goal is to help buyers **quickly determine whether a property is worth pursuing**, before investing time in viewings or negotiations.

The system is designed as a **lightweight multi-agent pipeline**, where each agent specialises in a different aspect of property evaluation.

---

# 🎯 Project Motivation

Buying a home requires evaluating many factors simultaneously:

* Does the layout match my lifestyle?
* Is the price reasonable for the area?
* Will the apartment get enough sunlight?
* Is the neighbourhood livable?
* Will my commute be manageable?

Real estate listings typically present fragmented information, forcing buyers to manually research multiple aspects across multiple tools.

Casa Agent automates this analysis by combining:

* listing parsing (URL fetch or paste)
* preference matching against user-defined house types
* financial analysis including negotiation assessment
* geospatial lookups via OpenStreetMap
* lifestyle and livability evaluation
* commute routing via Google Maps

into a single structured report.

---

# ⚙️ Inputs

### Property information

* **Listing URL** — auto-fetch attempted for any Italian listing portal
* **Listing text** — fallback if URL fetch is blocked (paste from browser)

Works with Idealista, Immobiliare.it, Casa.it and other Italian portals.

### Manual overrides

Optional fields that supplement what is extracted from the listing text:

* Floor level
* Year last renovated
* Heating type (autonomo / centralizzato / pompa di calore)

These are entered in a side panel on the evaluation page alongside the listing input.

---

### Preference configuration

Users define their requirements once and save them as a named profile for reuse.

#### House types (OR logic)

2–3 acceptable property configurations, for example:

* House type 1: Ground floor apartment with garden
* House type 2: 2nd–4th floor with balcony or terrace and elevator

Each house type defines:
* Floor range (min / max)
* Minimum size (sqm)
* Required outdoor spaces and minimum outdoor sqm
* Elevator requirement

A listing passes if it matches **at least one** house type.

---

#### Exposure

Compass widget — each of the 8 directions (N, NE, E, SE, S, SW, W, NW) is marked as:
* 🟢 preferred
* 🟡 acceptable
* 🔴 avoid

---

#### Other requirements

Per-property requirements with Essential / Not important toggle:
* Parking
* Cantina / Storage
* Quiet street

---

### Budget

* **Maximum purchase price** — with automatic 8% negotiation margin applied
* **Maximum condominio** (monthly)
* **Agency fee %** (default 4%)

A listing passes the budget check if:

```
listing_price ≤ max_price / 0.92
```

This accounts for the typical negotiation margin between asking and final sale price in Italy.

---

### Commute destinations

Multiple destinations with name + address:

```
name: "Office"
address: "Via del Corso 1, Roma"
```

Raw addresses are passed directly to Google Maps for routing — no intermediate geocoding that could introduce address errors.

---

# 🤖 Agents

Five agents run **sequentially**. Agents B–E execute only if Agent A passes.

---

## A. Preferences Agent

Acts as a gatekeeper. If this agent fails, the evaluation stops and no further processing runs.

Checks:
* Does the listing match at least one house type?
* Is the price within budget (with negotiation margin)?
* Does the exposure match preferences?
* Are dealbreakers satisfied (parking, cantina, quiet street)?

Output: **PASS / FAIL + plain-English explanation**

---

## B. Financial Agent

Analyses the economic picture of the property.

* Price per sqm vs zone average for Rome (LLM-assisted)
* Days on market and price reduction history
* Full purchase cost breakdown: price + agency fee + notary estimate + registration tax
* Condominio monthly cost
* Heating type note (autonomo vs centralizzato)
* Renovation flag if last renovation > 10 years ago
* **Negotiation assessment** with suggested offer price

Includes an interactive price calculator — enter any offer price and see the full cost breakdown instantly.

---

## C. Sunlight Agent

Estimates natural light availability based on orientation, floor level, and urban context.

* Base sun hours by compass direction (Rome latitude ~42°N)
* Floor adjustment (ground floor penalty, upper floor bonus)
* Urban obstruction penalty assessed by LLM knowledge of the specific street
* Multi-aspect properties handled (e.g. "sud, est")

Output:
* Estimated sun hours in summer and winter
* Light quality verdict: Excellent / Good / Moderate / Poor
* Best time of day to visit for accurate light assessment

---

## D. Livability Agent

Evaluates the neighbourhood environment using OpenStreetMap data with LLM fallback.

**Road noise assessment:**
* Primary method: OSM road classification query with retry
* Fallback: GPT-4o assessment based on knowledge of the specific street — fires automatically if the OSM query fails, ensuring road noise is never silently blank

**Neighbourhood summary** via LLM: character, walkability, vibe, weaknesses.

If OpenStreetMap is temporarily unavailable (API timeout), the agent retries up to 3 times and shows a clear warning rather than silently returning empty results.

---

## E. Commutability Agent

Evaluates transit access and commute routes.

**Transit lookup via OpenStreetMap Overpass API:**

Queries using Rome-specific network tags for reliable results:
* `network=Metropolitana di Roma` for metro stations
* `network=ATAC` for bus stops
* Fallback generic tags (`highway=bus_stop`, `railway=subway_entrance`) as safety net
* 1500m radius for metro, 1200m for bus
* Deduplication to avoid the same stop appearing multiple times
* Retry logic (3 attempts) for Overpass API reliability

**Commute routing via Google Maps:**
* Raw address strings passed directly to Google Maps — no intermediate geocoding
* This avoids the house-number snapping problem where Nominatim would resolve "Via X 22" to the nearest known address point (e.g. number 131), causing Google Maps to show the wrong location
* Embedded directional map per saved destination
* "Explore more options" link opens Google Maps with full transit/walking/driving options

**ATAC map integration:**
* Link to ATAC Viaggia con map opens in one click
* Property address displayed in a copy box for easy pasting into ATAC's "Cerca intorno" search allowing users to further investigate publi transport options

---

# 📊 Output structure

```
Preferences: PASS / FAIL
  └ explanation + dealbreakers + warnings

If PASS:
    Financial report
      └ price/sqm vs zone · cost breakdown · negotiation assessment

    Sunlight report
      └ sun hours · light quality · visit timing tip

    Livability report
      └ amenities · road noise · neighbourhood summary · map

    Commutability report
      └ nearest metro + bus · ATAC link · commute maps per destination
```

This gated design avoids unnecessary computation for properties that clearly fail requirements.

---

# 🧱 Repository structure

```
casa-agent/
├── app.py                    # Streamlit entry point
├── agents/
│   ├── preferences.py        # Agent A — gatekeeper
│   ├── financial.py          # Agent B — price analysis + costs
│   ├── sunlight.py           # Agent C — light estimation
│   ├── livability.py         # Agent D — neighbourhood + road noise
│   └── commutability.py      # Agent E — transit + commute maps
├── utils/
│   ├── listing_parser.py     # URL fetch + LLM extraction
│   ├── profile_manager.py    # Save/load buyer profiles (JSON)
│   └── history_manager.py    # Evaluation history (SQLite)
├── profiles/                 # Saved buyer profiles
├── data/
│   └── history.db            # SQLite evaluation history
├── .env                      # API keys (not committed)
├── requirements.txt
└── README.md
```

---

# 🧠 Design decisions

**Why a multi-agent pipeline rather than one large prompt?**
Each agent has a single responsibility and produces interpretable output. This avoids the black-box problem — every section of the report has a clear source and reasoning. It also allows early termination: if Agent A (Preferences) fails, there is no point running financial or livability analysis.

**Why does Agent A gate the rest?**
Evaluating a property's finances or sunlight is only useful if it already meets basic requirements. Stopping early saves API cost and makes results cleaner.

**Why use OpenStreetMap rather than a paid API?**
OSM covers Rome's transit network well enough for the use case and requires no API key. The tradeoff is occasional availability issues — mitigated by retry logic and LLM fallback for road noise.

**Why pass addresses directly to Google Maps rather than geocoding them first?**
Nominatim (OSM's geocoder) sometimes snaps addresses to the nearest known house number, causing Google Maps to show the wrong location. Passing the raw address string lets Google Maps use its own, more accurate geocoder.

**Why SQLite for history?**
It is built into Python, requires no setup, and is sufficient for a single-user tool. The full listing data and all agent results are stored as JSON, so any past evaluation can be fully replicated from history.

---

# 🚧 Potential extensions

* Multi-city support (currently defaults to Roma)
* Automatic listing scraping across multiple portals
* Historical price trend comparison
* Renovation cost estimation from listing description
* Sunlight simulation using actual building geometry (GIS)
* Neighbourhood safety and pollution metrics
* Learning user preferences over time from evaluation history

---

# 🛠 Tech stack

* **Python 3.11+**
* **Streamlit** — UI
* **LangChain + OpenAI GPT-4o** — listing extraction, financial analysis, neighbourhood summaries, sunlight context, road noise fallback
* **OpenStreetMap Overpass API** — transit stops, amenities, parks, road classification
* **Nominatim** — property geocoding (for OSM queries only)
* **Google Maps embed** — commute routing and livability map
* **httpx** — HTTP requests
* **BeautifulSoup** — listing HTML parsing
* **SQLite** — evaluation history

---

# Author

**Lucy Michaels**
Data Scientist – Applied Data Analysis & Generative AI

Master's Degree — *Analisi e Modellazione dei Dati e dei Processi*
Unitelma Sapienza – Rome, 12-2025

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?logo=linkedin)](https://www.linkedin.com/in/lucymichaels/)

---

# License

This project is licensed under the **MIT License**.
