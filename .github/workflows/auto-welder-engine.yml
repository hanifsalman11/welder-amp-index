import os
import json
import re
import time
import urllib.request
import urllib.parse
from datetime import datetime, timezone

# ==========================================
# ENVIRONMENT CONFIGURATION
# ==========================================
GEMINI_API_KEY = os.environ.get("GEMINI_API_KEY", "").strip()
SITE_BASE_URL = os.environ.get("SITE_BASE_URL", "https://hanifsalman11.github.io/welder-amp-index").rstrip("/")
INDEXNOW_KEY = os.environ.get("INDEXNOW_KEY", "a1c3e5f7b9d2e4a6c8f0b1d3e5a7c9f2").strip()
CUSTOM_WELDER = os.environ.get("CUSTOM_WELDER", "").strip()
PAGES_PER_RUN = int(os.environ.get("PAGES_PER_RUN", "3"))

DOCS_DIR = "docs"
DATA_DIR = "data"
PUBLISHED_FILE = os.path.join(DATA_DIR, "published_welders.json")
MONETIZATION_FILE = os.path.join(DATA_DIR, "monetization.json")

# Format: (Welder Model / Material Scenario, Max Output Amps, Duty Cycle %, Input Voltage, DefaultBreaker Amps, Process Type)
SEED_WELDERS = [
    ("Miller Millermatic 211 Auto-Set MIG Welder", 210, 40, "120V / 240V Dual", 30, "MIG / Flux-Core"),
    ("Lincoln Electric Power MIG 215 MPi Multi-Process", 215, 30, "120V / 240V Dual", 30, "MIG / TIG / Stick"),
    ("Hobart Handler 210 MVP MIG Welder", 210, 30, "115V / 230V MVP", 30, "MIG / Flux-Core"),
    ("Hobart Handler 140 115V MIG Welder", 140, 20, "115V Household", 20, "MIG / Flux-Core"),
    ("YesWelder MIG-205DS-B Multi-Process Inverter", 205, 60, "110V / 220V Dual", 30, "Synergic MIG / Lift TIG / Stick"),
    ("YesWelder TIG-200P AC/DC Aluminum TIG Welder", 200, 60, "110V / 220V Dual", 30, "AC/DC Pulse TIG / Stick"),
    ("PrimeWeld TIG225X 225A AC/DC Pulse TIG Welder", 225, 60, "110V / 220V Dual", 35, "AC/DC TIG / Stick"),
    ("PrimeWeld MIG180 180A Dual-Voltage Welder", 180, 35, "110V / 220V Dual", 30, "MIG / Flux-Core / Spool Gun"),
    ("Titanium Unlimited 200 Multi-Process Welder", 200, 30, "120V / 240V Dual", 30, "MIG / DC TIG / Stick"),
    ("Titanium Easy-Flux 125 Amp Welder", 125, 30, "120V Household", 20, "Gasless Flux-Core"),
    ("Lincoln Electric Weld-Pak 180i MP DV", 180, 30, "120V / 230V Dual", 30, "MIG / DC TIG / Stick"),
    ("Miller Multimatic 220 AC/DC Multi-Process", 220, 40, "120V / 240V Auto-Line", 30, "MIG / AC-DC TIG / Stick"),
    ("Miller Diversion 180 AC/DC TIG Welder", 180, 35, "115V / 230V Dual", 25, "AC/DC GTAW TIG"),
    ("ESAB Rebel EMP 215ic Multi-Process Welder", 240, 25, "120V / 230V sMIG", 30, "sMIG / Lift TIG / Stick"),
    ("ESAB Rogue ET 200iP Pro DC TIG Welder", 200, 25, "120V / 230V Dual", 25, "Pulsed DC TIG / Stick"),
    ("Everlast PowerTIG 200DV AC/DC TIG Welder", 200, 35, "120V / 240V Dual", 30, "AC/DC Pulse TIG / Stick"),
    ("Everlast Cyclone 200E MIG Welder", 200, 35, "120V / 240V Dual", 30, "Inverter MIG / Flux-Core"),
    ("Forney Easy Weld 140 MP Multi-Process", 140, 30, "120V Household", 20, "MIG / DC TIG / Stick"),
    ("Forney 220 MP Multi-Process Welder", 220, 30, "120V / 230V Dual", 30, "Synergic MIG / TIG / Stick"),
    (" Vulcan OmniPro 220 Multi-Process Welder", 220, 30, "120V / 240V Dual", 30, "MIG / DC TIG / Stick"),
    ("Vulcan ProTIG 205 AC/DC TIG Welder", 205, 25, "120V / 240V Dual", 30, "AC/DC TIG / Stick"),
    (" Eastwood Elite MP200i Multi-Process Welder", 200, 30, "120V / 240V Dual", 30, "MIG / TIG / Stick"),
    ("Tooliom 200M 200A Synergic MIG Welder", 200, 60, "110V / 220V Dual", 30, "MIG / Flux-Core / Lift TIG"),
    ("ARCCAPTAIN MIG200 6-in-1 Multi-Process Welder", 200, 60, "110V / 220V Dual", 30, "Synergic MIG / Pulse / TIG"),
    ("DEKOPRO 160A MMA Stick Arc Welder", 160, 40, "110V / 220V Dual", 25, "7018 / 6011 SMAW Stick"),
    ("1/4-Inch Mild Steel Plate MIG Welding Setup", 210, 40, "240V Single-Phase", 30, "Short-Circuit / Spray MIG"),
    ("3/16-Inch Structural Steel Tube MIG Setup", 175, 40, "240V Single-Phase", 30, "GMAW C25 Gas MIG"),
    ("1/8-Inch Mild Steel & Auto Frame Fabrication", 130, 35, "120V / 240V Dual", 20, "0.030 Solid Wire MIG"),
    ("16-Gauge Automotive Sheet Metal Bodywork", 65, 60, "120V Household", 20, "0.023 Wire Stitch MIG"),
    ("1/8-Inch 6061 Aluminum AC TIG Welding Setup", 145, 50, "240V AC High-Freq", 30, "AC GTAW Pure Argon")
]


def load_json(filepath, default):
    if os.path.exists(filepath):
        try:
            with open(filepath, "r", encoding="utf-8") as f:
                return json.load(f)
        except Exception as e:
            print(f"[WARN] Could not parse {filepath}: {e}")
    return default


def save_json(filepath, data):
    os.makedirs(os.path.dirname(filepath), exist_ok=True)
    with open(filepath, "w", encoding="utf-8") as f:
        json.dump(data, f, indent=2, ensure_ascii=False)


def slugify(text):
    text = text.lower().strip()
    text = re.sub(r"[^a-z0-9\s-]", "", text)
    return re.sub(r"[\s-]+", "-", text).strip("-")


def safe_float(val, default=200.0):
    try:
        cleaned = re.sub(r"[^0-9.]", "", str(val))
        return float(cleaned) if cleaned else float(default)
    except Exception:
        return float(default)


