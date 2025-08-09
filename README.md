import React, { useEffect, useMemo, useState } from "react";
import { motion, AnimatePresence } from "framer-motion";

// --- Minimal UI atoms (no external UI libs) ---
const Chip = ({ active, children, onClick }) => (
  <button
    onClick={onClick}
    className={
      `px-3 py-1 rounded-full border text-sm mr-2 mb-2 transition ` +
      (active
        ? "bg-black text-white border-black shadow"
        : "bg-white hover:bg-gray-100 border-gray-300 text-gray-800")
    }
  >
    {children}
  </button>
);

const Card = ({ children, className = "" }) => (
  <div className={`rounded-2xl shadow-sm border bg-white ${className}`}>{children}</div>
);

const Section = ({ title, children, right }) => (
  <Card className="p-4 md:p-5 mb-4">
    <div className="flex items-center justify-between mb-3">
      <h3 className="text-lg md:text-xl font-semibold">{title}</h3>
      {right}
    </div>
    {children}
  </Card>
);

const Button = ({ children, onClick, variant = "solid", disabled }) => {
  const base = "px-4 py-2 rounded-xl text-sm font-medium transition";
  const styles = {
    solid:
      "bg-black text-white hover:bg-gray-900 disabled:bg-gray-300 disabled:text-gray-600",
    outline:
      "border border-gray-300 hover:border-black hover:bg-gray-50 text-gray-900",
    ghost: "text-gray-700 hover:bg-gray-100",
  };
  return (
    <button className={`${base} ${styles[variant]}`} onClick={onClick} disabled={disabled}>
      {children}
    </button>
  );
};

// --- Sample Data ---
const MAKES = {
  Apex: {
    basePrice: 28000,
    models: {
      Comet: { base: 28000, engines: ["I4"], drivetrains: ["FWD", "AWD"] },
      Nova: { base: 34000, engines: ["I4", "V6"], drivetrains: ["FWD", "AWD"] },
      Titan: { base: 42000, engines: ["V6"], drivetrains: ["RWD", "AWD"] },
    },
  },
  Horizon: {
    basePrice: 32000,
    models: {
      Pulse: { base: 32000, engines: ["I4"], drivetrains: ["FWD"] },
      Rift: { base: 38500, engines: ["I4", "Hybrid"], drivetrains: ["FWD", "AWD"] },
      Vector: { base: 51000, engines: ["V6", "EV"], drivetrains: ["RWD", "AWD"] },
    },
  },
};

const ENGINES = {
  I4: { name: "2.0L Inline‑4 Turbo", price: 0, power: 190, torque: 210, type: "ICE" },
  V6: { name: "3.0L V6 Twin‑Turbo", price: 4500, power: 340, torque: 360, type: "ICE" },
  Hybrid: { name: "2.0L Hybrid", price: 3000, power: 230, torque: 250, type: "Hybrid" },
  EV: { name: "Dual‑Motor Electric", price: 8000, power: 380, torque: 480, type: "EV" },
};

const TRANSMISSIONS = {
  Auto8: { name: "8‑Speed Automatic", price: 0 },
  DCT7: { name: "7‑Speed DCT", price: 1200 },
  SingleSpeed: { name: "Single‑Speed (EV)", price: 0 },
};

const DRIVETRAINS = ["FWD", "RWD", "AWD"];

const COLORS = [
  { id: "#000000", name: "Obsidian Black", price: 500 },
  { id: "#ffffff", name: "Pearl White", price: 500, ring: true },
  { id: "#c1121f", name: "Crimson Red", price: 700 },
  { id: "#1d3557", name: "Deep Blue", price: 700 },
  { id: "#2a9d8f", name: "Teal Wave", price: 700 },
  { id: "#888888", name: "Space Gray", price: 0 },
];

const WHEELS = [
  { id: "Aero18", name: "Aero 18\"", price: 0, size: 18 },
  { id: "Split19", name: "Split‑5 19\"", price: 900, size: 19 },
  { id: "Forge20", name: "Forged 20\"", price: 1800, size: 20 },
];

const INTERIORS = [
  { id: "Fabric", name: "Fabric Black", price: 0 },
  { id: "Leather", name: "Leather Graphite", price: 1400 },
  { id: "Premium", name: "Premium Nappa", price: 2600 },
];

