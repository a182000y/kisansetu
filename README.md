# kisansetu
import { useState } from "react";
import {
  Leaf, TrendingUp, TrendingDown, Phone, Search, ShoppingBag,
  X, Mic, MapPin, CheckCircle2, ArrowRight,
} from "lucide-react";

// ---------- Data ----------
// Same 5 reference mandis for every crop, already sorted nearest → farthest.
const MANDIS = [
  { name: "Pimpalgaon Baswant", distance: 9 },
  { name: "Yeola", distance: 14 },
  { name: "Lasalgaon", distance: 18 },
  { name: "Nashik (Panchavati)", distance: 22 },
  { name: "Manmad", distance: 35 },
];

// Non-perishables listed first on purpose — the whole app leans toward them.
const CROPS = [
  { id: "wheat", name: "Wheat", category: "Grains", perishable: false, icon: "🌾", bestPriceLastYear: 2510, prices: [2330, 2350, 2360, 2410, 2380] },
  { id: "soybean", name: "Soybean", category: "Oilseeds", perishable: false, icon: "🌱", bestPriceLastYear: 5100, prices: [4590, 4610, 4650, 4680, 4720] },
  { id: "chana", name: "Chana (Gram)", category: "Pulses", perishable: false, icon: "🫘", bestPriceLastYear: 6100, prices: [5680, 5720, 5750, 5810, 5860] },
  { id: "cotton", name: "Cotton", category: "Cash Crop", perishable: false, icon: "☁️", bestPriceLastYear: 7550, prices: [6980, 7020, 7080, 7150, 7210] },
  { id: "turmeric", name: "Turmeric", category: "Spices", perishable: false, icon: "🟡", bestPriceLastYear: 10200, prices: [9250, 9380, 9460, 9580, 9700] },
  { id: "jowar", name: "Jowar", category: "Grains", perishable: false, icon: "🌾", bestPriceLastYear: 3150, prices: [2830, 2870, 2900, 2950, 3010] },
  { id: "onion", name: "Onion", category: "Perishable", perishable: true, icon: "🧅", bestPriceLastYear: 2200, prices: [1780, 1810, 1850, 1720, 1690] },
  { id: "tomato", name: "Tomato", category: "Perishable", perishable: true, icon: "🍅", bestPriceLastYear: 1900, prices: [1120, 1180, 1260, 1340, 1090] },
  { id: "grapes", name: "Grapes", category: "Perishable", perishable: true, icon: "🍇", bestPriceLastYear: 4850, prices: [3950, 3880, 4050, 4200, 3700] },
];