def build_amazon_url(search_keywords, mon_cfg):
    domain = mon_cfg.get("marketplace_domain", "www.amazon.com")
    tag = mon_cfg.get("amazon_associate_tag", "YOUR_AMAZON_TAG-20")
    encoded_q = urllib.parse.quote_plus(search_keywords)
    return f"https://{domain}/s?k={encoded_q}&tag={tag}"


def discover_google_welder_queries():
    """Mengambil kueri model mesin las & kalkulator pengaturan las dari Google Autocomplete US."""
    discovered = []
    seeds = [
        "mig welding settings for ",
        "miller millermatic ",
        "yeswelder ",
        "hobart handler ",
        "welding amperage calculator for "
    ]
    for seed in seeds:
        try:
            url = f"http://suggestqueries.google.com/complete/search?client=firefox&hl=en&gl=us&q={urllib.parse.quote(seed)}"
            req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
            with urllib.request.urlopen(req, timeout=8) as resp:
                data = json.loads(resp.read().decode("utf-8"))
                for suggestion in data[1]:
                    cleaned = suggestion.lower()
                    for pref in seeds:
                        cleaned = cleaned.replace(pref, "")
                    cleaned = re.sub(
                        r"\b(calculator|settings|chart|manual|review|reddit|for|amps)\b",
                        "",
                        cleaned
                    ).strip()
                    if 5 <= len(cleaned) <= 42:
                        amp_match = re.search(r"(125|140|180|200|205|210|215|220|225|250)", cleaned)
                        amp_val = int(amp_match.group(1)) if amp_match else 200
                        volt_str = "120V Household" if amp_val <= 140 else "120V / 240V Dual"
                        brk_val = 20 if amp_val <= 140 else 30
                        discovered.append((cleaned.title(), amp_val, 35, volt_str, brk_val, "MIG / TIG / Stick"))
        except Exception as e:
            print(f"[WARN] Google Suggest notice: {e}")
    return discovered


def build_deterministic_welder_data(weld_tuple):
    """Mesin Fisika Pengelasan AWS (1 Amp per 0.001") & Kelistrikan NEC Article 630 (100% bebas gagal)."""
    model_name, max_amps, duty_pct, input_volt, breaker_a, proc_type = weld_tuple

    # Ketebalan baja maksimum single-pass berdasarkan aturan 1 Amp = 0.001 inci
    max_thickness_in = round(max_amps * 0.001, 3)
    if max_amps >= 200:
        max_plate_label = "3/8-Inch (single/multi-pass) & 1/4-Inch (single-pass)"
        cord_awg = "8 AWG or 10 AWG 3-Conductor NEMA 6-50P to 6-50R (240V)"
    elif max_amps >= 170:
        max_plate_label = "1/4-Inch Mild Steel (single-pass)"
        cord_awg = "10 AWG 3-Conductor NEMA 6-50 Extension Cord (240V)"
    else:
        max_plate_label = "3/16-Inch Mild Steel (MIG/Flux-Core)"
        cord_awg = "12 AWG Heavy-Duty 120V 20A Extension Cord"

    # Parameter referensi untuk baja 1/8 inci (0.125" = 125 Amps) dengan kawat 0.030" (multiplier 1.7 IPM/Amp)
    ref_125_ipm = int(round(125 * 1.70))
    ref_125_volts = 18.5

    return {
        "welder_name": model_name,
        "process_type": proc_type,
        "seo_title": f"{model_name} Settings Calculator: Amps, Wire Speed (IPM) & Breaker (2026)",
        "meta_description": f"Interactive {model_name} settings chart & calculator. Compute exact MIG/TIG/Stick amperage (up to {max_amps}A), wire feed speed ({ref_125_ipm} IPM @ 1/8\"), voltage, and {breaker_a}A breaker size.",
        "specs": {
            "max_output_amps": max_amps,
            "duty_cycle_pct": duty_pct,
            "input_voltage": input_volt,
            "recommended_breaker_amps": breaker_a,
            "recommended_cord_spec": cord_awg,
            "max_steel_thickness": max_plate_label,
            "ref_eighth_inch_ipm": ref_125_ipm,
            "ref_eighth_inch_volts": ref_125_volts
        },
        "aws_verdict": f"Rated at {max_amps} Amps peak output ({duty_pct}% duty cycle) on {input_volt} power, the {model_name} ({proc_type}) comfortably welds up to {max_plate_label}. Using the American Welding Society (AWS) baseline of 1 Amp per 0.001\" of material thickness, welding 1/8-inch (0.125\") mild steel requires ~125 Amps, {ref_125_volts} Volts, and ~{ref_125_ipm} IPM wire feed speed with 0.030\" ER70S-6 wire and 25 CFH of 75% Argon / 25% CO2 (C25) shielding gas.",
        "hardware_bundle": [
            {
                "badge": f"{max_amps}A Welder Unit",
                "title": f"{model_name} ({input_volt})",
                "price_range": "$289 - $1,399",
                "amazon_query": f"{model_name} Welder",
                "weld_note": f"Delivers {max_amps}A output at {duty_pct}% duty cycle with smooth arc starts and thermal overload protection."
            },
            {
                "badge": "1/1/1/1 True-Color Hood",
                "title": "Panoramic 1/1/1/1 Optical Clarity True-Color Auto-Darkening Welding Helmet (4 Arc Sensors)",
                "price_range": "$79 - $289",
                "amazon_query": "True Color Auto Darkening Welding Helmet 1/1/1/1 4 Arc Sensor",
                "weld_note": "Eliminates the traditional green lens tint so you can clearly read the liquid puddle edge and toe tie-in on MIG and TIG welds."
            },
            {
                "badge": "NEC 630 Power & Gas Kit",
                "title": f"{cord_awg} + Brass CGA-580 Argon/CO2 Flowmeter Regulator",
                "price_range": "$69 - $169",
                "amazon_query": "NEMA 6-50 Extension Cord 8 AWG 50A Welder Argon Flowmeter Regulator",
                "weld_note": f"Prevents voltage drop during high-inrush arc striking on a {breaker_a}A circuit and meters exact 20–30 CFH shielding gas flow."
            },
            {
                "badge": "Wire, Tips & Rolling Cart",
                "title": "ER70S-6 .030\" & E71T-GS Flux-Core Wire Spools, Contact Tips & 3-Tier Tilted Welding Cart",
                "price_range": "$59 - $159",
                "amazon_query": "ER70S-6 030 MIG Welding Wire Heavy Duty 3 Tier Welding Cart",
                "weld_note": "Includes dual gas-cylinder safety chains and copper contact tips matched to 0.023\", 0.030\", and 0.035\" wire."
            }
        ],
        "fabrication_checklist": [
            "Grind all mill scale, rust, paint, and galvanized zinc coatings at least 1 inch back from the joint; bare shiny metal is mandatory for porosity-free short-circuit MIG and TIG welds.",
            "Set correct polarity before striking an arc: Solid Wire (ER70S-6) with C25 shielding gas requires DCEP (Direct Current Electrode Positive), while Gasless Flux-Core (E71T-GS) requires DCEN (Electrode Negative).",
            "Maintain a tight 3/8-inch (9.5mm) wire stick-out (CTWD) for short-circuit MIG; excessive stick-out drops amperage, causes ropey cold beads, and blows away shielding gas coverage.",
            f"Size your workshop circuit per NEC Article 630: with a {duty_pct}% duty cycle at {max_amps}A output, use a dedicated {breaker_a}A breaker and {cord_awg}."
        ],
        "faqs": [
            {
                "q": f"What thickness of steel can the {model_name} weld?",
                "a": f"With {max_amps} Amps of maximum output, the {model_name} can weld from 24-gauge sheet metal (at 30–40 Amps with 0.023\" wire) up to {max_plate_label} using 0.030\" or 0.035\" wire and proper joint beveling."
            },
            {
                "q": f"What wire speed (IPM) and voltage should I use for 1/8-inch steel on the {model_name}?",
                "a": f"For 1/8-inch (0.125\" / 11-gauge) mild steel using 0.030\" ER70S-6 solid wire and 75/25 Argon/CO2 gas, set voltage to {ref_125_volts}V – 19.5V and wire feed speed to {ref_125_ipm} – 230 Inches Per Minute (IPM) (~125 Amps)."
            },
            {
                "q": f"What size breaker and outlet do I need for the {model_name}?",
                "a": f"On 240V operation, install a NEMA 6-50R receptacle backed by a {breaker_a}-Amp 2-pole circuit breaker. If running on a 120V outlet, use a dedicated 20-Amp breaker with a 12 AWG cord so the breaker does not trip at 90–125A output."
            }
        ]
    }


