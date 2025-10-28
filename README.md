 [![Stand With Ukraine](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/banner2-direct.svg)](https://vshymanskyy.github.io/StandWithUkraine/)
 
# FleetLeaks — Global Sanctions Vessel Tracker

**Real-time maritime sanctions intelligence across six jurisdictions**  
Aggregating official designations from OFAC, EU, UK, Canada, Australia, and New Zealand into a single, searchable database.

> 🗺️ **Live platform**: [fleetleaks.com](https://fleetleaks.com)  
> 📊 **Interactive map**: [fleetleaks.com/vessel-map](https://fleetleaks.com/vessel-map)

---

## What Makes FleetLeaks Different

Most sanctions trackers are static lists or commercial services locked behind paywalls. **FleetLeaks is different**:

### 🔄 **Fully Automated Daily Updates**
- Zero manual intervention — the entire pipeline runs on autopilot
- Scrapes six government sources every 24 hours (00:00 UTC)
- Delta detection tracks what changed overnight: additions, removals, and modifications
- Public changelog shows exactly what moved ([view example](https://fleetleaks.com/vessels))

### 🎯 **Smart Normalization Layer**
- IMO number validation and cross-referencing
- Deduplicates vessels appearing on multiple lists (Panama-flagged tanker on both OFAC + EU? One entry, not two)
- Standardizes messy government data: conflicting names, missing fields, format inconsistencies
- Tracks designation history and source attribution

### 🌍 **Live AIS Integration**
- Real-time vessel positions overlaid on sanctions status
- Color-coded by recency: 🟢 updated <7 days, 🟠 7-30 days, 🔴 stale >30 days
- Directional markers show heading and speed
- Filter by sanctioner, vessel type, flag state, or broadcast status

### 📰 **AI-Powered News Pipeline**
- Monitors 30+ Google Alerts feeds in multiple languages
- Hybrid AI: OpenAI (GPT-4o-mini) for editorial review, Ollama (Llama 3.2) for vessel extraction
- LLM-based relevance scoring (shadow fleet operations, price cap violations, STS transfers)
- Automated publishing at 8 AM, 2 PM, 8 PM daily
- Cross-links vessel mentions to IMO profiles

### 🔓 **Completely Free & Open**
Built as a public resource for journalists, researchers, and compliance teams. No registration walls, no API limits for reasonable use.

---

## 🏗️ Architecture Overview

```
┌─────────────────────┐
│  Government APIs    │  OFAC, EU, UK, CA, AU, NZ
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ sanctions_pipeline  │  Python • Daily scrape + normalize
│  - IMO validation   │  - Delta detection
│  - Deduplication    │  - WordPress REST API push
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│    WordPress DB     │  Custom post type: 'vessel'
│  + Custom Plugins   │  Taxonomies: sanctioner, flag, type
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│   AIS Data Layer    │  Third-party stream (continuous)
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  fleetleaks.com     │  Public web interface
│  - Search & filters │  - Interactive map (Leaflet)
│  - Vessel profiles  │  - Changelog timeline
└─────────────────────┘
```

**Key components:**
- **`sanctions_pipeline.py`**: Core scraper + aggregator (Python 3.11+, runs via cron)
- **WordPress plugins**: Custom vessel post type, map shortcodes, REST endpoints
- **`news_pipeline.py`**: Hybrid AI article pipeline (OpenAI GPT-4o-mini + Ollama Llama 3.2)
- **Front-end**: Block theme + custom CSS (no framework bloat)

---

## 📊 Current Dataset

- **815 vessels** actively tracked
- **6 sanctioning authorities** (OFAC, EU, UK, Canada, Australia, New Zealand)
- **Daily refresh** ensures data is never >24 hours stale
- **AIS coverage**: ~85% of vessels broadcasting within last 30 days

**Most common patterns:**
- 🇷🇺 Russia-flagged tankers (47% of total)
- 🇵🇦 Panama / 🇱🇷 Liberia flags of convenience
- Crude oil tankers, product tankers, general cargo
- Dark fleet vessels with AIS manipulation behaviors

---

## 🔎 Example Vessel Profile

**Live example**: [KARAKUZ (IMO 9621558)](https://fleetleaks.com/vessels/imo-9621558/)

**What you'll see:**
- Sanctions timeline (which lists, when designated)
- Real-time position + 7-day track history
- Flag changes and ownership opacity
- Management company details (if available)
- Cross-links to official government records

*Note: Screenshots in `/docs/` show point-in-time data — live profiles update continuously.*

---

## 🗺️ Map Visualization

The [interactive map](https://fleetleaks.com/vessel-map) uses:
- **Leaflet.js** + **MarkerCluster** for performance
- **Filters**: Sanctioner, vessel type, flag state, last update age
- **Vessel popups**: Name, IMO, speed, heading, last broadcast time
- **Color coding**: Green (fresh), Orange (aging), Red (stale), Gray (dark)

**Technical specs:**
- Renders 700+ markers smoothly on mobile
- REST API endpoint: `/wp-json/fleetleaks/v1/vessels/map-data`
- Client-side clustering prevents map clutter
- Auto-refresh: positions update every 15 minutes

---

## 🤝 Use Cases

### For Journalists
- Cross-reference suspected dark fleet vessels with official lists
- Track when vessels appear/disappear from sanctions (why was X delisted?)
- Export vessel timelines for investigative pieces
- Real-time alerts when high-profile vessels broadcast position

### For Compliance Teams
- Quick IMO lookup: "Is this vessel sanctioned anywhere?"
- Multi-jurisdiction screening (one search, six sources)
- Historical record: "When was this vessel first designated?"
- API access for bulk screening (contact for rate limits)

### For Researchers
- Analyze sanctions evasion patterns (flag-hopping, AIS spoofing)
- Study coordination between jurisdictions (who designates first?)
- Correlate vessel movements with geopolitical events
- Dataset available for academic collaboration

---

## 📡 Data Schema (Public)

**Vessel record structure:**

```json
{
  "imo": "9313242",
  "name": "EXAMPLE VESSEL",
  "flag": "PA",
  "vessel_type": "Oil/Chemical Tanker",
  "sanctioners": ["OFAC", "EU", "UK"],
  "designation_date": "2025-01-15",
  "status": "Active",
  "latitude": 25.1234,
  "longitude": 55.5678,
  "speed_knots": 13.4,
  "heading_degrees": 92,
  "ais_last_update": "2025-10-15T12:30:00Z",
  "url": "https://fleetleaks.com/vessels/imo-9313242/"
}
```

**Notes:**
- Private enrichment fields (beneficial owners, insurance, port calls) intentionally omitted
- AIS positions are live and subject to vessel manipulation
- `sanctioners` array shows all jurisdictions where vessel appears

---

## 🔄 Update Cadence

| Data Type | Frequency | Source |
|-----------|-----------|--------|
| Sanctions lists | Daily (00:00 UTC) | OFAC, EU, UK, CA, AU, NZ |
| AIS positions | Continuous | Third-party stream |
| News articles | 3x daily | Google Alerts + LLM |
| Database refresh | On verified delta | Automatic |

**Changelog publication:**
- Daily digest at [fleetleaks.com/changelog](https://fleetleaks.com/changelog)
- RSS feed available for automated monitoring
- Vessel profiles show individual change history

---

## 🧩 Collaboration & Integration

FleetLeaks welcomes partnerships with:

**Journalists & OSINT practitioners:**
- Embed vessel profiles in your reporting
- API access for investigative projects
- Co-publish findings with source attribution

**Risk intelligence platforms:**
- Cross-link "sketchy score" with verified sanctions status
- Publish correlation studies: high-risk vessels → later sanctions?
- Lightweight JSON exchange (keyed by IMO)

**Academic researchers:**
- Bulk data access for peer-reviewed studies
- Anonymized pattern analysis
- Co-author publications on sanctions effectiveness

**If you maintain a maritime compliance dataset**, let's align schemas and avoid duplicate work.

📩 **Contact**: [fleetleaks@proton.me](mailto:fleetleaks@proton.me)  
*(GitHub issues work too — I'll close them after reading)*

---

## 🛡️ Data Quality & Limitations

**What FleetLeaks does well:**
- ✅ Mirrors official government sources with <24hr lag
- ✅ Catches cross-jurisdictional patterns invisible in single-source lists
- ✅ Tracks designation history and removals (not just current status)

**Known limitations:**
- ⚠️ AIS data can be spoofed or disabled by vessels
- ⚠️ Ownership data is incomplete (shell companies obscure trails)
- ⚠️ We aggregate but don't interpret legal nuances — always verify at source

**If you spot an error**, email with:
1. IMO number
2. What's wrong
3. Link to official source showing correct data

Corrections typically processed within 24 hours.

---

## 🎨 Brand Colors

For embeds, integrations, or derivative works:

```css
/* Primary gradient */
background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);

/* Status colors */
--fl-green: #10b981;   /* Fresh AIS */
--fl-orange: #f97316;  /* Aging data */
--fl-red: #ef4444;     /* Stale */
--fl-gray: #6b7280;    /* Dark vessel */

/* Neutrals */
--fl-dark: #111827;
--fl-light: #f8fafc;
--fl-border: #e5e7eb;
```

---

## 📄 Licensing

**Documentation & samples** (this README, `/docs/`, screenshots):  
© FleetLeaks — [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/)

**Code snippets** (if shared in `/examples/`):  
MIT License

**Commercial use** of docs/samples requires written consent.  
For broader licensing, contact [fleetleaks@proton.me](mailto:fleetleaks@proton.me).

---

## 🧭 Credits

**FleetLeaks** is maintained by **Ben**, an independent developer focused on:
- Open data transparency
- Maritime compliance intelligence
- Fighting opaque shipping practices that enable sanctions evasion

*Built with Python, WordPress, Leaflet, and an unhealthy amount of coffee.*

---

## 🔗 Links

- **Website**: [fleetleaks.com](https://fleetleaks.com)
- **Map**: [fleetleaks.com/vessel-map](https://fleetleaks.com/vessel-map)
- **Changelog**: [fleetleaks.com/changelog](https://fleetleaks.com/changelog)
- **Contact**: [fleetleaks@proton.me](mailto:fleetleaks@proton.me)

---

**If you cite FleetLeaks in your work**, please link to [fleetleaks.com](https://fleetleaks.com) and this repository. Attribution helps us reach more researchers and journalists who need this data.