const ADDONS = [
  { id: "TechPack", name: "Tech Pack (HUD, 360° Cam)", price: 1800 },
  { id: "DriverAssist", name: "Driver Assist+ (L2)", price: 2200 },
  { id: "Tow", name: "Tow Package", price: 900 },
  { id: "Sport", name: "Sport Suspension", price: 1200 },
  { id: "Brembo", name: "Performance Brakes", price: 1500 },
  { id: "Panoroof", name: "Panoramic Roof", price: 1300 },
];

// Components library (learn mode)
const COMPONENTS = [
  {
    key: "engine",
    name: "Engine",
    icon: "⚙️",
    summary:
      "Converts fuel (or electricity in EVs) into motion. Power and torque influence acceleration and towing.",
    details:
      "Engines mix air and fuel, ignite it, and drive pistons connected to a crankshaft. Turbochargers force more air for extra power. EVs replace this with electric motors delivering instant torque.",
  },
  {
    key: "transmission",
    name: "Transmission",
    icon: "🧰",
    summary:
      "Matches engine speed to wheel speed using gear ratios. DCTs shift faster; automatics are smoother.",
    details:
      "Automatic gearboxes use planetary gears and a torque converter. DCTs pre‑select gears for speed. EVs typically use a single reduction gear.",
  },
  {
    key: "drivetrain",
    name: "Drivetrain",
    icon: "🛞",
    summary:
      "Delivers power to the wheels: FWD, RWD, or AWD. Affects traction and handling.",
    details:
      "FWD is efficient and space‑saving, RWD balances handling, AWD adds traction using differentials to split torque.",
  },
  {
    key: "suspension",
    name: "Suspension",
    icon: "🧿",
    summary:
      "Springs, dampers, and links that keep tires on the road and ride comfortable.",
    details:
      "Sport setups reduce body roll; adaptive dampers change firmness on the fly.",
  },
  {
    key: "brakes",
    name: "Brakes",
    icon: "🛑",
    summary:
      "Convert kinetic energy to heat. Bigger rotors and multi‑piston calipers stop harder.",
    details:
      "Performance brakes resist fade during repeated stops; EVs add regen braking to recapture energy.",
  },
  {
    key: "aero",
    name: "Aerodynamics",
    icon: "💨",
    summary:
      "Shapes airflow to reduce drag and increase downforce for stability.",
    details:
      "Splitters, diffusers, and spoilers manage air; smooth wheels cut turbulence.",
  },
];

// --- Helpers ---
const fmt = (n) => n.toLocaleString(undefined, { style: "currency", currency: "USD" });
const sum = (arr) => arr.reduce((a, b) => a + b, 0);

const DEFAULT = {
  make: "Apex",
  model: "Comet",
  engine: "I4",
  drivetrain: "FWD",
  transmission: "Auto8",
  color: COLORS[5].id,
  wheels: "Aero18",
  interior: "Fabric",
  addons: ["TechPack"],
};