def generate_welder_data(weld_tuple):
    """Memanggil Gemini 2.5 Flash API; jika gagal otomatis memakai Deterministic AWS Welding Engine."""
    if not GEMINI_API_KEY:
        return build_deterministic_welder_data(weld_tuple)

    model_name, max_amps, duty_pct, input_volt, breaker_a, proc_type = weld_tuple
    ref_125_ipm = int(round(125 * 1.70))

    prompt = f"""
You are an AWS Certified Welding Inspector (CWI) & Industrial Electrical Engineer (NEC Article 630). Create an accurate 2026 welding settings calculator guide (Amps, Voltage, Wire Feed Speed IPM, Shielding Gas CFH, and Breaker Sizing) for: "{model_name}".
Baseline reference specs: Max Output={max_amps}A, Duty Cycle={duty_pct}%, Input Voltage={input_volt}, Breaker={breaker_a}A, Process={proc_type}, 1/8" Steel 0.030" Wire Speed={ref_125_ipm} IPM @ 18.5V.
Return ONLY a valid JSON object matching this exact schema:
{{
  "welder_name": "{model_name}",
  "process_type": "{proc_type}",
  "seo_title": "{model_name} Settings Calculator: Amps, Wire Speed (IPM) & Breaker (2026)",
  "meta_description": "Interactive {model_name} settings chart & calculator. Compute exact MIG/TIG/Stick amperage, wire feed speed (IPM), arc voltage, and 240V breaker size.",
  "specs": {{
    "max_output_amps": {max_amps},
    "duty_cycle_pct": {duty_pct},
    "input_voltage": "{input_volt}",
    "recommended_breaker_amps": {breaker_a},
    "recommended_cord_spec": "10 AWG NEMA 6-50 Extension Cord",
    "max_steel_thickness": "3/8-Inch Mild Steel",
    "ref_eighth_inch_ipm": {ref_125_ipm},
    "ref_eighth_inch_volts": 18.5
  }},
  "aws_verdict": "3-sentence technical summary stating max amps, duty cycle, max metal thickness, and exact voltage/IPM settings for 1/8-inch steel.",
  "hardware_bundle": [
    {{
      "badge": "Welder Machine",
      "title": "{model_name}",
      "price_range": "$299 - $1,399",
      "amazon_query": "{model_name} Welder",
      "weld_note": "1-sentence arc performance and duty cycle note"
    }},
    {{
      "badge": "1/1/1/1 True-Color Helmet",
      "title": "True-Color 1/1/1/1 Optical Clarity Auto-Darkening Welding Helmet",
      "price_range": "$79 - $289",
      "amazon_query": "True Color Auto Darkening Welding Helmet 1/1/1/1",
      "weld_note": "1-sentence puddle visibility note"
    }},
    {{
      "badge": "NEMA 6-50 Cord & Regulator",
      "title": "Heavy-Duty 8 AWG NEMA 6-50 Extension Cord + CGA-580 Argon/CO2 Flowmeter",
      "price_range": "$69 - $169",
      "amazon_query": "NEMA 6-50 Extension Cord 8 AWG Welder Argon Flowmeter",
      "weld_note": "1-sentence voltage drop and gas flow note"
    }},
    {{
      "badge": "Wire Spools & Welding Cart",
      "title": "ER70S-6 .030\" Solid & Flux-Core Wire Spools + 3-Tier Rolling Welding Cart",
      "price_range": "$59 - $159",
      "amazon_query": "ER70S-6 030 MIG Welding Wire 3 Tier Welding Cart",
      "weld_note": "1-sentence wire feed and cylinder storage note"
    }}
  ],
  "fabrication_checklist": [
    "Step 1 regarding metal preparation and mill scale removal",
    "Step 2 regarding DCEP (gas MIG) vs DCEN (gasless flux-core) polarity",
    "Step 3 regarding 3/8-inch contact tip to work distance (stick-out)",
    "Step 4 regarding NEC Article 630 breaker and extension cord gauge"
  ],
  "faqs": [
    {{"q": "What thickness of steel can the {model_name} weld?", "a": "Exact gauge and inch limits for single-pass and multi-pass."}},
    {{"q": "What wire speed and voltage for 1/8-inch steel on the {model_name}?", "a": "Exact Volts, IPM, wire diameter, and CFH gas flow."}},
    {{"q": "What size breaker and extension cord do I need for the {model_name}?", "a": "Exact 120V and 240V breaker amps and AWG cord size."}}
  ]
}}
"""

    for model_id in ["gemini-2.5-flash", "gemini-2.0-flash"]:
        try:
            endpoint = f"https://generativelanguage.googleapis.com/v1beta/models/{model_id}:generateContent?key={GEMINI_API_KEY}"
            payload = {
                "contents": [{"parts": [{"text": prompt}]}],
                "generationConfig": {
                    "temperature": 0.2,
                    "responseMimeType": "application/json"
                }
            }
            req = urllib.request.Request(
                endpoint,
                data=json.dumps(payload).encode("utf-8"),
                headers={"Content-Type": "application/json"},
                method="POST"
            )
            with urllib.request.urlopen(req, timeout=35) as resp:
                result = json.loads(resp.read().decode("utf-8"))
                raw_text = result["candidates"][0]["content"]["parts"][0]["text"].strip()
                raw_text = re.sub(r"^```(?:json)?\s*", "", raw_text)
                raw_text = re.sub(r"\s*```$", "", raw_text)
                parsed = json.loads(raw_text)
                if "specs" in parsed and "hardware_bundle" in parsed:
                    return parsed
        except Exception as e:
            print(f"[WARN] Gemini ({model_id}) notice for '{model_name}': {e}")

    return build_deterministic_welder_data(weld_tuple)


