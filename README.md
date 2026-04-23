# import { useState, useEffect, useRef, useCallback } from "react";

// ── Lucide-style SVG icons (inline, no import needed) ──────────────────────
const Icon = ({ d, size = 16, color = "currentColor", strokeWidth = 1.5 }) => (
  <svg width={size} height={size} viewBox="0 0 24 24" fill="none" stroke={color} strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round">
    <path d={d} />
  </svg>
);

// ── Color palette & theme ──────────────────────────────────────────────────
const C = {
  bg0: "#0B0E11",
  bg1: "#12161C",
  bg2: "#1E2329",
  bg3: "#2B3139",
  bg4: "#374151",
  accent: "#F0B90B",
  accentDark: "#C99B09",
  accentGlow: "rgba(240,185,11,0.15)",
  green: "#0ECB81",
  red: "#F6465D",
  text1: "#EAECEF",
  text2: "#848E9C",
  text3: "#5E6673",
  border: "#2B3139",
};

// ── Crypto data ─────────────────────────────────────────────────────────────
const COINS = [
  { id: "btc", name: "Bitcoin", symbol: "BTC", price: 67842.50, change: 2.34, vol: 28_400_000_000, mcap: 1_330_000_000_000, color: "#F7931A", icon: "₿" },
  { id: "eth", name: "Ethereum", symbol: "ETH", price: 3521.80, change: -1.12, vol: 14_200_000_000, mcap: 423_000_000_000, color: "#627EEA", icon: "Ξ" },
  { id: "bnb", name: "BNB", symbol: "BNB", price: 598.40, change: 3.87, vol: 2_100_000_000, mcap: 89_000_000_000, color: "#F0B90B", icon: "⬡" },
  { id: "sol", name: "Solana", symbol: "SOL", price: 178.30, change: 5.21, vol: 4_800_000_000, mcap: 78_000_000_000, color: "#9945FF", icon: "◎" },
  { id: "xrp", name: "XRP", symbol: "XRP", price: 0.6234, change: -0.87, vol: 1_900_000_000, mcap: 34_000_000_000, color: "#00AAE4", icon: "✕" },
  { id: "ada", name: "Cardano", symbol: "ADA", price: 0.4521, change: 1.45, vol: 520_000_000, mcap: 16_000_000_000, color: "#0033AD", icon: "₳" },
  { id: "doge", name: "Dogecoin", symbol: "DOGE", price: 0.1634, change: 8.92, vol: 3_200_000_000, mcap: 23_000_000_000, color: "#C2A633", icon: "Ð" },
  { id: "avax", name: "Avalanche", symbol: "AVAX", price: 38.72, change: -2.34, vol: 680_000_000, mcap: 16_000_000_000, color: "#E84142", icon: "▲" },
  { id: "link", name: "Chainlink", symbol: "LINK", price: 14.87, change: 4.56, vol: 890_000_000, mcap: 9_000_000_000, color: "#375BD2", icon: "⬡" },
  { id: "dot", name: "Polkadot", symbol: "DOT", price: 7.23, change: -1.98, vol: 310_000_000, mcap: 10_000_000_000, color: "#E6007A", icon: "●" },
  { id: "matic", name: "Polygon", symbol: "MATIC", price: 0.8934, change: 2.11, vol: 450_000_000, mcap: 8_800_000_000, color: "#8247E5", icon: "◆" },
  { id: "uni", name: "Uniswap", symbol: "UNI", price: 9.87, change: 6.34, vol: 230_000_000, mcap: 5_900_000_000, color: "#FF007A", icon: "🦄" },
];

const NETWORKS = {
  BTC: [{ id: "btc", name: "Bitcoin Network", fee: "0.0005 BTC", time: "10-30 min" }],
  ETH: [
    { id: "erc20", name: "ERC20", fee: "0.001 ETH", time: "1-5 min" },
    { id: "arb", name: "Arbitrum One", fee: "0.0001 ETH", time: "< 1 min" },
  ],
  BNB: [
    { id: "bep20", name: "BEP20 (BSC)", fee: "0.0005 BNB", time: "< 1 min" },
    { id: "bep2", name: "BEP2", fee: "0.001 BNB", time: "1-3 min" },
  ],
  SOL: [{ id: "sol", name: "Solana Network", fee: "0.000005 SOL", time: "< 30 sec" }],
  XRP: [{ id: "xrp", name: "XRP Ledger", fee: "0.00001 XRP", time: "< 10 sec" }],
  default: [
    { id: "erc20", name: "ERC20", fee: "0.001 ETH", time: "1-5 min" },
    { id: "trc20", name: "TRC20", fee: "1 TRX", time: "1-3 min" },
    { id: "bep20", name: "BEP20 (BSC)", fee: "0.0005 BNB", time: "< 1 min" },
  ],
};

const generateAddress = (coin, network) => {
  const prefixes = { btc: "bc1q", eth: "0x", bep20: "0x", trc20: "T", sol: "", xrp: "r" };
  const chars = "abcdefABCDEF0123456789";
  const prefix = prefixes[network?.id] || "0x";
  const len = network?.id === "btc" ? 38 : 40;
  return prefix + Array.from({ length: len }, () => chars[Math.floor(Math.random() * chars.length)]).join("");
};

const generateTxHash = () => "0x" + Array.from({ length: 64 }, () => "0123456789abcdef"[Math.floor(Math.random() * 16)]).join("");

// ── Helpers ─────────────────────────────────────────────────────────────────
const fmt = (n, d = 2) => n?.toLocaleString("en-IN", { minimumFractionDigits: d, maximumFractionDigits: d }) ?? "0";
const fmtUSD = (n) => n >= 1e9 ? `$${(n / 1e9).toFixed(2)}B` : n >= 1e6 ? `$${(n / 1e6).toFixed(2)}M` : `$${fmt(n)}`;
const fmtINR = (n) => (n * 83.5).toLocaleString("en-IN", { style: "currency", currency: "INR", maximumFractionDigits: 0 });
const sleep = (ms) => new Promise((r) => setTimeout(r, ms));

// ── Initial State ────────────────────────────────────────────────────────────
const INIT_PORTFOLIO = {
  BTC: { amount: 0.2341, locked: 0 },
  ETH: { amount: 1.8834, locked: 0 },
  BNB: { amount: 12.450, locked: 0 },
  SOL: { amount: 45.20, locked: 0 },
  USDT: { amount: 3420.50, locked: 0 },
};

const INIT_TRANSACTIONS = [
  { id: "tx1", type: "deposit", coin: "BTC", amount: 0.05, status: "completed", network: "Bitcoin Network", fee: "0.0005", hash: generateTxHash(), ts: Date.now() - 86400000 * 2, usdVal: 3392 },
  { id: "tx2", type: "withdrawal", coin: "ETH", amount: 0.5, status: "completed", network: "ERC20", fee: "0.001", hash: generateTxHash(), ts: Date.now() - 86400000, usdVal: 1760 },
  { id: "tx3", type: "deposit", coin: "USDT", amount: 1000, status: "pending", network: "TRC20", fee: "1", hash: generateTxHash(), ts: Date.now() - 3600000, usdVal: 1000 },
  { id: "tx4", type: "send", coin: "SOL", amount: 10, status: "completed", network: "Solana Network", fee: "0.000005", hash: generateTxHash(), ts: Date.now() - 7200000, usdVal: 1783 },
];

// ── Mini Sparkline ───────────────────────────────────────────────────────────
const Sparkline = ({ positive, width = 80, height = 32 }) => {
  const pts = Array.from({ length: 12 }, (_, i) => {
    const base = 50 + (positive ? i * 2 : -i * 2);
    return base + (Math.random() - 0.5) * 20;
  });
  const min = Math.min(...pts), max = Math.max(...pts);
  const normalize = (v) => ((v - min) / (max - min)) * (height - 4) + 2;
  const path = pts.map((v, i) => `${i === 0 ? "M" : "L"} ${(i / (pts.length - 1)) * width} ${height - normalize(v)}`).join(" ");
  return (
    <svg width={width} height={height} style={{ display: "block" }}>
      <path d={path} fill="none" stroke={positive ? C.green : C.red} strokeWidth="1.5" />
    </svg>
  );
};

// ── QR Code (simple pattern) ─────────────────────────────────────────────────
const QRCode = ({ value, size = 160 }) => {
  const cells = 21;
  const cell = size / cells;
  // Simple deterministic pattern based on value hash
  const hash = value.split("").reduce((a, c) => ((a << 5) - a + c.charCodeAt(0)) | 0, 0);
  const pattern = Array.from({ length: cells }, (_, r) =>
    Array.from({ length: cells }, (_, c) => {
      if (r < 7 && c < 7) return (r === 0 || r === 6 || c === 0 || c === 6 || (r >= 2 && r <= 4 && c >= 2 && c <= 4));
      if (r < 7 && c > cells - 8) return (r === 0 || r === 6 || c === cells - 1 || c === cells - 7 || (r >= 2 && r <= 4 && c >= cells - 5 && c <= cells - 3));
      if (r > cells - 8 && c < 7) return (r === cells - 1 || r === cells - 7 || c === 0 || c === 6 || (r >= cells - 5 && r <= cells - 3 && c >= 2 && c <= 4));
      return ((hash * (r + 1) * (c + 1)) % 3 === 0);
    })
  );
  return (
    <svg width={size} height={size} style={{ background: "white", padding: 8, borderRadius: 8 }}>
      {pattern.map((row, r) => row.map((filled, c) => filled ? (
        <rect key={`${r}-${c}`} x={c * cell} y={r * cell} width={cell} height={cell} fill="#000" />
      ) : null))}
    </svg>
  );
};

// ── Toast ────────────────────────────────────────────────────────────────────
const ToastContainer = ({ toasts }) => (
  <div style={{ position: "fixed", top: 72, right: 16, zIndex: 9999, display: "flex", flexDirection: "column", gap: 8 }}>
    {toasts.map((t) => (
      <div key={t.id} style={{
        background: t.type === "success" ? C.green : t.type === "error" ? C.red : C.bg3,
        color: "#fff", padding: "10px 16px", borderRadius: 8, fontSize: 13, fontWeight: 500,
        boxShadow: "0 4px 20px rgba(0,0,0,0.4)", minWidth: 260,
        animation: "slideIn 0.3s ease", display: "flex", alignItems: "center", gap: 8,
      }}>
        {t.type === "success" ? "✓" : t.type === "error" ? "✕" : "ℹ"} {t.msg}
      </div>
    ))}
  </div>
);

// ── Loading Skeleton ─────────────────────────────────────────────────────────
const Skeleton = ({ w = "100%", h = 16, r = 4 }) => (
  <div style={{ width: w, height: h, borderRadius: r, background: `linear-gradient(90deg, ${C.bg3} 25%, ${C.bg4} 50%, ${C.bg3} 75%)`, backgroundSize: "200% 100%", animation: "shimmer 1.5s infinite" }} />
);

// ── CoinIcon ─────────────────────────────────────────────────────────────────
const CoinIcon = ({ coin, size = 28 }) => {
  const c = COINS.find((x) => x.symbol === coin);
  return (
    <div style={{ width: size, height: size, borderRadius: "50%", background: c?.color || C.bg4, display: "flex", alignItems: "center", justifyContent: "center", fontSize: size * 0.45, color: "#fff", fontWeight: 700, flexShrink: 0 }}>
      {c?.icon || coin?.[0]}
    </div>
  );
};

// ── Status Badge ─────────────────────────────────────────────────────────────
const StatusBadge = ({ status }) => {
  const map = { completed: [C.green, "#0a2a1f"], pending: [C.accent, "#2a220a"], failed: [C.red, "#2a0a0f"], confirming: ["#3399ff", "#0a1a2a"] };
  const [color, bg] = map[status] || [C.text2, C.bg3];
  return <span style={{ background: bg, color, border: `1px solid ${color}40`, borderRadius: 4, padding: "2px 8px", fontSize: 11, fontWeight: 600, textTransform: "uppercase" }}>{status}</span>;
};