const INITIAL_LISTINGS = [
  { id: "wheat-1", cropId: "wheat", farmer: "Ramesh Patil", village: "Niphad", distance: 22, qty: 120, quality: "Grade A (FAQ)", price: 2430, isFPO: false },
  { id: "wheat-2", cropId: "wheat", farmer: "Niphad Farmer Producer Co.", village: "Niphad", distance: 22, qty: 420, quality: "Grade A (FAQ)", price: 2410, isFPO: true },
  { id: "wheat-3", cropId: "wheat", farmer: "Sunita Gaikwad", village: "Sinnar", distance: 31, qty: 60, quality: "Grade B", price: 2380, isFPO: false },
  { id: "soybean-1", cropId: "soybean", farmer: "Balaji Wagh", village: "Yeola", distance: 14, qty: 150, quality: "Grade A", price: 4700, isFPO: false },
  { id: "soybean-2", cropId: "soybean", farmer: "Sinnar Taluka FPO", village: "Sinnar", distance: 31, qty: 500, quality: "Grade A", price: 4680, isFPO: true },
  { id: "soybean-3", cropId: "soybean", farmer: "Anita More", village: "Niphad", distance: 22, qty: 45, quality: "Grade B", price: 4600, isFPO: false },
  { id: "chana-1", cropId: "chana", farmer: "Deepak Shelke", village: "Chandwad", distance: 27, qty: 90, quality: "Grade A", price: 5820, isFPO: false },
  { id: "chana-2", cropId: "chana", farmer: "Niphad Farmer Producer Co.", village: "Niphad", distance: 22, qty: 300, quality: "Grade A", price: 5790, isFPO: true },
  { id: "chana-3", cropId: "chana", farmer: "Meera Kale", village: "Yeola", distance: 14, qty: 35, quality: "Grade B", price: 5700, isFPO: false },
  { id: "cotton-1", cropId: "cotton", farmer: "Suresh Bhosale", village: "Malegaon", distance: 40, qty: 80, quality: "Grade A", price: 7180, isFPO: false },
  { id: "cotton-2", cropId: "cotton", farmer: "Malegaon Cotton FPO", village: "Malegaon", distance: 40, qty: 350, quality: "Export Grade", price: 7220, isFPO: true },
  { id: "cotton-3", cropId: "cotton", farmer: "Kavita Jadhav", village: "Chandwad", distance: 27, qty: 55, quality: "Grade B", price: 7050, isFPO: false },
  { id: "turmeric-1", cropId: "turmeric", farmer: "Ganesh Aher", village: "Kalwan", distance: 45, qty: 40, quality: "Export Grade", price: 9650, isFPO: false },
  { id: "turmeric-2", cropId: "turmeric", farmer: "Vaishali Pawar", village: "Yeola", distance: 14, qty: 25, quality: "Grade A", price: 9400, isFPO: false },
  { id: "turmeric-3", cropId: "turmeric", farmer: "Kalwan Spice FPO", village: "Kalwan", distance: 45, qty: 200, quality: "Export Grade", price: 9700, isFPO: true },
  { id: "jowar-1", cropId: "jowar", farmer: "Prakash Thorat", village: "Sinnar", distance: 31, qty: 70, quality: "Grade A", price: 2960, isFPO: false },
  { id: "jowar-2", cropId: "jowar", farmer: "Sinnar Taluka FPO", village: "Sinnar", distance: 31, qty: 260, quality: "Grade A", price: 2940, isFPO: true },
  { id: "jowar-3", cropId: "jowar", farmer: "Sarika Bagul", village: "Niphad", distance: 22, qty: 30, quality: "Grade B", price: 2870, isFPO: false },
  { id: "onion-1", cropId: "onion", farmer: "Vikas Shinde", village: "Lasalgaon", distance: 18, qty: 80, quality: "Grade A", price: 1840, isFPO: false },
  { id: "onion-2", cropId: "onion", farmer: "Lasalgaon APMC Growers Group", village: "Lasalgaon", distance: 18, qty: 260, quality: "Grade A", price: 1855, isFPO: true },
  { id: "onion-3", cropId: "onion", farmer: "Rekha Salve", village: "Pimpalgaon", distance: 9, qty: 35, quality: "Grade B", price: 1770, isFPO: false },
  { id: "tomato-1", cropId: "tomato", farmer: "Ajay Bhalerao", village: "Pimpalgaon", distance: 9, qty: 25, quality: "Grade A", price: 1310, isFPO: false },
  { id: "tomato-2", cropId: "tomato", farmer: "Shubhangi Pagar", village: "Niphad", distance: 22, qty: 18, quality: "Grade B", price: 1240, isFPO: false },
  { id: "grapes-1", cropId: "grapes", farmer: "Sahyadri Grape Growers", village: "Niphad", distance: 22, qty: 22, quality: "Export Grade", price: 4180, isFPO: true },
  { id: "grapes-2", cropId: "grapes", farmer: "Nitin Kadam", village: "Pimpalgaon", distance: 9, qty: 12, quality: "Grade A", price: 3980, isFPO: false },
];

const BUYER_TYPES = [
  { id: "retailer", label: "Retailer" },
  { id: "fpo", label: "FPO" },
  { id: "exporter", label: "Exporter" },
  { id: "hotel", label: "Hotel & Hostel Bulk Buyer" },
];

const NON_PERISHABLE_CATEGORIES = ["Grains", "Oilseeds", "Pulses", "Cash Crop", "Spices"];

function isRecommended(listing, crop, buyerType) {
  if (buyerType === "exporter") return listing.quality.includes("Export") || listing.quality.includes("Grade A");
  if (buyerType === "fpo") return listing.isFPO || listing.qty >= 200;
  if (buyerType === "retailer") return listing.qty >= 10 && listing.qty <= 150;
  if (buyerType === "hotel") return crop.perishable;
  return false;
}