def build_welder_html_page(data, slug, mon_cfg):
    welder_name = data.get("welder_name", "Multi-Process Welder")
    proc_type = data.get("process_type", "MIG / TIG / Stick")
    sp = data.get("specs", {})
    max_amps = int(safe_float(sp.get("max_output_amps", 200), 200))
    duty_pct = int(safe_float(sp.get("duty_cycle_pct", 35), 35))
    input_volt = sp.get("input_voltage", "120V / 240V Dual")
    breaker_a = int(safe_float(sp.get("recommended_breaker_amps", 30), 30))
    max_steel = sp.get("max_steel_thickness", "3/8-Inch Mild Steel")

    header_cta = mon_cfg.get("header_cta", {})
    header_cta_url = build_amazon_url(header_cta.get("search_query", "Multi+Process+MIG+TIG+Stick+Welder"), mon_cfg)

    schema_graph = {
        "@context": "https://schema.org",
        "@graph": [
            {
                "@type": "SoftwareApplication",
                "name": f"{welder_name} Welding Settings & Wire Speed Calculator",
                "applicationCategory": "UtilitiesApplication",
                "operatingSystem": "Web Browser",
                "offers": {"@type": "Offer", "price": "0", "priceCurrency": "USD"}
            },
            {
                "@type": "FAQPage",
                "mainEntity": [
                    {
                        "@type": "Question",
                        "name": f.get("q", ""),
                        "acceptedAnswer": {"@type": "Answer", "text": f.get("a", "")}
                    } for f in data.get("faqs", [])
                ]
            }
        ]
    }

    bundle_html = ""
    for item in data.get("hardware_bundle", []):
        amz_url = build_amazon_url(item.get("amazon_query", item.get("title", "MIG TIG Stick Welder")), mon_cfg)
        bundle_html += f"""
        <div class="bg-zinc-900 border border-zinc-800 rounded-xl p-5 flex flex-col justify-between hover:border-red-500/60 transition">
          <div>
            <div class="flex items-center justify-between gap-2 mb-2">
              <span class="text-xs font-bold uppercase tracking-wider bg-red-500/10 text-red-400 border border-red-500/20 px-2.5 py-0.5 rounded">{item.get('badge', 'Welding Gear')}</span>
              <span class="text-sm font-bold text-emerald-400">{item.get('price_range', '$69+')}</span>
            </div>
            <h3 class="text-base font-bold text-white mt-2">{item.get('title', '')}</h3>
            <p class="text-xs text-zinc-400 mt-2 leading-relaxed">🔥 {item.get('weld_note', '')}</p>
          </div>
          <a href="{amz_url}" target="_blank" rel="nofollow sponsored" class="mt-5 block text-center bg-red-500 hover:bg-red-400 text-white font-extrabold py-2.5 px-4 rounded-lg text-sm transition shadow-lg">
            Check Price on Amazon →
          </a>
        </div>"""

    # Build AWS Metal Thickness Reference Chart (20ga, 16ga, 1/8", 3/16", 1/4", 3/8")
    thickness_tiers = [
        ("18 Gauge (0.048\" / 1.2mm)", 50, "15.5V – 16.5V", "0.023\" Solid", "155 IPM", "20 CFH C25"),
        ("16 Gauge (0.060\" / 1.5mm)", 65, "16.5V – 17.5V", "0.023\" / 0.030\"", "180 IPM", "20 CFH C25"),
        ("1/8 Inch (0.125\" / 3.2mm)", 125, "18.5V – 19.5V", "0.030\" Solid", "220 IPM", "25 CFH C25"),
        ("3/16 Inch (0.188\" / 4.8mm)", 180, "20.5V – 21.5V", "0.030\" / 0.035\"", "285 IPM", "25 CFH C25"),
        ("1/4 Inch (0.250\" / 6.4mm)", 215, "22.5V – 24.0V", "0.035\" Solid", "340 IPM", "30 CFH C25"),
        ("3/8 Inch (0.375\" / 9.5mm)", 275, "25.0V – 27.0V", "0.035\" / 0.045\"", "410 IPM", "35 CFH C25")
    ]
    matrix_rows_html = ""
    for thk_s, amp_req, v_range, wire_s, ipm_s, gas_s in thickness_tiers:
        can_run = "✅ Single-Pass" if amp_req <= max_amps else "⚠️ Bevel / Multi-Pass"
        matrix_rows_html += f"""
        <tr class="border-b border-zinc-800 hover:bg-zinc-800/40">
          <td class="py-3 px-4 font-bold text-white">{thk_s}</td>
          <td class="py-3 px-4 font-bold text-red-400">{amp_req} Amps</td>
          <td class="py-3 px-4 text-zinc-300">{v_range}</td>
          <td class="py-3 px-4 text-emerald-400 font-semibold">{ipm_s} ({wire_s})</td>
          <td class="py-3 px-4 text-xs text-zinc-300">{can_run}</td>
        </tr>"""

    checklist_html = ""
    for idx, step_text in enumerate(data.get("fabrication_checklist", []), start=1):
        checklist_html += f"""
        <label class="flex items-start gap-3.5 bg-zinc-900 border border-zinc-800 rounded-xl p-4 cursor-pointer hover:border-zinc-700 transition">
          <input type="checkbox" class="mt-1 w-5 h-5 accent-red-500 rounded cursor-pointer weld-check">
          <span class="text-sm text-zinc-200 leading-relaxed"><strong>AWS Step {idx}:</strong> {step_text}</span>
        </label>"""

    faqs_html = "".join([
        f"<div class='bg-zinc-900 border border-zinc-800 rounded-xl p-5'><h3 class='font-bold text-white text-base mb-2'>{f.get('q', '')}</h3><p class='text-sm text-zinc-300 leading-relaxed'>{f.get('a', '')}</p></div>"
        for f in data.get("faqs", [])
    ])

    canonical_url = f"{SITE_BASE_URL}/{slug}.html"
    amz_domain = mon_cfg.get("marketplace_domain", "www.amazon.com")
    amz_tag = mon_cfg.get("amazon_associate_tag", "YOUR_AMAZON_TAG-20")

    html = f"""<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{data.get('seo_title', welder_name)}</title>
  <meta name="description" content="{data.get('meta_description', '')}">
  <link rel="canonical" href="{canonical_url}">
  <meta property="og:title" content="{data.get('seo_title', welder_name)}">
  <meta property="og:description" content="{data.get('meta_description', '')}">
  <meta property="og:type" content="article">
  <meta property="og:url" content="{canonical_url}">
  <script src="https://cdn.tailwindcss.com"></script>
  <script type="application/ld+json">{json.dumps(schema_graph)}</script>
</head>
<body class="bg-zinc-950 text-zinc-100 antialiased min-h-screen flex flex-col">
  <header class="border-b border-zinc-800 bg-zinc-900/80 backdrop-blur sticky top-0 z-30">
    <div class="max-w-5xl mx-auto px-4 py-3.5 flex items-center justify-between">
      <a href="index.html" class="text-lg font-black tracking-tight text-white flex items-center gap-2">
        <span class="bg-red-500 text-white px-2 py-0.5 rounded font-black">WELD</span>AmpIndex
      </a>
      <a href="{header_cta_url}" target="_blank" rel="nofollow sponsored" class="text-xs sm:text-sm bg-red-500 hover:bg-red-400 text-white font-extrabold px-3.5 py-2 rounded-lg transition">
        {header_cta.get('cta_text', 'Shop Multi-Process Welders →')}
      </a>
    </div>
  </header>

  <main class="max-w-5xl mx-auto px-4 py-10 flex-1 w-full">
    <div class="mb-8">
      <div class="flex flex-wrap items-center gap-2.5 mb-3">
        <span class="text-xs font-extrabold uppercase tracking-widest bg-red-500 text-white px-3 py-1 rounded-full">Max Output: {max_amps}A ({duty_pct}% Duty)</span>
        <span class="text-xs font-semibold bg-emerald-500/10 text-emerald-400 border border-emerald-500/20 px-3 py-1 rounded-full">{proc_type}</span>
        <span class="text-xs font-semibold bg-zinc-800 text-zinc-300 px-3 py-1 rounded-full">{input_volt} | {breaker_a}A Breaker | Max: {max_steel}</span>
      </div>
      <h1 class="text-3xl sm:text-4xl font-black text-white leading-tight">{data.get('seo_title', welder_name)}</h1>
      <div class="mt-4 bg-zinc-900 border-l-4 border-red-500 p-4 rounded-r-xl text-sm text-zinc-200 leading-relaxed">
        <strong>AWS CWI Welding Parameters Summary:</strong> {data.get('aws_verdict', '')}
      </div>
    </div>

    <!-- INTERACTIVE MIG/TIG/STICK AMPERAGE, VOLTAGE & WIRE FEED SPEED CALCULATOR -->
    <section class="bg-zinc-900 border border-zinc-800 rounded-2xl p-6 sm:p-8 mb-12 shadow-2xl">
      <h2 class="text-xl font-extrabold text-white mb-2">Interactive {welder_name} Dial-In Settings Calculator</h2>
      <p class="text-xs text-zinc-400 mb-6">Select your metal thickness, welding process, and wire/electrode diameter to calculate exact Amperage, Arc Voltage, Wire Feed Speed (IPM), and Shielding Gas flow (CFH).</p>

      <div class="grid md:grid-cols-2 gap-8 items-center">
        <div class="space-y-5">
          <div>
            <div class="flex justify-between text-sm font-medium mb-1.5">
              <label for="thickSlider" class="text-zinc-300">Metal Thickness (Thousandths of an Inch)</label>
              <span id="thickLabel" class="text-red-400 font-bold">1/8" (0.125" / 3.2mm)</span>
            </div>
            <input id="thickSlider" type="range" min="40" max="375" step="5" value="125" class="w-full accent-red-500 cursor-pointer">
          </div>

          <div class="grid grid-cols-2 gap-3">
            <div>
              <label for="procSelect" class="block text-xs font-medium text-zinc-400 mb-1">Welding Process & Metal</label>
              <select id="procSelect" class="w-full bg-zinc-950 border border-zinc-700 rounded-lg px-3 py-2 text-xs text-white focus:outline-none focus:border-red-500">
                <option value="mig_steel" selected>MIG Solid Wire (C25 Gas - Steel)</option>
                <option value="flux_steel">Flux-Core Gasless (E71T-GS - Steel)</option>
                <option value="tig_steel">DC TIG (100% Argon - Steel/Stainless)</option>
                <option value="tig_alum">AC TIG (100% Argon - 6061 Aluminum)</option>
                <option value="stick_7018">Stick SMAW (E7018 / E6011 Rod)</option>
              </select>
            </div>
            <div>
              <label for="wireSelect" class="block text-xs font-medium text-zinc-400 mb-1">Wire / Tungsten / Rod Size</label>
              <select id="wireSelect" class="w-full bg-zinc-950 border border-zinc-700 rounded-lg px-3 py-2 text-xs text-white focus:outline-none focus:border-red-500">
                <option value="0.023">0.023" (0.6mm) Wire / 1/16" Tungsten</option>
                <option value="0.030" selected>0.030" (0.8mm) Wire / 3/32" Tungsten</option>
                <option value="0.035">0.035" (0.9mm) Wire / 1/8" Rod</option>
                <option value="0.045">0.045" (1.2mm) Heavy Wire / 5/32" Rod</option>
              </select>
            </div>
          </div>
        </div>

        <div class="grid grid-cols-2 gap-4">
          <div class="bg-zinc-950 border border-red-500/30 rounded-xl p-4">
            <span class="text-xs uppercase text-red-400 font-bold">Target Amps & Volts</span>
            <div class="text-2xl sm:text-3xl font-black text-white mt-1"><span id="ampOut">125</span>A <span class="text-base font-bold text-red-400">@ <span id="voltOut">19.0</span>V</span></div>
            <p id="capStatusOut" class="text-xs text-emerald-400 font-semibold mt-1">✅ Within {max_amps}A Limit</p>
          </div>

          <div class="bg-zinc-950 border border-emerald-500/30 rounded-xl p-4">
            <span class="text-xs uppercase text-emerald-400 font-bold">Wire Speed & Gas</span>
            <div id="ipmOut" class="text-2xl sm:text-3xl font-black text-emerald-400 mt-1">215 <span class="text-sm font-normal text-zinc-400">IPM</span></div>
            <p id="gasOut" class="text-xs text-zinc-400 mt-1">Gas: 25 CFH (75/25 C25)</p>
          </div>
        </div>
      </div>

      <div class="mt-6 bg-red-950/40 border border-red-500/30 rounded-xl p-4 flex flex-col sm:flex-row items-center justify-between gap-4">
        <div id="weldAdviceBanner" class="text-sm text-red-100 font-medium"></div>
        <a id="dynamicWeldBtn" href="{header_cta_url}" target="_blank" rel="nofollow sponsored" class="shrink-0 bg-red-500 hover:bg-red-400 text-white font-extrabold px-4 py-2.5 rounded-lg text-xs uppercase tracking-wider transition">
          Shop Matched Welding Consumables on Amazon →
        </a>
      </div>
    </section>

    <!-- 4-CARD HARDWARE BUNDLE -->
    <section class="mb-12">
      <div class="flex items-center justify-between mb-4">
        <h2 class="text-2xl font-extrabold text-white">Complete {welder_name} Shop Setup & Consumable Kit</h2>
        <span class="text-xs text-zinc-400">AWS D1.1 & NEC Article 630 Rated</span>
      </div>
      <div class="grid sm:grid-cols-2 lg:grid-cols-4 gap-4">{bundle_html}</div>
    </section>

    <!-- METAL THICKNESS & SETTINGS CHART TABLE -->
    <section class="mb-12">
      <h2 class="text-2xl font-extrabold text-white mb-4">{welder_name} Mild Steel MIG Reference Chart (75/25 Argon/CO2)</h2>
      <div class="overflow-x-auto bg-zinc-900 border border-zinc-800 rounded-xl">
        <table class="w-full text-left border-collapse text-sm">
          <thead>
            <tr class="border-b border-zinc-800 bg-zinc-950/60 text-zinc-400 uppercase text-xs">
              <th class="py-3.5 px-4">Steel Thickness</th>
              <th class="py-3.5 px-4 text-red-400">Target Amperage</th>
              <th class="py-3.5 px-4">Arc Voltage Range</th>
              <th class="py-3.5 px-4 text-emerald-400">Wire Feed Speed (IPM)</th>
              <th class="py-3.5 px-4">Machine Capability</th>
            </tr>
          </thead>
          <tbody>{matrix_rows_html}</tbody>
        </table>
      </div>
    </section>

    <!-- FABRICATION CHECKLIST -->
    <section class="mb-12">
      <div class="flex items-center justify-between mb-4">
        <h2 class="text-2xl font-extrabold text-white">AWS CWI Pre-Weld Polarity & Setup Checklist</h2>
        <span id="weldBadge" class="text-xs font-bold bg-zinc-800 text-red-400 px-3 py-1 rounded-full">0 / 4 Completed</span>
      </div>
      <div class="space-y-3">{checklist_html}</div>
    </section>

    <!-- FAQ -->
    <section class="mb-12">
      <h2 class="text-2xl font-extrabold text-white mb-5">Frequently Asked Questions ({welder_name})</h2>
      <div class="space-y-4">{faqs_html}</div>
    </section>
  </main>

  <footer class="border-t border-zinc-800 py-8 text-center text-xs text-zinc-500 px-4">
    <p>© {datetime.now(timezone.utc).year} WeldAmpIndex. Independent AWS Welding Parameters & Electrical Sizing Database. As an Amazon Associate, we earn from qualifying purchases.</p>
  </footer>

  <script>
    const machineMaxAmps = {max_amps};
    const amzDomain = "{amz_domain}";
    const amzTag = "{amz_tag}";

    const thickSlider = document.getElementById('thickSlider');
    const procSelect = document.getElementById('procSelect');
    const wireSelect = document.getElementById('wireSelect');
    const thickLabel = document.getElementById('thickLabel');
    const ampOut = document.getElementById('ampOut');
    const voltOut = document.getElementById('voltOut');
    const capStatusOut = document.getElementById('capStatusOut');
    const ipmOut = document.getElementById('ipmOut');
    const gasOut = document.getElementById('gasOut');
    const weldAdviceBanner = document.getElementById('weldAdviceBanner');
    const dynamicWeldBtn = document.getElementById('dynamicWeldBtn');

    function formatThicknessLabel(thou) {{
      const mm = (thou * 0.0254).toFixed(1);
      if (thou <= 50) return `18 Gauge (0.${{thou.toString().padStart(3, '0')}}" / ${{mm}}mm)`;
      if (thou <= 65) return `16 Gauge (0.${{thou.toString().padStart(3, '0')}}" / ${{mm}}mm)`;
      if (thou <= 80) return `14 Gauge (0.${{thou.toString().padStart(3, '0')}}" / ${{mm}}mm)`;
      if (thou <= 110) return `12 Gauge (0.${{thou}}"" / ${{mm}}mm)`;
      if (thou <= 135) return `1/8" Plate (0.${{thou}}" / ${{mm}}mm)`;
      if (thou <= 195) return `3/16" Plate (0.${{thou}}" / ${{mm}}mm)`;
      if (thou <= 260) return `1/4" Plate (0.${{thou}}" / ${{mm}}mm)`;
      return `3/8" Heavy Plate (0.${{thou}}" / ${{mm}}mm)`;
    }}

    function recalculateWeld() {{
      const thou = parseInt(thickSlider.value, 10);
      const proc = procSelect.value;
      const wireDia = wireSelect.value;

      thickLabel.textContent = formatThicknessLabel(thou);

      let calcAmps = thou;
      if (proc === 'tig_alum') calcAmps = Math.round(thou * 1.15);
      else if (proc === 'tig_steel') calcAmps = Math.round(thou * 0.90);
      else if (proc === 'flux_steel') calcAmps = Math.round(thou * 0.95);

      const calcVolts = (14.0 + (calcAmps * 0.042)).toFixed(1);

      let ipmMult = 1.70;
      if (wireDia === '0.023') ipmMult = 3.10;
      else if (wireDia === '0.030') ipmMult = 1.75;
      else if (wireDia === '0.035') ipmMult = 1.35;
      else ipmMult = 0.85;

      const calcIpm = Math.min(650, Math.round(calcAmps * ipmMult));

      ampOut.textContent = calcAmps;
      voltOut.textContent = calcVolts;

      if (calcAmps <= machineMaxAmps) {{
        capStatusOut.textContent = `✅ Single-Pass OK (${{calcAmps}}A / ${{machineMaxAmps}}A Max)`;
        capStatusOut.className = "text-xs text-emerald-400 font-semibold mt-1";
      }} else {{
        capStatusOut.textContent = `⚠️ Needs Bevel & Multi-Pass (${{machineMaxAmps}}A Max)`;
        capStatusOut.className = "text-xs text-amber-400 font-bold mt-1";
      }}

      let searchItem = "ER70S-6+030+MIG+Welding+Wire+Spool";
      if (proc === 'mig_steel') {{
        ipmOut.innerHTML = `${{calcIpm}} <span class="text-sm font-normal text-zinc-400">IPM</span>`;
        gasOut.textContent = "DCEP | 25 CFH (75% Argon / 25% CO2)";
        searchItem = `ER70S-6+${{wireDia.replace('0.', '')}}+MIG+Welding+Wire+Spool`;
        weldAdviceBanner.innerHTML = `🔥 <strong>MIG Setup Spec:</strong> Run <strong>DCEP Polarity</strong> with <strong>${{wireDia}}" ER70S-6 Solid Wire</strong> at <strong>${{calcVolts}}V / ${{calcIpm}} IPM</strong> and 25 CFH C25 gas.`;
      }} else if (proc === 'flux_steel') {{
        ipmOut.innerHTML = `${{calcIpm}} <span class="text-sm font-normal text-zinc-400">IPM</span>`;
        gasOut.textContent = "DCEN Polarity | No Gas Required";
        searchItem = `E71T-GS+${{wireDia.replace('0.', '')}}+Gasless+Flux+Core+Welding+Wire`;
        weldAdviceBanner.innerHTML = `🔥 <strong>Flux-Core Spec:</strong> Switch welder lugs to <strong>DCEN (Electrode Negative)</strong> with <strong>${{wireDia}}" E71T-GS Wire</strong> at <strong>${{calcVolts}}V / ${{calcIpm}} IPM</strong>.`;
      }} else if (proc === 'tig_steel') {{
        ipmOut.innerHTML = `3/32" <span class="text-sm font-normal text-zinc-400">2% Lanthanated</span>`;
        gasOut.textContent = "DCEN | 18 CFH (100% Pure Argon)";
        searchItem = "3/32+2+Percent+Lanthanated+Tungsten+Electrodes+ER70S-2";
        weldAdviceBanner.innerHTML = `🔥 <strong>DC TIG Spec:</strong> Use <strong>DCEN Polarity</strong>, a sharp <strong>3/32" Blue 2% Lanthanated Tungsten</strong>, #7 Gas Lens Cup, and 18 CFH Pure Argon.`;
      }} else if (proc === 'tig_alum') {{
        ipmOut.innerHTML = `3/32" <span class="text-sm font-normal text-zinc-400">AC Balance 70%</span>`;
        gasOut.textContent = "AC High-Freq | 22 CFH (100% Argon)";
        searchItem = "ER4043+ER5356+TIG+Aluminum+Filler+Rod+Tungsten";
        weldAdviceBanner.innerHTML = `🔥 <strong>AC Aluminum TIG Spec:</strong> Set <strong>AC Frequency to 120Hz</strong>, <strong>AC Balance to 70% EN</strong>, and feed <strong>3/32" ER4043 / ER5356 Filler Rod</strong>.`;
      }} else {{
        ipmOut.innerHTML = `1/8" <span class="text-sm font-normal text-zinc-400">E7018 / E6011</span>`;
        gasOut.textContent = "DCEP | Flux-Coated SMAW Electrode";
        searchItem = "E7018+1/8+Stick+Welding+Rods+Electrode+Oven";
        weldAdviceBanner.innerHTML = `🔥 <strong>Stick SMAW Spec:</strong> Run <strong>DCEP Polarity</strong> at <strong>${{calcAmps}} Amps</strong> with dry <strong>E7018 Low-Hydrogen</strong> or deep-penetrating <strong>E6011 Rods</strong>.`;
      }}

      dynamicWeldBtn.href = `https://${{amzDomain}}/s?k=${{searchItem}}&tag=${{amzTag}}`;
    }}

    thickSlider.addEventListener('input', recalculateWeld);
    procSelect.addEventListener('change', recalculateWeld);
    wireSelect.addEventListener('change', recalculateWeld);
    recalculateWeld();

    const checks = document.querySelectorAll('.weld-check');
    const badge = document.getElementById('weldBadge');
    checks.forEach(ch => {{
      ch.addEventListener('change', () => {{
        const done = Array.from(checks).filter(c => c.checked).length;
        badge.textContent = `${{done}} / ${{checks.length}} Completed`;
      }});
    }});
  </script>
</body>
</html>"""

    os.makedirs(DOCS_DIR, exist_ok=True)
    with open(os.path.join(DOCS_DIR, f"{slug}.html"), "w", encoding="utf-8") as f:
        f.write(html)


