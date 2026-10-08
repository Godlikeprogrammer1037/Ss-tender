name: SS Bar Tenders (daily update)
on:
  schedule:
    - cron: "30 0 * * *"   # 6:00 AM IST every day
  workflow_dispatch:
  push:
    paths: [".github/workflows/daily.yml"]
permissions:
  contents: write
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
      - run: pip install requests
      - name: Build data and website
        run: |
          python - <<'PYEOF'
          import json, re, datetime, os, requests
          BASE = "https://bidplus.gem.gov.in"
          KEYWORDS = ["stainless steel bright bar","stainless steel black bar","SS bright bar","AISI 304 bar","AISI 316 bar","SS 410 bar","SS 420 bar","SS 431 bar","SS 304 round bar"]
          GRADE = re.compile(r"\b(3[0-4][0-9]|4[0-4][0-9])[A-Z]{0,2}\b")
          SS = re.compile(r"stainless|\bss\b|\bss ?\d|aisi", re.I)
          BAR = re.compile(r"\b(bar|bars|rod|rods)\b", re.I)
          SEED = [
          {"bid_no":"GEM/2026/R/696759","item":"Supply of SS 304H and SS 321H stainless steel round bars (50 mm dia)","buyer":"JV of CIL, NTPC, IOCL, FCIL & HFCL (Dept. of Fertilizers)","qty":"52","end":"2026-07-15"},
          {"bid_no":"GEM/2026/B/7549231","item":"AISI 304L stainless steel rods, various diameters (VSSC)","buyer":"Indian Space Research Organisation, Kerala","qty":"33500","end":"2026-06-09"},
          {"bid_no":"GEM/2026/B/7397664","item":"Stainless steel bars (austenitic, Ti stabilised), solution annealed, 70x40 mm","buyer":"BHEL Haridwar","qty":"","end":"2026-04-18"},
          {"bid_no":"GEM/2025/B/7028107","item":"AISI 304 black bars and pipes (HMT raw material)","buyer":"HMT","qty":"","end":"2026-01-01"}]
          def first(v):
              return v[0] if isinstance(v, list) and v else (v or "")
          def fetch(kw, page):
              s = requests.Session(); s.headers["User-Agent"] = "Mozilla/5.0"
              s.get(BASE + "/all-bids", timeout=30)
              csrf = s.cookies.get("csrf_cookie_name") or s.cookies.get("csrf_bd_gem_nk", "")
              p = {"page": page, "param": {"searchBid": kw, "searchType": "fullText"},
                   "filter": {"bidStatusType": "ongoing_bids", "byType": "all", "highBidValue": "", "byEndDate": {"from": "", "to": ""}, "sort": "Bid-Start-Date-Latest"}}
              r = s.post(BASE + "/all-bids-data", data={"payload": json.dumps(p), "csrf_bd_gem_nk": csrf}, timeout=60)
              return r.json()["response"]["response"]["docs"]
          found = {}
          for kw in KEYWORDS:
              for page in (1, 2):
                  try:
                      docs = fetch(kw, page)
                  except Exception as e:
                      print("skip", kw, page, e); break
                  for d in docs:
                      no = first(d.get("b_bid_number")); item = str(first(d.get("b_category_name")))
                      if no and SS.search(item) and BAR.search(item) and GRADE.search(item):
                          found[no] = {"bid_no": no, "item": item,
                              "buyer": first(d.get("ba_official_details_deptName")) or first(d.get("ba_official_details_minName")),
                              "qty": str(first(d.get("b_total_quantity"))), "end": str(first(d.get("final_end_date_sort")))[:10]}
          try:
              old = json.load(open("data.json"))["bids"]
          except Exception:
              old = SEED
          for b in old:
              found.setdefault(b["bid_no"], b)
          bids = sorted(found.values(), key=lambda b: b.get("end", ""), reverse=True)
          json.dump({"updated": datetime.date.today().isoformat(), "bids": bids}, open("data.json", "w"), indent=1, ensure_ascii=False)
          HTML = r'''<!DOCTYPE html>
          <html lang="en"><head><meta charset="utf-8"><meta name="viewport" content="width=device-width, initial-scale=1">
          <title>SS Bar Tenders - GeM</title>
          <style>
          :root{--bg:#eef1f4;--card:#fff;--ink:#17202b;--mute:#5d6b7a;--line:#d3dae1;--accent:#2b5f8e;--hot:#b4531a}
          @media (prefers-color-scheme:dark){:root{--bg:#11171e;--card:#1a232d;--ink:#e8edf2;--mute:#9aa8b6;--line:#2c3844;--accent:#6fa6d6;--hot:#e0894f}}
          *{box-sizing:border-box}body{margin:0;background:var(--bg);color:var(--ink);font:16px/1.5 system-ui,sans-serif}
          main{max-width:820px;margin:0 auto;padding:20px 16px 40px}h1{font:700 1.6rem Georgia,serif;margin:0 0 4px}
          .sub{color:var(--mute);margin:0 0 14px}input,select{font:inherit;color:var(--ink);background:var(--card);border:1px solid var(--line);border-radius:6px;padding:9px 10px}
          .bar{display:flex;flex-wrap:wrap;gap:8px;margin-bottom:12px}input{flex:1 1 200px}
          .item{background:var(--card);border:1px solid var(--line);border-left:5px solid var(--accent);border-radius:6px;padding:12px 14px;margin-bottom:10px}
          .item.closed{opacity:.6;border-left-color:var(--mute)}.item.open{border-left-color:var(--hot)}
          .t{font-weight:600;margin:0}.m{font-size:.88rem;color:var(--mute);margin-top:6px;display:flex;flex-wrap:wrap;gap:4px 14px}.m b{color:var(--ink)}
          .tag{font-size:.75rem;padding:2px 8px;border-radius:999px;background:var(--hot);color:#fff;margin-left:6px}.closed .tag{background:var(--mute)}
          </style></head><body><main>
          <h1>Stainless Steel Bar Tenders</h1>
          <p class="sub">Bright / black bars, 300 and 400 series. GeM, all India. Updated daily: <span id="u">...</span></p>
          <div class="bar"><input id="q" type="search" placeholder="Search grade, buyer, item" aria-label="Search">
          <select id="f"><option value="open">Open bids only</option><option value="all">Open + closed</option></select></div>
          <p class="sub" id="c"></p><div id="l"></div>
          <script>
          var B=[],today=new Date().toISOString().slice(0,10);
          function esc(s){return String(s||"").replace(/[&<>"]/g,function(c){return{"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;"}[c]})}
          function draw(){var q=document.getElementById("q").value.toLowerCase(),f=document.getElementById("f").value;
          var r=B.filter(function(b){var o=!b.end||b.end>=today;return (f==="all"||o)&&(b.item+b.buyer+b.bid_no).toLowerCase().indexOf(q)>-1});
          document.getElementById("c").textContent=r.length+" bid(s)";
          document.getElementById("l").innerHTML=r.length?r.map(function(b){var o=!b.end||b.end>=today;
          return '<div class="item '+(o?'open':'closed')+'"><p class="t">'+esc(b.item)+'<span class="tag">'+(o?'Open':'Closed')+'</span></p><div class="m"><span>Bid: <b>'+esc(b.bid_no)+'</b></span><span>Buyer: <b>'+esc(b.buyer)+'</b></span>'+(b.qty?'<span>Qty: <b>'+esc(b.qty)+'</b></span>':'')+(b.end?'<span>Last date: <b>'+esc(b.end)+'</b></span>':'')+'</div></div>'}).join(""):'<p class="sub">No open bids right now. Choose "Open + closed" to see past ones.</p>'}
          fetch("data.json").then(function(r){return r.json()}).then(function(d){B=d.bids;document.getElementById("u").textContent=d.updated;draw()});
          document.getElementById("q").oninput=draw;document.getElementById("f").onchange=draw;
          </script></main></body></html>'''
          open("index.html", "w", encoding="utf-8").write(HTML)
          print(len(bids), "bids saved")
          PYEOF
      - name: Save changes
        run: |
          git config user.name "bot"
          git config user.email "bot@users.noreply.github.com"
          git add data.json index.html
          git commit -m "Daily update" || echo "no changes"
          git push
          