const translations = {
  en: { sellingLabel: "I'm selling", current: "Current price", currentSub: "avg. of your 3 nearest mandis", lastYear: "Best price last year", nearest: "Your 3 nearest mandis", listTitle: "List this for buyers", qtyLabel: "Quantity (quintals)", qualityLabel: "Quality grade", listBtn: "List for buyers", callBtn: "Hear prices by phone", holdTip: "Long shelf life — you can afford to wait for a better price.", sellTip: "Perishable — sell within days, don't wait for the peak." },
  hi: { sellingLabel: "मैं बेच रहा हूँ", current: "वर्तमान भाव", currentSub: "आपकी 3 नज़दीकी मंडियों का औसत", lastYear: "पिछले साल का सबसे अच्छा भाव", nearest: "आपकी 3 नज़दीकी मंडियाँ", listTitle: "खरीदारों के लिए सूचीबद्ध करें", qtyLabel: "मात्रा (क्विंटल)", qualityLabel: "गुणवत्ता", listBtn: "सूचीबद्ध करें", callBtn: "फ़ोन पर भाव सुनें", holdTip: "लंबे समय तक टिकता है — बेहतर भाव का इंतज़ार कर सकते हैं।", sellTip: "जल्दी खराब होता है — जल्द बेचें, इंतज़ार न करें।" },
  mr: { sellingLabel: "मी विकत आहे", current: "सध्याचा भाव", currentSub: "तुमच्या 3 जवळच्या बाजारांची सरासरी", lastYear: "मागच्या वर्षीचा सर्वोत्तम भाव", nearest: "तुमचे 3 जवळचे बाजार", listTitle: "खरेदीदारांसाठी नोंदवा", qtyLabel: "प्रमाण (क्विंटल)", qualityLabel: "प्रत", listBtn: "नोंदवा", callBtn: "फोनवर भाव ऐका", holdTip: "जास्त काळ टिकते — चांगल्या भावाची वाट पाहू शकता.", sellTip: "लवकर खराब होते — लवकर विका, वाट पाहू नका." },
};