def rebuild_portal_sitemap_and_rss(published, mon_cfg):
    header_cta = mon_cfg.get("header_cta", {})
    header_cta_url = build_amazon_url(header_cta.get("search_query", "Multi+Process+MIG+TIG+Stick+Welder"), mon_cfg)

    cards_html = ""
    for item in reversed(published):
        cards_html += f"""
        <a href="{item['slug']}.html" class="block bg-zinc-900 border border-zinc-800 hover:border-red-500/60 rounded-xl p-5 transition group">
          <div class="flex items-center justify-between mb-2">
            <span class="text-xs font-black uppercase bg-red-500 text-white px-2.5 py-0.5 rounded">{item.get('max_amps', 200)}A Max | {item.get('duty_pct', 35)}% Duty</span>
            <span class="text-xs text-zinc-400">{item.get('breaker_a', 30)}A Breaker</span>
          </div>
          <h2 class="text-base font-bold text-white group-hover:text-red-400 transition">{item['welder_name']} Settings Calculator</h2>
          <p class="text-xs text-zinc-400 mt-2 line-clamp-2">{item['description']}</p>
          <span class="inline-block mt-4 text-xs font-extrabold text-red-400">Open Wire Speed & Amperage Chart →</span>
        </a>"""

    index_html = f"""<!DOCTYPE html>
<html lang="en" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>WeldAmpIndex — MIG/TIG/Stick Welder Settings, Wire Speed & Breaker Calculator (2026)</title>
  <meta name="description" content="Interactive welding amperage, wire feed speed (IPM), arc voltage, and 240V NEMA 6-50 breaker calculators for Miller, Lincoln, Hobart, YesWelder, PrimeWeld, and ESAB machines.">
  <link rel="canonical" href="{SITE_BASE_URL}/">
  <link rel="alternate" type="application/rss+xml" title="WeldAmpIndex RSS Feed" href="{SITE_BASE_URL}/rss.xml">
  <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-zinc-950 text-zinc-100 min-h-screen flex flex-col">
  <header class="border-b border-zinc-800 bg-zinc-900/80 sticky top-0 z-30 backdrop-blur">
    <div class="max-w-5xl mx-auto px-4 py-3.5 flex items-center justify-between">
      <div class="text-lg font-black text-white flex items-center gap-2">
        <span class="bg-red-500 text-white px-2 py-0.5 rounded font-black">WELD</span>AmpIndex
      </div>
      <a href="{header_cta_url}" target="_blank" rel="nofollow sponsored" class="text-xs sm:text-sm bg-red-500 hover:bg-red-400 text-white font-extrabold px-3.5 py-2 rounded-lg transition">
        {header_cta.get('cta_text', 'Shop Multi-Process Welders →')}
      </a>
    </div>
  </header>

  <main class="max-w-5xl mx-auto px-4 py-12 flex-1 w-full">
    <div class="text-center max-w-2xl mx-auto mb-10">
      <h1 class="text-3xl sm:text-5xl font-black text-white tracking-tight">MIG, TIG & Stick Welder Settings & Circuit Database</h1>
      <p class="text-zinc-400 mt-3 text-base sm:text-lg">Select your welder model or metal thickness below to dial in exact Amperage, Wire Feed Speed (IPM), Arc Voltage, Shielding Gas CFH, and NEC 240V breaker sizing.</p>
      <div class="mt-6">
        <input id="filterInput" type="text" placeholder="Search welder or metal (e.g. Millermatic 211, Lincoln 215, YesWelder 205, Hobart 140, 1/4 Steel...)" class="w-full bg-zinc-900 border border-zinc-700 rounded-xl px-4 py-3.5 text-sm text-white focus:outline-none focus:border-red-500">
      </div>
    </div>

    <div id="weldGrid" class="grid sm:grid-cols-2 lg:grid-cols-3 gap-5">{cards_html}</div>
  </main>

  <footer class="border-t border-zinc-800 py-8 text-center text-xs text-zinc-500">
    <p>© {datetime.now(timezone.utc).year} WeldAmpIndex. As an Amazon Associate, we earn from qualifying purchases.</p>
  </footer>

  <script>
    const filterInput = document.getElementById('filterInput');
    const cards = Array.from(document.getElementById('weldGrid').children);
    filterInput.addEventListener('input', (e) => {{
      const q = e.target.value.toLowerCase();
      cards.forEach(c => {{
        c.style.display = c.textContent.toLowerCase().includes(q) ? 'block' : 'none';
      }});
    }});
  </script>
</body>
</html>"""

    os.makedirs(DOCS_DIR, exist_ok=True)
    with open(os.path.join(DOCS_DIR, "index.html"), "w", encoding="utf-8") as f:
        f.write(index_html)

    today = datetime.now(timezone.utc).strftime("%Y-%m-%d")
    urls_xml = f"""  <url>
    <loc>{SITE_BASE_URL}/</loc>
    <lastmod>{today}</lastmod>
    <changefreq>daily</changefreq>
    <priority>1.0</priority>
  </url>\n"""

    for item in published:
        urls_xml += f"""  <url>
    <loc>{SITE_BASE_URL}/{item['slug']}.html</loc>
    <lastmod>{item.get('date', today)}</lastmod>
    <changefreq>monthly</changefreq>
    <priority>0.9</priority>
  </url>\n"""

    with open(os.path.join(DOCS_DIR, "sitemap.xml"), "w", encoding="utf-8") as f:
        f.write(f"""<?xml version="1.0" encoding="UTF-8"?>\n<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">\n{urls_xml}</urlset>""")

    rss_items = ""
    for item in list(reversed(published))[:30]:
        rss_items += f"""    <item>
      <title>{item['title']}</title>
      <link>{SITE_BASE_URL}/{item['slug']}.html</link>
      <guid>{SITE_BASE_URL}/{item['slug']}.html</guid>
      <description>{item['description']}</description>
    </item>\n"""

    rss_xml = f"""<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
  <channel>
    <title>WeldAmpIndex AWS Welding Settings & Circuit Database</title>
    <link>{SITE_BASE_URL}/</link>
    <description>Interactive MIG, TIG & Stick Welder Amperage, Wire Feed Speed & Breaker Calculators</description>
    <atom:link href="{SITE_BASE_URL}/rss.xml" rel="self" type="application/rss+xml" />
    <atom:link href="https://pubsubhubbub.appspot.com/" rel="hub" />
{rss_items}  </channel>
</rss>"""

    with open(os.path.join(DOCS_DIR, "rss.xml"), "w", encoding="utf-8") as f:
        f.write(rss_xml)

    with open(os.path.join(DOCS_DIR, "robots.txt"), "w", encoding="utf-8") as f:
        f.write(f"User-agent: *\nAllow: /\nSitemap: {SITE_BASE_URL}/sitemap.xml\n")

    with open(os.path.join(DOCS_DIR, f"{INDEXNOW_KEY}.txt"), "w", encoding="utf-8") as f:
        f.write(INDEXNOW_KEY)


