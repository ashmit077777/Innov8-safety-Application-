# Innov8-safety-Application-
An Application for Safety 

<!DOCTYPE html>
<html lang="en" class="scroll-smooth">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Innov8 - Night Safety Travel App</title>
    <!-- Tailwind + Inter Font -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Leaflet -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <style>
        body { font-family: 'Inter', sans-serif; background-color: #020617; color: #E2E8F0; }
        .glass-nav { background-color: rgba(15, 23, 42, 0.9); backdrop-filter: blur(12px); border-bottom: 1px solid rgba(51, 65, 85, 0.7); }
        .feature-card { background-color: #020617; border: 1px solid #1f2937; transition: 0.25s; }
        .feature-card:hover { transform: translateY(-4px); border-color: #a855f7; }
        .cta-button:hover { transform: scale(1.05); }
        #map { background: radial-gradient(circle at top, #1d1b3a 0, #020617 55%, #000 100%); }
        .scanlines {
            pointer-events: none;
            position: fixed;
            inset: 0;
            background: repeating-linear-gradient(to bottom,
                rgba(15,23,42,0.18),
                rgba(15,23,42,0.18) 1px,
                transparent 2px,
                transparent 3px
            );
            opacity: 0.6;
            z-index: 5;
        }
        /* stealth visual class still available (feature preserved) */
        .stealth-mode { filter: brightness(0.5) grayscale(0.4); }
        .nv-filter { filter: hue-rotate(90deg) saturate(1.4) contrast(1.1); }
        .thermal-filter { filter: hue-rotate(-50deg) saturate(2) contrast(1.3); }
        .sonar-filter { filter: hue-rotate(200deg) saturate(1.8) contrast(1.15); }
        .sos-active {
            box-shadow: 0 0 25px rgba(248, 113, 113, 0.9),
                        0 0 60px rgba(248, 113, 113, 0.7);
        }
        /* About modal specific */
        .about-modal-backdrop {
            background: linear-gradient(180deg, rgba(2,6,23,0.85), rgba(2,6,23,0.95));
            backdrop-filter: blur(8px);
        }
        /* Compact header actions */
        .header-actions { display:flex; gap:0.5rem; align-items:center; }
        .hdr-btn {
            display:inline-flex;
            align-items:center;
            justify-content:center;
            gap:0.5rem;
            padding:0.45rem 0.8rem;
            border-radius:999px;
            border:1px solid rgba(148,163,184,0.12);
            font-size:0.78rem;
            font-weight:600;
            background: radial-gradient(circle at 0% 0%, rgba(129,140,248,0.25), transparent 55%);
            box-shadow: 0 0 0 1px rgba(15,23,42,0.9);
            transition: all .18s ease-out;
        }
        .hdr-btn:hover {
            transform: translateY(-1px) scale(1.03);
            box-shadow: 0 10px 30px rgba(15,23,42,0.9);
        }
        .hdr-btn .dot { width:8px; height:8px; border-radius:999px; display:inline-block; }
        /* Theme colors */
        .btn-openmap { background:linear-gradient(120deg,#4f46e5,#7c3aed); color:white; border-color: rgba(124,58,237,0.35); }
        .btn-sos { background:linear-gradient(120deg,#ef4444,#fb7185); color:white; border-color: rgba(239,68,68,0.4); }
        .btn-about { background: radial-gradient(circle at 0 0, rgba(56,189,248,0.18), transparent 55%); color:#93c5fd; border-color: rgba(99,102,241,0.25); }
        /* New: Profile button neon */
        .btn-profile {
            background: linear-gradient(120deg,#22c55e,#14b8a6);
            color:#ecfeff;
            border-color: rgba(34,197,94,0.45);
            position: relative;
            overflow:hidden;
        }
        .btn-profile::before {
            content:"";
            position:absolute;
            inset:-40%;
            background: conic-gradient(from 220deg, rgba(16,185,129,0.05), rgba(45,212,191,0.18), transparent 55%);
            opacity:0;
            transition: opacity .2s ease-out;
        }
        .btn-profile:hover::before { opacity:1; }
        /* Social buttons inside modal */
        .social-row { display:flex; gap:0.6rem; margin-top:6px; }
        .social-btn {
            display:inline-flex;
            align-items:center;
            justify-content:center;
            width:38px;
            height:38px;
            border-radius:10px;
            border:1px solid rgba(148,163,184,0.06);
            background: rgba(255,255,255,0.02);
            transition: transform .12s ease, box-shadow .12s ease;
            text-decoration:none;
        }
        .social-btn:hover { transform: translateY(-3px); box-shadow: 0 6px 18px rgba(0,0,0,0.5); }
        /* specific brand accents */
        .ig { background: linear-gradient(135deg, #f58529, #dd2a7b 50%, #8134af); color:white; border:none; }
        .tg { background: linear-gradient(135deg,#2AABEE,#0088CC); color:white; border:none; }
        .member-card { background: rgba(255,255,255,0.02); padding:10px; border-radius:10px; border:1px solid rgba(148,163,184,0.04); }
        /* New: Profile slide-in panel + backdrop */
        .profile-backdrop {
            position:fixed;
            inset:0;
            background: radial-gradient(circle at top, rgba(15,23,42,0.85), rgba(2,6,23,0.96));
            backdrop-filter: blur(10px);
            opacity:0;
            pointer-events:none;
            transition: opacity .18s ease-out;
            z-index:45;
        }
        .profile-backdrop.open {
            opacity:1;
            pointer-events:auto;
        }
        .profile-panel {
            position:fixed;
            top:0;
            right:0;
            height:100%;
            width: min(340px, 100%);
            background:
                radial-gradient(circle at top, rgba(56,189,248,0.15), transparent 55%),
                radial-gradient(circle at bottom, rgba(94,234,212,0.12), transparent 60%),
                rgba(15,23,42,0.98);
            border-left:1px solid rgba(45,212,191,0.35);
            box-shadow: -20px 0 45px rgba(0,0,0,0.8);
            transform: translateX(100%);
            transition: transform .22s ease-out;
            z-index:50;
            padding:1.25rem 1.1rem;
        }
        .profile-panel.open {
            transform: translateX(0);
        }
        .profile-avatar {
            width:44px;
            height:44px;
            border-radius:999px;
            background: conic-gradient(from 200deg, #22c55e, #0ea5e9, #22c55e);
            padding:2px;
            display:flex;
            align-items:center;
            justify-content:center;
        }
        .profile-avatar-inner {
            width:100%;
            height:100%;
            border-radius:999px;
            background: radial-gradient(circle at 30% 0, #1e293b, #020617);
            display:flex;
            align-items:center;
            justify-content:center;
            font-size:0.9rem;
            font-weight:700;
            color:#a5f3fc;
        }
        .profile-pill {
            font-size:0.65rem;
            text-transform:uppercase;
            letter-spacing:0.08em;
            padding:0.25rem 0.5rem;
            border-radius:999px;
            border:1px solid rgba(45,212,191,0.4);
            background: rgba(15,23,42,0.9);
            color:#6ee7b7;
        }
        .profile-section-title {
            font-size:0.75rem;
            text-transform:uppercase;
            letter-spacing:0.12em;
            color:#64748b;
            margin-bottom:0.3rem;
        }
        .profile-card {
            border-radius:0.9rem;
            border:1px solid rgba(148,163,184,0.14);
            background: rgba(15,23,42,0.96);
            padding:0.6rem 0.7rem;
        }
        .profile-quick-btn {
            display:flex;
            align-items:center;
            gap:0.35rem;
            font-size:0.72rem;
            padding:0.35rem 0.55rem;
            border-radius:999px;
            border:1px solid rgba(148,163,184,0.18);
            background: radial-gradient(circle at 0 0, rgba(59,130,246,0.22), transparent 55%);
            cursor:pointer;
        }
        .profile-quick-btn span.icon {
            width:16px;
            height:16px;
            border-radius:999px;
            display:flex;
            align-items:center;
            justify-content:center;
            font-size:0.65rem;
        }
        /* Emergency contacts inside profile */
        .contact-row {
            display:flex;
            align-items:center;
            justify-content:space-between;
            gap:0.4rem;
            padding:0.45rem 0.55rem;
            border-radius:0.75rem;
            border:1px solid rgba(148,163,184,0.24);
            background: radial-gradient(circle at 0 0, rgba(248,250,252,0.02), transparent 60%);
        }
        .contact-main {
            display:flex;
            flex-direction:column;
            gap:0.1rem;
            font-size:0.78rem;
        }
        .contact-actions {
            display:flex;
            flex-direction:column;
            gap:0.25rem;
        }
        .contact-btn {
            font-size:0.65rem;
            padding:0.15rem 0.5rem;
            border-radius:999px;
            border:1px solid rgba(148,163,184,0.35);
            background: rgba(15,23,42,0.95);
            white-space:nowrap;
            cursor:pointer;
        }
        .contact-btn.alert {
            border-color: rgba(248,113,113,0.65);
            background: radial-gradient(circle at 0 0, rgba(248,113,113,0.16), transparent 60%);
            color:#fecaca;
        }
        .contact-btn.delete {
            border-color: rgba(148,163,184,0.45);
            color:#cbd5f5;
        }
        /* Neon mic button near destination input */
        .neon-input-wrap {
            display:flex;
            gap:0.4rem;
            align-items:center;
            position: relative; /* for absolute suggestions */
        }
        .neon-mic-btn {
            width:42px;
            height:42px;
            border-radius:999px;
            border:1px solid rgba(129,140,248,0.6);
            background:
                radial-gradient(circle at 30% 0, rgba(248,250,252,0.18), transparent 55%),
                radial-gradient(circle at 100% 100%, rgba(79,70,229,0.75), rgba(30,64,175,0.95));
            display:flex;
            align-items:center;
            justify-content:center;
            cursor:pointer;
            box-shadow: 0 0 20px rgba(79,70,229,0.85);
            transition: transform .16s ease-out, box-shadow .16s ease-out, border-color .16s;
        }
        .neon-mic-btn svg {
            width:18px;
            height:18px;
        }
        .neon-mic-btn:hover {
            transform: translateY(-1px) scale(1.05);
        }
        .neon-mic-btn.active {
            border-color: rgba(34,197,94,0.9);
            box-shadow: 0 0 24px rgba(34,197,94,0.9);
        }
        /* Neon autocomplete dropdown */
        #suggestionsBox {
            background: rgba(15,23,42,0.96);
            border: 1px solid rgba(124,58,237,0.55);
            backdrop-filter: blur(10px);
            border-radius: 0.75rem;
            overflow: hidden;
            max-height: 230px;
            box-shadow: 0 18px 45px rgba(15,23,42,0.95);
        }
        .suggestion-item {
            padding: 9px 12px;
            font-size: 0.8rem;
            cursor: pointer;
            border-bottom: 1px solid rgba(124,58,237,0.18);
            color: #e5e7eb;
            transition: background 0.15s ease-out, color 0.15s ease-out;
            white-space: nowrap;
            text-overflow: ellipsis;
            overflow: hidden;
        }
        .suggestion-item:last-child {
            border-bottom: none;
        }
        .suggestion-item:hover {
            background: rgba(124,58,237,0.28);
            color: #e9d5ff;
        }
        /* Mobility / speed widget */
        .mobility-widget {
            position:absolute;
            bottom:1.1rem;
            left:1.1rem;
            padding:0.6rem 0.75rem;
            border-radius:1rem;
            border:1px solid rgba(56,189,248,0.55);
            background:
                radial-gradient(circle at 0 0, rgba(56,189,248,0.28), transparent 55%),
                radial-gradient(circle at 100% 100%, rgba(15,23,42,0.95), rgba(2,6,23,0.98));
            font-size:0.7rem;
            min-width:170px;
            box-shadow: 0 14px 40px rgba(15,23,42,0.85);
            z-index:30;
        }
        .mobility-header {
            display:flex;
            justify-content:space-between;
            align-items:center;
            margin-bottom:0.25rem;
        }
        .mobility-title {
            font-size:0.7rem;
            text-transform:uppercase;
            letter-spacing:0.16em;
            color:#e0f2fe;
        }
        .mobility-pill {
            font-size:0.6rem;
            padding:0.15rem 0.45rem;
            border-radius:999px;
            border:1px solid rgba(45,212,191,0.6);
            background: rgba(15,23,42,0.9);
            color:#a5f3fc;
        }
        .mobility-row {
            display:flex;
            justify-content:space-between;
            align-items:center;
            margin-top:0.1rem;
        }
        .mobility-label {
            color:#94a3b8;
            font-size:0.7rem;
        }
        .mobility-value {
            font-size:0.7rem;
            font-weight:600;
        }
        .mobility-value.main {
            font-size:0.85rem;
            color:#e0f2fe;
        }
        /* Guardian AI chat styles (added) */
        .guardian-chat-log {
            max-height: 220px;
            overflow-y: auto;
            padding: 0.25rem 0;
        }
        .guardian-msg {
            margin-bottom: 0.35rem;
            display: flex;
        }
        .guardian-msg.user {
            justify-content: flex-end;
        }
        .guardian-msg.ai {
            justify-content: flex-start;
        }
        .guardian-bubble {
            border-radius: 0.75rem;
            padding: 0.35rem 0.6rem;
            font-size: 0.7rem;
            max-width: 85%;
            line-height: 1.35;
        }
        .guardian-msg.user .guardian-bubble {
            background: #4f46e5;
            color: #e5e7eb;
        }
        .guardian-msg.ai .guardian-bubble {
            background: #020617;
            border: 1px solid rgba(148,163,184,0.6);
            color: #e2e8f0;
        }
        /* Responsive spacing */
        @media (min-width: 768px) {
            .header-actions { gap:0.75rem; }
            .hdr-btn { padding:0.55rem 0.95rem; font-size:0.9rem; }
            .social-btn { width:44px; height:44px; border-radius:12px; }
            .mobility-widget { bottom:1.4rem; left:1.4rem; }
        }
    </style>
</head>
<body class="antialiased">
<div class="scanlines"></div>
<!-- BOOT SCREEN -->
<div id="bootScreen" class="hidden fixed inset-0 z-50 flex flex-col items-center justify-center bg-black">
    <div class="text-center space-y-4">
        <p class="text-xs text-emerald-400 tracking-[0.3em] uppercase">initializing</p>
        <h2 class="text-3xl font-bold text-indigo-300">NIGHT PROTOCOL v3.2</h2>
        <div class="w-64 h-1 bg-slate-800 rounded-full overflow-hidden mx-auto">
            <div id="bootProgress" class="h-full w-0 bg-indigo-500 transition-all duration-300"></div>
        </div>
        <p id="bootLog" class="text-[11px] text-slate-400 font-mono mt-2">&gt; BOOT SEQUENCE START...</p>
    </div>
</div>
<!-- LOGIN PAGE -->
<section id="loginPage" class="flex items-center justify-center min-h-screen p-6">
    <div class="w-full max-w-md bg-slate-900/90 border border-slate-700 p-8 rounded-2xl shadow-2xl">
        <h1 class="text-3xl font-bold text-center mb-4 text-indigo-300">Innov8 Login</h1>
        <p class="text-center text-slate-400 text-sm mb-6">Enter ANY email & password to continue</p>
        <form class="space-y-5" onsubmit="handleLogin(event)">
            <div>
                <label class="text-sm">Email</label>
                <input id="email" type="email" required
                    class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-600" />
            </div>
            <div>
                <label class="text-sm">Password</label>
                <input id="password" type="password" required
                    class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-600" />
            </div>
            <button class="w-full py-3 bg-indigo-600 hover:bg-indigo-700 rounded-lg font-semibold cta-button">
                Login
            </button>
        </form>
        <p id="loginMsg" class="text-center text-sm mt-4 h-5"></p>
    </div>
</section>
<div id="appContent" class="hidden">
<!-- ===================== HEADER ===================== -->
<header class="glass-nav sticky top-0 z-40">
    <nav class="container mx-auto max-w-6xl px-6 py-3 flex justify-between items-center">
        <h1 class="text-2xl font-bold flex items-center text-white">
            <svg class="w-7 h-7 mr-2 text-indigo-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M9 12l2 2 4-4m5.618-4.016A11.955 11.955 0 0112 2.944a11.955 11.955 0 01-8.618 3.04A12.02 12.02 0 003 9c0 5.591 3.824 10.29 9 11.622 5.176-1.332 9-6.03 9-11.622 0-1.042-.133-2.052-.382-3.016z"></path>
            </svg>
            Innov8
        </h1>
        <!-- NAV LINKS -->
        <div class="hidden md:flex items-center gap-6 text-sm text-slate-300">
            <a href="#features" class="hover:text-indigo-400">Features</a>
            <a href="#guardianSection" class="hover:text-indigo-400">AI Guardian</a>
            <a href="#sosSection" class="hover:text-indigo-400">SOS</a>
        </div>
        <!-- HEADER ACTIONS -->
        <div class="header-actions">
            <!-- SOS -->
            <button class="hdr-btn btn-sos" onclick="triggerSOS()" id="hdrSosBtn" title="Activate SOS">
                <span class="dot" style="background:#ff6b6b;"></span>
                SOS
            </button>
            <!-- Open Map -->
            <button class="hdr-btn btn-openmap" onclick="showMap()" title="Open Map">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-[14px] h-[14px]" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 20l-5.447-2.724A2 2 0 013 15.382V6.618a2 2 0 011.553-1.894L9 2m6 18l5.447-2.724A2 2 0 0021 15.382V6.618a2 2 0 00-1.553-1.894L15 2M9 2v18M15 2v18" /></svg>
                <span class="hidden md:inline">Map</span>
            </button>
            <!-- About -->
            <button class="hdr-btn btn-about" onclick="showAbout()" id="hdrAboutBtn" title="Know more about us">
                <svg xmlns="http://www.w3.org/2000/svg" class="w-[13px] h-[13px]" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M12 20a8 8 0 100-16 8 8 0 000 16z"/></svg>
                <span class="hidden md:inline">About</span>
            </button>
            <!-- Profile -->
            <button class="hdr-btn btn-profile" onclick="openProfilePanel()" title="Operator Panel">
                <span class="dot" style="background:#4ade80;"></span>
                <span class="hidden md:inline">Profile</span>
                <svg xmlns="http://www.w3.org/2000/svg" class="w-[14px] h-[14px]" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15c3.866 0 7 1.567 7 3.5V20H5v-1.5C5 16.567 8.134 15 12 15z" />
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 12a4 4 0 100-8 4 4 0 000 8z" />
                </svg>
            </button>
        </div>
    </nav>
</header>
<!-- PROFILE PANEL -->
<div id="profileBackdrop" class="profile-backdrop"></div>
<div id="profilePanel" class="profile-panel">
    <div class="flex items-center justify-between mb-4">
        <div class="flex items-center gap-3">
            <div class="profile-avatar">
                <div class="profile-avatar-inner">
                    IN
                </div>
            </div>
            <div>
                <div class="text-sm font-semibold text-slate-100">Night Operator</div>
                <div class="text-xs text-slate-400">Innov8 Safety Console</div>
            </div>
        </div>
        <button onclick="closeProfilePanel()" class="text-xs text-slate-400 hover:text-slate-100">
            ✕
        </button>
    </div>
    <div class="flex items-center justify-between mb-3">
        <span class="profile-pill">PROFILE ONLINE</span>
        <span class="text-[10px] text-slate-400">
            Last login:
            <span id="lastLoginLabel" class="text-slate-200">Just now</span>
        </span>
    </div>
    <!-- Quick Actions -->
    <div class="mb-4">
        <div class="profile-section-title">Quick Actions</div>
        <div class="flex flex-wrap gap-2">
            <button class="profile-quick-btn" type="button" onclick="scrollToSection('mapSection')">
                <span class="icon bg-sky-500/20 text-sky-300">🗺️</span>
                Map & Routes
            </button>
            <button class="profile-quick-btn" type="button" onclick="scrollToSection('sosSection')">
                <span class="icon bg-rose-500/20 text-rose-300">⚠️</span>
                SOS Panel
            </button>
            <button class="profile-quick-btn" type="button" onclick="scrollToSection('guardianSection')">
                <span class="icon bg-emerald-500/20 text-emerald-300">🛡️</span>
                AI Guardian
            </button>
        </div>
    </div>
    <!-- Operator stats -->
    <div class="profile-card mb-4">
        <div class="flex items-center justify-between mb-2">
            <span class="text-xs text-slate-400">Operator Snapshot</span>
            <span class="text-[10px] text-emerald-300">Live</span>
        </div>
        <div class="grid grid-cols-3 gap-2 text-[11px]">
            <div>
                <div class="text-slate-400">Sessions</div>
                <div class="text-slate-100 font-semibold" id="profileSessionCount">1</div>
            </div>
            <div>
                <div class="text-slate-400">Risk Mode</div>
                <div class="text-slate-100 font-semibold" id="profileRiskMode">Adaptive</div>
            </div>
            <div>
                <div class="text-slate-400">Stealth</div>
                <div class="text-slate-100 font-semibold" id="profileStealthState">Off</div>
            </div>
        </div>
    </div>
    <!-- Emergency Contacts Manager -->
    <div class="mb-3">
        <div class="profile-section-title">Emergency Contacts</div>
        <form class="space-y-2 mb-3" onsubmit="handleAddContact(event)">
            <div class="grid grid-cols-5 gap-2">
                <input id="contactName" type="text" required
                       class="col-span-2 px-2 py-1.5 rounded-md bg-slate-900 border border-slate-700 text-xs"
                       placeholder="Name" />
                <input id="contactPhone" type="tel" required
                       class="col-span-2 px-2 py-1.5 rounded-md bg-slate-900 border border-slate-700 text-xs"
                       placeholder="Phone" />
                <button type="submit"
                        class="col-span-1 text-[11px] rounded-md border border-emerald-500/60 bg-emerald-500/15 text-emerald-200">
                    Add
                </button>
            </div>
            <p class="text-[10px] text-slate-500">
                Saved locally on this device. Use for quick distress messages.
            </p>
        </form>
        <div id="contactsList" class="space-y-1.5 max-h-48 overflow-y-auto pr-1 text-xs">
            <!-- Filled by JS -->
        </div>
    </div>
    <!-- Route share helper -->
    <div class="profile-card mt-3">
        <div class="flex items-center justify-between mb-1">
            <span class="text-xs text-slate-400">Share Location</span>
            <span class="text-[10px] text-sky-300">Beta</span>
        </div>
        <p class="text-[11px] text-slate-400 mb-2">
            Use an emergency contact's <span class="text-rose-300">"Alert SMS"</span> to send a prefilled message with your live coordinates.
        </p>
        <button type="button"
                onclick="copyLocationToClipboard()"
                class="w-full text-[11px] mt-1 py-1.5 rounded-md border border-sky-500/50 bg-sky-500/10 text-sky-100">
            Copy map location link
        </button>
    </div>
</div>
<!-- HERO -->
<section class="relative pt-24 pb-32 text-center overflow-hidden">
    <div class="absolute inset-0 opacity-40 pointer-events-none"
         style="background: radial-gradient(circle at 10% 20%, rgba(129, 140, 248, 0.3) 0, transparent 55%),
                        radial-gradient(circle at 90% 20%, rgba(236, 72, 153, 0.3) 0, transparent 55%);">
    </div>
    <div class="relative z-10">
        <h1 class="text-5xl md:text-6xl font-extrabold text-white mb-6 tracking-tight">
            Night Safety. <span class="text-indigo-400">Augmented.</span>
        </h1>
        <p class="text-slate-300 max-w-2xl mx-auto text-lg">
            A tactical, AI-assisted night travel companion that maps safer paths, connects cabs,
            and triggers help when needed.
        </p>
        <div class="mt-10 flex flex-col sm:flex-row gap-4 justify-center">
            <button onclick="showMap()" class="px-8 py-3 bg-indigo-600 rounded-lg font-semibold cta-button">
                Launch Navigation Console
            </button>
            <a href="#sosSection"
               class="px-8 py-3 border border-rose-500/60 text-rose-300 rounded-lg font-semibold hover:bg-rose-500/10">
                Open SOS / Panic Panel
            </a>
        </div>
        <!-- HUD -->
        <div class="mt-10 flex flex-wrap gap-4 justify-center text-xs">
            <div class="px-4 py-2 bg-slate-900/70 border border-slate-700 rounded-full flex items-center gap-2">
                <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span>
                <span id="heroRiskLabel">Risk: Calculating...</span>
            </div>
            <div class="px-4 py-2 bg-slate-900/70 border border-slate-700 rounded-full">
                KP3 Grid • Night Mode Online
            </div>
        </div>
    </div>
</section>
<!-- MAP SECTION -->
<section id="mapSection" class="hidden p-6 pb-20 max-w-6xl mx-auto">
    <div class="flex flex-col lg:flex-row lg:items-start lg:gap-6">
        <!-- LEFT PANEL -->
        <div class="lg:w-1/3 space-y-4 mb-6 lg:mb-0">
            <h2 class="text-3xl font-semibold mb-2">Navigation Console</h2>
            <p class="text-slate-400 text-sm">
                Default origin: <span class="text-indigo-300">Knowledge Park III, Greater Noida</span>.
            </p>
            <div class="space-y-3">
                <input id="currentInput" disabled
                       class="w-full px-4 py-3 rounded-lg bg-slate-800 border border-slate-700 text-sm"
                       placeholder="Fetching your location..." />
                <!-- Destination + Neon Mic + Suggestions -->
                <div class="neon-input-wrap">
                    <input id="destinationInput"
                           class="flex-1 px-4 py-3 rounded-lg bg-slate-800 border border-slate-700 text-sm"
                           placeholder="Enter destination (e.g. Alpha 1, Pari Chowk)" />
                    <button type="button" class="neon-mic-btn" onclick="startVoiceDestination()" title="Speak destination">
                        <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none">
                            <path d="M12 14a3 3 0 0 0 3-3V7a3 3 0 0 0-6 0v4a3 3 0 0 0 3 3z" stroke="rgba(15,23,42,0.95)" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"/>
                            <path d="M19 11a7 7 0 0 1-14 0m7 7v3" stroke="rgba(15,23,42,0.95)" stroke-width="1.7" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                    </button>
                    <div id="suggestionsBox" class="absolute z-50 w-full hidden left-0 top-full mt-2"></div>
                </div>
                <button onclick="searchDestination()"
                        class="w-full py-3 bg-indigo-600 hover:bg-indigo-700 rounded-lg text-sm font-semibold">
                    Find Destination & Draw Route
                </button>
                <button onclick="openGoogleRoute()"
                        class="w-full py-3 bg-emerald-600 hover:bg-emerald-700 rounded-lg text-sm font-semibold">
                    Start Route in Google Maps
                </button>
                <button onclick="showCabOptions()"
                        class="w-full py-3 bg-yellow-500 hover:bg-yellow-600 rounded-lg text-sm font-semibold text-black">
                    Find Cab (Uber / Ola / Rapido)
                </button>
                <div id="cabOptions"
                     class="hidden bg-slate-900 border border-slate-700 rounded-lg p-3 space-y-2 text-sm">
                    <button onclick="openUber()" class="w-full py-2 bg-indigo-600 hover:bg-indigo-700 rounded-lg">Uber</button>
                    <button onclick="openOla()"  class="w-full py-2 bg-green-600  hover:bg-green-700  rounded-lg">Ola</button>
                    <button onclick="openRapido()" class="w-full py-2 bg-orange-500 hover:bg-orange-600 rounded-lg">Rapido</button>
                </div>
                <button onclick="recenterMap()
                        " class="w-full py-2 bg-slate-800 hover:bg-slate-700 rounded-lg text-xs border border-slate-600">
                    Recenter to My Location
                </button>
                <div class="flex gap-2">
                    <button onclick="toggleHeatmap()
                            " class="w-1/2 py-2 bg-rose-500/20 hover:bg-rose-500/30 border border-rose-500/50 rounded-lg text-[11px]">
                        Toggle Heatmap
                    </button>
                    <button onclick="toggleRadar()"
                            class="w-1/2 py-2 bg-sky-500/20 hover:bg-sky-500/30 border border-sky-500/50 rounded-lg text-[11px]">
                        Toggle Radar
                    </button>
                </div>
            </div>
            <p id="distanceInfo" class="text-sm text-indigo-300 font-semibold mt-2 min-h-[1.5rem]"></p>
            <!-- Guardian Panel -->
            <div id="guardianPanel"
                 class="mt-4 bg-slate-900/80 border border-slate-700 rounded-xl p-4 text-xs space-y-2">
                <div class="flex items-center justify-between mb-1">
                    <p class="font-semibold text-slate-200 text-sm">AI Guardian</p>
                    <span id="guardianRiskBadge"
                          class="px-2 py-1 rounded-full text-[10px] bg-slate-800 border border-slate-600 text-slate-300">
                        RISK: UNKNOWN
                    </span>
                </div>
                <p id="guardianSummary" class="text-slate-300 text-[12px]">
                    Waiting for destination to analyze route safety...
                </p>
                <p id="guardianContext" class="text-slate-500 text-[11px]">
                    Time • Distance • Night factor • Movement
                </p>
                <p id="guardianTip" class="text-indigo-300 text-[11px]">
                    Tip: Prefer busy, well-lit areas after 8PM.
                </p>
            </div>
        </div>
        <!-- MAP DISPLAY -->
        <div class="lg:w-2/3">
            <div id="mapWrapper" class="relative w-full h-[70vh] border border-slate-800 rounded-2xl overflow-hidden bg-black">
                <div id="map" class="w-full h-full"></div>
                <!-- Compass -->
                <div id="compassWidget"
                     class="absolute top-3 left-3 bg-slate-900/80 border border-slate-700 rounded-xl px-3 py-2 text-[10px]">
                    <div class="relative w-14 h-14 rounded-full border border-slate-600 flex items-center justify-center">
                        <div id="compassRing" class="absolute inset-0 border border-indigo-500/60 rounded-full"></div>
                        <span class="text-[9px] text-slate-300">N</span>
                    </div>
                    <span id="compassHeadingText" class="text-[10px] text-indigo-300">--°</span>
                </div>
                <!-- Tactical HUD -->
                <div class="absolute top-3 right-3 bg-slate-900/80 border border-slate-700 rounded-xl px-3 py-2 text-[10px] space-y-1">
                    <div class="flex justify-between"><span>Mode:</span><span class="text-indigo-400">NIGHT WATCH</span></div>
                    <div class="flex justify-between"><span>Threat:</span><span id="hudThreat" class="text-emerald-400">LOW</span></div>
                    <div class="flex justify-between"><span>Radar:</span><span id="hudRadar" class="text-slate-400">OFF</span></div>
                </div>
                <!-- Radar -->
                <div id="radarOverlay" class="hidden absolute inset-0 pointer-events-none flex items-center justify-center">
                    <div class="relative w-64 h-64 rounded-full border border-sky-400/50"></div>
                </div>
                <!-- Mobility / speed widget -->
                <div id="mobilityWidget" class="mobility-widget">
                    <div class="mobility-header">
                        <span class="mobility-title">Mobility HUD</span>
                        <span class="mobility-pill" id="mobilityStatusBadge">Tracking</span>
                    </div>
                    <div class="mobility-row">
                        <span class="mobility-label">Speed</span>
                        <span class="mobility-value main" id="mobilitySpeed">-- km/h</span>
                    </div>
                    <div class="mobility-row">
                        <span class="mobility-label">Movement</span>
                        <span class="mobility-value" id="mobilityMovement">Standing</span>
                    </div>
                    <div class="mobility-row">
                        <span class="mobility-label">Steps (approx)</span>
                        <span class="mobility-value" id="mobilitySteps">0</span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</section>
<!-- FEATURES -->
<section id="features" class="py-20 bg-slate-950">
    <div class="container mx-auto max-w-6xl px-6">
        <h2 class="text-3xl font-semibold mb-10 text-center">Core Safety Systems</h2>
        <div class="grid md:grid-cols-3 gap-8">
            <div class="feature-card p-6 rounded-xl">
                <h3 class="text-2xl font-semibold mb-2 text-indigo-300">Safe Route Engine</h3>
                <p class="text-slate-300 text-sm">
                    OSRM-powered neon route drawing between KP3 and your destination.
                </p>
            </div>
            <div class="feature-card p-6 rounded-xl">
                <h3 class="text-2xl font-semibold mb-2 text-sky-300">Live Mobility + Cabs</h3>
                <p class="text-slate-300 text-sm">
                    Track yourself in real-time, jump directly to Uber / Ola / Rapido.
                </p>
            </div>
            <div class="feature-card p-6 rounded-xl">
                <h3 class="text-2xl font-semibold mb-2 text-rose-300">SOS / Panic Layer</h3>
                <p class="text-slate-300 text-sm">
                    Activate alarms, sirens and glowing UI pulse during emergencies.
                </p>
            </div>
        </div>
    </div>
</section>
<!-- GUARDIAN SECTION -->
<section id="guardianSection" class="py-20 bg-slate-950/90 border-t border-slate-800">
    <div class="max-w-5xl mx-auto px-6 grid md:grid-cols-2 gap-8 items-start">
        <div>
            <h2 class="text-3xl font-semibold mb-4 text-indigo-300">AI Guardian Overview</h2>
            <p class="text-slate-300 text-sm mb-4">
                The Guardian estimates your route safety based on distance, time of night,
                and whether you are walking or standing still.
            </p>
            <ul class="text-slate-400 text-sm space-y-2 list-disc list-inside">
                <li>Higher caution after 10 PM</li>
                <li>Warnings for long isolated routes</li>
                <li>Advises when standing still too long</li>
                <li>Suggests safer alternatives</li>
            </ul>
        </div>
        <div class="bg-slate-900/80 border border-slate-700 rounded-2xl p-5 text-xs space-y-3">
            <p class="text-slate-300 font-semibold mb-2">Live Guardian Snapshot</p>
            <p><span class="text-slate-500">Last Risk:</span> <span id="guardianLiveRisk" class="text-indigo-300">Waiting…</span></p>
            <p><span class="text-slate-500">Distance:</span> <span id="guardianLiveDistance" class="text-slate-300">Unknown</span></p>
            <p><span class="text-slate-500">Time Window:</span> <span id="guardianLiveTime" class="text-slate-300">Detecting…</span></p>
            <p><span class="text-slate-500">Movement:</span> <span id="guardianLiveMotion" class="text-slate-300">Standing</span></p>
            <p class="text-[11px] text-slate-500 border-t border-slate-800 pt-2"></p>
            <!-- Guardian AI Chat (added, without touching existing snapshot content) -->
            <div class="mt-2 rounded-xl border border-slate-700 bg-slate-950/60 p-3">
                <div class="flex items-center justify-between mb-2">
                    <span class="text-[11px] text-slate-300 font-semibold">Guardian AI Chat</span>
                    <span class="text-[10px] text-emerald-300">On-device helper</span>
                </div>
                <div id="guardianChatLog" class="guardian-chat-log"></div>
                <div class="mt-2 flex items-end gap-2">
                    <textarea id="guardianChatInput" rows="2"
                              class="flex-1 text-[11px] px-2 py-1.5 rounded-md bg-slate-900 border border-slate-700 focus:outline-none focus:border-indigo-500 resize-none"
                              placeholder="Ask about your route, cab safety, or what to do if you feel unsafe..."></textarea>
                    <button id="guardianChatSend" type="button"
                            class="px-3 py-1.5 text-[11px] rounded-md bg-indigo-600 hover:bg-indigo-700 font-semibold">
                        Send
                    </button>
                </div>
                <p class="mt-1 text-[10px] text-slate-500">
                    Uses smart local rules, no API keys or accounts.
                </p>
            </div>
        </div>
    </div>
</section>
<!-- SOS SECTION -->
<section id="sosSection" class="py-20 bg-gradient-to-b from-slate-950 to-black">
    <div class="max-w-3xl mx-auto px-6 text-center">
        <h2 class="text-3xl font-bold mb-4 text-rose-400">SOS / Panic Mode</h2>
        <p class="text-slate-300 mb-8 text-sm">
            Use SOS only when you genuinely feel unsafe.  
            It triggers a loud alarm + red flashing glow.
        </p>
        <button id="sosButton" onclick="triggerSOS()"
            class="relative inline-flex items-center justify-center px-10 py-4 bg-rose-600 hover:bg-rose-700 rounded-full text-lg font-bold text-white tracking-wide transition">
            <span class="absolute inline-flex h-full w-full rounded-full bg-rose-500 opacity-60 animate-ping"></span>
            <span class="relative z-10">ACTIVATE SOS</span>
        </button>
        <p id="sosStatus" class="text-slate-400 mt-4 text-sm h-6"></p>
        <audio id="sosAudio" src="https://actions.google.com/sounds/v1/alarms/alarm_clock.ogg" preload="auto"></audio>
        <audio id="ambientAudio" src="https://actions.google.com/sounds/v1/ambiences/night_crickets.ogg" preload="auto"></audio>
        <div class="mt-10 text-left text-xs text-slate-500 space-y-1">
            <p class="font-semibold text-slate-300">Emergency Tips:</p>
            <p>• Move toward well-lit areas</p>
            <p>• Call trusted friends or emergency contacts</p>
            <p>• Share your live location immediately</p>
            <p>• Avoid isolated shortcuts</p>
        </div>
    </div>
</section>
<!-- ABOUT MODAL -->
<div id="aboutModal" class="hidden fixed inset-0 z-50 flex items-center justify-center about-modal-backdrop">
    <div class="max-w-md w-full mx-4 bg-slate-900/95 border border-slate-700 rounded-2xl p-6 shadow-xl text-slate-200">
        <div class="flex justify-between items-start">
            <div>
                <h3 class="text-2xl font-bold text-indigo-300">Know more about us</h3>
                <p class="text-xs text-slate-400 mt-1">Team & members behind Innov8</p>
            </div>
            <button onclick="hideAbout()" class="ml-4 text-slate-400 hover:text-white text-sm">Close ✕</button>
        </div>
        <div class="mt-4 space-y-4 text-sm">
            <p class="text-slate-300 font-semibold">Team</p>
            <p class="text-indigo-300 font-bold">Innov8</p>
            <p class="text-slate-300 font-semibold mt-3">Team Members</p>
            <!-- ASHMIT -->
            <div class="member-card">
                <div class="flex items-center justify-between">
                    <div>
                        <div class="text-slate-300 font-medium">ASHMIT SAXENA</div>
                    </div>
                    <div class="social-row">
                        <a class="social-btn ig" href="https://www.instagram.com/_a_s_h_m_i_t_0_7_/" target="_blank" rel="noopener noreferrer" aria-label="Ashmit Instagram" title="Instagram" onclick="event.stopPropagation();">
                            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none">
                                <path d="M7 2h10a5 5 0 0 1 5 5v10a5 5 0 0 1-5 5H7a5 5 0 0 1-5-5V7a5 5 0 0 1 5-5z" stroke="rgba(255,255,255,0.9)" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                                <path d="M12 8.5a3.5 3.5 0 1 0 0 7 3.5 3.5 0 0 0 0-7z" stroke="rgba(255,255,255,0.95)" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                                <circle cx="17.5" cy="6.5" r="0.7" fill="rgba(255,255,255,0.95)"/>
                            </svg>
                        </a>
                        <a class="social-btn tg" href="https://www.linkedin.com/in/ashmit-saxena-84799132a?utm_source=share&utm_campaign=share_via&utm_content=profile&utm_medium=android_app" target="_blank" rel="noopener noreferrer" aria-label="Ashmit Telegram/LinkedIn" title="Telegram / LinkedIn" onclick="event.stopPropagation();">
                            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none">
                                <path d="M22 2L11 13" stroke="rgba(255,255,255,0.95)" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                                <path d="M22 2L15 22l-4-8-8-4 19-8z" stroke="rgba(255,255,255,0.85)" stroke-width="1.1" stroke-linecap="round" stroke-linejoin="round"/>
                            </svg>
                        </a>
                    </div>
                </div>
            </div>
            <!-- AYUSH -->
            <div class="member-card">
                <div class="flex items-center justify-between">
                    <div>
                        <div class="text-slate-300 font-medium">AYUSH</div>
                    </div>
                    <div class="social-row">
                        <a class="social-btn ig" href="https://www.instagram.com/ayushhhhhhhh.exe/" target="_blank" rel="noopener noreferrer" aria-label="Ayush Instagram" title="Instagram" onclick="event.stopPropagation();">
                            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none">
                                <path d="M7 2h10a5 5 0 0 1 5 5v10a5 5 0 0 1-5 5H7a5 5 0 0 1-5-5V7a5 5 0 0 1 5-5z" stroke="rgba(255,255,255,0.9)" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                                <path d="M12 8.5a3.5 3.5 0 1 0 0 7 3.5 3.5 0 0 0 0-7z" stroke="rgba(255,255,255,0.95)" stroke-width="1.2" stroke-linecap="round" stroke-linejoin="round"/>
                                <circle cx="17.5" cy="6.5" r="0.7" fill="rgba(255,255,255,0.95)"/>
                            </svg>
                        </a>
                        <a class="social-btn tg" href="https://www.linkedin.com/in/ayush-tiwari-2879b7315?utm_source=share&utm_campaign=share_via&utm_content=profile&utm_medium=android_app" target="_blank" rel="noopener noreferrer" aria-label="Ayush Telegram/LinkedIn" title="Telegram / LinkedIn" onclick="event.stopPropagation();">
                            <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none">
                                <path d="M22 2L11 13" stroke="rgba(255,255,255,0.95)" stroke-width="1.6" stroke-linecap="round" stroke-linejoin="round"/>
                                <path d="M22 2L15 22l-4-8-8-4 19-8z" stroke="rgba(255,255,255,0.85)" stroke-width="1.1" stroke-linecap="round" stroke-linejoin="round"/>
                            </svg>
                        </a>
                    </div>
                </div>
            </div>
            <p class="text-[11px] text-slate-500 mt-1">
                Team listing. You can update names and links here anytime.
            </p>
        </div>
        <div class="mt-6 flex justify-end">
            <button onclick="hideAbout()" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded-lg text-sm font-semibold">
                Close
            </button>
        </div>
    </div>
</div>
<!-- FOOTER -->
<footer class="py-10 bg-black border-t border-slate-800 text-center text-slate-500 text-xs">
    <p>&copy; 2025 Innov8 — Night Safety Project UI</p>
</footer>
</div>  <!-- END OF APP CONTENT -->
<!-- ===================== JS LOGIC ===================== -->
<!-- We'll include the full original JS but with the Guardian AI replaced/enhanced to call Google API directly (using your provided key), and fall back locally if needed. -->
<!-- math.js for local math support -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/11.8.0/math.min.js" crossorigin="anonymous"></script>

<script>
// ========== LOGIN ==========
function handleLogin(e) {
    e.preventDefault();
    const msg = document.getElementById("loginMsg");
    msg.textContent = "Login successful!";
    msg.className = "text-center text-green-400";
    setTimeout(() => {
        document.getElementById("loginPage").classList.add("hidden");
        const boot = document.getElementById("bootScreen");
        boot.classList.remove("hidden");
        runBootSequence();
    }, 500);
}
// ========== BOOT SEQUENCE ==========
function runBootSequence() {
    const boot = document.getElementById("bootScreen");
    const progress = document.getElementById("bootProgress");
    const log = document.getElementById("bootLog");
    const steps = [
        { percent: 15, text: "> LINKING GPS • • •", delay: 400 },
        { percent: 35, text: "> LOADING NIGHT GRID SHADERS • •", delay: 450 },
        { percent: 55, text: "> ARMING AI GUARDIAN SUBROUTINES •", delay: 450 },
        { percent: 80, text: "> DECRYPTING ROUTE DATABASE • • •", delay: 500 },
        { percent: 100, text: "> NIGHT PROTOCOL v3.2 READY. ENGAGE.", delay: 500 }
    ];
    let i = 0;
    function nextStep() {
        if (i < steps.length) {
            progress.style.width = steps[i].percent + "%";
            log.textContent = steps[i].text;
            i++;
            setTimeout(nextStep, steps[i - 1].delay);
        } else {
            setTimeout(() => {
                boot.classList.add("hidden");
                document.getElementById("appContent").classList.remove("hidden");
                updateGuardianPanel();
            }, 350);
        }
    }
    nextStep();
}
// ========== STEALTH MODE ==========
let stealthOn = false;
function toggleStealthMode() {
    stealthOn = !stealthOn;
    document.body.classList.toggle("stealth-mode", stealthOn);
    const stealthState = document.getElementById("profileStealthState");
    if (stealthState) stealthState.textContent = stealthOn ? "On" : "Off";
}
// keyboard shortcut Shift+S
document.addEventListener('keydown', (e) => {
    if (e.shiftKey && (e.key === 'S' || e.key === 's')) {
        toggleStealthMode();
    }
});
// ========== MAP + LOCATION ==========
let mapLoaded = false;
let map, userMarker, destMarker, routeLine;
let currentLat = 28.4743;  // KP3 default
let currentLon = 77.5040;  // KP3 default
let lastDistanceKm = null;
// Mobility tracking
let lastMotionPos = null;
let lastMotionTime = null;
let totalDistanceM = 0;
let totalSteps = 0;
const STEP_LENGTH_M = 0.78;
// heatmap / radar
let heatmapOn = false;
let radarOn = false;
let heatLayers = [];
let dangerZones = [];
// Autocomplete globals
let destInput, suggestionBox;
function showMap() {
    const section = document.getElementById("mapSection");
    section.classList.remove("hidden");
    if (!mapLoaded) {
        mapLoaded = true;
        initMap();
    }
}
function initMap() {
    map = L.map("map").setView([currentLat, currentLon], 15);
    L.tileLayer("https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png", {
        maxZoom: 19,
        attribution: "&copy; OpenStreetMap, &copy; CartoDB"
    }).addTo(map);
    userMarker = L.circleMarker([currentLat, currentLon], {
        radius: 8,
        color: "#22d3ee",
        fillColor: "#a855f7",
        fillOpacity: 0.9,
        weight: 3
    }).addTo(map).bindPopup("You are near KP3");
    document.getElementById("currentInput").value =
        `Lat: ${currentLat.toFixed(4)}, Lon: ${currentLon.toFixed(4)}`;
    initDangerZones();
    initCompass();
    updateGuardianPanel();
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
            (pos) => {
                currentLat = pos.coords.latitude;
                currentLon = pos.coords.longitude;
                updateUserLocationMarker();
                map.setView([currentLat, currentLon], 15);
            },
            () => {}
        );
        navigator.geolocation.watchPosition(
            handleMotionUpdate,
            () => {},
            { enableHighAccuracy: true, maximumAge: 5000, timeout: 10000 }
        );
    }
}
function updateUserLocationMarker() {
    if (!userMarker) return;
    userMarker.setLatLng([currentLat, currentLon]);
    const inp = document.getElementById("currentInput");
    if (inp) {
        inp.value = `Lat: ${currentLat.toFixed(4)}, Lon: ${currentLon.toFixed(4)}`;
    }
}
function recenterMap() {
    if (!map) return;
    map.setView([currentLat, currentLon], 15);
    if (userMarker) userMarker.openPopup();
}
// ========== DISTANCE + ROUTING ==========
function getDistance(lat1, lon1, lat2, lon2) {
    const R = 6371; // km
    const dLat = (lat2 - lat1) * Math.PI / 180;
    const dLon = (lon2 - lon1) * Math.PI / 180;
    const a =
        Math.sin(dLat / 2) ** 2 +
        Math.cos(lat1 * Math.PI / 180) *
        Math.cos(lat2 * Math.PI / 180) *
        Math.sin(dLon / 2) ** 2;
    return R * (2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a)));
}
function searchDestination() {
    const query = document.getElementById("destinationInput").value.trim();
    if (!query) {
        alert("Enter a destination first");
        return;
    }
    fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(query)}`)
        .then(res => res.json())
        .then(data => {
            if (!data || !data.length) {
                alert("Destination not found");
                return;
            }
            const lat = parseFloat(data[0].lat);
            const lon = parseFloat(data[0].lon);
            if (destMarker) map.removeLayer(destMarker);
            destMarker = L.circleMarker([lat, lon], {
                radius: 7,
                color: "#f472b6",
                fillColor: "#f9a8d4",
                fillOpacity: 0.9,
                weight: 3
            }).addTo(map).bindPopup("Destination").openPopup();
            const dist = getDistance(currentLat, currentLon, lat, lon);
            lastDistanceKm = dist;
            document.getElementById("distanceInfo").textContent =
                `Distance: ${dist.toFixed(2)} km (approx)`;
            drawRoute(lat, lon);
            updateGuardianPanel();
        })
        .catch(() => {
            alert("Error while finding destination");
        });
}
function drawRoute(destLat, destLon) {
    if (!map) return;
    const url =
        `https://router.project-osrm.org/route/v1/driving/${currentLon},${currentLat};${destLon},${destLat}?overview=full&geometries=geojson`;
    fetch(url)
        .then(res => res.json())
        .then(data => {
            if (!data.routes || !data.routes.length) return;
            const coords = data.routes[0].geometry.coordinates.map(c => [c[1], c[0]]);
            if (routeLine) map.removeLayer(routeLine);
            routeLine = L.polyline(coords, {
                color: "#f97316",
                weight: 4,
                opacity: 0.9
            }).addTo(map);
            map.fitBounds(routeLine.getBounds(), { padding: [30, 30] });
        })
        .catch(() => {
            console.log("OSRM error");
        });
}
function openGoogleRoute() {
    const destination = document.getElementById("destinationInput").value.trim();
    if (!destination) {
        alert("Enter a destination first!");
        return;
    }
    const origin = `${currentLat},${currentLon}`;
    const url =
        `https://www.google.com/maps/dir/?api=1&origin=${origin}&destination=${encodeURIComponent(destination)}&travelmode=walking`;
    window.open(url, "_blank");
}
// ========== CABS ==========
function showCabOptions() {
    document.getElementById("cabOptions").classList.toggle("hidden");
}
function openUber() {
    const dest = document.getElementById("destinationInput").value.trim();
    if (!dest) return alert("Enter destination first");
    const url =
        `https://m.uber.com/ul/?action=setPickup&pickup=${currentLat},${currentLon}&dropoff[formatted_address]=${encodeURIComponent(dest)}`;
    window.open(url, "_blank");
}
function openOla() {
    const dest = document.getElementById("destinationInput").value.trim();
    if (!dest) return alert("Enter destination first");
    const url =
        `https://book.olacabs.com/?lat=${currentLat}&lng=${currentLon}&drop_lat=&drop_lng=&ds=${encodeURIComponent(dest)}`;
    window.open(url, "_blank");
}
function openRapido() {
    const dest = document.getElementById("destinationInput").value.trim();
    if (!dest) return alert("Enter destination first");
    const url =
        `https://rapido.bike/book?pickup_lat=${currentLat}&pickup_lng=${currentLon}&drop_text=${encodeURIComponent(dest)}`;
    window.open(url, "_blank");
}
// ========== DANGER ZONES + HEATMAP ==========
function initDangerZones() {
    dangerZones = [
        { lat: 28.4725, lon: 77.5025, level: "HIGH" },
        { lat: 28.4760, lon: 77.5075, level: "MEDIUM" },
        { lat: 28.4785, lon: 77.5005, level: "LOW" }
    ];
}
function toggleHeatmap() {
    if (!map) return;
    heatmapOn = !heatmapOn;
    if (heatmapOn) {
        dangerZones.forEach(z => {
            const color = z.level === "HIGH"
                ? "#ef4444"
                : z.level === "MEDIUM"
                ? "#f97316"
                : "#eab308";
            const radius = z.level === "HIGH"
                ? 250
                : z.level === "MEDIUM"
                ? 200
                : 150;
            const layer = L.circle([z.lat, z.lon], {
                radius,
                color,
                fillColor: color,
                fillOpacity: 0.22,
                weight: 1,
                dashArray: "4 6"
            }).addTo(map);
            heatLayers.push(layer);
        });
    } else {
        heatLayers.forEach(l => map.removeLayer(l));
        heatLayers = [];
    }
}
// ========== RADAR ==========
function toggleRadar() {
    radarOn = !radarOn;
    const overlay = document.getElementById("radarOverlay");
    const hudRadar = document.getElementById("hudRadar");
    if (radarOn) {
        overlay.classList.remove("hidden");
        hudRadar.textContent = "ON";
        hudRadar.className = "text-sky-300";
    } else {
        overlay.classList.add("hidden");
        hudRadar.textContent = "OFF";
        hudRadar.className = "text-slate-400";
    }
}
// ========== COMPASS ==========
let compassHeading = 0;
let compassFallbackInterval = null;
function initCompass() {
    if (window.DeviceOrientationEvent) {
        window.addEventListener("deviceorientation", (event) => {
            if (typeof event.alpha === "number") {
                compassHeading = event.alpha;
                updateCompassUI();
            }
        });
        setTimeout(() => {
            if (compassHeading === 0 && !compassFallbackInterval) {
                compassFallbackInterval = setInterval(() => {
                    compassHeading = (compassHeading + 3) % 360;
                    updateCompassUI();
                }, 200);
            }
        }, 2000);
    } else {
        compassFallbackInterval = setInterval(() => {
            compassHeading = (compassHeading + 3) % 360;
            updateCompassUI();
        }, 200);
    }
}
function updateCompassUI() {
    const ring = document.getElementById("compassRing");
    const text = document.getElementById("compassHeadingText");
    if (!ring || !text) return;
    ring.style.transform = `rotate(${(-compassHeading).toFixed(0)}deg)`;
    text.textContent = `${compassHeading.toFixed(0)}°`;
}
// ========== AI GUARDIAN PANEL ==========
function updateGuardianPanel() {
    const now = new Date();
    const hour = now.getHours();
    let riskScore = 0;
    let timeBucket = "";
    if (hour >= 22 || hour < 5) {
        timeBucket = "Late Night (High caution)";
        riskScore += 2;
    } else if (hour >= 19 && hour < 22) {
        timeBucket = "Evening / Early Night";
        riskScore += 1;
    } else {
        timeBucket = "Daytime / Safer window";
    }
    let distanceLabel = "No route yet";
    if (lastDistanceKm != null) {
        if (lastDistanceKm < 2) {
            distanceLabel = "Short walk (< 2 km)";
        } else if (lastDistanceKm < 7) {
            distanceLabel = "Medium route (2–7 km)";
            riskScore += 1;
        } else {
            distanceLabel = "Long route (> 7 km)";
            riskScore += 2;
        }
    }
    let baseRisk = "LOW";
    if (riskScore >= 4) baseRisk = "HIGH";
    else if (riskScore >= 2) baseRisk = "MEDIUM";
    const badge = document.getElementById("guardianRiskBadge");
    const heroRiskLabel = document.getElementById("heroRiskLabel");
    const hudThreat = document.getElementById("hudThreat");
    const summary = document.getElementById("guardianSummary");
    const context = document.getElementById("guardianContext");
    const tip = document.getElementById("guardianTip");
    const liveRisk = document.getElementById("guardianLiveRisk");
    const liveDistance = document.getElementById("guardianLiveDistance");
    const liveTime = document.getElementById("guardianLiveTime");
    const liveMotion = document.getElementById("guardianLiveMotion");
    let badgeColor =
        "px-2 py-1 rounded-full text-[10px] bg-emerald-600/30 border border-emerald-500/60 text-emerald-200";
    let riskText = `RISK: LOW`;
    if (baseRisk === "MEDIUM") {
        badgeColor =
            "px-2 py-1 rounded-full text-[10px] bg-amber-600/30 border border-amber-500/60 text-amber-200";
        riskText = "RISK: MEDIUM";
    } else if (baseRisk === "HIGH") {
        badgeColor =
            "px-2 py-1 rounded-full text-[10px] bg-rose-600/30 border border-rose-500/60 text-rose-200";
        riskText = "RISK: HIGH";
    }
    if (badge) {
        badge.className = badgeColor;
        badge.textContent = riskText;
    }
    if (heroRiskLabel) heroRiskLabel.textContent = `Risk Level: ${baseRisk}`;
    if (hudThreat) {
        hudThreat.textContent = baseRisk;
        hudThreat.className =
            "text-xs " +
            (baseRisk === "LOW"
                ? "text-emerald-400"
                : baseRisk === "MEDIUM"
                ? "text-amber-400"
                : "text-rose-400");
    }
    if (summary) {
        if (lastDistanceKm == null) {
            summary.textContent =
                "No active destination. Set a route for a better safety estimate.";
        } else {
            summary.textContent =
                `Estimated risk is ${baseRisk.toLowerCase()} for this ${lastDistanceKm.toFixed(1)} km route at this time. Stay alert and avoid empty shortcuts.`;
        }
    }
    if (context)
        context.textContent = `Time: ${timeBucket} • Distance: ${distanceLabel}`;
    if (tip) {
        tip.textContent =
            baseRisk === "HIGH"
                ? "Tip: Prefer main roads, avoid isolated lanes, and consider taking a cab instead of walking."
                : baseRisk === "MEDIUM"
                ? "Tip: Share your live location and stay on well-lit, busy paths."
                : "Tip: Even in safer windows, keep your phone accessible and stay aware of surroundings.";
    }
    if (liveRisk) liveRisk.textContent = baseRisk;
    if (liveDistance) liveDistance.textContent = distanceLabel;
    if (liveTime) liveTime.textContent = timeBucket;
    if (liveMotion) liveMotion.textContent = "Assuming walking pace";
}
// ========== SOS / PANIC ==========
let sosOn = false;
function triggerSOS() {
    const btn = document.getElementById("sosButton");
    const hdrBtn = document.getElementById("hdrSosBtn");
    const status = document.getElementById("sosStatus");
    const alarm = document.getElementById("sosAudio");
    sosOn = !sosOn;
    if (hdrBtn) {
        hdrBtn.classList.toggle('sos-active', sosOn);
    }
    if (sosOn) {
        if (btn) btn.classList.add("sos-active");
        if (status) {
            status.textContent = "SOS ACTIVE – Alarm and visual alert ON.";
            status.className = "text-rose-400 mt-4 text-sm";
        }
        if (alarm) {
            alarm.currentTime = 0;
            alarm.loop = true;
            alarm.play().catch(() => {});
        }
    } else {
        if (btn) btn.classList.remove("sos-active");
        if (status) {
            status.textContent = "SOS turned off.";
            status.className = "text-slate-400 mt-4 text-sm";
        }
        if (alarm) alarm.pause();
    }
}
// ========== ABOUT MODAL ==========
function showAbout() {
    const m = document.getElementById("aboutModal");
    if (m) m.classList.remove("hidden");
}
function hideAbout() {
    const m = document.getElementById("aboutModal");
    if (m) m.classList.add("hidden");
}
document.addEventListener("keydown", (e) => {
    if (e.key === "Escape") {
        hideAbout();
        closeProfilePanel();
    }
});
document.addEventListener("click", (ev) => {
    const modal = document.getElementById("aboutModal");
    if (!modal) return;
    if (!modal.classList.contains("hidden") && ev.target === modal) hideAbout();
});
// ========== PROFILE PANEL ==========
function openProfilePanel() {
    const panel = document.getElementById("profilePanel");
    const backdrop = document.getElementById("profileBackdrop");
    if (panel) panel.classList.add("open");
    if (backdrop) backdrop.classList.add("open");
}
function closeProfilePanel() {
    const panel = document.getElementById("profilePanel");
    const backdrop = document.getElementById("profileBackdrop");
    if (panel) panel.classList.remove("open");
    if (backdrop) backdrop.classList.remove("open");
}
function scrollToSection(id) {
    const el = document.getElementById(id);
    if (!el) return;
    closeProfilePanel();
    el.scrollIntoView({ behavior: "smooth" });
}
// ========== EMERGENCY CONTACTS ==========
let emergencyContacts = [];
function loadContactsFromStorage() {
    try {
        const raw = localStorage.getItem("innov8_emergency_contacts");
        if (!raw) {
            emergencyContacts = [];
        } else {
            emergencyContacts = JSON.parse(raw) || [];
        }
    } catch (e) {
        emergencyContacts = [];
    }
    renderContacts();
}
function saveContactsToStorage() {
    try {
        localStorage.setItem("innov8_emergency_contacts", JSON.stringify(emergencyContacts));
    } catch (e) {}
}
function handleAddContact(e) {
    e.preventDefault();
    const nameEl = document.getElementById("contactName");
    const phoneEl = document.getElementById("contactPhone");
    if (!nameEl || !phoneEl) return;
    const name = nameEl.value.trim();
    const phone = phoneEl.value.trim();
    if (!name || !phone) return;
    emergencyContacts.push({ name, phone });
    saveContactsToStorage();
    renderContacts();
    nameEl.value = "";
    phoneEl.value = "";
}
function renderContacts() {
    const list = document.getElementById("contactsList");
    if (!list) return;
    if (!emergencyContacts.length) {
        list.innerHTML = `<p class="text-[11px] text-slate-500">No emergency contacts added yet.</p>`;
        return;
    }
    list.innerHTML = emergencyContacts
        .map((c, idx) => `
            <div class="contact-row">
                <div class="contact-main">
                    <span class="text-slate-100 font-medium">${c.name}</span>
                    <span class="text-slate-400 text-[11px]">${c.phone}</span>
                </div>
                <div class="contact-actions">
                    <button type="button" class="contact-btn alert" onclick="smsContact(${idx})">
                        Alert SMS
                    </button>
                    <button type="button" class="contact-btn delete" onclick="deleteContact(${idx})">
                        Remove
                    </button>
                </div>
            </div>
        `)
        .join("");
}
function deleteContact(index) {
    if (index < 0 || index >= emergencyContacts.length) return;
    emergencyContacts.splice(index, 1);
    saveContactsToStorage();
    renderContacts();
}
function smsContact(index) {
    const c = emergencyContacts[index];
    if (!c) return;
    const body = `This is an alert from Innov8. I may need help. My last known location: https://www.google.com/maps?q=${currentLat},${currentLon}`;
    const link = `sms:${encodeURIComponent(c.phone)}?&body=${encodeURIComponent(body)}`;
    window.location.href = link;
}
function copyLocationToClipboard() {
    const text = `https://www.google.com/maps?q=${currentLat},${currentLon}`;
    if (navigator.clipboard && navigator.clipboard.writeText) {
        navigator.clipboard.writeText(text).then(() => {
            alert("Location link copied to clipboard.");
        }).catch(() => {
            alert("Could not copy. You can manually copy from the address bar after opening Maps.");
        });
    } else {
        alert("Clipboard not available. You can manually share the location from Google Maps.");
    }
}
// ========== LAST LOGIN ==========
function setLastLoginLabel() {
    const el = document.getElementById("lastLoginLabel");
    if (!el) return;
    const now = new Date();
    el.textContent = now.toLocaleString();
}
// ========== VOICE DESTINATION INPUT ==========
let voiceRecognition = null;
function startVoiceDestination() {
    const micBtn = document.querySelector(".neon-mic-btn");
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
        alert("Voice input is not supported on this browser/device.");
        return;
    }
    if (!voiceRecognition) {
        voiceRecognition = new SpeechRecognition();
        voiceRecognition.lang = "en-IN";
        voiceRecognition.interimResults = false;
        voiceRecognition.maxAlternatives = 1;
        voiceRecognition.onresult = (e) => {
            const transcript = e.results[0][0].transcript;
            const destInputLocal = document.getElementById("destinationInput");
            if (destInputLocal) destInputLocal.value = transcript;
        };
        voiceRecognition.onstart = () => {
            if (micBtn) micBtn.classList.add("active");
        };
        voiceRecognition.onend = () => {
            if (micBtn) micBtn.classList.remove("active");
        };
        voiceRecognition.onerror = () => {
            if (micBtn) micBtn.classList.remove("active");
        };
    }
    try {
        voiceRecognition.start();
    } catch (err) {}
}
// ========== MOBILITY WIDGET ==========
function handleMotionUpdate(pos) {
    const lat = pos.coords.latitude;
    const lon = pos.coords.longitude;
    const nowMs = pos.timestamp ? pos.timestamp : Date.now();
    currentLat = lat;
    currentLon = lon;
    updateUserLocationMarker();
    let speedKmh = 0;
    let movement = "Standing";
    if (lastMotionPos && lastMotionTime) {
        const dt = (nowMs - lastMotionTime) / 1000;
        const distKm = getDistance(lastMotionPos.lat, lastMotionPos.lon, lat, lon);
        const distM = distKm * 1000;
        if (dt > 0.5 && distM > 0.5) {
            speedKmh = (distM / dt) * 3.6;
            totalDistanceM += distM;
            totalSteps += distM / STEP_LENGTH_M;
            if (speedKmh < 0.5) {
                movement = "Standing";
            } else if (speedKmh < 6) {
                movement = "Walking";
            } else if (speedKmh < 15) {
                movement = "Running";
            } else {
                movement = "In vehicle";
            }
        }
    }
    lastMotionPos = { lat, lon };
    lastMotionTime = nowMs;
    updateMobilityWidget(speedKmh, Math.round(totalSteps), movement);
}
function updateMobilityWidget(speedKmh, steps, movement) {
    const speedEl = document.getElementById("mobilitySpeed");
    const stepsEl = document.getElementById("mobilitySteps");
    const movementEl = document.getElementById("mobilityMovement");
    const badgeEl = document.getElementById("mobilityStatusBadge");
    if (!speedEl || !stepsEl || !movementEl || !badgeEl) return;
    speedEl.textContent = `${speedKmh.toFixed(1)} km/h`;
    stepsEl.textContent = steps.toString();
    movementEl.textContent = movement;
    if (movement === "Standing") {
        badgeEl.textContent = "Idle";
        badgeEl.style.borderColor = "rgba(148,163,184,0.7)";
        badgeEl.style.color = "#cbd5f5";
    } else if (movement === "Walking") {
        badgeEl.textContent = "On foot";
        badgeEl.style.borderColor = "rgba(34,197,94,0.85)";
        badgeEl.style.color = "#bbf7d0";
    } else if (movement === "Running") {
        badgeEl.textContent = "High Motion";
        badgeEl.style.borderColor = "rgba(249,115,22,0.9)";
        badgeEl.style.color = "#fed7aa";
    } else {
        badgeEl.textContent = "Vehicle";
        badgeEl.style.borderColor = "rgba(59,130,246,0.9)";
        badgeEl.style.color = "#bfdbfe";
    }
}
// ========== AUTOCOMPLETE FOR DESTINATION ==========
function fetchSuggestions(q) {
    fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${encodeURIComponent(q)}&addressdetails=1&limit=8`)
        .then(res => res.json())
        .then(data => {
            if (data && data.length) {
                renderSuggestions(data);
            } else if (suggestionBox) {
                suggestionBox.classList.add("hidden");
            }
        })
        .catch(() => {
            if (suggestionBox) suggestionBox.classList.add("hidden");
        });
}
function renderSuggestions(results) {
    if (!suggestionBox) return;
    suggestionBox.innerHTML = "";
    suggestionBox.classList.remove("hidden");
    results.forEach(r => {
        const item = document.createElement("div");
        item.className = "suggestion-item";
        item.textContent = r.display_name;
        item.onclick = () => selectSuggestion(r.display_name);
        suggestionBox.appendChild(item);
    });
}
function selectSuggestion(name) {
    if (!destInput || !suggestionBox) return;
    destInput.value = name;
    suggestionBox.classList.add("hidden");
}
document.addEventListener("click", (ev) => {
    if (!suggestionBox || !destInput) return;
    if (!suggestionBox.contains(ev.target) && ev.target !== destInput) {
        suggestionBox.classList.add("hidden");
    }
});
// ========== GUARDIAN AI CHAT (NEW: tries Google Generative API, falls back to local advanced assistant) ==========
/*
  This implementation will:
  1) Attempt to call Google Generative API (generateText) directly from the client using the API key you provided.
  2) If that call fails (network/CORS/limits), it will fall back to a capable on-device assistant that handles:
     - math (via math.js),
     - Wikipedia quick summaries,
     - route/safety advice (your original rules),
     - small conversions.
 
  NOTES:
  - You provided the key and asked to use it. I embedded it below.
  - In production, it's safer to call an app-hosted proxy that stores the key server-side.
*/

// -------- CONFIG: your API key (you provided this) --------
const GOOGLE_API_KEY = "AIzaSyCqoZSs5kIISGh4SpPPPijXq_Ftzd5d8lM"; // user's key
const GEMINI_MO
