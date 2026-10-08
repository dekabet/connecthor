# ⚡ ConnecTHOR

Pentagon Pizza Index + geopolitical prediction-market intel on a 3D globe, in a single static `index.html`. Inspired by [PizzINT](https://www.pizzint.watch/).

## Features

- **🍕 Pentagon Pizza Watch** - "Popular Times"-style busyness charts for pizza shops near the Pentagon, each scored against its usual level for that hour (QUIET / NOMINAL / BUSY / SPIKE)
- **DOUGHCON level** - a DEFCON-style 5→1 alert level driven by the combined pizza index, with a 24-hour history sparkline
- **📉 Nothing Ever Happens Index** - 0-100 tension score from the volume-weighted odds of conflict/escalation markets (ceasefire markets inverted), with the top driving markets
- **Interactive 3D Globe** - Polymarket markets, Pentagon beacon whose pulse speeds up with DOUGHCON, pizza shops, arcs to hotspots
- **Intel Feed** - live world news (BBC, NYT, Al Jazeera, NPR, Guardian via RSS) tagged MILITARY / DIPLOMACY / ECONOMY, plus a breaking ticker
- **Markets tab** - live Polymarket odds (falls back to sample markets if the API is unreachable)

### About the pizza data

Google Maps Popular Times has no public API and can't be read from a static page, so out of the box the pizza busyness is **modeled**: typical hourly curves per shop plus deterministic noise and occasional correlated surges (every viewer sees the same values at the same time). The panel is badged `MODELED` while this is the case.

To use real data, run your own collector and set `PIZZA_DATA_URL` in `index.html` to a JSON endpoint returning:

```json
{"updated":"2026-10-08T20:00:00Z","shops":[{"id":"dominos-pc","live":72,"typical":[0,0,0,0,0,0,0,0,0,0,10,25,45,40,30,28,35,55,70,72,60,45,30,15]}]}
```

`id` must match an entry in `SHOPS`; `typical` is 24 hourly values (0-100, ET).

## 🚀 Deploy to GitHub Pages

### Quick Setup (2 minutes)

1. **Create a new repository** on GitHub
   - Go to [github.com/new](https://github.com/new)
   - Name it `polyglobe` (or any name)
   - Make it **Public**
   - Click "Create repository"

2. **Upload the files**
   ```bash
   git clone https://github.com/YOUR_USERNAME/polyglobe.git
   cd polyglobe
   # Copy index.html to this folder
   git add .
   git commit -m "Initial Polyglobe deployment"
   git push origin main
   ```

3. **Enable GitHub Pages**
   - Go to your repo → **Settings** → **Pages**
   - Under "Source", select **main** branch
   - Click **Save**
   - Wait 1-2 minutes for deployment

4. **Access your site**
   ```
   https://YOUR_USERNAME.github.io/polyglobe/
   ```

### Alternative: GitHub UI Upload

1. Create repo on GitHub
2. Click "Add file" → "Upload files"
3. Drag `index.html` into the upload area
4. Click "Commit changes"
5. Enable Pages in Settings

## 📊 API Integration

### Polymarket Gamma API

The app can fetch live data from Polymarket's public API:

```javascript
// Fetch active geopolitical markets
fetch('https://gamma-api.polymarket.com/events?tag=politics&closed=false&limit=50')
  .then(res => res.json())
  .then(data => {
    // Process markets
    data.forEach(event => {
      console.log(event.title, event.markets);
    });
  });
```

### Key Endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /events` | List events with markets |
| `GET /markets` | Individual market details |
| `GET /markets?slug={slug}` | Get market by slug |

### CORS Note

For production use, you'll need a CORS proxy or backend service to fetch Polymarket data, as the API doesn't support browser CORS.

Options:
- Use a serverless function (Vercel, Netlify Functions)
- Set up a simple proxy with Cloudflare Workers
- Use [cors-anywhere](https://github.com/Rob--W/cors-anywhere)

## 🛠️ Customization

### Add Your Own Markets

Edit the `MARKETS` array in `index.html`:

```javascript
const SAMPLE_MARKETS = [
  {
    id: 'unique-id',
    question: 'Your market question?',
    outcomePrices: '0.65,0.35',  // YES,NO probabilities
    volume: '1000000',
    slug: 'polymarket-slug',
    endDate: '2026-12-31'
  },
  // ... more markets
];
```

### Add Location Keywords

Expand the `LOCS` object to map new keywords:

```javascript
const LOCATIONS = {
  // ... existing locations
  'your_keyword': { lat: 40.7128, lng: -74.0060, country: 'New York' },
};
```

### Customize Colors

Modify the color logic in `getColor`:

```javascript
const getColor = (market) => {
  if (market.yesPrice >= 0.7) return '#22c55e';  // Green
  if (market.yesPrice >= 0.4) return '#eab308';  // Yellow  
  return '#ef4444';  // Red
};
```

## 📁 Project Structure

```
polyglobe/
├── index.html      # Complete standalone app
└── README.md       # This file
```

## 🔧 Technology Stack

- **React 18** - UI framework (via CDN)
- **globe.gl** - 3D WebGL globe visualization
- **Three.js** - 3D rendering engine
- **Babel** - JSX transformation (browser)

## 📜 License

MIT License - Feel free to use and modify.

## 🙏 Credits

- [Polymarket](https://polymarket.com) - Prediction market data
- [globe.gl](https://globe.gl) - Globe visualization library
- [Three.js](https://threejs.org) - 3D graphics
- Inspired by [PizzINT Polyglobe](https://pizzint.watch/polyglobe)

---

**Disclaimer:** This is an educational project. Trading on prediction markets involves financial risk. Always do your own research.