def notify_search_engines(new_urls):
    if not SITE_BASE_URL or not new_urls:
        return
    parsed_host = urllib.parse.urlparse(SITE_BASE_URL).netloc
    payload = {
        "host": parsed_host,
        "key": INDEXNOW_KEY,
        "keyLocation": f"{SITE_BASE_URL}/{INDEXNOW_KEY}.txt",
        "urlList": new_urls
    }
    try:
        req = urllib.request.Request(
            "https://api.indexnow.org/indexnow",
            data=json.dumps(payload).encode("utf-8"),
            headers={"Content-Type": "application/json; charset=utf-8"},
            method="POST"
        )
        with urllib.request.urlopen(req, timeout=15) as resp:
            print(f"[INDEXNOW] Pushed {len(new_urls)} URLs (HTTP {resp.status})")
    except Exception as e:
        print(f"[WARN] IndexNow ping notice: {e}")

    try:
        hub_data = urllib.parse.urlencode({
            "hub.mode": "publish",
            "hub.url": f"{SITE_BASE_URL}/rss.xml"
        }).encode("utf-8")
        req_hub = urllib.request.Request(
            "https://pubsubhubbub.appspot.com/",
            data=hub_data,
            headers={"Content-Type": "application/x-www-form-urlencoded"},
            method="POST"
        )
        with urllib.request.urlopen(req_hub, timeout=15) as resp_hub:
            print(f"[WEBSUB] Google PubSubHubbub ping status: {resp_hub.status}")
    except Exception as e:
        print(f"[WARN] WebSub ping notice: {e}")