// ─────────────────────────────────────────────────────────────────────────────
// MAIN COMPONENT
// ─────────────────────────────────────────────────────────────────────────────
export default function CryptoDashboard() {
  // Auth
  const [authed, setAuthed] = useState(false);
  const [authMode, setAuthMode] = useState("login"); // login | signup | otp
  const [authForm, setAuthForm] = useState({ email: "", password: "", otp: "" });
  const [otpSent, setOtpSent] = useState(false);
  const [generatedOTP, setGeneratedOTP] = useState("");

  // Navigation
  const [page, setPage] = useState("dashboard");
  const [tradePair, setTradePair] = useState("BTC/USDT");

  // Currency toggle
  const [currency, setCurrency] = useState("USD"); // USD | INR

  // Portfolio & prices
  const [portfolio, setPortfolio] = useState(INIT_PORTFOLIO);
  const [prices, setPrices] = useState(() => Object.fromEntries(COINS.map((c) => [c.symbol, c.price])));
  const [priceChanges, setPriceChanges] = useState(() => Object.fromEntries(COINS.map((c) => [c.symbol, c.change])));
  const [coinData, setCoinData] = useState(COINS);
  const [transactions, setTransactions] = useState(INIT_TRANSACTIONS);
  const [favorites, setFavorites] = useState(new Set(["BTC", "ETH", "BNB"]));
  const [loading, setLoading] = useState(true);

  // UI state
  const [toasts, setToasts] = useState([]);
  const [modalOpen, setModalOpen] = useState(null); // null | "deposit" | "withdraw" | "send" | "receive" | "confirm"
  const [flowCoin, setFlowCoin] = useState("BTC");
  const [flowNetwork, setFlowNetwork] = useState(null);
  const [flowAddress, setFlowAddress] = useState("");
  const [flowAmount, setFlowAmount] = useState("");
  const [flowStep, setFlowStep] = useState(1); // 1=select, 2=details, 3=confirm, 4=processing, 5=done
  const [generatedAddr, setGeneratedAddr] = useState("");
  const [walletTab, setWalletTab] = useState("assets");
  const [marketSearch, setMarketSearch] = useState("");
  const [marketFilter, setMarketFilter] = useState("all"); // all | favorites | gainers | losers
  const [tradeTab, setTradeTab] = useState("buy");
  const [orderType, setOrderType] = useState("market");
  const [tradeAmount, setTradeAmount] = useState("");
  const [tradePrice, setTradePrice] = useState("");
  const [settingsTab, setSettingsTab] = useState("security");
  const [pinValue, setPinValue] = useState("");
  const [showBalance, setShowBalance] = useState(true);
  const [sidebarOpen, setSidebarOpen] = useState(false);

  // Admin panel
  const [adminMode, setAdminMode] = useState(false);
  const [adminKey, setAdminKey] = useState("");
  const [adminCoin, setAdminCoin] = useState("BTC");
  const [adminAmount, setAdminAmount] = useState("");
  const [adminAction, setAdminAction] = useState("add_balance");
  const [syncProgress, setSyncProgress] = useState(67);
  const [deviceAStatus, setDeviceAStatus] = useState("syncing");

  // Deposit address
  const [depositAddr, setDepositAddr] = useState("");

  // Live price simulation
  useEffect(() => {
    const interval = setInterval(() => {
      setCoinData((prev) =>
        prev.map((coin) => {
          const delta = (Math.random() - 0.49) * coin.price * 0.003;
          const newPrice = Math.max(coin.price + delta, 0.0001);
          return { ...coin, price: newPrice };
        })
      );
    }, 2000);
    setTimeout(() => setLoading(false), 1200);
    return () => clearInterval(interval);
  }, []);

  useEffect(() => {
    const map = {};
    coinData.forEach((c) => (map[c.symbol] = c.price));
    setPrices(map);
  }, [coinData]);

  // Toast helper
  const toast = useCallback((msg, type = "info", duration = 3000) => {
    const id = Date.now();
    setToasts((p) => [...p, { id, msg, type }]);
    setTimeout(() => setToasts((p) => p.filter((t) => t.id !== id)), duration);
  }, []);

  // Portfolio total
  const totalUSD = Object.entries(portfolio).reduce((sum, [sym, { amount }]) => {
    const p = prices[sym] || (sym === "USDT" ? 1 : 0);
    return sum + amount * p;
  }, 0);

  const dailyPnL = totalUSD * 0.0234; // mock

  // ── Auth Handlers ──────────────────────────────────────────────────────────
  const handleLogin = () => {
    if (!authForm.email || !authForm.password) { toast("Please fill all fields", "error"); return; }
    const otp = Math.floor(100000 + Math.random() * 900000).toString();
    setGeneratedOTP(otp);
    setOtpSent(true);
    setAuthMode("otp");
    toast(`OTP sent: ${otp}`, "info", 8000);
  };

  const handleOTP = () => {
    if (authForm.otp === generatedOTP) {
      setAuthed(true);
      toast("Welcome back!", "success");
    } else {
      toast("Invalid OTP", "error");
    }
  };

  // ── Admin Handler ──────────────────────────────────────────────────────────
  const ADMIN_PASSKEY = "ADMIN@CRYPTO2024";
  const handleAdminKey = () => {
    if (adminKey === ADMIN_PASSKEY) {
      setAdminMode(true);
      toast("Admin panel unlocked", "success");
    } else {
      toast("Invalid passkey", "error");
    }
  };

  const handleAdminAction = () => {
    const amt = parseFloat(adminAmount);
    if (isNaN(amt)) { toast("Invalid amount", "error"); return; }
    if (adminAction === "add_balance") {
      setPortfolio((p) => ({ ...p, [adminCoin]: { ...p[adminCoin], amount: (p[adminCoin]?.amount || 0) + amt } }));
      toast(`Added ${amt} ${adminCoin}`, "success");
    } else if (adminAction === "set_balance") {
      setPortfolio((p) => ({ ...p, [adminCoin]: { amount: amt, locked: 0 } }));
      toast(`Balance set to ${amt} ${adminCoin}`, "success");
    } else if (adminAction === "approve_deposit") {
      setTransactions((p) => p.map((t) => t.status === "pending" ? { ...t, status: "completed" } : t));
      toast("All pending deposits approved", "success");
    } else if (adminAction === "add_transaction") {
      const tx = { id: `tx${Date.now()}`, type: "deposit", coin: adminCoin, amount: amt, status: "completed", network: "Admin", fee: "0", hash: generateTxHash(), ts: Date.now(), usdVal: amt * (prices[adminCoin] || 1) };
      setTransactions((p) => [tx, ...p]);
      toast("Transaction added", "success");
    } else if (adminAction === "sync_progress") {
      setSyncProgress(Math.min(100, Math.max(0, amt)));
      toast(`Sync progress set to ${amt}%`, "success");
    }
    setAdminAmount("");
  };

  // ── Deposit flow ───────────────────────────────────────────────────────────
  const startDeposit = (coin = "BTC") => {
    setFlowCoin(coin);
    setFlowStep(1);
    setFlowNetwork(null);
    setDepositAddr("");
    setModalOpen("deposit");
  };

  const handleGenerateAddress = () => {
    if (!flowNetwork) { toast("Select a network", "error"); return; }
    const addr = generateAddress(flowCoin, flowNetwork);
    setDepositAddr(addr);
    setFlowStep(2);
  };

  // ── Withdraw/Send flow ─────────────────────────────────────────────────────
  const startWithdraw = (coin = "BTC") => {
    setFlowCoin(coin);
    setFlowStep(1);
    setFlowNetwork(null);
    setFlowAddress("");
    setFlowAmount("");
    setModalOpen("withdraw");
  };

  const handleWithdrawConfirm = async () => {
    setFlowStep(4);
    await sleep(2500);
    const amt = parseFloat(flowAmount);
    if (amt > (portfolio[flowCoin]?.amount || 0)) {
      setFlowStep(1);
      toast("Insufficient balance", "error");
      return;
    }
    setPortfolio((p) => ({ ...p, [flowCoin]: { ...p[flowCoin], amount: p[flowCoin].amount - amt } }));
    const tx = { id: `tx${Date.now()}`, type: "withdrawal", coin: flowCoin, amount: amt, status: "pending", network: flowNetwork?.name || "Unknown", fee: flowNetwork?.fee || "0", hash: generateTxHash(), ts: Date.now(), usdVal: amt * (prices[flowCoin] || 1) };
    setTransactions((p) => [tx, ...p]);
    setFlowStep(5);
    toast("Withdrawal submitted!", "success");
  };

  // ── Receive ────────────────────────────────────────────────────────────────
  const startReceive = (coin = "BTC") => {
    setFlowCoin(coin);
    const nets = NETWORKS[coin] || NETWORKS.default;
    setFlowNetwork(nets[0]);
    setDepositAddr(generateAddress(coin, nets[0]));
    setModalOpen("receive");
  };

  // ── Trade ──────────────────────────────────────────────────────────────────
  const handleTrade = () => {
    const amt = parseFloat(tradeAmount);
    if (!amt || amt <= 0) { toast("Enter valid amount", "error"); return; }
    const [base, quote] = tradePair.split("/");
    const price = prices[base] || coinData.find((c) => c.symbol === base)?.price || 0;
    if (tradeTab === "buy") {
      const cost = amt * price;
      if ((portfolio.USDT?.amount || 0) < cost) { toast("Insufficient USDT", "error"); return; }
      setPortfolio((p) => ({
        ...p,
        [base]: { ...p[base], amount: (p[base]?.amount || 0) + amt },
        USDT: { ...p.USDT, amount: p.USDT.amount - cost },
      }));
      const tx = { id: `tx${Date.now()}`, type: "buy", coin: base, amount: amt, status: "completed", network: "Binance", fee: `${(cost * 0.001).toFixed(4)} USDT`, hash: generateTxHash(), ts: Date.now(), usdVal: cost };
      setTransactions((p) => [tx, ...p]);
      toast(`Bought ${amt} ${base}`, "success");
    } else {
      if ((portfolio[base]?.amount || 0) < amt) { toast(`Insufficient ${base}`, "error"); return; }
      const revenue = amt * price;
      setPortfolio((p) => ({
        ...p,
        [base]: { ...p[base], amount: p[base].amount - amt },
        USDT: { ...p.USDT, amount: (p.USDT?.amount || 0) + revenue },
      }));
      const tx = { id: `tx${Date.now()}`, type: "sell", coin: base, amount: amt, status: "completed", network: "Binance", fee: `${(revenue * 0.001).toFixed(4)} USDT`, hash: generateTxHash(), ts: Date.now(), usdVal: revenue };
      setTransactions((p) => [tx, ...p]);
      toast(`Sold ${amt} ${base}`, "success");
    }
    setTradeAmount("");
  };

  // ── Network list for coin ──────────────────────────────────────────────────
  const getNets = (coin) => NETWORKS[coin] || NETWORKS.default;

  // ── Filtered markets ───────────────────────────────────────────────────────
  const filteredCoins = coinData.filter((c) => {
    const matchSearch = c.name.toLowerCase().includes(marketSearch.toLowerCase()) || c.symbol.toLowerCase().includes(marketSearch.toLowerCase());
    if (!matchSearch) return false;
    if (marketFilter === "favorites") return favorites.has(c.symbol);
    if (marketFilter === "gainers") return c.change > 0;
    if (marketFilter === "losers") return c.change < 0;
    return true;
  });

  // ── CSS ────────────────────────────────────────────────────────────────────
  const css = `
    @import url('https://fonts.googleapis.com/css2?family=Syne:wght@400;600;700;800&family=JetBrains+Mono:wght@400;500;600&display=swap');
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { background: ${C.bg0}; color: ${C.text1}; font-family: 'Syne', sans-serif; }
    ::-webkit-scrollbar { width: 4px; height: 4px; }
    ::-webkit-scrollbar-track { background: ${C.bg1}; }
    ::-webkit-scrollbar-thumb { background: ${C.bg4}; border-radius: 2px; }
    @keyframes shimmer { 0%{background-position:200% 0} 100%{background-position:-200% 0} }
    @keyframes slideIn { from{transform:translateX(20px);opacity:0} to{transform:translateX(0);opacity:1} }
    @keyframes fadeIn { from{opacity:0;transform:translateY(8px)} to{opacity:1;transform:translateY(0)} }
    @keyframes spin { to{transform:rotate(360deg)} }
    @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:0.5} }
    @keyframes ping { 0%{transform:scale(1);opacity:1} 75%,100%{transform:scale(2);opacity:0} }
    .page { animation: fadeIn 0.3s ease; }
    .hover-row:hover { background: ${C.bg2} !important; cursor: pointer; }
    .nav-item:hover { color: ${C.accent} !important; }
    .btn-primary { background: ${C.accent}; color: #000; border: none; border-radius: 6px; padding: 10px 20px; font-family: 'Syne', sans-serif; font-weight: 700; font-size: 14px; cursor: pointer; transition: all 0.2s; }
    .btn-primary:hover { background: ${C.accentDark}; transform: translateY(-1px); }
    .btn-ghost { background: transparent; color: ${C.text1}; border: 1px solid ${C.border}; border-radius: 6px; padding: 8px 16px; font-family: 'Syne', sans-serif; font-weight: 600; font-size: 13px; cursor: pointer; transition: all 0.2s; }
    .btn-ghost:hover { border-color: ${C.accent}; color: ${C.accent}; }
    .btn-red { background: ${C.red}; color: #fff; border: none; border-radius: 6px; padding: 10px 20px; font-family: 'Syne', sans-serif; font-weight: 700; font-size: 14px; cursor: pointer; transition: all 0.2s; }
    .btn-red:hover { opacity: 0.85; }
    .input { background: ${C.bg3}; border: 1px solid ${C.border}; border-radius: 6px; padding: 10px 14px; color: ${C.text1}; font-family: 'Syne', sans-serif; font-size: 14px; width: 100%; outline: none; transition: border 0.2s; }
    .input:focus { border-color: ${C.accent}; }
    .card { background: ${C.bg1}; border: 1px solid ${C.border}; border-radius: 12px; padding: 20px; }
    .mono { font-family: 'JetBrains Mono', monospace; }
    .green { color: ${C.green}; }
    .red { color: ${C.red}; }
    .accent { color: ${C.accent}; }
    .tab { padding: 8px 16px; border-radius: 6px; cursor: pointer; font-weight: 600; font-size: 13px; transition: all 0.2s; color: ${C.text2}; border: none; background: transparent; font-family: 'Syne', sans-serif; }
    .tab.active { background: ${C.accentGlow}; color: ${C.accent}; }
    .tab:hover:not(.active) { color: ${C.text1}; }
    .modal-overlay { position: fixed; inset: 0; background: rgba(0,0,0,0.7); z-index: 1000; display: flex; align-items: center; justify-content: center; padding: 16px; backdrop-filter: blur(4px); }
    .modal { background: ${C.bg1}; border: 1px solid ${C.border}; border-radius: 16px; width: 100%; max-width: 480px; max-height: 90vh; overflow-y: auto; animation: fadeIn 0.2s ease; }
    .select { background: ${C.bg3}; border: 1px solid ${C.border}; border-radius: 6px; padding: 10px 14px; color: ${C.text1}; font-family: 'Syne', sans-serif; font-size: 14px; outline: none; cursor: pointer; }
    .price-up { animation: flashGreen 0.5s ease; }
    .price-down { animation: flashRed 0.5s ease; }
    @keyframes flashGreen { 0%{color:${C.green}} 100%{color:inherit} }
    @keyframes flashRed { 0%{color:${C.red}} 100%{color:inherit} }
    .sidebar-link { display: flex; align-items: center; gap: 10px; padding: 10px 16px; border-radius: 8px; cursor: pointer; transition: all 0.2s; color: ${C.text2}; font-weight: 600; font-size: 14px; border: none; background: transparent; font-family: 'Syne', sans-serif; width: 100%; text-align: left; }
    .sidebar-link:hover { background: ${C.bg3}; color: ${C.text1}; }
    .sidebar-link.active { background: ${C.accentGlow}; color: ${C.accent}; }
    .spinner { width: 40px; height: 40px; border: 3px solid ${C.bg3}; border-top-color: ${C.accent}; border-radius: 50%; animation: spin 0.8s linear infinite; }
    @media (max-width: 768px) {
      .sidebar-desktop { display: none !important; }
      .main-content { margin-left: 0 !important; }
    }
    @media (min-width: 769px) {
      .bottom-nav { display: none !important; }
    }
  `;

  // ── Auth Screen ────────────────────────────────────────────────────────────
  if (!authed) {
    return (
      <>
        <style>{css}</style>
        <div style={{ minHeight: "100vh", background: C.bg0, display: "flex", alignItems: "center", justifyContent: "center", padding: 16 }}>
          <div style={{ width: "100%", maxWidth: 420 }}>
            {/* Logo */}
            <div style={{ textAlign: "center", marginBottom: 40 }}>
              <div style={{ display: "flex", alignItems: "center", justifyContent: "center", gap: 10, marginBottom: 8 }}>
                <div style={{ background: C.accent, borderRadius: 8, width: 36, height: 36, display: "flex", alignItems: "center", justifyContent: "center" }}>
                  <span style={{ color: "#000", fontWeight: 800, fontSize: 18 }}>B</span>
                </div>
                <span style={{ fontSize: 28, fontWeight: 800, color: C.accent }}>Binance</span>
              </div>
              <p style={{ color: C.text2, fontSize: 13 }}>The World's Leading Crypto Exchange</p>
            </div>

            <div className="card" style={{ borderRadius: 16 }}>
              {authMode !== "otp" ? (
                <>
                  <div style={{ display: "flex", gap: 8, marginBottom: 24 }}>
                    {["login", "signup"].map((m) => (
                      <button key={m} className={`tab ${authMode === m ? "active" : ""}`} onClick={() => setAuthMode(m)} style={{ flex: 1, textTransform: "capitalize" }}>
                        {m === "login" ? "Log In" : "Sign Up"}
                      </button>
                    ))}
                  </div>
                  <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
                    <input className="input" placeholder="Email address" value={authForm.email} onChange={(e) => setAuthForm((p) => ({ ...p, email: e.target.value }))} />
                    <input className="input" type="password" placeholder="Password" value={authForm.password} onChange={(e) => setAuthForm((p) => ({ ...p, password: e.target.value }))} />
                    {authMode === "signup" && <input className="input" type="password" placeholder="Confirm Password" />}
                    <button className="btn-primary" onClick={handleLogin} style={{ width: "100%", marginTop: 8 }}>
                      {authMode === "login" ? "Log In" : "Create Account"}
                    </button>
                  </div>
                  <p style={{ color: C.text3, fontSize: 11, textAlign: "center", marginTop: 16 }}>
                    Demo: any email/password → OTP shown in notification
                  </p>
                </>
              ) : (
                <>
                  <h3 style={{ marginBottom: 8, fontSize: 18 }}>OTP Verification</h3>
                  <p style={{ color: C.text2, fontSize: 13, marginBottom: 20 }}>Enter the 6-digit code sent to {authForm.email}</p>
                  <input className="input mono" placeholder="Enter OTP" value={authForm.otp} onChange={(e) => setAuthForm((p) => ({ ...p, otp: e.target.value }))} style={{ letterSpacing: 8, textAlign: "center", fontSize: 20 }} maxLength={6} />
                  <button className="btn-primary" onClick={handleOTP} style={{ width: "100%", marginTop: 16 }}>Verify</button>
                  <button className="btn-ghost" onClick={() => setAuthMode("login")} style={{ width: "100%", marginTop: 8 }}>Back</button>
                </>
              )}
            </div>
            <p style={{ textAlign: "center", color: C.text3, fontSize: 11, marginTop: 16, lineHeight: 1.6 }}>
              ⚠️ DISCLAIMER: This is a Binance UI clone for portfolio/demo purposes only. No real cryptocurrency transactions are processed.
            </p>
          </div>
        </div>
        <ToastContainer toasts={toasts} />
      </>
    );
  }

  // ── Admin Panel ────────────────────────────────────────────────────────────
  if (page === "admin") {
    return (
      <>
        <style>{css}</style>
        <div style={{ minHeight: "100vh", background: C.bg0, padding: 24 }}>
          <div style={{ maxWidth: 600, margin: "0 auto" }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 24 }}>
              <h2 style={{ color: C.accent, fontWeight: 800 }}>🔑 Admin Panel</h2>
              <button className="btn-ghost" onClick={() => setPage("dashboard")}>← Back</button>
            </div>
            {!adminMode ? (
              <div className="card">
                <h3 style={{ marginBottom: 16 }}>Enter Admin Passkey</h3>
                <input className="input mono" type="password" placeholder="Admin passkey" value={adminKey} onChange={(e) => setAdminKey(e.target.value)} />
                <button className="btn-primary" onClick={handleAdminKey} style={{ marginTop: 12, width: "100%" }}>Unlock</button>
                <p style={{ color: C.text3, fontSize: 11, marginTop: 12 }}>Hint: ADMIN@CRYPTO2024</p>
              </div>
            ) : (
              <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
                <div className="card" style={{ borderColor: C.accent }}>
                  <h3 style={{ marginBottom: 4, color: C.accent }}>✅ Admin Access Granted</h3>
                  <p style={{ color: C.text2, fontSize: 13 }}>Full control over balances, transactions, and sync.</p>
                </div>
                <div className="card">
                  <h3 style={{ marginBottom: 16 }}>Balance Control</h3>
                  <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
                    <div style={{ display: "flex", gap: 10 }}>
                      <select className="select" value={adminCoin} onChange={(e) => setAdminCoin(e.target.value)} style={{ flex: 1 }}>
                        {[...COINS.map((c) => c.symbol), "USDT"].map((s) => <option key={s}>{s}</option>)}
                      </select>
                      <select className="select" value={adminAction} onChange={(e) => setAdminAction(e.target.value)} style={{ flex: 2 }}>
                        <option value="add_balance">Add Balance</option>
                        <option value="set_balance">Set Balance</option>
                        <option value="approve_deposit">Approve All Pending</option>
                        <option value="add_transaction">Add Transaction</option>
                        <option value="sync_progress">Set Sync %</option>
                      </select>
                    </div>
                    <input className="input" placeholder="Amount" value={adminAmount} onChange={(e) => setAdminAmount(e.target.value)} type="number" />
                    <button className="btn-primary" onClick={handleAdminAction}>Execute</button>
                  </div>
                </div>
                <div className="card">
                  <h3 style={{ marginBottom: 12 }}>Current Balances</h3>
                  {Object.entries(portfolio).map(([sym, { amount }]) => (
                    <div key={sym} style={{ display: "flex", justifyContent: "space-between", padding: "6px 0", borderBottom: `1px solid ${C.border}` }}>
                      <span>{sym}</span>
                      <span className="mono accent">{fmt(amount, 6)}</span>
                    </div>
                  ))}
                </div>
                <div className="card">
                  <h3 style={{ marginBottom: 12 }}>Device Sync</h3>
                  <p style={{ color: C.text2, fontSize: 13, marginBottom: 12 }}>Device A Sync: {syncProgress}%</p>
                  <div style={{ background: C.bg3, borderRadius: 8, height: 8, overflow: "hidden" }}>
                    <div style={{ width: `${syncProgress}%`, height: "100%", background: C.accent, transition: "width 0.5s" }} />
                  </div>
                  <div style={{ display: "flex", gap: 8, marginTop: 12 }}>
                    <button className="btn-ghost" onClick={() => { setSyncProgress(100); setDeviceAStatus("synced"); toast("Sync completed!", "success"); }} style={{ flex: 1 }}>Complete Sync</button>
                    <button className="btn-ghost" onClick={() => { setSyncProgress(0); setDeviceAStatus("syncing"); }} style={{ flex: 1 }}>Reset Sync</button>
                  </div>
                </div>
                <div className="card">
                  <h3 style={{ marginBottom: 12 }}>Transaction Control</h3>
                  {transactions.slice(0, 5).map((tx) => (
                    <div key={tx.id} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "6px 0", borderBottom: `1px solid ${C.border}` }}>
                      <span style={{ fontSize: 13 }}>{tx.type} {tx.amount} {tx.coin}</span>
                      <div style={{ display: "flex", gap: 6, alignItems: "center" }}>
                        <StatusBadge status={tx.status} />
                        {tx.status === "pending" && (
                          <button className="btn-primary" style={{ padding: "3px 8px", fontSize: 11 }} onClick={() => {
                            setTransactions((p) => p.map((t) => t.id === tx.id ? { ...t, status: "completed" } : t));
                            toast("Transaction approved", "success");
                          }}>Approve</button>
                        )}
                      </div>
                    </div>
                  ))}
                </div>
              </div>
            )}
          </div>
        </div>
        <ToastContainer toasts={toasts} />
      </>
    );
  }

  // ── Page Content ───────────────────────────────────────────────────────────
  const renderPage = () => {
    switch (page) {
      // ── DASHBOARD ──────────────────────────────────────────────────────────
      case "dashboard": return (
        <div className="page" style={{ display: "flex", flexDirection: "column", gap: 20 }}>
          {/* Portfolio Overview */}
          <div className="card" style={{ background: `linear-gradient(135deg, ${C.bg2} 0%, ${C.bg1} 100%)`, borderColor: C.border }}>
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "flex-start", flexWrap: "wrap", gap: 16 }}>
              <div>
                <p style={{ color: C.text2, fontSize: 12, fontWeight: 600, textTransform: "uppercase", letterSpacing: 1 }}>Total Portfolio Value</p>
                <div style={{ display: "flex", alignItems: "center", gap: 12, marginTop: 8 }}>
                  {showBalance ? (
                    <h1 className="mono" style={{ fontSize: "clamp(24px, 5vw, 40px)", fontWeight: 800, color: C.text1 }}>
                      {currency === "USD" ? `$${fmt(totalUSD)}` : fmtINR(totalUSD)}
                    </h1>
                  ) : (
                    <h1 style={{ fontSize: 40, fontWeight: 800, color: C.text3 }}>••••••</h1>
                  )}
                  <button onClick={() => setShowBalance((p) => !p)} style={{ background: "none", border: "none", color: C.text2, cursor: "pointer", fontSize: 18 }}>{showBalance ? "👁" : "👁‍🗨"}</button>
                </div>
                <div style={{ display: "flex", alignItems: "center", gap: 8, marginTop: 6 }}>
                  <span style={{ color: C.green, fontWeight: 700, fontSize: 14 }}>▲ +${fmt(dailyPnL)} (+2.34%)</span>
                  <span style={{ color: C.text3, fontSize: 12 }}>Today</span>
                </div>
              </div>
              <div style={{ display: "flex", gap: 8 }}>
                <button className={`tab ${currency === "USD" ? "active" : ""}`} onClick={() => setCurrency("USD")}>USD</button>
                <button className={`tab ${currency === "INR" ? "active" : ""}`} onClick={() => setCurrency("INR")}>INR</button>
              </div>
            </div>

            {/* Quick Actions */}
            <div style={{ display: "flex", gap: 10, marginTop: 24, flexWrap: "wrap" }}>
              {[
                { label: "Deposit", icon: "↓", action: () => startDeposit(), color: C.green },
                { label: "Withdraw", icon: "↑", action: () => startWithdraw(), color: C.red },
                { label: "Send", icon: "→", action: () => { setFlowCoin("BTC"); setModalOpen("send"); setFlowStep(1); } },
                { label: "Receive", icon: "←", action: () => startReceive() },
              ].map(({ label, icon, action, color }) => (
                <button key={label} onClick={action} style={{ display: "flex", alignItems: "center", gap: 8, background: C.bg3, border: `1px solid ${C.border}`, borderRadius: 10, padding: "10px 18px", cursor: "pointer", color: color || C.text1, fontFamily: "'Syne', sans-serif", fontWeight: 700, fontSize: 13, transition: "all 0.2s" }}
                  onMouseEnter={(e) => e.currentTarget.style.borderColor = color || C.accent}
                  onMouseLeave={(e) => e.currentTarget.style.borderColor = C.border}>
                  <span style={{ fontSize: 16 }}>{icon}</span> {label}
                </button>
              ))}
            </div>
          </div>

          {/* Holdings */}
          <div style={{ display: "grid", gridTemplateColumns: "1fr 1fr", gap: 20 }}>
            <div className="card" style={{ gridColumn: "1 / -1" }}>
              <h3 style={{ marginBottom: 16, fontWeight: 700 }}>My Holdings</h3>
              {loading ? (
                <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
                  {[1, 2, 3].map((i) => <Skeleton key={i} h={48} r={8} />)}
                </div>
              ) : (
                Object.entries(portfolio).filter(([, { amount }]) => amount > 0).map(([sym, { amount }]) => {
                  const price = prices[sym] || (sym === "USDT" ? 1 : 0);
                  const usdVal = amount * price;
                  const pct = totalUSD > 0 ? (usdVal / totalUSD) * 100 : 0;
                  const coin = COINS.find((c) => c.symbol === sym);
                  return (
                    <div key={sym} className="hover-row" style={{ display: "flex", alignItems: "center", gap: 14, padding: "12px 8px", borderRadius: 8, transition: "background 0.2s" }}>
                      {sym === "USDT" ? (
                        <div style={{ width: 36, height: 36, borderRadius: "50%", background: "#26A17B", display: "flex", alignItems: "center", justifyContent: "center", fontSize: 16, color: "#fff", fontWeight: 700, flexShrink: 0 }}>₮</div>
                      ) : (
                        <CoinIcon coin={sym} size={36} />
                      )}
                      <div style={{ flex: 1 }}>
                        <div style={{ display: "flex", justifyContent: "space-between" }}>
                          <span style={{ fontWeight: 700 }}>{sym}</span>
                          <span className="mono" style={{ fontWeight: 600 }}>{currency === "USD" ? `$${fmt(usdVal)}` : fmtINR(usdVal)}</span>
                        </div>
                        <div style={{ display: "flex", justifyContent: "space-between", marginTop: 4 }}>
                          <span style={{ color: C.text2, fontSize: 12 }} className="mono">{fmt(amount, sym === "USDT" ? 2 : 6)} {sym}</span>
                          <span style={{ color: C.text2, fontSize: 12 }}>{fmt(pct, 1)}%</span>
                        </div>
                        <div style={{ height: 3, background: C.bg3, borderRadius: 2, marginTop: 6 }}>
                          <div style={{ width: `${pct}%`, height: "100%", background: coin?.color || C.accent, borderRadius: 2, transition: "width 0.5s" }} />
                        </div>
                      </div>
                    </div>
                  );
                })
              )}
            </div>
          </div>

          {/* Recent Transactions */}
          <div className="card">
            <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 16 }}>
              <h3 style={{ fontWeight: 700 }}>Recent Transactions</h3>
              <button className="tab" onClick={() => setPage("history")} style={{ fontSize: 12 }}>View All →</button>
            </div>
            {transactions.slice(0, 4).map((tx) => (
              <div key={tx.id} style={{ display: "flex", alignItems: "center", gap: 14, padding: "10px 0", borderBottom: `1px solid ${C.border}` }}>
                <div style={{ width: 36, height: 36, borderRadius: "50%", background: tx.type === "deposit" || tx.type === "receive" ? `${C.green}20` : `${C.red}20`, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 16 }}>
                  {tx.type === "deposit" || tx.type === "receive" ? "↓" : "↑"}
                </div>
                <div style={{ flex: 1 }}>
                  <div style={{ display: "flex", justifyContent: "space-between" }}>
                    <span style={{ fontWeight: 600, textTransform: "capitalize" }}>{tx.type} {tx.coin}</span>
                    <span className={`mono ${tx.type === "deposit" || tx.type === "receive" ? "green" : "red"}`} style={{ fontWeight: 700 }}>
                      {tx.type === "deposit" || tx.type === "receive" ? "+" : "-"}{tx.amount} {tx.coin}
                    </span>
                  </div>
                  <div style={{ display: "flex", justifyContent: "space-between", marginTop: 4 }}>
                    <span style={{ color: C.text3, fontSize: 12 }}>{new Date(tx.ts).toLocaleString()}</span>
                    <StatusBadge status={tx.status} />
                  </div>
                </div>
              </div>
            ))}
          </div>

          {/* Device Sync */}
          <div className="card" style={{ borderColor: syncProgress < 100 ? C.accent + "40" : C.green + "40" }}>
            <h3 style={{ marginBottom: 16, fontWeight: 700 }}>Device Sync Status</h3>
            <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
              {[
                { name: "Device A (This device)", status: syncProgress < 100 ? `Syncing... ${syncProgress}%` : "Synced ✓", color: syncProgress < 100 ? C.accent : C.green },
                { name: "Device B (Mobile)", status: syncProgress < 100 ? "Waiting for authorization..." : "Authorized ✓", color: syncProgress < 100 ? C.text2 : C.green },
              ].map(({ name, status, color }) => (
                <div key={name} style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
                  <div>
                    <p style={{ fontWeight: 600, fontSize: 14 }}>{name}</p>
                    <p style={{ color, fontSize: 12, marginTop: 2 }}>{status}</p>
                  </div>
                  {syncProgress < 100 && name.includes("Device A") && (
                    <div style={{ width: 100, background: C.bg3, borderRadius: 4, height: 6 }}>
                      <div style={{ width: `${syncProgress}%`, height: "100%", background: C.accent, borderRadius: 4, transition: "width 0.5s" }} />
                    </div>
                  )}
                </div>
              ))}
            </div>
          </div>
        </div>
      );

      // ── MARKETS ────────────────────────────────────────────────────────────
      case "markets": return (
        <div className="page">
          <div className="card">
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 16, flexWrap: "wrap", gap: 12 }}>
              <h3 style={{ fontWeight: 700 }}>Crypto Markets</h3>
              <div style={{ display: "flex", gap: 8, flexWrap: "wrap" }}>
                {["all", "favorites", "gainers", "losers"].map((f) => (
                  <button key={f} className={`tab ${marketFilter === f ? "active" : ""}`} onClick={() => setMarketFilter(f)} style={{ textTransform: "capitalize" }}>{f === "favorites" ? "⭐ Watchlist" : f}</button>
                ))}
              </div>
            </div>
            <input className="input" placeholder="🔍 Search coins..." value={marketSearch} onChange={(e) => setMarketSearch(e.target.value)} style={{ marginBottom: 16 }} />
            <div style={{ overflowX: "auto" }}>
              <table style={{ width: "100%", borderCollapse: "collapse", fontSize: 13 }}>
                <thead>
                  <tr style={{ color: C.text2, textAlign: "left", borderBottom: `1px solid ${C.border}` }}>
                    {["#", "Name", "Price", "24h %", "Volume", "Market Cap", "7D Chart", ""].map((h) => (
                      <th key={h} style={{ padding: "8px 10px", fontWeight: 600 }}>{h}</th>
                    ))}
                  </tr>
                </thead>
                <tbody>
                  {filteredCoins.map((coin, i) => (
                    <tr key={coin.id} className="hover-row" style={{ borderBottom: `1px solid ${C.border}20`, transition: "background 0.2s" }} onClick={() => { setTradePair(`${coin.symbol}/USDT`); setPage("trade"); }}>
                      <td style={{ padding: "12px 10px", color: C.text3 }}>
                        <button onClick={(e) => { e.stopPropagation(); setFavorites((p) => { const n = new Set(p); n.has(coin.symbol) ? n.delete(coin.symbol) : n.add(coin.symbol); return n; }); }} style={{ background: "none", border: "none", cursor: "pointer", fontSize: 14 }}>
                          {favorites.has(coin.symbol) ? "⭐" : "☆"}
                        </button>
                        {i + 1}
                      </td>
                      <td style={{ padding: "12px 10px" }}>
                        <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                          <CoinIcon coin={coin.symbol} size={28} />
                          <div>
                            <p style={{ fontWeight: 700 }}>{coin.symbol}</p>
                            <p style={{ color: C.text2, fontSize: 11 }}>{coin.name}</p>
                          </div>
                        </div>
                      </td>
                      <td className="mono" style={{ padding: "12px 10px", fontWeight: 600 }}>${fmt(coin.price, coin.price < 1 ? 4 : 2)}</td>
                      <td style={{ padding: "12px 10px" }}>
                        <span style={{ color: coin.change >= 0 ? C.green : C.red, fontWeight: 700 }}>
                          {coin.change >= 0 ? "▲" : "▼"} {Math.abs(coin.change).toFixed(2)}%
                        </span>
                      </td>
                      <td style={{ padding: "12px 10px", color: C.text2 }}>{fmtUSD(coin.vol)}</td>
                      <td style={{ padding: "12px 10px", color: C.text2 }}>{fmtUSD(coin.mcap)}</td>
                      <td style={{ padding: "12px 10px" }}><Sparkline positive={coin.change >= 0} /></td>
                      <td style={{ padding: "12px 10px" }}>
                        <button className="btn-primary" style={{ padding: "5px 14px", fontSize: 12 }} onClick={(e) => { e.stopPropagation(); setTradePair(`${coin.symbol}/USDT`); setPage("trade"); }}>Trade</button>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      );

      // ── TRADE ──────────────────────────────────────────────────────────────
      case "trade": {
        const [baseCoin] = tradePair.split("/");
        const coinInfo = coinData.find((c) => c.symbol === baseCoin);
        const currentPrice = prices[baseCoin] || coinInfo?.price || 0;
        const estTotal = (parseFloat(tradeAmount) || 0) * (orderType === "market" ? currentPrice : parseFloat(tradePrice) || currentPrice);
        const orderBook = Array.from({ length: 10 }, (_, i) => ({
          price: currentPrice * (tradeTab === "buy" ? 1 - (i + 1) * 0.001 : 1 + (i + 1) * 0.001),
          amount: Math.random() * 5 + 0.1,
          total: 0,
        }));
        return (
          <div className="page" style={{ display: "flex", flexDirection: "column", gap: 16 }}>
            {/* Pair selector */}
            <div className="card" style={{ padding: "12px 20px" }}>
              <div style={{ display: "flex", alignItems: "center", gap: 16, flexWrap: "wrap" }}>
                <div style={{ display: "flex", alignItems: "center", gap: 10 }}>
                  <CoinIcon coin={baseCoin} size={32} />
                  <div>
                    <p style={{ fontWeight: 800, fontSize: 18 }}>{tradePair}</p>
                    <p style={{ color: coinInfo?.change >= 0 ? C.green : C.red, fontSize: 12, fontWeight: 700 }}>
                      {coinInfo?.change >= 0 ? "▲" : "▼"} {Math.abs(coinInfo?.change || 0).toFixed(2)}%
                    </p>
                  </div>
                </div>
                <h2 className="mono" style={{ fontSize: 28, fontWeight: 800, color: coinInfo?.change >= 0 ? C.green : C.text1 }}>
                  ${fmt(currentPrice, currentPrice < 1 ? 4 : 2)}
                </h2>
                <div style={{ marginLeft: "auto", display: "flex", gap: 8, flexWrap: "wrap" }}>
                  {coinData.slice(0, 5).map((c) => (
                    <button key={c.id} className={`tab ${baseCoin === c.symbol ? "active" : ""}`} onClick={() => setTradePair(`${c.symbol}/USDT`)} style={{ fontSize: 12 }}>
                      {c.symbol}/USDT
                    </button>
                  ))}
                </div>
              </div>
            </div>

            <div style={{ display: "grid", gridTemplateColumns: "1fr 300px", gap: 16 }}>
              {/* Chart area (mock) */}
              <div className="card" style={{ minHeight: 360, position: "relative", overflow: "hidden" }}>
                <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 12 }}>
                  <div style={{ display: "flex", gap: 6 }}>
                    {["1m", "5m", "15m", "1h", "4h", "1D"].map((t) => (
                      <button key={t} className="tab" style={{ fontSize: 11, padding: "4px 8px" }}>{t}</button>
                    ))}
                  </div>
                </div>
                {/* Mock candlestick chart */}
                <svg width="100%" height="280" viewBox="0 0 600 280" preserveAspectRatio="none">
                  <defs>
                    <linearGradient id="chartGrad" x1="0" y1="0" x2="0" y2="1">
                      <stop offset="0%" stopColor={C.accent} stopOpacity="0.3" />
                      <stop offset="100%" stopColor={C.accent} stopOpacity="0" />
                    </linearGradient>
                  </defs>
                  {Array.from({ length: 40 }, (_, i) => {
                    const x = (i / 39) * 580 + 10;
                    const baseH = 140 + Math.sin(i * 0.5) * 60 + Math.random() * 30;
                    const open = baseH;
                    const close = baseH + (Math.random() - 0.48) * 20;
                    const high = Math.min(open, close) - Math.random() * 10;
                    const low = Math.max(open, close) + Math.random() * 10;
                    const isGreen = close < open;
                    return (
                      <g key={i}>
                        <line x1={x} y1={high} x2={x} y2={low} stroke={isGreen ? C.green : C.red} strokeWidth="1" />
                        <rect x={x - 5} y={Math.min(open, close)} width={10} height={Math.abs(close - open) || 1} fill={isGreen ? C.green : C.red} />
                      </g>
                    );
                  })}
                  <text x="10" y="20" fill={C.text2} fontSize="12">${fmt(currentPrice, 2)}</text>
                </svg>
                <p style={{ color: C.text3, fontSize: 11, textAlign: "center", marginTop: 8 }}>Live price simulation — chart for visualization purposes</p>
              </div>

              {/* Order panel */}
              <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
                <div className="card">
                  <div style={{ display: "flex", gap: 6, marginBottom: 16 }}>
                    <button className={`tab ${tradeTab === "buy" ? "active" : ""}`} onClick={() => setTradeTab("buy")} style={{ flex: 1, color: tradeTab === "buy" ? C.green : undefined }}>Buy</button>
                    <button className={`tab ${tradeTab === "sell" ? "active" : ""}`} onClick={() => setTradeTab("sell")} style={{ flex: 1, color: tradeTab === "sell" ? C.red : undefined }}>Sell</button>
                  </div>
                  <div style={{ display: "flex", gap: 6, marginBottom: 14 }}>
                    {["market", "limit", "stop"].map((o) => (
                      <button key={o} className={`tab ${orderType === o ? "active" : ""}`} onClick={() => setOrderType(o)} style={{ flex: 1, fontSize: 11, textTransform: "capitalize" }}>{o}</button>
                    ))}
                  </div>
                  <div style={{ display: "flex", flexDirection: "column", gap: 10 }}>
                    {orderType !== "market" && (
                      <div>
                        <label style={{ color: C.text2, fontSize: 12, marginBottom: 4, display: "block" }}>Price (USDT)</label>
                        <input className="input mono" placeholder={fmt(currentPrice)} value={tradePrice} onChange={(e) => setTradePrice(e.target.value)} />
                      </div>
                    )}
                    <div>
                      <label style={{ color: C.text2, fontSize: 12, marginBottom: 4, display: "block" }}>Amount ({baseCoin})</label>
                      <input className="input mono" placeholder="0.00" value={tradeAmount} onChange={(e) => setTradeAmount(e.target.value)} />
                    </div>
                    <div style={{ display: "flex", gap: 4 }}>
                      {[25, 50, 75, 100].map((p) => (
                        <button key={p} className="btn-ghost" style={{ flex: 1, padding: "4px 0", fontSize: 11 }} onClick={() => {
                          if (tradeTab === "buy") {
                            const usdtBal = portfolio.USDT?.amount || 0;
                            setTradeAmount(((usdtBal * p / 100) / currentPrice).toFixed(6));
                          } else {
                            const coinBal = portfolio[baseCoin]?.amount || 0;
                            setTradeAmount((coinBal * p / 100).toFixed(6));
                          }
                        }}>{p}%</button>
                      ))}
                    </div>
                    <div style={{ background: C.bg3, borderRadius: 6, padding: "8px 12px" }}>
                      <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12 }}>
                        <span style={{ color: C.text2 }}>Est. Total</span>
                        <span className="mono">{fmt(estTotal, 2)} USDT</span>
                      </div>
                      <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, marginTop: 4 }}>
                        <span style={{ color: C.text2 }}>Fee (0.1%)</span>
                        <span className="mono">{fmt(estTotal * 0.001, 4)} USDT</span>
                      </div>
                    </div>
                    <div style={{ display: "flex", justifyContent: "space-between", fontSize: 12, color: C.text2 }}>
                      <span>Available</span>
                      <span className="mono">{tradeTab === "buy" ? `${fmt(portfolio.USDT?.amount || 0)} USDT` : `${fmt(portfolio[baseCoin]?.amount || 0, 6)} ${baseCoin}`}</span>
                    </div>
                    <button className={tradeTab === "buy" ? "btn-primary" : "btn-red"} style={{ width: "100%", marginTop: 4 }} onClick={handleTrade}>
                      {tradeTab === "buy" ? `Buy ${baseCoin}` : `Sell ${baseCoin}`}
                    </button>
                  </div>
                </div>

                {/* Order book mini */}
                <div className="card" style={{ padding: 12 }}>
                  <h4 style={{ marginBottom: 10, fontSize: 13, fontWeight: 700 }}>Order Book</h4>
                  <div style={{ fontSize: 11, display: "flex", justifyContent: "space-between", color: C.text3, marginBottom: 6 }}>
                    <span>Price (USDT)</span><span>Amount</span>
                  </div>
                  {Array.from({ length: 6 }, (_, i) => ({
                    price: currentPrice * (1 + (6 - i) * 0.0008),
                    amt: (Math.random() * 3 + 0.1).toFixed(4),
                    type: "sell",
                  })).map((row, i) => (
                    <div key={`s${i}`} style={{ display: "flex", justifyContent: "space-between", fontSize: 11, padding: "2px 0", position: "relative" }}>
                      <div style={{ position: "absolute", right: 0, top: 0, height: "100%", width: `${Math.random() * 60 + 20}%`, background: `${C.red}15`, borderRadius: 2 }} />
                      <span className="mono red">{fmt(row.price, 2)}</span>
                      <span className="mono" style={{ color: C.text2 }}>{row.amt}</span>
                    </div>
                  ))}
                  <div style={{ textAlign: "center", padding: "6px 0", background: C.bg3, borderRadius: 4, margin: "4px 0" }}>
                    <span className="mono" style={{ fontWeight: 700, color: coinInfo?.change >= 0 ? C.green : C.red }}>${fmt(currentPrice, 2)}</span>
                  </div>
                  {Array.from({ length: 6 }, (_, i) => ({
                    price: currentPrice * (1 - (i + 1) * 0.0008),
                    amt: (Math.random() * 3 + 0.1).toFixed(4),
                  })).map((row, i) => (
                    <div key={`b${i}`} style={{ display: "flex", justifyContent: "space-between", fontSize: 11, padding: "2px 0", position: "relative" }}>
                      <div style={{ position: "absolute", right: 0, top: 0, height: "100%", width: `${Math.random() * 60 + 20}%`, background: `${C.green}15`, borderRadius: 2 }} />
                      <span className="mono green">{fmt(row.price, 2)}</span>
                      <span className="mono" style={{ color: C.text2 }}>{row.amt}</span>
                    </div>
                  ))}
                </div>
              </div>
            </div>

            {/* Recent Trades */}
            <div className="card">
              <h4 style={{ marginBottom: 12, fontWeight: 700 }}>Recent Trades</h4>
              <div style={{ overflowX: "auto" }}>
                <table style={{ width: "100%", fontSize: 12, borderCollapse: "collapse" }}>
                  <thead>
                    <tr style={{ color: C.text2, borderBottom: `1px solid ${C.border}` }}>
                      {["Time", "Type", "Price", "Amount", "Total"].map((h) => <th key={h} style={{ padding: "6px 10px", textAlign: "left", fontWeight: 600 }}>{h}</th>)}
                    </tr>
                  </thead>
                  <tbody>
                    {transactions.filter((t) => t.coin === baseCoin && (t.type === "buy" || t.type === "sell")).slice(0, 6).map((t) => (
                      <tr key={t.id} className="hover-row">
                        <td style={{ padding: "8px 10px", color: C.text3 }}>{new Date(t.ts).toLocaleTimeString()}</td>
                        <td style={{ padding: "8px 10px", color: t.type === "buy" ? C.green : C.red, fontWeight: 700, textTransform: "uppercase" }}>{t.type}</td>
                        <td className="mono" style={{ padding: "8px 10px" }}>${fmt(t.usdVal / t.amount, 2)}</td>
                        <td className="mono" style={{ padding: "8px 10px" }}>{fmt(t.amount, 6)}</td>
                        <td className="mono" style={{ padding: "8px 10px" }}>${fmt(t.usdVal)}</td>
                      </tr>
                    ))}
                    {transactions.filter((t) => t.coin === baseCoin).length === 0 && (
                      <tr><td colSpan={5} style={{ padding: 20, textAlign: "center", color: C.text3 }}>No trades yet</td></tr>
                    )}
                  </tbody>
                </table>
              </div>
            </div>
          </div>
        );
      }

      // ── WALLET ─────────────────────────────────────────────────────────────
      case "wallet": return (
        <div className="page" style={{ display: "flex", flexDirection: "column", gap: 16 }}>
          <div className="card">
            <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20, flexWrap: "wrap", gap: 12 }}>
              <div>
                <h3 style={{ fontWeight: 800, fontSize: 20 }}>My Wallet</h3>
                <p className="mono" style={{ color: C.text2, marginTop: 4 }}>Total: {currency === "USD" ? `$${fmt(totalUSD)}` : fmtINR(totalUSD)}</p>
              </div>
              <div style={{ display: "flex", gap: 8 }}>
                <button className="btn-primary" style={{ padding: "8px 16px" }} onClick={() => startDeposit()}>+ Deposit</button>
                <button className="btn-ghost" onClick={() => startWithdraw()}>Withdraw</button>
              </div>
            </div>
            <div style={{ display: "flex", gap: 8, marginBottom: 20 }}>
              <button className={`tab ${walletTab === "assets" ? "active" : ""}`} onClick={() => setWalletTab("assets")}>Crypto Assets</button>
              <button className={`tab ${walletTab === "history" ? "active" : ""}`} onClick={() => setWalletTab("history")}>Transaction History</button>
            </div>

            {walletTab === "assets" ? (
              <div style={{ overflowX: "auto" }}>
                <table style={{ width: "100%", borderCollapse: "collapse", fontSize: 13 }}>
                  <thead>
                    <tr style={{ color: C.text2, borderBottom: `1px solid ${C.border}` }}>
                      {["Coin", "Balance", "Available", "In Order", "Value (USD)", "Action"].map((h) => (
                        <th key={h} style={{ padding: "8px 12px", textAlign: "left", fontWeight: 600, whiteSpace: "nowrap" }}>{h}</th>
                      ))}
                    </tr>
                  </thead>
                  <tbody>
                    {Object.entries(portfolio).filter(([, { amount }]) => amount > 0).map(([sym, { amount, locked }]) => {
                      const price = prices[sym] || (sym === "USDT" ? 1 : 0);
                      const usdVal = amount * price;
                      return (
                        <tr key={sym} className="hover-row" style={{ borderBottom: `1px solid ${C.border}20` }}>
                          <td style={{ padding: "12px" }}>
                            <div style={{ display: "flex", alignItems: "center", gap: 8 }}>
                              {sym === "USDT" ? <div style={{ width: 32, height: 32, borderRadius: "50%", background: "#26A17B", display: "flex", alignItems: "center", justifyContent: "center", color: "#fff", fontWeight: 700 }}>₮</div> : <CoinIcon coin={sym} size={32} />}
                              <div>
                                <p style={{ fontWeight: 700 }}>{sym}</p>
                                <p style={{ color: C.text2, fontSize: 11 }}>{COINS.find((c) => c.symbol === sym)?.name || "Tether"}</p>
                              </div>
                            </div>
                          </td>
                          <td className="mono" style={{ padding: "12px" }}>{fmt(amount, 6)}</td>
                          <td className="mono" style={{ padding: "12px", color: C.green }}>{fmt(amount - (locked || 0), 6)}</td>
                          <td className="mono" style={{ padding: "12px", color: C.text3 }}>{fmt(locked || 0, 6)}</td>
                          <td className="mono" style={{ padding: "12px", fontWeight: 700 }}>${fmt(usdVal)}</td>
                          <td style={{ padding: "12px" }}>
                            <div style={{ display: "flex", gap: 6, flexWrap: "wrap" }}>
                              <button className="btn-primary" style={{ padding: "4px 10px", fontSize: 11 }} onClick={() => startDeposit(sym)}>Deposit</button>
                              <button className="btn-ghost" style={{ padding: "4px 10px", fontSize: 11 }} onClick={() => startWithdraw(sym)}>Withdraw</button>
                              <button className="btn-ghost" style={{ padding: "4px 10px", fontSize: 11 }} onClick={() => { setTradePair(`${sym}/USDT`); setPage("trade"); }}>Trade</button>
                            </div>
                          </td>
                        </tr>
                      );
                    })}
                  </tbody>
                </table>
              </div>
            ) : (
              <div>
                {transactions.map((tx) => (
                  <div key={tx.id} style={{ display: "flex", gap: 14, padding: "14px 0", borderBottom: `1px solid ${C.border}`, alignItems: "center", flexWrap: "wrap" }}>
                    <div style={{ width: 40, height: 40, borderRadius: "50%", background: ["deposit", "receive", "buy"].includes(tx.type) ? `${C.green}20` : `${C.red}20`, display: "flex", alignItems: "center", justifyContent: "center", fontSize: 18, flexShrink: 0 }}>
                      {["deposit", "receive", "buy"].includes(tx.type) ? "↓" : "↑"}
                    </div>
                    <div style={{ flex: 1, minWidth: 200 }}>
                      <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 4, flexWrap: "wrap", gap: 4 }}>
                        <span style={{ fontWeight: 700, textTransform: "capitalize" }}>{tx.type} {tx.coin}</span>
                        <span className={`mono ${["deposit", "receive", "buy"].includes(tx.type) ? "green" : "red"}`} style={{ fontWeight: 700 }}>
                          {["deposit", "receive", "buy"].includes(tx.type) ? "+" : "-"}{fmt(tx.amount, 6)} {tx.coin}
                        </span>
                      </div>
                      <div style={{ display: "flex", gap: 12, flexWrap: "wrap", fontSize: 12, color: C.text3 }}>
                        <span>{tx.network}</span>
                        <span>Fee: {tx.fee}</span>
                        <span className="mono" style={{ fontSize: 11 }}>{tx.hash?.slice(0, 18)}...</span>
                      </div>
                      <div style={{ display: "flex", justifyContent: "space-between", marginTop: 6, alignItems: "center" }}>
                        <span style={{ color: C.text3, fontSize: 12 }}>{new Date(tx.ts).toLocaleString()}</span>
                        <StatusBadge status={tx.status} />
                      </div>
                    </div>
                  </div>
                ))}
              </div>
            )}
          </div>
        </div>
      );

      // ── HISTORY ────────────────────────────────────────────────────────────
      case "history": return (
        <div className="page">
          <div className="card">
            <h3 style={{ fontWeight: 700, marginBottom: 20 }}>Transaction History</h3>
            {transactions.map((tx) => (
              <div key={tx.id} style={{ borderBottom: `1px solid ${C.border}`, padding: "16px 0" }}>
                <div style={{ display: "flex", justifyContent: "space-between", flexWrap: "wrap", gap: 8 }}>
                  <div style={{ display: "flex", gap: 12, alignItems: "center" }}>
                    <CoinIcon coin={tx.coin} size={36} />
                    <div>
                      <p style={{ fontWeight: 700, textTransform: "capitalize" }}>{tx.type} {tx.coin}</p>
                      <p style={{ color: C.text2, fontSize: 12 }}>{tx.network} · {new Date(tx.ts).toLocaleString()}</p>
                    </div>
                  </div>
                  <div style={{ textAlign: "right" }}>
                    <p className={`mono ${["deposit", "receive", "buy"].includes(tx.type) ? "green" : "red"}`} style={{ fontWeight: 700 }}>
                      {["deposit", "receive", "buy"].includes(tx.type) ? "+" : "-"}{fmt(tx.amount, 6)} {tx.coin}
                    </p>
                    <p style={{ color: C.text2, fontSize: 12 }}>${fmt(tx.usdVal)}</p>
                  </div>
                </div>
                <div style={{ marginTop: 10, padding: "10px 12px", background: C.bg2, borderRadius: 8 }}>
                  <div style={{ display: "grid", gridTemplateColumns: "repeat(auto-fit, minmax(180px, 1fr))", gap: 8, fontSize: 12 }}>
                    {[["Status", <StatusBadge status={tx.status} />], ["Fee", `${tx.fee} (${tx.network})`], ["TX Hash", <span className="mono" style={{ fontSize: 11 }}>{tx.hash?.slice(0, 26)}...</span>]].map(([label, val]) => (
                      <div key={label}>
                        <p style={{ color: C.text3, marginBottom: 2 }}>{label}</p>
                        <p style={{ fontWeight: 600 }}>{val}</p>
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            ))}
          </div>
        </div>
      );

      // ── SETTINGS ───────────────────────────────────────────────────────────
      case "settings": return (
        <div className="page" style={{ display: "flex", flexDirection: "column", gap: 16 }}>
          <div className="card">
            <div style={{ display: "flex", gap: 8, marginBottom: 24 }}>
              {["security", "preferences", "devices"].map((t) => (
                <button key={t} className={`tab ${settingsTab === t ? "active" : ""}`} onClick={() => setSettingsTab(t)} style={{ textTransform: "capitalize" }}>{t}</button>
              ))}
            </div>

            {settingsTab === "security" && (
              <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "14px 16px", background: C.bg2, borderRadius: 10 }}>
                  <div>
                    <p style={{ fontWeight: 700 }}>Two-Factor Authentication (2FA)</p>
                    <p style={{ color: C.text2, fontSize: 13, marginTop: 2 }}>Add an extra layer of security</p>
                  </div>
                  <button className="btn-primary" style={{ padding: "6px 14px", fontSize: 12 }}>Enable</button>
                </div>
                <div style={{ padding: "14px 16px", background: C.bg2, borderRadius: 10 }}>
                  <p style={{ fontWeight: 700, marginBottom: 10 }}>Change PIN</p>
                  <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
                    <input className="input" type="password" placeholder="Current PIN" />
                    <input className="input" type="password" placeholder="New PIN (6 digits)" maxLength={6} />
                    <input className="input" type="password" placeholder="Confirm new PIN" maxLength={6} />
                    <button className="btn-primary" style={{ alignSelf: "flex-start" }} onClick={() => toast("PIN updated successfully", "success")}>Update PIN</button>
                  </div>
                </div>
                <div style={{ padding: "14px 16px", background: C.bg2, borderRadius: 10 }}>
                  <p style={{ fontWeight: 700, marginBottom: 8 }}>Login History</p>
                  {[{ device: "Chrome / Windows", ip: "192.168.1.1", time: "Just now" }, { device: "Safari / iPhone", ip: "10.0.0.1", time: "2h ago" }].map((s, i) => (
                    <div key={i} style={{ display: "flex", justifyContent: "space-between", padding: "8px 0", borderBottom: `1px solid ${C.border}`, fontSize: 13 }}>
                      <div>
                        <p style={{ fontWeight: 600 }}>{s.device}</p>
                        <p style={{ color: C.text3, fontSize: 11 }}>IP: {s.ip}</p>
                      </div>
                      <p style={{ color: C.text3, fontSize: 12 }}>{s.time}</p>
                    </div>
                  ))}
                </div>
              </div>
            )}

            {settingsTab === "preferences" && (
              <div style={{ display: "flex", flexDirection: "column", gap: 16 }}>
                <div style={{ padding: "14px 16px", background: C.bg2, borderRadius: 10 }}>
                  <p style={{ fontWeight: 700, marginBottom: 12 }}>Display Currency</p>
                  <div style={{ display: "flex", gap: 8 }}>
                    {["USD", "INR", "EUR", "GBP"].map((c) => (
                      <button key={c} className={`tab ${currency === c ? "active" : ""}`} onClick={() => { setCurrency(c); toast(`Currency set to ${c}`, "success"); }}>{c}</button>
                    ))}
                  </div>
                </div>
                <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "14px 16px", background: C.bg2, borderRadius: 10 }}>
                  <p style={{ fontWeight: 700 }}>Show Balance</p>
                  <button onClick={() => setShowBalance((p) => !p)} style={{ background: showBalance ? C.accent : C.bg4, width: 48, height: 26, borderRadius: 13, border: "none", cursor: "pointer", position: "relative", transition: "background 0.2s" }}>
                    <div style={{ position: "absolute", width: 20, height: 20, borderRadius: "50%", background: "#fff", top: 3, left: showBalance ? 25 : 3, transition: "left 0.2s" }} />
                  </button>
                </div>
              </div>
            )}

            {settingsTab === "devices" && (
              <div style={{ display: "flex", flexDirection: "column", gap: 12 }}>
                {[
                  { name: "Chrome / Windows (This device)", status: "Active now", verified: true },
                  { name: "Binance App / iPhone", status: `Sync: ${syncProgress}%`, verified: syncProgress >= 100 },
                  { name: "Binance App / Android", status: "Waiting authorization", verified: false },
                ].map((d, i) => (
                  <div key={i} style={{ display: "flex", justifyContent: "space-between", alignItems: "center", padding: "14px 16px", background: C.bg2, borderRadius: 10 }}>
                    <div>
                      <p style={{ fontWeight: 700, fontSize: 14 }}>{d.name}</p>
                      <p style={{ color: d.verified ? C.green : C.accent, fontSize: 12, marginTop: 2 }}>{d.status}</p>
                    </div>
                    {!d.verified && <button className="btn-ghost" style={{ fontSize: 12 }} onClick={() => toast("Authorization request sent", "success")}>Authorize</button>}
                    {d.verified && <span style={{ color: C.green, fontSize: 13 }}>✓ Verified</span>}
                  </div>
                ))}
              </div>
            )}
          </div>
        </div>
      );

      default: return null;
    }
  };

  // ── Modal: Deposit ─────────────────────────────────────────────────────────
  const renderDepositModal = () => (
    <div className="modal-overlay" onClick={() => setModalOpen(null)}>
      <div className="modal" onClick={(e) => e.stopPropagation()}>
        <div style={{ padding: 24 }}>
          <div style={{ display: "flex", justifyContent: "space-between", alignItems: "center", marginBottom: 20 }}>
            <h3 style={{ fontWeight: 800 }}>Deposit Crypto</h3>
            <button onClick={() => setModalOpen(null)} style={{ background: "none", border: "none", color: C.text2, fontSize: 20, cursor: "pointer" }}>✕</button>
          </div>

          {flowStep === 1 ? (
            <>
              <div style={{ marginBottom: 16 }}>
                <label style={{ color: C.text2, fontSize: 12, display: "block", marginBottom: 6 }}>Select Coin</label>
                <select className="select" style={{ width: "100%" }} value={flowCoin} onChange={(e) => { setFlowCoin(e.target.value); setFlowNetwork(null); }}>
                  {[...COINS.map((c) => c.symbol), "USDT"].map((s) => <option key={s}>{s}</option>)}
                </select>
              </div>
              <div style={{ marginBottom: 20 }}>
                <label style={{ color: C.text2, fontSize: 12, display: "block", marginBottom: 6 }}>Select Network</label>
                {getNets(flowCoin).map((net) => (
                  <div key={net.id} onClick={() => setFlowNetwork(net)} style={{ padding: "12px 14px", background: flowNetwork?.id === net.id ? C.accentGlow : C.bg2, border: `1px solid ${flowNetwork?.id === net.id ? C.accent : C.border}`, borderRadius: 8, marginBottom: 8, cursor: "pointer", transition: "all 0.2s" }}>
                    <div style={{ display: "flex", justifyContent: "space-between" }}>
                      <span style={{ fontWeight: 700 }}>{net.name}</span>
                      {flowNetwork?.id === net.id && <span style={{ color: C.accent }}>✓</span>}
                    </div>
                    <div style={{ display: "flex", gap: 16, marginTop: 4, fontSize: 12, color: C.text2 }}>
                      <span>Fee: {net.fee}</span><span>~{net.time}</span>
                    </div>
                  </div>
                ))}
              </div>
              <div style={{ background: `${C.accent}15`, border: `1px solid ${C.accent}40`, borderRadius: 8, padding: "10px 14px", marginBottom: 20, fontSize: 13, color: C.accent }}>
                ⚠️ Only send {flowCoin} on the selected network. Wrong network = permanent loss.
              </div>
              <button className="btn-primary" onClick={handleGenerateAddress} style={{ width: "100%" }}>Generate Address</button>
            </>
          ) : (
            <>
              <div style={{ textAlign: "center", marginBottom: 20 }}>
                <p style={{ color: C.text2, fontSize: 13, marginBottom: 12 }}>Send {flowCoin} to this address ({flowNetwork?.name})</p>
                <div style={{ display: "flex", justifyContent: "center", marginBottom: 16 }}>
                  <QRCode value={depositAddr} size={160} />
                </div>
                <div style={{ background: C.bg2, borderRadius: 8, padding: "10px 14px", display: "flex", alignItems: "center", gap: 8 }}>
                  <span className="mono" style={{ fontSize: 12, flex: 1, wordBreak: "break-all", textAlign: "left", color: C.text1 }}>{depositAddr}</span>
                  <button onClick={() => { navigator.clipboard?.writeText(depositAddr); toast("Address copied!", "success"); }} style={{ background: C.bg4, border: "none", borderRadius: 6, padding: "6px 10px", cursor: "pointer", color: C.accent, fontSize: 12, fontFamily: "Syne", fontWeight: 600 }}>Copy</button>
                </div>
              </div>
              <div style={{ display: "flex", flexDirection: "column", gap: 8, marginBottom: 20 }}>
                {[["Network", flowNetwork?.name], ["Minimum Deposit", `0.001 ${flowCoin}`], ["Expected Arrival", flowNetwork?.time], ["Fee", flowNetwork?.fee]].map(([l, v]) => (
                  <div key={l} style={{ display: "flex", justifyContent: "space-between", fontSize: 13 }}>
                    <span style={{ color: C.text2 }}>{l}</span>
                    <span style={{ fontWeight: 600 }}>{v}</span>
                  </div>
                ))}
              </div>
              <div style={{ background: `${C.green}15`, border: `1px solid ${C.green}40`, borderRadius: 8, padding: "10px 14px", fontSize: 13, color: C.green, marginBottom: 16 }}>
                ✓ Deposits will be credited after network confirmations
              </div>
              <button className="btn-ghost" onClick={() => setFlowStep(1)} style={{ width: "100%" }}>← Back</button>
            </>
          )}
        </div>
      </div>
    </div>
  );

  // ── Modal: Withdraw/Send ───────────────────────────────────────────────────
  const renderWithdrawModal = () => {
    const isOpen = modalOpen === "withdraw" || modalOpen === "send";
    if (!isOpen) return null;
    const fee = flowNetwork?.fee || "0.001";
    const feeNum = parseFloat(fee.split(" ")[0]) || 0.001;
    const amtNum = parseFloat(flowAmount) || 0;
    const netReceived = Math.max(0, amtNum - feeNum);
    return (
      <div className="modal-overlay" onClick={() => setModalOpen(null)}>
        <div className="modal" onClick={(e) => e.stopPropagation()}>
          <div style={{ padding: 24 }}>
            <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 20 }}>
              <h3 style={{ fontWeight: 800 }}>Withdraw {flowCoin}</h3>
              <button onClick={() => setModalOpen(null)} style={{ background: "none", border: "none", color: C.text2, fontSize: 20, cursor: "pointer" }}>✕</button>
            </div>

            {flowStep === 4 ? (
              <div style={{ textAlign: "center", padding: "40px 0" }}>
                <div className="spinner" style={{ margin: "0 auto 20px" }} />
                <p style={{ fontWeight: 700 }}>Processing withdrawal...</p>
                <p style={{ color: C.text2, fontSize: 13, marginTop: 8 }}>This may take a moment</p>
              </div>
            ) : flowStep === 5 ? (
              <div style={{ textAlign: "center", padding: "30px 0" }}>
                <div style={{ fontSize: 60, marginBottom: 16 }}>✅</div>
                <h3 style={{ marginBottom: 8 }}>Withdrawal Submitted!</h3>
                <p style={{ color: C.text2, fontSize: 13 }}>Your withdrawal is being processed. You'll receive a confirmation once it's complete.</p>
                <button className="btn-primary" style={{ marginTop: 20 }} onClick={() => { setModalOpen(null); setFlowStep(1); }}>Done</button>
              </div>
            ) : flowStep === 3 ? (
              <>
                <div style={{ background: C.bg2, borderRadius: 10, padding: 16, marginBottom: 16 }}>
                  <h4 style={{ marginBottom: 12, color: C.text2, fontSize: 13, textTransform: "uppercase", letterSpacing: 1 }}>Confirm Withdrawal</h4>
                  {[["Coin", flowCoin], ["To Address", flowAddress.slice(0, 20) + "..."], ["Network", flowNetwork?.name], ["Amount", `${flowAmount} ${flowCoin}`], ["Fee", flowNetwork?.fee], ["Net Received", `${fmt(netReceived, 6)} ${flowCoin}`]].map(([l, v]) => (
                    <div key={l} style={{ display: "flex", justifyContent: "space-between", padding: "8px 0", borderBottom: `1px solid ${C.border}`, fontSize: 13 }}>
                      <span style={{ color: C.text2 }}>{l}</span>
                      <span style={{ fontWeight: 600 }}>{v}</span>
                    </div>
                  ))}
                </div>
                <div style={{ background: `${C.red}10`, border: `1px solid ${C.red}40`, borderRadius: 8, padding: "10px 14px", fontSize: 13, color: C.red, marginBottom: 16 }}>
                  ⚠️ This action cannot be undone. Verify the address carefully.
                </div>
                <div style={{ display: "flex", gap: 8 }}>
                  <button className="btn-ghost" onClick={() => setFlowStep(2)} style={{ flex: 1 }}>Back</button>
                  <button className="btn-red" onClick={handleWithdrawConfirm} style={{ flex: 1 }}>Confirm Withdrawal</button>
                </div>
              </>
            ) : (
              <>
                <div style={{ marginBottom: 14 }}>
                  <label style={{ color: C.text2, fontSize: 12, display: "block", marginBottom: 6 }}>Coin</label>
                  <select className="select" style={{ width: "100%" }} value={flowCoin} onChange={(e) => { setFlowCoin(e.target.value); setFlowNetwork(null); }}>
                    {Object.keys(portfolio).filter((s) => portfolio[s].amount > 0).map((s) => <option key={s}>{s}</option>)}
                  </select>
                </div>
                <div style={{ marginBottom: 14 }}>
                  <label style={{ color: C.text2, fontSize: 12, display: "block", marginBottom: 6 }}>Network</label>
                  <select className="select" style={{ width: "100%" }} value={flowNetwork?.id || ""} onChange={(e) => setFlowNetwork(getNets(flowCoin).find((n) => n.id === e.target.value))}>
                    <option value="">Select network</option>
                    {getNets(flowCoin).map((n) => <option key={n.id} value={n.id}>{n.name} — Fee: {n.fee}</option>)}
                  </select>
                </div>
                <div style={{ marginBottom: 14 }}>
                  <label style={{ color: C.text2, fontSize: 12, display: "block", marginBottom: 6 }}>Recipient Address</label>
                  <input className="input mono" placeholder="Enter wallet address" value={flowAddress} onChange={(e) => setFlowAddress(e.target.value)} style={{ fontSize: 12 }} />
                </div>
                <div style={{ marginBottom: 16 }}>
                  <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 6 }}>
                    <label style={{ color: C.text2, fontSize: 12 }}>Amount</label>
                    <span style={{ color: C.text2, fontSize: 12 }}>Available: {fmt(portfolio[flowCoin]?.amount || 0, 6)} {flowCoin}</span>
                  </div>
                  <div style={{ position: "relative" }}>
                    <input className="input mono" placeholder="0.00" value={flowAmount} onChange={(e) => setFlowAmount(e.target.value)} />
                    <button onClick={() => setFlowAmount(fmt(portfolio[flowCoin]?.amount || 0, 6))} style={{ position: "absolute", right: 10, top: "50%", transform: "translateY(-50%)", background: "none", border: "none", color: C.accent, cursor: "pointer", fontSize: 12, fontFamily: "Syne", fontWeight: 700 }}>MAX</button>
                  </div>
                </div>
                {amtNum > 0 && (
                  <div style={{ background: C.bg2, borderRadius: 8, padding: "10px 14px", marginBottom: 16, fontSize: 13 }}>
                    <div style={{ display: "flex", justifyContent: "space-between" }}><span style={{ color: C.text2 }}>Network fee</span><span>{fee}</span></div>
                    <div style={{ display: "flex", justifyContent: "space-between", marginTop: 6, fontWeight: 700 }}><span style={{ color: C.text2 }}>You'll receive</span><span style={{ color: C.green }}>{fmt(netReceived, 6)} {flowCoin}</span></div>
                  </div>
                )}
                <button className="btn-primary" style={{ width: "100%" }} onClick={() => {
                  if (!flowAddress) { toast("Enter recipient address", "error"); return; }
                  if (!flowAmount || parseFloat(flowAmount) <= 0) { toast("Enter valid amount", "error"); return; }
                  if (!flowNetwork) { toast("Select a network", "error"); return; }
                  setFlowStep(3);
                }}>Continue</button>
              </>
            )}
          </div>
        </div>
      </div>
    );
  };

  // ── Modal: Receive ─────────────────────────────────────────────────────────
  const renderReceiveModal = () => (
    <div className="modal-overlay" onClick={() => setModalOpen(null)}>
      <div className="modal" onClick={(e) => e.stopPropagation()}>
        <div style={{ padding: 24 }}>
          <div style={{ display: "flex", justifyContent: "space-between", marginBottom: 20 }}>
            <h3 style={{ fontWeight: 800 }}>Receive Crypto</h3>
            <button onClick={() => setModalOpen(null)} style={{ background: "none", border: "none", color: C.text2, fontSize: 20, cursor: "pointer" }}>✕</button>
          </div>
          <div style={{ marginBottom: 16 }}>
            <label style={{ color: C.text2, fontSize: 12, display: "block", marginBottom: 6 }}>Select Coin</label>
            <select className="select" style={{ width: "100%" }} value={flowCoin} onChange={(e) => {
              setFlowCoin(e.target.value);
              const nets = getNets(e.target.value);
              setFlowNetwork(nets[0]);
              setDepositAddr(generateAddress(e.target.value, nets[0]));
            }}>
              {[...COINS.map((c) => c.symbol), "USDT"].map((s) => <option key={s}>{s}</option>)}
            </select>
          </div>
          <div style={{ marginBottom: 16 }}>
            <label style={{ color: C.text2, fontSize: 12, display: "block", marginBottom: 6 }}>Network</label>
            <div style={{ display: "flex", gap: 6, flexWrap: "wrap" }}>
              {getNets(flowCoin).map((net) => (
                <button key={net.id} className={`tab ${flowNetwork?.id === net.id ? "active" : ""}`} onClick={() => { setFlowNetwork(net); setDepositAddr(generateAddress(flowCoin, net)); }} style={{ fontSize: 12 }}>{net.name}</button>
              ))}
            </div>
          </div>
          <div style={{ textAlign: "center", marginBottom: 20 }}>
            <div style={{ display: "flex", justifyContent: "center", marginBottom: 16 }}>
              <QRCode value={depositAddr} size={160} />
            </div>
            <p style={{ color: C.text2, fontSize: 12, marginBottom: 8 }}>Your {flowCoin} deposit address ({flowNetwork?.name})</p>
            <div style={{ background: C.bg2, borderRadius: 8, padding: "10px 14px", display: "flex", gap: 8, alignItems: "center" }}>
              <span className="mono" style={{ fontSize: 11, flex: 1, wordBreak: "break-all", textAlign: "left" }}>{depositAddr}</span>
              <button onClick={() => { navigator.clipboard?.writeText(depositAddr); toast("Copied!", "success"); }} style={{ background: C.accent, border: "none", borderRadius: 6, padding: "6px 12px", cursor: "pointer", color: "#000", fontFamily: "Syne", fontWeight: 700, fontSize: 12 }}>Copy</button>
            </div>
          </div>
          <div style={{ background: `${C.accent}15`, border: `1px solid ${C.accent}40`, borderRadius: 8, padding: "10px 14px", fontSize: 13, color: C.accent }}>
            ⚠️ Only send {flowCoin} via {flowNetwork?.name}. Other assets sent to this address will be lost.
          </div>
        </div>
      </div>
    </div>
  );

  // ── NAV ITEMS ──────────────────────────────────────────────────────────────
  const navPages = [
    { id: "dashboard", label: "Dashboard", icon: "⊞" },
    { id: "markets", label: "Markets", icon: "◈" },
    { id: "trade", label: "Trade", icon: "⇅" },
    { id: "wallet", label: "Wallet", icon: "◉" },
    { id: "history", label: "History", icon: "⊡" },
    { id: "settings", label: "Settings", icon: "⚙" },
  ];

  // ── RENDER ─────────────────────────────────────────────────────────────────
  return (
    <>
      <style>{css}</style>

      {/* TOP NAV */}
      <nav style={{ position: "fixed", top: 0, left: 0, right: 0, height: 56, background: C.bg1, borderBottom: `1px solid ${C.border}`, display: "flex", alignItems: "center", padding: "0 20px", zIndex: 100, gap: 16 }}>
        <div style={{ display: "flex", alignItems: "center", gap: 8, cursor: "pointer" }} onClick={() => setPage("dashboard")}>
          <div style={{ background: C.accent, borderRadius: 6, width: 28, height: 28, display: "flex", alignItems: "center", justifyContent: "center" }}>
            <span style={{ color: "#000", fontWeight: 800, fontSize: 15 }}>B</span>
          </div>
          <span style={{ fontWeight: 800, fontSize: 18, color: C.accent }}>Binance</span>
        </div>

        {/* Desktop nav links */}
        <div style={{ display: "flex", gap: 4, marginLeft: 24 }} className="sidebar-desktop">
          {["markets", "trade", "wallet", "settings"].map((p) => (
            <button key={p} className={`nav-item tab ${page === p ? "active" : ""}`} onClick={() => setPage(p)} style={{ textTransform: "capitalize" }}>{p}</button>
          ))}
        </div>

        <div style={{ flex: 1 }} />

        {/* Live indicator */}
        <div style={{ display: "flex", alignItems: "center", gap: 6, fontSize: 12, color: C.green }}>
          <div style={{ width: 6, height: 6, borderRadius: "50%", background: C.green, animation: "pulse 1.5s infinite" }} />
          Live
        </div>

        {/* Currency */}
        <button className="tab" onClick={() => setCurrency((c) => c === "USD" ? "INR" : "USD")} style={{ fontSize: 12 }}>{currency}</button>

        {/* Admin (hidden) */}
        <button onClick={() => setPage("admin")} style={{ background: "none", border: "none", color: C.text3, cursor: "pointer", fontSize: 11, opacity: 0.3 }} title="Admin">⚙</button>

        {/* Profile */}
        <div style={{ width: 32, height: 32, borderRadius: "50%", background: C.accent, display: "flex", alignItems: "center", justifyContent: "center", fontWeight: 800, color: "#000", cursor: "pointer", fontSize: 14 }}>
          {authForm.email?.[0]?.toUpperCase() || "U"}
        </div>
      </nav>

      {/* SIDEBAR (desktop) */}
      <aside className="sidebar-desktop" style={{ position: "fixed", left: 0, top: 56, bottom: 0, width: 200, background: C.bg1, borderRight: `1px solid ${C.border}`, padding: "16px 8px", display: "flex", flexDirection: "column", gap: 4, overflowY: "auto", zIndex: 90 }}>
        {navPages.map(({ id, label, icon }) => (
          <button key={id} className={`sidebar-link ${page === id ? "active" : ""}`} onClick={() => setPage(id)}>
            <span style={{ fontSize: 16 }}>{icon}</span> {label}
          </button>
        ))}
        <div style={{ flex: 1 }} />
        <button className="sidebar-link" onClick={() => { setAuthed(false); setAdminMode(false); }}>
          <span>⎋</span> Logout
        </button>
      </aside>

      {/* MAIN */}
      <main className="main-content" style={{ marginLeft: 200, marginTop: 56, padding: "24px 20px", minHeight: "calc(100vh - 56px)" }}>
        <div style={{ maxWidth: 1200, margin: "0 auto" }}>
          {/* Disclaimer */}
          <div style={{ background: `${C.accent}10`, border: `1px solid ${C.accent}30`, borderRadius: 8, padding: "8px 14px", marginBottom: 20, fontSize: 12, color: C.text2, display: "flex", alignItems: "center", gap: 8 }}>
            <span style={{ color: C.accent }}>⚠️</span>
            <span><strong style={{ color: C.accent }}>DISCLAIMER:</strong> This is a Binance UI clone for demo/portfolio purposes only. No real cryptocurrency transactions or payments are processed.</span>
          </div>
          {renderPage()}
        </div>
      </main>

      {/* BOTTOM NAV (mobile) */}
      <nav className="bottom-nav" style={{ position: "fixed", bottom: 0, left: 0, right: 0, background: C.bg1, borderTop: `1px solid ${C.border}`, display: "flex", zIndex: 100 }}>
        {navPages.slice(0, 5).map(({ id, label, icon }) => (
          <button key={id} onClick={() => setPage(id)} style={{ flex: 1, display: "flex", flexDirection: "column", alignItems: "center", justifyContent: "center", padding: "8px 0", background: "none", border: "none", cursor: "pointer", color: page === id ? C.accent : C.text3, fontFamily: "Syne", fontSize: 10, fontWeight: 600, gap: 2 }}>
            <span style={{ fontSize: 18 }}>{icon}</span>
            {label}
          </button>
        ))}
      </nav>

      {/* MODALS */}
      {modalOpen === "deposit" && renderDepositModal()}
      {(modalOpen === "withdraw" || modalOpen === "send") && renderWithdrawModal()}
      {modalOpen === "receive" && renderReceiveModal()}

      <ToastContainer toasts={toasts} />
    </>
  );
}