// --- SVG Car Preview ---
function CarSVG({ color = "#888", wheel = "Aero18", labelHotspots = false, onHotspot }) {
  const wheelSize = { Aero18: 18, Split19: 19, Forge20: 20 }[wheel] || 18;
  const r = 18 + (wheelSize - 18) * 0.7; // visual exaggeration

  return (
    <svg viewBox="0 0 760 300" className="w-full h-auto" role="img" aria-label="Car preview">
      {/* body */}
      <g>
        <motion.path
          initial={{ opacity: 0.6 }}
          animate={{ opacity: 1 }}
          transition={{ duration: 0.4 }}
          d="M60 210 C120 120, 220 90, 370 90 L520 90 C580 90, 650 130, 690 180 L700 210 C705 220, 700 230, 690 230 L70 230 C55 230, 50 220, 60 210 Z"
          fill={color}
          stroke="#222"
          strokeWidth="3"
        />
        {/* windows */}
        <path d="M210 120 L350 110 L520 110 L560 160 L240 160 Z" fill="#e6f0ff" stroke="#222" strokeWidth="3" />
      </g>
      {/* wheels */}
      <g>
        <circle cx="210" cy="240" r={r} fill="#222" />
        <circle cx="540" cy="240" r={r} fill="#222" />
        <circle cx="210" cy="240" r={r - 6} fill="#bbb" />
        <circle cx="540" cy="240" r={r - 6} fill="#bbb" />
        {/* simple spokes */}
        {[0, 36, 72, 108, 144, 180].map((deg, i) => (
          <g key={i} transform={`rotate(${deg} 210 240)`}>
            <rect x="208" y={240 - (r - 10)} width="4" height={r - 10} fill="#777" />
          </g>
        ))}
        {[0, 36, 72, 108, 144, 180].map((deg, i) => (
          <g key={i} transform={`rotate(${deg} 540 240)`}>
            <rect x="538" y={240 - (r - 10)} width="4" height={r - 10} fill="#777" />
          </g>
        ))}
      </g>

      {/* Hotspots for learn mode */}
      {labelHotspots && (
        <g>
          <Hotspot x={390} y={150} label="Engine" onClick={() => onHotspot?.("engine")} />
          <Hotspot x={410} y={205} label="Transmission" onClick={() => onHotspot?.("transmission")} />
          <Hotspot x={375} y={230} label="Drivetrain" onClick={() => onHotspot?.("drivetrain")} />
          <Hotspot x={610} y={110} label="Aerodynamics" onClick={() => onHotspot?.("aero")} />
          <Hotspot x={210} y={270} label="Brakes" onClick={() => onHotspot?.("brakes")} />
          <Hotspot x={160} y={170} label="Suspension" onClick={() => onHotspot?.("suspension")} />
        </g>
      )}
    </svg>
  );
}

function Hotspot({ x, y, label, onClick }) {
  return (
    <g className="cursor-pointer" onClick={onClick}>
      <circle cx={x} cy={y} r={14} fill="white" stroke="#111" strokeWidth="2" />
      <text x={x} y={y + 4} textAnchor="middle" fontFamily="mono" fontSize="12">
        i
      </text>
      <text x={x} y={y - 18} textAnchor="middle" fontFamily="system-ui" fontSize="12" fill="#111">
        {label}
      </text>
    </g>
  );
}