def main():
    published = load_json(PUBLISHED_FILE, [])
    mon_cfg = load_json(MONETIZATION_FILE, {})
    published_slugs = {item["slug"] for item in published}

    candidates = []
    if CUSTOM_WELDER:
        candidates.append((CUSTOM_WELDER.strip(), 200, 35, "120V / 240V Dual", 30, "MIG / TIG / Stick"))
    candidates.extend(SEED_WELDERS)
    candidates.extend(discover_google_welder_queries())

    new_urls = []
    count = 0

    for weld_tuple in candidates:
        if count >= PAGES_PER_RUN:
            break
        w_name = weld_tuple[0]
        slug = slugify(f"{w_name}-settings-wire-speed-calculator")
        if slug in published_slugs:
            continue

        print(f"[ENGINE] Generating AWS welding settings & circuit calculator for: {w_name}...")
        try:
            data = generate_welder_data(weld_tuple)
            build_welder_html_page(data, slug, mon_cfg)
            today = datetime.now(timezone.utc).strftime("%Y-%m-%d")
            sp = data.get("specs", {})

            published.append({
                "welder_name": data.get("welder_name", w_name),
                "slug": slug,
                "title": data.get("seo_title", f"{w_name} Settings Calculator"),
                "description": data.get("meta_description", ""),
                "max_amps": int(safe_float(sp.get("max_output_amps", weld_tuple[1]), weld_tuple[1])),
                "duty_pct": int(safe_float(sp.get("duty_cycle_pct", weld_tuple[2]), weld_tuple[2])),
                "breaker_a": int(safe_float(sp.get("recommended_breaker_amps", weld_tuple[4]), weld_tuple[4])),
                "date": today
            })
            published_slugs.add(slug)
            if SITE_BASE_URL:
                new_urls.append(f"{SITE_BASE_URL}/{slug}.html")
            count += 1
            time.sleep(2)
        except Exception as e:
            print(f"[ERROR] Failed building {w_name}: {e}")

    save_json(PUBLISHED_FILE, published)
    rebuild_portal_sitemap_and_rss(published, mon_cfg)
    if new_urls and SITE_BASE_URL:
        new_urls.append(f"{SITE_BASE_URL}/")
        notify_search_engines(new_urls)
    print(f"[DONE] Published {count} new welder settings calculator pages.")


if __name__ == "__main__":
    main()