// ---------- Root ----------
export default function KisanConnectMarketplace() {
  const [view, setView] = useState("farmer");
  const [listings, setListings] = useState(INITIAL_LISTINGS);
  const [toast, setToast] = useState("");

  return (
    <div style={{ background: "var(--bg)", minHeight: "100%" }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,500;9..144,600;9..144,700&family=Inter:wght@400;500;600;700&display=swap');
        :root {
          --bg:#FAF6EC; --ink:#21301C; --green-deep:#2F5233; --green-mid:#6B8F5E;
          --gold:#C88A2E; --clay:#A85C32; --sky:#3E6E86; --card:#FFFFFF; --line:#E4DCC4;
          --font-display:'Fraunces',serif; --font-body:'Inter',sans-serif;
        }
        * { box-sizing: border-box; }
        .kc-wrap { max-width: 460px; margin: 0 auto; padding: 20px 16px 60px; color: var(--ink); font-family: var(--font-body); }
        .kc-header { display:flex; align-items:center; justify-content:space-between; margin-bottom: 14px; }
        .kc-brand { display:flex; align-items:center; gap:8px; }
        .kc-brand h1 { font-family: var(--font-display); font-size: 21px; font-weight: 600; margin: 0; }
        .role-toggle { display:flex; gap:5px; background:#EFEADB; padding:4px; border-radius:12px; margin-bottom:18px; }
        .role-toggle button { flex:1; border:none; background:transparent; padding:9px 6px; border-radius:9px; font-size:13px; font-weight:600; cursor:pointer; font-family:var(--font-body); color:var(--ink); }
        .role-toggle button.active { background:var(--green-deep); color:#fff; }
        .kc-lang { display:flex; gap:4px; }
        .kc-lang button { border:1px solid var(--line); background:var(--card); border-radius:7px; padding:4px 8px; font-size:11.5px; cursor:pointer; color:var(--ink); font-family:var(--font-body); }
        .kc-lang button.active { background:var(--green-deep); color:#fff; border-color:var(--green-deep); }
        .kc-field { flex:1; }
        .kc-field label { display:block; font-size:11.5px; color:var(--green-mid); margin-bottom:4px; font-weight:600; }
        .kc-field select, .kc-field input { width:100%; padding:9px 10px; border-radius:9px; border:1px solid var(--line); background:var(--card); font-size:14px; font-family:var(--font-body); color:var(--ink); }
        .crop-tip { display:inline-flex; align-items:center; gap:6px; font-size:11.5px; padding:6px 10px; border-radius:8px; margin: 10px 0 16px; font-weight:500; }
        .crop-tip.hold { background:#E4EFDD; color:var(--green-deep); }
        .crop-tip.sell { background:#FBE7DD; color:var(--clay); }
        .kc-hero { background: var(--green-deep); border-radius:16px; padding:20px; color:#F3EFE1; margin-bottom:16px; }
        .kc-hero-label { font-size:12.5px; opacity:0.85; }
        .kc-hero-value { font-family: var(--font-display); font-size:36px; font-weight:600; line-height:1.15; }
        .kc-hero-sub { font-size:12px; opacity:0.8; margin-top:2px; }
        .kc-compare { display:flex; align-items:center; gap:8px; margin-top:14px; background:rgba(255,255,255,0.1); border-radius:10px; padding:10px 12px; }
        .kc-compare-label { font-size:11.5px; opacity:0.85; }
        .kc-compare-value { font-family: var(--font-display); font-size:17px; font-weight:600; }
        .kc-pct { margin-left:auto; display:flex; align-items:center; gap:4px; font-size:12.5px; font-weight:700; padding:3px 9px; border-radius:16px; }
        .kc-pct.up { background: rgba(200,138,46,0.3); color:var(--gold); }
        .kc-pct.down { background: rgba(255,255,255,0.15); color:#F3EFE1; }
        .kc-section-title { font-family: var(--font-display); font-size:16px; font-weight:600; margin: 4px 0 10px; }
        .mandi-row { display:flex; justify-content:space-between; align-items:center; padding:9px 12px; background:var(--card); border:1px solid var(--line); border-radius:10px; margin-bottom:6px; font-size:12.5px; }
        .mandi-name { font-weight:600; }
        .mandi-dist { color:#6B6F5F; font-size:11px; }
        .mandi-price { font-weight:700; color: var(--green-deep); }
        .kc-card { background:var(--card); border:1px solid var(--line); border-radius:14px; padding:16px; margin: 18px 0; }
        .kc-btn { border:none; border-radius:9px; padding:10px 14px; font-size:13px; font-weight:600; cursor:pointer; font-family:var(--font-body); display:inline-flex; align-items:center; gap:6px; }
        .kc-btn-gold { background:var(--gold); color:#fff; width:100%; justify-content:center; }
        .kc-btn-sky { background:var(--sky); color:#fff; }
        .kc-row2 { display:flex; gap:10px; margin-bottom:12px; }
        .call-strip { display:flex; align-items:center; gap:10px; background:var(--card); border:1px solid var(--line); border-radius:12px; padding:12px 14px; margin-top: 16px; }
        .call-icon { background:var(--sky); color:#fff; border-radius:9px; width:34px; height:34px; display:flex; align-items:center; justify-content:center; flex-shrink:0; }

        .search-bar { display:flex; align-items:center; gap:8px; background:var(--card); border:1px solid var(--line); border-radius:12px; padding:9px 12px; margin-bottom:12px; }
        .search-bar input { border:none; outline:none; flex:1; font-size:13.5px; font-family:var(--font-body); background:transparent; }
        .chip-row { display:flex; gap:8px; overflow-x:auto; margin-bottom: 12px; padding-bottom:3px; }
        .chip { flex-shrink:0; border:1px solid var(--line); background:var(--card); border-radius:20px; padding:7px 13px; font-size:12px; font-weight:600; cursor:pointer; white-space:nowrap; color: var(--ink); font-family: var(--font-body); }
        .chip.active-buyer { background:var(--clay); color:#fff; border-color:var(--clay); }
        .chip.active-cat { background:var(--green-deep); color:#fff; border-color:var(--green-deep); }
        .group-heading { font-size:12.5px; font-weight:700; color:var(--green-mid); margin:18px 0 8px; display:flex; align-items:center; gap:6px; }
        .product-card { background:var(--card); border:1px solid var(--line); border-radius:14px; padding:14px; margin-bottom:12px; }
        .product-top { display:flex; align-items:flex-start; gap:10px; }
        .product-icon-wrap { width:42px; height:42px; border-radius:10px; background:#F3EFE1; display:flex; align-items:center; justify-content:center; font-size:21px; flex-shrink:0; }
        .product-title-block { flex:1; }
        .product-name { font-weight:600; font-size:14.5px; margin:0 0 2px; }
        .product-farmer { font-size:11.5px; color:#6B6F5F; margin:0; display:flex; align-items:center; gap:4px; }
        .product-price { font-weight:700; color:var(--green-deep); font-size:15.5px; text-align:right; white-space:nowrap; }
        .product-price small { display:block; font-weight:500; font-size:10px; color:#8A8D7C; }
        .product-tags { display:flex; flex-wrap:wrap; gap:5px; margin:9px 0; }
        .tag { font-size:10px; padding:3px 8px; border-radius:6px; font-weight:600; }
        .tag-nonperishable { background:#E4EFDD; color:var(--green-deep); }
        .tag-perishable { background:#FBE7DD; color:var(--clay); }
        .tag-quality { background:#EAF2F5; color:var(--sky); }
        .tag-recommended { background:var(--gold); color:#fff; }
        .tag-fpo { background:#EFEADB; color:#7A6A2E; }
        .product-bottom { display:flex; justify-content:space-between; align-items:center; margin-top:4px; }
        .product-meta { font-size:11.5px; color:#6B6F5F; }
        .quote-btn { background:var(--sky); color:#fff; border:none; border-radius:8px; padding:7px 12px; font-size:12px; font-weight:600; cursor:pointer; }
        .quote-btn.added { background:var(--green-deep); }
        .cart-fab { position:fixed; bottom:20px; right:20px; background:var(--gold); color:#fff; border:none; border-radius:30px; padding:12px 18px; display:flex; align-items:center; gap:8px; font-weight:700; font-size:13px; box-shadow:0 6px 18px rgba(0,0,0,0.18); cursor:pointer; z-index:40; }
        .toast { position:fixed; top:14px; left:50%; transform:translateX(-50%); background:var(--green-deep); color:#fff; padding:10px 16px; border-radius:10px; font-size:13px; display:flex; align-items:center; gap:8px; z-index:60; box-shadow:0 6px 16px rgba(0,0,0,0.2); }
        .toast button { background:none; border:none; color:#fff; cursor:pointer; opacity:0.8; }
        .kc-modal-backdrop { position:fixed; inset:0; background:rgba(33,48,28,0.55); display:flex; align-items:center; justify-content:center; padding:20px; z-index:50; }
        .kc-modal { background:var(--card); border-radius:18px; padding:22px; max-width:360px; width:100%; max-height:80vh; overflow-y:auto; }
        .kc-modal-head { display:flex; justify-content:space-between; align-items:center; margin-bottom:14px; }
        .kc-modal-head h3 { font-family: var(--font-display); font-size:17px; margin:0; }
        .kc-close { background:none; border:none; cursor:pointer; color:var(--ink); }
        .cart-line { display:flex; justify-content:space-between; align-items:center; padding:10px 0; border-bottom:1px solid var(--line); font-size:13px; }
        .cart-line:last-of-type { border-bottom:none; }
        .kc-line { display:flex; gap:9px; margin-bottom:12px; font-size:13px; line-height:1.4; }
        .kc-line .n { width:20px; height:20px; border-radius:50%; background:var(--sky); color:#fff; font-size:10.5px; display:flex; align-items:center; justify-content:center; flex-shrink:0; margin-top:1px; }
      `}</style>

      <div className="kc-wrap">
        <div className="kc-header">
          <div className="kc-brand">
            <Leaf size={20} color="var(--green-deep)" />
            <h1>KisanConnect</h1>
          </div>
        </div>

        <div className="role-toggle">
          <button className={view === "farmer" ? "active" : ""} onClick={() => setView("farmer")}>Farmer</button>
          <button className={view === "buyer" ? "active" : ""} onClick={() => setView("buyer")}>Buyer</button>
        </div>

        {view === "farmer" ? (
          <FarmerView listings={listings} setListings={setListings} setToast={setToast} setView={setView} />
        ) : (
          <BuyerView listings={listings} setToast={setToast} />
        )}
      </div>

      {toast && (
        <div className="toast">
          <CheckCircle2 size={15} /> {toast}
          <button onClick={() => setToast("")}><X size={14} /></button>
        </div>
      )}
    </div>
  );
}

// ---------- Farmer view ----------
function FarmerView({ listings, setListings, setToast, setView }) {
  const [lang, setLang] = useState("en");
  const [cropId, setCropId] = useState("wheat");
  const [qty, setQty] = useState("20");
  const [quality, setQuality] = useState("Grade A");
  const [showCall, setShowCall] = useState(false);
  const t = translations[lang];

  const crop = CROPS.find((c) => c.id === cropId);
  const nearestThree = crop.prices.slice(0, 3);
  const currentAvg = Math.round(nearestThree.reduce((a, b) => a + b, 0) / 3);
  const diff = currentAvg - crop.bestPriceLastYear;
  const diffPct = Math.round((diff / crop.bestPriceLastYear) * 100);

  const handleList = () => {
    const id = `${cropId}-you-${Date.now()}`;
    setListings((prev) => [
      { id, cropId, farmer: "You", village: "Niphad", distance: 0, qty: Number(qty) || 1, quality, price: currentAvg, isFPO: false },
      ...prev,
    ]);
    setToast("Listed! Buyers can now see this in the marketplace.");
  };

  return (
    <div>
      <div className="kc-header" style={{ marginBottom: 4 }}>
        <span style={{ fontSize: 12.5, color: "#6B6F5F" }}>Niphad, Nashik</span>
        <div className="kc-lang">
          <button className={lang === "en" ? "active" : ""} onClick={() => setLang("en")}>EN</button>
          <button className={lang === "hi" ? "active" : ""} onClick={() => setLang("hi")}>हिं</button>
          <button className={lang === "mr" ? "active" : ""} onClick={() => setLang("mr")}>मरा</button>
        </div>
      </div>

      <div className="kc-field" style={{ marginTop: 14 }}>
        <label>{t.sellingLabel}</label>
        <select value={cropId} onChange={(e) => setCropId(e.target.value)}>
          {CROPS.map((c) => (
            <option key={c.id} value={c.id}>{c.icon} {c.name} · {c.category}</option>
          ))}
        </select>
      </div>

      <div className={`crop-tip ${crop.perishable ? "sell" : "hold"}`}>
        {crop.perishable ? t.sellTip : t.holdTip}
      </div>

      <div className="kc-hero">
        <div className="kc-hero-label">{t.current}</div>
        <div className="kc-hero-value">₹{currentAvg.toLocaleString("en-IN")}</div>
        <div className="kc-hero-sub">{t.currentSub}</div>

        <div className="kc-compare">
          <div>
            <div className="kc-compare-label">{t.lastYear}</div>
            <div className="kc-compare-value">₹{crop.bestPriceLastYear.toLocaleString("en-IN")}</div>
          </div>
          <div className={`kc-pct ${diff >= 0 ? "up" : "down"}`}>
            {diff >= 0 ? <TrendingUp size={13} /> : <TrendingDown size={13} />}
            {Math.abs(diffPct)}%
          </div>
        </div>
      </div>

      <h2 className="kc-section-title">{t.nearest}</h2>
      {MANDIS.slice(0, 3).map((m, i) => (
        <div className="mandi-row" key={m.name}>
          <div>
            <div className="mandi-name">{m.name}</div>
            <div className="mandi-dist">{m.distance} km</div>
          </div>
          <div className="mandi-price">₹{crop.prices[i]}</div>
        </div>
      ))}

      <div className="call-strip">
        <div className="call-icon"><Phone size={16} /></div>
        <div style={{ flex: 1, fontSize: 12.5 }}>{t.callBtn}</div>
        <button className="kc-btn kc-btn-sky" onClick={() => setShowCall(true)}>{t.callBtn}</button>
      </div>

      <div className="kc-card">
        <h2 className="kc-section-title" style={{ marginTop: 0 }}>{t.listTitle}</h2>
        <div className="kc-row2">
          <div className="kc-field">
            <label>{t.qtyLabel}</label>
            <input type="number" value={qty} onChange={(e) => setQty(e.target.value)} />
          </div>
          <div className="kc-field">
            <label>{t.qualityLabel}</label>
            <select value={quality} onChange={(e) => setQuality(e.target.value)}>
              <option>Grade A</option>
              <option>Grade B</option>
              <option>Export Grade</option>
            </select>
          </div>
        </div>
        <button className="kc-btn kc-btn-gold" onClick={handleList}>{t.listBtn} <ArrowRight size={14} /></button>
        <div style={{ textAlign: "center", marginTop: 10 }}>
          <button
            onClick={() => setView("buyer")}
            style={{ background: "none", border: "none", color: "var(--sky)", fontSize: 12, cursor: "pointer", fontFamily: "var(--font-body)", fontWeight: 600 }}
          >
            See it in Buyer view →
          </button>
        </div>
      </div>

      {showCall && (
        <div className="kc-modal-backdrop" onClick={() => setShowCall(false)}>
          <div className="kc-modal" onClick={(e) => e.stopPropagation()}>
            <div className="kc-modal-head">
              <h3>📞 KisanConnect IVR</h3>
              <button className="kc-close" onClick={() => setShowCall(false)}><X size={18} /></button>
            </div>
            <div className="kc-line"><div className="n">1</div><div>Welcome to the KisanConnect price line.</div></div>
            <div className="kc-line"><div className="n">2</div><div>Today's average price for {crop.name} across your 3 nearest mandis is ₹{currentAvg} per quintal.</div></div>
            <div className="kc-line"><div className="n">3</div><div>Last year's best price for {crop.name} was ₹{crop.bestPriceLastYear} per quintal.</div></div>
            <div className="kc-line" style={{ color: "var(--sky)", fontWeight: 600 }}>
              <div className="n" style={{ background: "var(--gold)" }}><Mic size={11} /></div>
              <div>Stay on the line after the beep to list your produce by voice.</div>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}

// ---------- Buyer view ----------
function BuyerView({ listings, setToast }) {
  const [buyerType, setBuyerType] = useState("retailer");
  const [categoryFilter, setCategoryFilter] = useState("All");
  const [search, setSearch] = useState("");
  const [cart, setCart] = useState([]);
  const [showCart, setShowCart] = useState(false);

  const categories = buyerType === "hotel"
    ? ["All", "Perishable", ...NON_PERISHABLE_CATEGORIES]
    : ["All", ...NON_PERISHABLE_CATEGORIES, "Perishable"];

  const withMeta = listings.map((l) => ({ ...l, crop: CROPS.find((c) => c.id === l.cropId) }));

  const filtered = withMeta.filter((l) => {
    const matchesCategory = categoryFilter === "All" || l.crop.category === categoryFilter;
    const matchesSearch = search === "" ||
      l.crop.name.toLowerCase().includes(search.toLowerCase()) ||
      l.farmer.toLowerCase().includes(search.toLowerCase());
    return matchesCategory && matchesSearch;
  });

  const sortFn = (a, b) => {
    const ra = isRecommended(a, a.crop, buyerType) ? 0 : 1;
    const rb = isRecommended(b, b.crop, buyerType) ? 0 : 1;
    if (ra !== rb) return ra - rb;
    return a.distance - b.distance;
  };

  const nonPerishGroup = filtered.filter((l) => !l.crop.perishable).sort(sortFn);
  const perishGroup = filtered.filter((l) => l.crop.perishable).sort(sortFn);
  const groups = buyerType === "hotel"
    ? [["Fresh & perishable", perishGroup], ["Storage-friendly", nonPerishGroup]]
    : [["Storage-friendly, non-perishable", nonPerishGroup], ["Perishable — move fast", perishGroup]];

  const toggleCart = (id) => {
    setCart((prev) => (prev.includes(id) ? prev.filter((x) => x !== id) : [...prev, id]));
  };

  const sendEnquiry = () => {
    setCart([]);
    setShowCart(false);
    setToast("Bulk enquiry sent to all selected farmers.");
  };

  const cartListings = withMeta.filter((l) => cart.includes(l.id));

  return (
    <div>
      <div className="chip-row">
        {BUYER_TYPES.map((b) => (
          <button
            key={b.id}
            className={`chip ${buyerType === b.id ? "active-buyer" : ""}`}
            onClick={() => setBuyerType(b.id)}
          >
            {b.label}
          </button>
        ))}
      </div>

      <div className="search-bar">
        <Search size={15} color="#8A8D7C" />
        <input placeholder="Search produce or farmer" value={search} onChange={(e) => setSearch(e.target.value)} />
      </div>

      <div className="chip-row">
        {categories.map((c) => (
          <button
            key={c}
            className={`chip ${categoryFilter === c ? "active-cat" : ""}`}
            onClick={() => setCategoryFilter(c)}
          >
            {c}
          </button>
        ))}
      </div>

      {categoryFilter === "All" ? (
        groups.map(([label, group]) => group.length > 0 && (
          <div key={label}>
            <div className="group-heading">{label}</div>
            {group.map((l) => (
              <ProductCard key={l.id} listing={l} buyerType={buyerType} inCart={cart.includes(l.id)} onToggle={() => toggleCart(l.id)} />
            ))}
          </div>
        ))
      ) : (
        filtered.sort(sortFn).map((l) => (
          <ProductCard key={l.id} listing={l} buyerType={buyerType} inCart={cart.includes(l.id)} onToggle={() => toggleCart(l.id)} />
        ))
      )}

      {cart.length > 0 && (
        <button className="cart-fab" onClick={() => setShowCart(true)}>
          <ShoppingBag size={16} /> {cart.length} in enquiry
        </button>
      )}

      {showCart && (
        <div className="kc-modal-backdrop" onClick={() => setShowCart(false)}>
          <div className="kc-modal" onClick={(e) => e.stopPropagation()}>
            <div className="kc-modal-head">
              <h3>Bulk enquiry</h3>
              <button className="kc-close" onClick={() => setShowCart(false)}><X size={18} /></button>
            </div>
            {cartListings.map((l) => (
              <div className="cart-line" key={l.id}>
                <span>{l.crop.icon} {l.crop.name} — {l.farmer}</span>
                <span>₹{l.price} × {l.qty}q</span>
              </div>
            ))}
            <button className="kc-btn kc-btn-gold" style={{ marginTop: 14 }} onClick={sendEnquiry}>
              Send bulk enquiry to {cartListings.length} farmer{cartListings.length > 1 ? "s" : ""}
            </button>
          </div>
        </div>
      )}
    </div>
  );
}

function ProductCard({ listing, buyerType, inCart, onToggle }) {
  const crop = listing.crop;
  const recommended = isRecommended(listing, crop, buyerType);
  return (
    <div className="product-card">
      <div className="product-top">
        <div className="product-icon-wrap">{crop.icon}</div>
        <div className="product-title-block">
          <p className="product-name">{crop.name}</p>
          <p className="product-farmer"><MapPin size={11} /> {listing.farmer}, {listing.village} · {listing.distance} km</p>
        </div>
        <div className="product-price">
          ₹{listing.price}
          <small>per quintal</small>
        </div>
      </div>
      <div className="product-tags">
        {recommended && <span className="tag tag-recommended">Recommended for you</span>}
        <span className={`tag ${crop.perishable ? "tag-perishable" : "tag-nonperishable"}`}>
          {crop.perishable ? "Perishable" : "Non-perishable"}
        </span>
        <span className="tag tag-quality">{listing.quality}</span>
        {listing.isFPO && <span className="tag tag-fpo">FPO lot</span>}
      </div>
      <div className="product-bottom">
        <span className="product-meta">{listing.qty} quintals available</span>
        <button className={`quote-btn ${inCart ? "added" : ""}`} onClick={onToggle}>
          {inCart ? "Added ✓" : "Request quote"}
        </button>
      </div>
    </div>
  );
}