// --- Main App ---
export default function DreamCarBuilderApp() {
  const [tab, setTab] = useState("build"); // build | learn | saved
  const [conf, setConf] = useState(() => {
    const fromURL = parseConfigFromURL();
    const saved = localStorage.getItem("dreamcar:last");
    return fromURL || JSON.parse(saved || "null") || DEFAULT;
  });

  const makeMeta = MAKES[conf.make];
  const modelMeta = makeMeta.models[conf.model];

  useEffect(() => {
    localStorage.setItem("dreamcar:last", JSON.stringify(conf));
    window.history.replaceState({}, "", buildShareURL(conf));
  }, [conf]);

  const basePrice = modelMeta.base;
  const price = useMemo(() => {
    const engineP = ENGINES[conf.engine]?.price || 0;
    const txP = TRANSMISSIONS[conf.transmission]?.price || 0;
    const colorP = COLORS.find((c) => c.id === conf.color)?.price || 0;
    const wheelP = WHEELS.find((w) => w.id === conf.wheels)?.price || 0;
    const interiorP = INTERIORS.find((i) => i.id === conf.interior)?.price || 0;
    const addonsP = sum(conf.addons.map((a) => ADDONS.find((x) => x.id === a)?.price || 0));
    return basePrice + engineP + txP + colorP + wheelP + interiorP + addonsP;
  }, [conf, basePrice]);

  const steps = [
    { key: "model", name: "Model" },
    { key: "power", name: "Powertrain" },
    { key: "style", name: "Style" },
    { key: "interior", name: "Interior" },
    { key: "addons", name: "Add‑ons" },
    { key: "summary", name: "Summary" },
  ];
  const [step, setStep] = useState(0);

  // --- Handlers ---
  const setField = (field, value) => setConf((c) => ({ ...c, [field]: value }));
  const toggleAddon = (id) =>
    setConf((c) => ({ ...c, addons: c.addons.includes(id) ? c.addons.filter((x) => x !== id) : [...c.addons, id] }));

  const saveBuild = () => {
    const all = JSON.parse(localStorage.getItem("dreamcar:saved") || "[]");
    const withId = { id: `Build-${Date.now()}`, createdAt: new Date().toISOString(), conf };
    localStorage.setItem("dreamcar:saved", JSON.stringify([withId, ...all]));
    alert("Saved! Check the ‘Saved’ tab.");
  };

  const exportJSON = () => {
    const blob = new Blob([JSON.stringify({ conf, price }, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = `dreamcar_${conf.model}_${Date.now()}.json`;
    a.click();
    URL.revokeObjectURL(url);
  };

  const pickMake = (mk) => {
    const firstModel = Object.keys(MAKES[mk].models)[0];
    const defaultEngine = MAKES[mk].models[firstModel].engines[0];
    const defaultDrive = MAKES[mk].models[firstModel].drivetrains[0];
    setConf((c) => ({ ...c, make: mk, model: firstModel, engine: defaultEngine, drivetrain: defaultDrive }));
  };

  return (
    <div className="min-h-screen bg-gradient-to-b from-gray-50 to-gray-100 text-gray-900">
      <header className="sticky top-0 z-20 backdrop-blur bg-white/80 border-b">
        <div className="max-w-7xl mx-auto px-4 md:px-6 py-3 flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="w-9 h-9 rounded-2xl bg-black text-white grid place-items-center font-bold">DC</div>
            <div>
              <h1 className="text-xl md:text-2xl font-semibold leading-tight">Dream Car Builder</h1>
              <p className="text-xs text-gray-500 -mt-0.5">Build your ride • Learn the parts • Share it</p>
            </div>
          </div>
          <div className="flex items-center gap-2">
            <Button variant={tab === "build" ? "solid" : "outline"} onClick={() => setTab("build")}>Build</Button>
            <Button variant={tab === "learn" ? "solid" : "outline"} onClick={() => setTab("learn")}>Learn</Button>
            <Button variant={tab === "saved" ? "solid" : "outline"} onClick={() => setTab("saved")}>Saved</Button>
          </div>
        </div>
      </header>

      <main className="max-w-7xl mx-auto px-4 md:px-6 py-6 md:py-8">
        {tab === "build" && (
          <div className="grid lg:grid-cols-12 gap-4 md:gap-6">
            {/* Left: Configurator */}
            <div className="lg:col-span-5 space-y-4">
              <Section
                title="Choose Make"
                right={<span className="text-sm text-gray-500">Base from {fmt(MAKES[conf.make].basePrice)}</span>}
              >
                <div className="flex flex-wrap">
                  {Object.keys(MAKES).map((mk) => (
                    <Chip key={mk} active={conf.make === mk} onClick={() => pickMake(mk)}>
                      {mk}
                    </Chip>
                  ))}
                </div>
              </Section>

              <Section title="{steps[step].name}">
                {/* Step content */}
                {step === 0 && (
                  <div>
                    <div className="flex flex-wrap mb-4">
                      {Object.keys(MAKES[conf.make].models).map((m) => (
                        <Chip key={m} active={conf.model === m} onClick={() => setField("model", m)}>
                          {m} · Base {fmt(MAKES[conf.make].models[m].base)}
                        </Chip>
                      ))}
                    </div>
                  </div>
                )}

                {step === 1 && (
                  <div className="space-y-3">
                    <div>
                      <div className="font-medium mb-1">Engine</div>
                      <div className="flex flex-wrap">
                        {modelMeta.engines.map((id) => (
                          <Chip key={id} active={conf.engine === id} onClick={() => setField("engine", id)}>
                            {ENGINES[id].name} {ENGINES[id].price ? `(+${fmt(ENGINES[id].price)})` : ""}
                          </Chip>
                        ))}
                      </div>
                    </div>
                    <div>
                      <div className="font-medium mb-1">Drivetrain</div>
                      <div className="flex flex-wrap">
                        {modelMeta.drivetrains.map((d) => (
                          <Chip key={d} active={conf.drivetrain === d} onClick={() => setField("drivetrain", d)}>
                            {d}
                          </Chip>
                        ))}
                      </div>
                    </div>
                    <div>
                      <div className="font-medium mb-1">Transmission</div>
                      <div className="flex flex-wrap">
                        {Object.entries(TRANSMISSIONS)
                          .filter(([k]) => (ENGINES[conf.engine].type === "EV" ? k === "SingleSpeed" : k !== "SingleSpeed"))
                          .map(([id, t]) => (
                            <Chip key={id} active={conf.transmission === id} onClick={() => setField("transmission", id)}>
                              {t.name} {t.price ? `(+${fmt(t.price)})` : ""}
                            </Chip>
                          ))}
                      </div>
                    </div>
                  </div>
                )}

                {step === 2 && (
                  <div className="space-y-3">
                    <div>
                      <div className="font-medium mb-1">Paint</div>
                      <div className="flex flex-wrap items-center gap-3">
                        {COLORS.map((c) => (
                          <button
                            key={c.id}
                            onClick={() => setField("color", c.id)}
                            className={`flex items-center gap-2 border rounded-xl px-2 py-1 pr-3 mr-1 mb-2 ${
                              conf.color === c.id ? "ring-2 ring-black" : ""
                            }`}
                            aria-label={c.name}
                          >
                            <span
                              className="inline-block w-7 h-7 rounded-lg border"
                              style={{ backgroundColor: c.id }}
                            />
                            <span className="text-sm">{c.name} {c.price ? `(+${fmt(c.price)})` : ""}</span>
                          </button>
                        ))}
                      </div>
                    </div>
                    <div>
                      <div className="font-medium mb-1">Wheels</div>
                      <div className="flex flex-wrap">
                        {WHEELS.map((w) => (
                          <Chip key={w.id} active={conf.wheels === w.id} onClick={() => setField("wheels", w.id)}>
                            {w.name} {w.price ? `(+${fmt(w.price)})` : ""}
                          </Chip>
                        ))}
                      </div>
                    </div>
                  </div>
                )}

                {step === 3 && (
                  <div>
                    <div className="flex flex-wrap">
                      {INTERIORS.map((i) => (
                        <Chip key={i.id} active={conf.interior === i.id} onClick={() => setField("interior", i.id)}>
                          {i.name} {i.price ? `(+${fmt(i.price)})` : ""}
                        </Chip>
                      ))}
                    </div>
                  </div>
                )}

                {step === 4 && (
                  <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">
                    {ADDONS.map((a) => (
                      <label key={a.id} className="flex items-center gap-3 border rounded-xl p-3 cursor-pointer">
                        <input
                          type="checkbox"
                          checked={conf.addons.includes(a.id)}
                          onChange={() => toggleAddon(a.id)}
                        />
                        <div className="text-sm">
                          <div className="font-medium">{a.name}</div>
                          <div className="text-gray-600">{fmt(a.price)}</div>
                        </div>
                      </label>
                    ))}
                  </div>
                )}

                {step === 5 && (
                  <div className="space-y-3 text-sm">
                    <div className="grid grid-cols-2 gap-2">
                      <div className="text-gray-600">Make</div>
                      <div>{conf.make}</div>
                      <div className="text-gray-600">Model</div>
                      <div>{conf.model}</div>
                      <div className="text-gray-600">Engine</div>
                      <div>{ENGINES[conf.engine].name}</div>
                      <div className="text-gray-600">Transmission</div>
                      <div>{TRANSMISSIONS[conf.transmission].name}</div>
                      <div className="text-gray-600">Drivetrain</div>
                      <div>{conf.drivetrain}</div>
                      <div className="text-gray-600">Wheels</div>
                      <div>{WHEELS.find((w) => w.id === conf.wheels)?.name}</div>
                      <div className="text-gray-600">Interior</div>
                      <div>{INTERIORS.find((i) => i.id === conf.interior)?.name}</div>
                      <div className="text-gray-600">Paint</div>
                      <div>{COLORS.find((c) => c.id === conf.color)?.name}</div>
                    </div>
                    <div>
                      <div className="font-medium mb-1">Add‑ons</div>
                      <ul className="list-disc list-inside text-gray-800">
                        {conf.addons.length ? (
                          conf.addons.map((a) => <li key={a}>{ADDONS.find((x) => x.id === a)?.name}</li>)
                        ) : (
                          <li className="text-gray-500">None selected</li>
                        )}
                      </ul>
                    </div>
                  </div>
                )}

                <div className="flex items-center justify-between mt-4">
                  <div className="text-lg font-semibold">Total: {fmt(price)}</div>
                  <div className="flex gap-2">
                    <Button variant="outline" onClick={() => setStep(Math.max(0, step - 1))} disabled={step === 0}>
                      Back
                    </Button>
                    <Button onClick={() => setStep(Math.min(steps.length - 1, step + 1))}>
                      {step === steps.length - 1 ? "Finish" : "Next"}
                    </Button>
                  </div>
                </div>
              </Section>

              <div className="flex items-center gap-2">
                <Button variant="outline" onClick={saveBuild}>Save build</Button>
                <Button variant="outline" onClick={exportJSON}>Export JSON</Button>
                <Button
                  variant="ghost"
                  onClick={() => {
                    navigator.clipboard.writeText(window.location.href);
                    alert("Share link copied!");
                  }}
                >
                  Copy share link
                </Button>
              </div>
            </div>

            {/* Right: Live Preview & Learn Hotspots */}
            <div className="lg:col-span-7">
              <Card className="p-4 md:p-6">
                <div className="flex items-start justify-between gap-4 flex-col lg:flex-row">
                  <div className="w-full lg:w-2/3">
                    <CarSVG color={conf.color} wheel={conf.wheels} labelHotspots={true} onHotspot={(k) => {
                      setTab("learn");
                      setTimeout(() => {
                        const el = document.getElementById(`comp-${k}`);
                        el?.scrollIntoView({ behavior: "smooth", block: "center" });
                        el?.classList.add("ring-2", "ring-black");
                        setTimeout(() => el?.classList.remove("ring-2", "ring-black"), 1400);
                      }, 60);
                    }} />
                  </div>
                  <div className="w-full lg:w-1/3">
                    <div className="text-sm text-gray-600">Configuration</div>
                    <div className="mt-2 space-y-1 text-sm">
                      <div className="flex justify-between"><span>{conf.make} {conf.model}</span><span>{fmt(basePrice)}</span></div>
                      <Row label={ENGINES[conf.engine].name} price={ENGINES[conf.engine].price} />
                      <Row label={TRANSMISSIONS[conf.transmission].name} price={TRANSMISSIONS[conf.transmission].price} />
                      <Row label={`${conf.drivetrain} Drivetrain`} price={0} />
                      <Row label={`${COLORS.find((c)=>c.id===conf.color)?.name} Paint`} price={COLORS.find((c)=>c.id===conf.color)?.price} />
                      <Row label={`${WHEELS.find((w)=>w.id===conf.wheels)?.name} Wheels`} price={WHEELS.find((w)=>w.id===conf.wheels)?.price} />
                      <Row label={`${INTERIORS.find((i)=>i.id===conf.interior)?.name} Interior`} price={INTERIORS.find((i)=>i.id===conf.interior)?.price} />
                      {conf.addons.map((a) => (
                        <Row key={a} label={ADDONS.find((x)=>x.id===a)?.name} price={ADDONS.find((x)=>x.id===a)?.price} />
                      ))}
                    </div>
                    <div className="border-t mt-3 pt-3 flex items-center justify-between">
                      <div className="font-semibold">Total</div>
                      <div className="text-lg font-bold">{fmt(price)}</div>
                    </div>
                  </div>
                </div>
              </Card>
            </div>
          </div>
        )}

        {tab === "learn" && (
          <LearnLibrary />
        )}

        {tab === "saved" && <SavedBuilds onLoad={(c)=>{ setConf(c); setTab("build"); }} />}
      </main>

      <footer className="max-w-7xl mx-auto px-4 md:px-6 pb-10 text-xs text-gray-500">
        <div className="flex flex-col md:flex-row items-center justify-between gap-2">
          <div>© {new Date().getFullYear()} Dream Car Builder</div>
          <div className="flex items-center gap-3">
            <a className="hover:underline" href="#" onClick={(e)=>{e.preventDefault(); window.scrollTo({top:0, behavior:'smooth'})}}>Back to top</a>
            <span>•</span>
            <span>Tip: Click the little “i” dots on the car to learn parts.</span>
          </div>
        </div>
      </footer>
    </div>
  );
}

function Row({ label, price }) {
  return (
    <div className="flex justify-between text-sm">
      <span>{label}</span>
      <span>{price ? `+${fmt(price)}` : "Included"}</span>
    </div>
  );
}

function LearnLibrary() {
  const [query, setQuery] = useState("");
  const items = COMPONENTS.filter(
    (c) => c.name.toLowerCase().includes(query.toLowerCase()) || c.summary.toLowerCase().includes(query.toLowerCase())
  );

  return (
    <div>
      <div className="flex items-center justify-between mb-3">
        <h2 className="text-xl md:text-2xl font-semibold">Learn: Car Components</h2>
        <div className="text-sm text-gray-500">Tap a card to expand</div>
      </div>

      <div className="mb-4 flex items-center gap-2">
        <input
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          placeholder="Search brakes, suspension, turbo…"
          className="w-full md:w-96 border rounded-xl px-3 py-2"
        />
      </div>

      <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-4">
        {items.map((c) => (
          <ComponentCard key={c.key} comp={c} />
        ))}
      </div>
    </div>
  );
}

function ComponentCard({ comp }) {
  const [open, setOpen] = useState(false);
  return (
    <Card id={`comp-${comp.key}`} className="p-4">
      <button className="w-full text-left" onClick={() => setOpen((o) => !o)}>
        <div className="flex items-center justify-between">
          <div className="flex items-center gap-3">
            <div className="text-2xl" aria-hidden>{comp.icon}</div>
            <div>
              <div className="font-semibold">{comp.name}</div>
              <div className="text-sm text-gray-600">{comp.summary}</div>
            </div>
          </div>
          <div className="text-gray-500 text-sm">{open ? "Hide" : "Details"}</div>
        </div>
      </button>
      <AnimatePresence initial={false}>
        {open && (
          <motion.div initial={{ height: 0, opacity: 0 }} animate={{ height: "auto", opacity: 1 }} exit={{ height: 0, opacity: 0 }} className="mt-3 text-sm text-gray-800">
            {comp.details}
          </motion.div>
        )}
      </AnimatePresence>
    </Card>
  );
}

function SavedBuilds({ onLoad }) {
  const [items, setItems] = useState(() => JSON.parse(localStorage.getItem("dreamcar:saved") || "[]"));

  const del = (id) => {
    const next = items.filter((x) => x.id !== id);
    setItems(next);
    localStorage.setItem("dreamcar:saved", JSON.stringify(next));
  };

  return (
    <div>
      <div className="flex items-center justify-between mb-3">
        <h2 className="text-xl md:text-2xl font-semibold">Saved Builds</h2>
        <Button variant="outline" onClick={() => { localStorage.removeItem("dreamcar:saved"); setItems([]); }}>Clear all</Button>
      </div>
      {items.length === 0 ? (
        <Card className="p-6 text-center text-gray-600">No saved builds yet. Save one from the Build tab.</Card>
      ) : (
        <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-4">
          {items.map(({ id, createdAt, conf }) => (
            <Card key={id} className="p-4">
              <div className="text-sm text-gray-500">{new Date(createdAt).toLocaleString()}</div>
              <div className="font-semibold mt-1">{conf.make} {conf.model}</div>
              <div className="text-sm text-gray-700">{ENGINES[conf.engine].name}</div>
              <div className="flex items-center gap-2 mt-2">
                <div className="w-6 h-6 rounded border" style={{ backgroundColor: conf.color }} />
                <div className="text-sm">{conf.wheels}</div>
              </div>
              <div className="mt-3 flex gap-2">
                <Button onClick={() => onLoad(conf)}>Load</Button>
                <Button variant="outline" onClick={() => del(id)}>Delete</Button>
              </div>
            </Card>
          ))}
        </div>
      )}
    </div>
  );
}

// --- URL helpers ---
function buildShareURL(conf) {
  const base = window.location.origin + window.location.pathname;
  const q = new URLSearchParams(conf).toString();
  return `${base}?${q}`;
}
function parseConfigFromURL() {
  const qs = new URLSearchParams(window.location.search);
  if ([...qs.keys()].length === 0) return null;
  const obj = Object.fromEntries(qs.entries());
  // coerce arrays
  if (obj.addons && typeof obj.addons === "string") obj.addons = obj.addons.split(",");
  return { ...DEFAULT, ...obj };
}

// --- Global styles (Tailwind required in host) ---
const style = document.createElement("style");
style.innerHTML = `
  :root { --ring: 0 0% 0%; }
  body { font-feature-settings: "cv02","cv03"; }
`;
if (typeof document !== "undefined") document.head.appendChild(style);
