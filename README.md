<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Sequence — Classic Tabletop Multiplayer</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
    <style>
        * {
            box-sizing: border-box;
            -webkit-tap-highlight-color: transparent;
            user-select: none;
            font-family: 'Inter', sans-serif;
        }
        html, body {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            overflow: hidden;
            background-color: #0b1120;
            color: #f8fafc;
        }
        /* Custom sleek scrollbar */
        ::-webkit-scrollbar { width: 4px; height: 4px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #334155; border-radius: 9999px; }

        /* Board Aspect Ratio & Maximum Screen Usage */
        #board-wrapper {
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 4px;
            overflow: hidden;
        }
        #sequence-board {
            display: grid;
            grid-template-columns: repeat(10, 1fr);
            grid-template-rows: repeat(10, 1fr);
            gap: 2px;
            background-color: #1e293b;
            border: 2px solid #334155;
            border-radius: 8px;
            padding: 2px;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.6), 0 8px 10px -6px rgba(0, 0, 0, 0.6);
            aspect-ratio: 1 / 1;
            max-width: 100%;
            max-height: 100%;
        }
        @media (min-width: 640px) {
            #sequence-board {
                gap: 3px;
                padding: 4px;
                border-radius: 12px;
            }
        }

        /* Individual Board Cell */
        .board-cell {
            position: relative;
            background-color: #f8fafc;
            border-radius: 3px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            cursor: pointer;
            transition: transform 0.08s ease, background-color 0.12s ease;
            overflow: hidden;
            line-height: 1;
        }
        .board-cell:active {
            transform: scale(0.92);
        }
        .board-cell.corner-cell {
            background: linear-gradient(135deg, #fbbf24 0%, #d97706 100%);
            color: #1e293b;
            font-weight: 900;
        }
        .cell-rank {
            font-weight: 800;
            font-size: clamp(9px, 2.5vw, 15px);
            letter-spacing: -0.04em;
        }
        .cell-suit {
            font-size: clamp(9px, 2.4vw, 15px);
            margin-top: -1px;
        }
        .suit-red { color: #dc2626; }
        .suit-black { color: #0f172a; }

        /* Chips on Board */
        .chip {
            position: absolute;
            width: 82%;
            height: 82%;
            border-radius: 9999px;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 2px 4px rgba(0,0,0,0.5), inset 0 2px 3px rgba(255,255,255,0.45);
            border: 2px solid rgba(255,255,255,0.85);
            animation: chipDrop 0.18s cubic-bezier(0.34, 1.56, 0.64, 1);
            z-index: 10;
        }
        .chip::after {
            content: '';
            width: 58%;
            height: 58%;
            border-radius: 9999px;
            border: 1.5px dashed rgba(255,255,255,0.65);
        }
        .chip-blue { background: radial-gradient(circle at 30% 30%, #3b82f6, #1d4ed8); }
        .chip-green { background: radial-gradient(circle at 30% 30%, #22c55e, #15803d); }
        .chip-red { background: radial-gradient(circle at 30% 30%, #ef4444, #b91c1c); }
        .chip-yellow { background: radial-gradient(circle at 30% 30%, #eab308, #a16207); }

        /* Locked Sequence Indicator */
        .chip.locked-seq {
            box-shadow: 0 0 0 2px #fde047, 0 0 10px 2px rgba(250, 204, 21, 0.85);
            border-color: #fef08a;
        }
        .chip.locked-seq::after {
            content: '★';
            border: none;
            color: #fef9c3;
            font-size: clamp(8px, 2vw, 13px);
            display: flex;
            align-items: center;
            justify-content: center;
            width: 100%;
            height: 100%;
        }

        /* Last played subtle marker */
        .last-move-dot {
            position: absolute;
            top: 2px;
            right: 2px;
            width: 6px;
            height: 6px;
            border-radius: 9999px;
            background-color: #facc15;
            box-shadow: 0 0 4px #000;
            z-index: 20;
        }

        @keyframes chipDrop {
            0% { transform: scale(1.45); opacity: 0.4; }
            100% { transform: scale(1); opacity: 1; }
        }

        @keyframes shakeBoard {
            0%, 100% { transform: translateX(0); }
            20%, 60% { transform: translateX(-5px); }
            40%, 80% { transform: translateX(5px); }
        }
        .shake {
            animation: shakeBoard 0.28s ease-in-out;
        }

        /* Hand Cards */
        .hand-card {
            min-width: 52px;
            width: 56px;
            height: 78px;
            background-color: #ffffff;
            border-radius: 8px;
            border: 2px solid #cbd5e1;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 4px 5px;
            cursor: pointer;
            transition: transform 0.15s ease, box-shadow 0.15s ease, border-color 0.15s ease;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.4);
            position: relative;
            flex-shrink: 0;
        }
        @media (min-width: 640px) {
            .hand-card {
                width: 66px;
                height: 92px;
                padding: 5px 7px;
            }
        }
        .hand-card.selected {
            transform: translateY(-10px) scale(1.05);
            border-color: #facc15;
            box-shadow: 0 0 0 2px #facc15, 0 10px 15px -3px rgba(0, 0, 0, 0.6);
            z-index: 20;
        }
        .hand-card.dead-card {
            background-color: #fee2e2;
            border-color: #f87171;
        }
    </style>
</head>
<body class="flex flex-col h-screen w-screen overflow-hidden bg-slate-950 text-slate-100">

    <!-- Toast Notification Banner -->
    <div id="toast-container" class="fixed top-12 left-1/2 -translate-x-1/2 z-50 pointer-events-none flex flex-col items-center gap-1.5 w-11/12 max-w-sm"></div>

    <!-- ==================== VIEW 1: SIMPLE MAIN MENU ==================== -->
    <div id="view-menu" class="flex-1 flex flex-col items-center justify-center p-4 overflow-y-auto">
        <div class="w-full max-w-sm bg-slate-900/95 border border-slate-800 rounded-2xl p-6 shadow-2xl space-y-5">
            <!-- Header -->
            <div class="text-center space-y-1">
                <div class="inline-flex items-center justify-center gap-1.5 px-3 py-1 rounded-full bg-amber-500/10 border border-amber-500/30 text-amber-400 text-xs font-bold tracking-wider uppercase">
                    <span>♠</span><span>♥</span><span>♣</span><span>♦</span> Official Rules
                </div>
                <h1 class="text-3xl font-black tracking-tight text-white">SEQUENCE</h1>
                <p class="text-xs text-slate-400">Play 1v1 up to 6v6 on any phone or PC</p>
            </div>

            <!-- Connection Status Badge -->
            <div id="conn-badge" class="flex items-center justify-center gap-2 text-xs py-1.5 px-3 rounded-lg bg-slate-800/80 text-slate-300">
                <span id="conn-dot" class="w-2 h-2 rounded-full bg-amber-400 animate-pulse"></span>
                <span id="conn-text">Connecting to game server...</span>
            </div>

            <!-- Player Name Input -->
            <div class="space-y-1.5">
                <label for="player-name-input" class="block text-xs font-semibold uppercase tracking-wider text-slate-400">Your Name</label>
                <input id="player-name-input" type="text" maxlength="14" placeholder="Enter your nickname"
                    class="w-full px-3.5 py-3 rounded-xl bg-slate-800 border border-slate-700 text-white placeholder-slate-500 font-semibold text-base focus:outline-none focus:border-amber-400 transition" />
            </div>

            <!-- Host Game Button -->
            <button id="btn-create-room"
                class="w-full py-3.5 px-4 rounded-xl bg-gradient-to-r from-amber-500 to-orange-500 hover:from-amber-400 hover:to-orange-400 active:scale-[0.98] text-slate-950 font-extrabold text-base shadow-lg shadow-amber-500/20 transition flex items-center justify-center gap-2">
                <span>Create Game (Host)</span>
            </button>

            <!-- Divider -->
            <div class="relative flex py-1 items-center">
                <div class="flex-grow border-t border-slate-800"></div>
                <span class="flex-shrink mx-3 text-xs font-semibold text-slate-500 uppercase">Or Join Room</span>
                <div class="flex-grow border-t border-slate-800"></div>
            </div>

            <!-- Join Room Form -->
            <div class="flex gap-2">
                <input id="room-code-input" type="text" maxlength="5" placeholder="CODE"
                    class="w-1/2 px-3 py-3 rounded-xl bg-slate-800 border border-slate-700 text-center text-white placeholder-slate-500 font-black tracking-widest uppercase text-base focus:outline-none focus:border-blue-400 transition" />
                <button id="btn-join-room"
                    class="w-1/2 py-3 px-4 rounded-xl bg-blue-600 hover:bg-blue-500 active:scale-[0.98] text-white font-bold text-base shadow-lg shadow-blue-600/20 transition">
                    Join Game
                </button>
            </div>

            <!-- Bottom Utility Links -->
            <div class="pt-2 flex items-center justify-between border-t border-slate-800/80 text-xs text-slate-400">
                <button id="btn-open-rules-menu" class="hover:text-white underline underline-offset-2 py-1">How to Play</button>
                <button id="btn-download-html" class="hover:text-amber-400 flex items-center gap-1 py-1" title="Save this game as a single HTML file to run anywhere">
                    <span>⬇ Save Offline File</span>
                </button>
            </div>
        </div>
    </div>

    <!-- ==================== VIEW 2: LOBBY ROOM ==================== -->
    <div id="view-lobby" class="hidden flex-1 flex flex-col p-3 sm:p-5 max-w-lg mx-auto w-full overflow-y-auto">
        <div class="bg-slate-900 border border-slate-800 rounded-2xl p-4 sm:p-6 shadow-2xl space-y-4 my-auto">
            <!-- Top Room Code Header -->
            <div class="flex items-center justify-between border-b border-slate-800 pb-3">
                <div>
                    <span class="text-[11px] font-bold uppercase tracking-wider text-slate-400 block">Room Code</span>
                    <div class="flex items-center gap-2">
                        <span id="lobby-room-code" class="text-2xl sm:text-3xl font-black tracking-widest text-amber-400">----</span>
                        <button id="btn-copy-code" class="px-2.5 py-1 rounded-lg bg-slate-800 hover:bg-slate-700 text-xs font-semibold text-slate-200 border border-slate-700 active:scale-95 transition">
                            Copy
                        </button>
                    </div>
                </div>
                <button id="btn-leave-lobby" class="px-3 py-1.5 rounded-lg bg-rose-500/10 hover:bg-rose-500/20 text-rose-400 border border-rose-500/30 text-xs font-bold">
                    Leave
                </button>
            </div>

            <!-- Host Game Configuration -->
            <div id="host-settings-panel" class="space-y-3 bg-slate-800/50 p-3 rounded-xl border border-slate-700/60">
                <div class="flex items-center justify-between">
                    <span class="text-xs font-bold uppercase tracking-wider text-amber-400">Match Setup</span>
                    <span id="lobby-role-badge" class="text-[11px] px-2 py-0.5 rounded bg-slate-700 text-slate-300 font-semibold">Host</span>
                </div>

                <div class="grid grid-cols-2 gap-2.5">
                    <div>
                        <label class="block text-[11px] text-slate-400 mb-1 font-semibold">Game Mode</label>
                        <select id="select-game-mode" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2.5 py-2 text-xs sm:text-sm font-bold text-white focus:outline-none focus:border-amber-400">
                            <option value="1v1">1v1 (2 Players • 7 Cards)</option>
                            <option value="2v2" selected>2v2 (4 Players • 6 Cards)</option>
                            <option value="1v1v1">1v1v1 (3 Players • 6 Cards)</option>
                            <option value="3v3">3v3 (6 Players • 5 Cards)</option>
                            <option value="4v4_2">4v4 — 2 Teams (8 Players • 4 Cards)</option>
                            <option value="4v4_4">2v2v2v2 — 4 Teams (8 Players • 4 Cards)</option>
                            <option value="6v6_2">6v6 — 2 Teams (12 Players • 3 Cards)</option>
                            <option value="6v6_3">4v4v4 — 3 Teams (12 Players • 3 Cards)</option>
                        </select>
                    </div>
                    <div>
                        <label class="block text-[11px] text-slate-400 mb-1 font-semibold">Sequences to Win</label>
                        <select id="select-win-seq" class="w-full bg-slate-900 border border-slate-700 rounded-lg px-2.5 py-2 text-xs sm:text-sm font-bold text-white focus:outline-none focus:border-amber-400">
                            <option value="1">1 Sequence</option>
                            <option value="2" selected>2 Sequences (Official)</option>
                            <option value="3">3 Sequences</option>
                        </select>
                    </div>
                </div>
            </div>

            <!-- Team Switcher for Current Player -->
            <div class="space-y-1.5">
                <span class="block text-xs font-bold uppercase tracking-wider text-slate-400">Choose Your Team Chip</span>
                <div id="team-picker-buttons" class="grid grid-cols-4 gap-2">
                    <!-- Populated dynamically based on active teams -->
                </div>
            </div>

            <!-- Connected Players List -->
            <div class="space-y-1.5">
                <div class="flex justify-between items-center">
                    <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Players in Room</span>
                    <span id="lobby-player-count" class="text-xs font-bold text-slate-300">0 / 4</span>
                </div>
                <div id="lobby-players-list" class="space-y-1.5 max-h-48 overflow-y-auto pr-1">
                    <!-- Player rows -->
                </div>
            </div>

            <!-- Start Game CTA (Host Only) -->
            <div class="pt-1">
                <button id="btn-start-game" class="hidden w-full py-3.5 rounded-xl bg-emerald-500 hover:bg-emerald-400 active:scale-[0.98] text-slate-950 font-black text-base shadow-lg shadow-emerald-500/20 transition">
                    Start Game
                </button>
                <p id="waiting-host-msg" class="text-center text-xs text-slate-400 py-2 animate-pulse">
                    Waiting for host to configure teams and start...
                </p>
            </div>
        </div>
    </div>

    <!-- ==================== VIEW 3: ACTIVE GAME BOARD ==================== -->
    <div id="view-game" class="hidden flex-1 flex flex-col w-full h-full overflow-hidden">

        <!-- Ultra-Compact Top HUD Bar -->
        <header class="h-11 sm:h-12 px-2 sm:px-4 bg-slate-900 border-b border-slate-800 flex items-center justify-between shrink-0 gap-1">
            <!-- Left: Room Code & Menu -->
            <div class="flex items-center gap-1.5">
                <button id="btn-game-menu" class="p-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs font-bold border border-slate-700" title="Menu & Rules">
                    ☰
                </button>
                <span id="hud-room-code" class="text-xs font-black tracking-wider text-amber-400 bg-slate-800/90 px-2 py-1 rounded border border-slate-700/70">----</span>
            </div>

            <!-- Center: Team Scores (Sequences) -->
            <div id="hud-team-scores" class="flex items-center gap-1.5 sm:gap-2 overflow-x-auto py-0.5">
                <!-- Populated dynamically: e.g. Blue 1/2, Green 0/2 -->
            </div>

            <!-- Right: Turn Indicator -->
            <div id="hud-turn-pill" class="px-2.5 py-1 rounded-full text-[11px] sm:text-xs font-extrabold truncate max-w-[130px] sm:max-w-[200px] bg-slate-800 text-slate-200 border border-slate-700">
                Waiting...
            </div>
        </header>

        <!-- Compact Opponents Strip (Shows ONLY card counts, never cards!) -->
        <div id="hud-players-strip" class="h-7 px-2 bg-slate-950/90 border-b border-slate-800/80 flex items-center gap-2 overflow-x-auto shrink-0 text-[11px]">
            <!-- Populated with player chip dot, name, and card count icon -->
        </div>

        <!-- Center Massive 10x10 Board Area -->
        <main id="board-wrapper" class="flex-1 relative bg-slate-950">
            <div id="sequence-board">
                <!-- 100 cells rendered via JS -->
            </div>
        </main>

        <!-- Bottom Player Hand & Actions Tray -->
        <footer class="bg-slate-900 border-t border-slate-800 px-2 pt-2 pb-2.5 sm:px-4 sm:py-3 shrink-0 flex flex-col gap-1.5">
            <!-- Action / Info Sub-bar -->
            <div class="flex items-center justify-between text-xs px-1">
                <div class="flex items-center gap-2">
                    <span id="my-team-badge" class="inline-flex items-center gap-1.5 font-extrabold text-xs px-2 py-0.5 rounded bg-slate-800">
                        <span id="my-team-dot" class="w-2.5 h-2.5 rounded-full bg-blue-500"></span>
                        <span id="my-team-label">Blue Team</span>
                    </span>
                    <span id="deck-count-label" class="text-[11px] text-slate-400">Deck: 104</span>
                </div>

                <div class="flex items-center gap-2">
                    <!-- Dead Card Exchange Button -->
                    <button id="btn-dead-card" class="hidden px-2.5 py-1 rounded-lg bg-rose-600 hover:bg-rose-500 text-white font-bold text-[11px] shadow animate-pulse">
                        ♻ Swap Dead Card
                    </button>
                    <!-- Last Discard Pill -->
                    <div id="last-discard-box" class="flex items-center gap-1 text-[11px] text-slate-400 bg-slate-800 px-2 py-0.5 rounded border border-slate-700">
                        <span>Last:</span>
                        <span id="last-discard-card" class="font-black text-white">--</span>
                    </div>
                </div>
            </div>

            <!-- Private Cards Row (ONLY visible to this player) -->
            <div id="my-hand-container" class="flex items-center justify-start sm:justify-center gap-1.5 sm:gap-2.5 overflow-x-auto py-1 px-1 min-h-[84px] sm:min-h-[98px]">
                <!-- Private cards rendered here -->
            </div>
        </footer>
    </div>

    <!-- ==================== MODAL: VICTORY OVERLAY ==================== -->
    <div id="modal-victory" class="hidden fixed inset-0 z-50 bg-slate-950/85 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="w-full max-w-sm bg-slate-900 border-2 border-amber-400 rounded-2xl p-6 text-center space-y-4 shadow-2xl">
            <div class="text-5xl">🏆</div>
            <h2 id="victory-title" class="text-2xl font-black text-white">Blue Team Wins!</h2>
            <p id="victory-subtitle" class="text-sm text-slate-300">Completed 2 Sequences</p>
            <div class="pt-2 flex flex-col gap-2">
                <button id="btn-play-again" class="hidden w-full py-3 rounded-xl bg-amber-500 hover:bg-amber-400 text-slate-950 font-black text-base">
                    Return to Lobby (Play Again)
                </button>
                <button id="btn-victory-leave" class="w-full py-2.5 rounded-xl bg-slate-800 hover:bg-slate-700 text-slate-300 font-bold text-sm">
                    Exit to Main Menu
                </button>
            </div>
        </div>
    </div>

    <!-- ==================== MODAL: IN-GAME MENU ==================== -->
    <div id="modal-game-menu" class="hidden fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="w-full max-w-xs bg-slate-900 border border-slate-700 rounded-2xl p-5 space-y-3 shadow-2xl">
            <div class="flex justify-between items-center border-b border-slate-800 pb-2">
                <h3 class="font-extrabold text-base text-white">Game Menu</h3>
                <button id="btn-close-game-menu" class="text-slate-400 hover:text-white text-lg font-bold px-2">✕</button>
            </div>
            <button id="btn-menu-rules" class="w-full py-2.5 px-3 rounded-xl bg-slate-800 hover:bg-slate-700 text-left text-sm font-semibold text-white">
                📖 Official Rules & Jack Guide
            </button>
            <button id="btn-menu-copy-code" class="w-full py-2.5 px-3 rounded-xl bg-slate-800 hover:bg-slate-700 text-left text-sm font-semibold text-amber-400">
                📋 Copy Room Code
            </button>
            <button id="btn-menu-download" class="w-full py-2.5 px-3 rounded-xl bg-slate-800 hover:bg-slate-700 text-left text-sm font-semibold text-slate-200">
                ⬇ Save Offline HTML File
            </button>
            <button id="btn-menu-leave-game" class="w-full py-2.5 px-3 rounded-xl bg-rose-600/20 hover:bg-rose-600/30 border border-rose-500/40 text-left text-sm font-bold text-rose-400">
                🚪 Leave Match
            </button>
        </div>
    </div>

    <!-- ==================== MODAL: OFFICIAL RULES ==================== -->
    <div id="modal-rules" class="hidden fixed inset-0 z-50 bg-slate-950/85 backdrop-blur-sm flex items-center justify-center p-4">
        <div class="w-full max-w-md bg-slate-900 border border-slate-700 rounded-2xl p-5 space-y-3 shadow-2xl max-h-[85vh] overflow-y-auto">
            <div class="flex justify-between items-center border-b border-slate-800 pb-2">
                <h3 class="font-black text-lg text-amber-400">Official Sequence Rules</h3>
                <button id="btn-close-rules" class="text-slate-400 hover:text-white text-lg font-bold px-2">✕</button>
            </div>
            <div class="space-y-2.5 text-xs sm:text-sm text-slate-300 leading-relaxed">
                <p><strong class="text-white">🎯 Goal:</strong> Form the required number of <strong>Sequences</strong> (5 chips of your team color in a continuous horizontal, vertical, or diagonal line).</p>
                <p><strong class="text-amber-400">★ Corner FREE Spaces:</strong> The 4 corners are wild and count as a chip for every team. You only need 4 chips + a corner to complete a sequence!</p>
                <p><strong class="text-emerald-400">👁️👁️ Two-Eyed Jacks (J♣, J♦):</strong> <strong>WILD!</strong> Play on any open space on the board to place your chip.</p>
                <p><strong class="text-rose-400">👁️ One-Eyed Jacks (J♠, J♥):</strong> <strong>REMOVE!</strong> Remove any 1 opponent chip from the board. <em>You CANNOT remove a chip that is already part of a completed/locked Sequence.</em></p>
                <p><strong class="text-sky-400">♻ Dead Cards:</strong> If both matching spaces for a card in your hand are already occupied, select that card and tap <strong>Swap Dead Card</strong> (once per turn before playing) to replace it.</p>
                <p><strong class="text-purple-400">🔗 Overlapping Sequences:</strong> When 2 sequences are needed to win, your second sequence may share at most <strong>1 chip</strong> with your first sequence.</p>
                <p><strong class="text-slate-400">👀 True Tabletop Mode:</strong> Selecting a card does not highlight spaces on the board—scan carefully and tap the exact matching card!</p>
            </div>
            <button id="btn-rules-got-it" class="w-full py-2.5 rounded-xl bg-amber-500 text-slate-950 font-extrabold text-sm mt-2">
                Got It
            </button>
        </div>
    </div>

    <!-- ==================== FIREBASE & COMPLETE GAME ENGINE ==================== -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // =========================================================================
        // 1. OFFICIAL 10x10 SEQUENCE BOARD LAYOUT (EXACT STANDARD LAYOUT)
        // =========================================================================
        const BOARD_LAYOUT = [
            ["FREE", "2S", "3S", "4S", "5S", "6S", "7S", "8S", "9S", "FREE"],
            ["6C",   "5C", "4C", "3C", "2C", "AH", "KH", "QH", "10H","10S"],
            ["7C",   "AS", "2D", "3D", "4D", "5D", "6D", "7D", "9H", "QS"],
            ["8C",   "KS", "6C", "5C", "4C", "3C", "2C", "8D", "8H", "KS"],
            ["9C",   "QS", "7C", "6H", "5H", "4H", "AH", "9D", "7H", "AS"],
            ["10C",  "10S","8C", "7H", "2H", "3H", "KH", "10D","6H", "2D"],
            ["QC",   "9S", "9C", "8H", "9H", "10H","QH", "QD", "5H", "3D"],
            ["KC",   "8S", "10C","QC", "KC", "AC", "AD", "KD", "4H", "4D"],
            ["AC",   "7S", "6S", "5S", "4S", "3S", "2S", "2H", "3H", "5D"],
            ["FREE", "AD", "KD", "QD", "10D","9D", "8D", "7D", "6D", "FREE"]
        ];

        const SUIT_SYMBOLS = { S: "♠", H: "♥", D: "♦", C: "♣" };
        const RED_SUITS = new Set(["H", "D"]);

        const TEAM_META = {
            blue:   { name: "Blue",   dot: "bg-blue-500",    chip: "chip-blue",   badge: "bg-blue-500/20 text-blue-300 border-blue-500/40" },
            green:  { name: "Green",  dot: "bg-emerald-500", chip: "chip-green",  badge: "bg-emerald-500/20 text-emerald-300 border-emerald-500/40" },
            red:    { name: "Red",    dot: "bg-rose-500",    chip: "chip-red",    badge: "bg-rose-500/20 text-rose-300 border-rose-500/40" },
            yellow: { name: "Yellow", dot: "bg-amber-400",   chip: "chip-yellow", badge: "bg-amber-500/20 text-amber-300 border-amber-500/40" }
        };

        const GAME_MODES = {
            "1v1":   { label: "1v1",          maxPlayers: 2,  teams: ["blue", "green"],                 handSize: 7, defaultSeq: 2 },
            "2v2":   { label: "2v2",          maxPlayers: 4,  teams: ["blue", "green"],                 handSize: 6, defaultSeq: 2 },
            "1v1v1": { label: "1v1v1",        maxPlayers: 3,  teams: ["blue", "green", "red"],          handSize: 6, defaultSeq: 1 },
            "3v3":   { label: "3v3",          maxPlayers: 6,  teams: ["blue", "green", "red"],          handSize: 5, defaultSeq: 1 },
            "4v4_2": { label: "4v4 (2 Teams)",maxPlayers: 8,  teams: ["blue", "green"],                 handSize: 4, defaultSeq: 2 },
            "4v4_4": { label: "2v2v2v2 (4T)", maxPlayers: 8,  teams: ["blue", "green", "red", "yellow"],handSize: 4, defaultSeq: 1 },
            "6v6_2": { label: "6v6 (2 Teams)",maxPlayers: 12, teams: ["blue", "green"],                 handSize: 3, defaultSeq: 2 },
            "6v6_3": { label: "4v4v4 (3T)",   maxPlayers: 12, teams: ["blue", "green", "red"],          handSize: 3, defaultSeq: 1 }
        };

        // =========================================================================
        // 2. FIREBASE / CLOUD STATE & LOCAL FALLBACK INITIALIZATION
        // =========================================================================
        let db = null;
        let auth = null;
        let currentUser = null;
        let appId = typeof __app_id !== "undefined" ? __app_id : "sequence-official-v1";
        let cloudReady = false;

        let localPlayerId = "p_" + Math.random().toString(36).substring(2, 10);
        let localPlayerName = "";
        let activeRoomCode = null;
        let roomUnsubscribe = null;
        let currentRoomData = null;

        // UI Selection State (Private to this client)
        let selectedCardIndex = null;
        let usedDeadCardThisTurn = false;
        let lastTurnKey = "";

        async function initBackend() {
            const connDot = document.getElementById("conn-dot");
            const connText = document.getElementById("conn-text");
            try {
                if (typeof __firebase_config !== "undefined" && __firebase_config) {
                    const firebaseConfig = JSON.parse(__firebase_config);
                    const app = initializeApp(firebaseConfig);
                    auth = getAuth(app);
                    db = getFirestore(app);

                    if (typeof __initial_auth_token !== "undefined" && __initial_auth_token) {
                        await signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await signInAnonymously(auth);
                    }
                    currentUser = auth.currentUser;
                    if (currentUser && currentUser.uid) {
                        localPlayerId = currentUser.uid;
                    }
                    cloudReady = true;
                    connDot.className = "w-2 h-2 rounded-full bg-emerald-400";
                    connText.textContent = "Online Multiplayer Ready (Any Device)";
                } else {
                    throw new Error("Offline standalone mode");
                }
            } catch (err) {
                cloudReady = false;
                connDot.className = "w-2 h-2 rounded-full bg-sky-400";
                connText.textContent = "Standalone / Same-Device Mode Ready";
            }
        }

        // =========================================================================
        // 3. DECK & OFFICIAL SEQUENCE RULE HELPERS
        // =========================================================================
        function createDoubleDeck() {
            const suits = ["S", "H", "D", "C"];
            const ranks = ["2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K", "A"];
            const deck = [];
            for (let d = 0; d < 2; d++) {
                for (const s of suits) {
                    for (const r of ranks) {
                        deck.push(r + s);
                    }
                }
            }
            // Fisher-Yates Shuffle
            for (let i = deck.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [deck[i], deck[j]] = [deck[j], deck[i]];
            }
            return deck;
        }

        function isTwoEyedJack(card) {
            return card === "JC" || card === "JD";
        }

        function isOneEyedJack(card) {
            return card === "JS" || card === "JH";
        }

        function isCorner(r, c) {
            return (r === 0 || r === 9) && (c === 0 || c === 9);
        }

        function isCardDead(card, chipsGrid) {
            if (!card || card.startsWith("J")) return false;
            let openSpaces = 0;
            for (let r = 0; r < 10; r++) {
                for (let c = 0; c < 10; c++) {
                    if (BOARD_LAYOUT[r][c] === card && !chipsGrid[r][c]) {
                        openSpaces++;
                    }
                }
            }
            return openSpaces === 0;
        }

        /**
         * Official Sequence Detection Engine:
         * - Scans horizontal, vertical, and both diagonals for 5-in-a-row of `teamColor` (corners count as wild for all teams).
         * - Respects existing locked sequence cells: a new sequence can share AT MOST 1 cell with previously locked sequences.
         * - Returns updated { sequencesCount, lockedCellsMap } for that team.
         */
        function evaluateTeamSequences(teamColor, chipsGrid, existingLockedMap) {
            // Collect all valid 5-cell lines for teamColor
            const candidateLines = [];
            const dirs = [
                [0, 1],  // Horizontal
                [1, 0],  // Vertical
                [1, 1],  // Diagonal down-right
                [1, -1]  // Diagonal down-left
            ];

            for (let r = 0; r < 10; r++) {
                for (let c = 0; c < 10; c++) {
                    for (const [dr, dc] of dirs) {
                        const rEnd = r + dr * 4;
                        const cEnd = c + dc * 4;
                        if (rEnd < 0 || rEnd >= 10 || cEnd < 0 || cEnd >= 10) continue;

                        const lineCells = [];
                        let valid = true;
                        for (let k = 0; k < 5; k++) {
                            const nr = r + dr * k;
                            const nc = c + dc * k;
                            if (isCorner(nr, nc) || chipsGrid[nr][nc] === teamColor) {
                                lineCells.push(`${nr},${nc}`);
                            } else {
                                valid = false;
                                break;
                            }
                        }
                        if (valid) {
                            candidateLines.push(lineCells);
                        }
                    }
                }
            }

            // Re-verify existing locked sequences and greedily lock any newly completed valid ones
            // Official rule: 2nd sequence can share at most 1 chip with an already completed sequence.
            const lockedLines = [];
            const lockedCellSet = new Set();

            // First, preserve lines that were already locked in previous turns so locked chips stay locked
            for (const line of candidateLines) {
                const allPreviouslyLocked = line.every(key => existingLockedMap[key] === teamColor);
                if (allPreviouslyLocked) {
                    const overlap = line.filter(key => lockedCellSet.has(key)).length;
                    if (overlap <= 1) {
                        lockedLines.push(line);
                        line.forEach(k => lockedCellSet.add(k));
                    }
                }
            }

            // Second, check any newly formed lines
            for (const line of candidateLines) {
                const alreadyAdded = lockedLines.some(l => l.join("|") === line.join("|"));
                if (alreadyAdded) continue;

                const overlap = line.filter(key => lockedCellSet.has(key)).length;
                if (overlap <= 1) {
                    lockedLines.push(line);
                    line.forEach(k => lockedCellSet.add(k));
                }
            }

            const updatedLockedForTeam = {};
            lockedCellSet.forEach(key => {
                if (!key.startsWith("0,0") && !key.startsWith("0,9") && !key.startsWith("9,0") && !key.startsWith("9,9")) {
                    updatedLockedForTeam[key] = teamColor;
                }
            });

            return {
                count: lockedLines.length,
                lockedCells: updatedLockedForTeam
            };
        }

        // =========================================================================
        // 4. STORAGE ABSTRACTION (FIRESTORE + LOCAL CHANNEL FALLBACK)
        // =========================================================================
        function getRoomDocRef(code) {
            return doc(db, "artifacts", appId, "public", "data", "sequence_rooms", code);
        }

        async function writeRoomData(code, data) {
            if (cloudReady && db) {
                await setDoc(getRoomDocRef(code), data);
            } else {
                sessionStorage.setItem("seq_room_" + code, JSON.stringify(data));
                handleRoomSnapshot(data);
            }
        }

        async function patchRoomData(code, partial) {
            if (cloudReady && db) {
                await updateDoc(getRoomDocRef(code), partial);
            } else {
                const raw = sessionStorage.getItem("seq_room_" + code);
                if (!raw) return;
                const merged = { ...JSON.parse(raw), ...partial };
                sessionStorage.setItem("seq_room_" + code, JSON.stringify(merged));
                handleRoomSnapshot(merged);
            }
        }

        async function fetchRoomOnce(code) {
            if (cloudReady && db) {
                const snap = await getDoc(getRoomDocRef(code));
                return snap.exists() ? snap.data() : null;
            } else {
                const raw = sessionStorage.getItem("seq_room_" + code);
                return raw ? JSON.parse(raw) : null;
            }
        }

        function subscribeToRoom(code) {
            if (roomUnsubscribe) {
                roomUnsubscribe();
                roomUnsubscribe = null;
            }
            if (cloudReady && db) {
                roomUnsubscribe = onSnapshot(
                    getRoomDocRef(code),
                    (snap) => {
                        if (snap.exists()) {
                            handleRoomSnapshot(snap.data());
                        } else {
                            showToast("Room closed by host.", "error");
                            exitToMenu();
                        }
                    },
                    (err) => {
                        console.error("Sync error:", err);
                        showToast("Connection hiccup — retrying...", "error");
                    }
                );
            } else {
                const interval = setInterval(() => {
                    const raw = sessionStorage.getItem("seq_room_" + code);
                    if (raw) handleRoomSnapshot(JSON.parse(raw));
                }, 600);
                roomUnsubscribe = () => clearInterval(interval);
            }
        }

        // =========================================================================
        // 5. ROOM CREATION, JOINING & LOBBY ACTIONS
        // =========================================================================
        function generateRoomCode() {
            const chars = "ABCDEFGHJKLMNPQRSTUVWXYZ23456789";
            let code = "";
            for (let i = 0; i < 4; i++) {
                code += chars[Math.floor(Math.random() * chars.length)];
            }
            return code;
        }

        function getPlayerName() {
            const input = document.getElementById("player-name-input").value.trim();
            return input || "Player_" + Math.floor(100 + Math.random() * 900);
        }

        async function handleCreateRoom() {
            localPlayerName = getPlayerName();
            const code = generateRoomCode();
            const modeKey = "2v2";
            const modeCfg = GAME_MODES[modeKey];

            const initialRoom = {
                roomCode: code,
                hostId: localPlayerId,
                status: "lobby", // "lobby" | "playing" | "finished"
                mode: modeKey,
                winSequences: modeCfg.defaultSeq,
                players: [
                    {
                        id: localPlayerId,
                        name: localPlayerName,
                        team: "blue",
                        hand: []
                    }
                ],
                turnOrder: [],
                turnIndex: 0,
                deck: [],
                discard: [],
                lastDiscard: "",
                lastMoveCell: "",
                chipsJson: JSON.stringify(Array.from({ length: 10 }, () => Array(10).fill(null))),
                lockedCells: {},
                teamScores: { blue: 0, green: 0, red: 0, yellow: 0 },
                winnerTeam: ""
            };

            try {
                await writeRoomData(code, initialRoom);
                activeRoomCode = code;
                subscribeToRoom(code);
                switchView("lobby");
            } catch (e) {
                showToast("Failed to create room. Try again.", "error");
            }
        }

        async function handleJoinRoom() {
            const codeInput = document.getElementById("room-code-input").value.trim().toUpperCase();
            if (codeInput.length < 4) {
                showToast("Enter a valid 4-character room code.", "error");
                return;
            }
            localPlayerName = getPlayerName();

            try {
                const room = await fetchRoomOnce(codeInput);
                if (!room) {
                    showToast("Room not found! Check the code.", "error");
                    return;
                }

                const existingIdx = room.players.findIndex(p => p.id === localPlayerId);
                if (existingIdx === -1) {
                    if (room.status !== "lobby") {
                        showToast("Game already in progress!", "error");
                        return;
                    }
                    const modeCfg = GAME_MODES[room.mode] || GAME_MODES["2v2"];
                    if (room.players.length >= modeCfg.maxPlayers) {
                        showToast(`Room is full (${modeCfg.maxPlayers} max).`, "error");
                        return;
                    }

                    // Auto-assign to team with fewest players
                    const counts = {};
                    modeCfg.teams.forEach(t => counts[t] = 0);
                    room.players.forEach(p => {
                        if (counts[p.team] !== undefined) counts[p.team]++;
                    });
                    let bestTeam = modeCfg.teams[0];
                    for (const t of modeCfg.teams) {
                        if (counts[t] < counts[bestTeam]) bestTeam = t;
                    }

                    room.players.push({
                        id: localPlayerId,
                        name: localPlayerName,
                        team: bestTeam,
                        hand: []
                    });
                    await patchRoomData(codeInput, { players: room.players });
                }

                activeRoomCode = codeInput;
                subscribeToRoom(codeInput);
                switchView(room.status === "lobby" ? "lobby" : "game");
            } catch (e) {
                showToast("Could not join room.", "error");
            }
        }

        async function handleSelectTeam(teamColor) {
            if (!currentRoomData || currentRoomData.status !== "lobby") return;
            const updatedPlayers = currentRoomData.players.map(p =>
                p.id === localPlayerId ? { ...p, team: teamColor } : p
            );
            await patchRoomData(activeRoomCode, { players: updatedPlayers });
        }

        async function handleHostSettingChange() {
            if (!currentRoomData || currentRoomData.hostId !== localPlayerId) return;
            const newMode = document.getElementById("select-game-mode").value;
            const newWinSeq = parseInt(document.getElementById("select-win-seq").value, 10) || 2;
            const modeCfg = GAME_MODES[newMode];

            // Ensure all players are on valid teams for this mode
            const updatedPlayers = currentRoomData.players.map((p, idx) => {
                if (!modeCfg.teams.includes(p.team)) {
                    return { ...p, team: modeCfg.teams[idx % modeCfg.teams.length] };
                }
                return p;
            });

            await patchRoomData(activeRoomCode, {
                mode: newMode,
                winSequences: newWinSeq,
                players: updatedPlayers
            });
        }

        async function handleStartGame() {
            if (!currentRoomData || currentRoomData.hostId !== localPlayerId) return;
            const players = [...currentRoomData.players];
            if (players.length < 2) {
                showToast("Need at least 2 players to start!", "error");
                return;
            }

            const modeCfg = GAME_MODES[currentRoomData.mode] || GAME_MODES["2v2"];
            const activeTeams = new Set(players.map(p => p.team));
            if (activeTeams.size < 2) {
                showToast("Players must be on at least 2 different teams!", "error");
                return;
            }

            // Interleave turn order by team so teammates alternate fairly (Official Sequence seating!)
            const buckets = {};
            modeCfg.teams.forEach(t => buckets[t] = []);
            players.forEach(p => {
                if (!buckets[p.team]) buckets[p.team] = [];
                buckets[p.team].push(p.id);
            });

            const turnOrder = [];
            let added = true;
            while (added) {
                added = false;
                for (const t of modeCfg.teams) {
                    if (buckets[t] && buckets[t].length > 0) {
                        turnOrder.push(buckets[t].shift());
                        added = true;
                    }
                }
            }

            // Shuffle double deck and deal private hands
            const deck = createDoubleDeck();
            const handSize = modeCfg.handSize;
            const dealtPlayers = players.map(p => ({
                ...p,
                hand: deck.splice(0, handSize)
            }));

            const emptyGrid = Array.from({ length: 10 }, () => Array(10).fill(null));

            await patchRoomData(activeRoomCode, {
                status: "playing",
                players: dealtPlayers,
                turnOrder,
                turnIndex: 0,
                deck,
                discard: [],
                lastDiscard: "",
                lastMoveCell: "",
                chipsJson: JSON.stringify(emptyGrid),
                lockedCells: {},
                teamScores: { blue: 0, green: 0, red: 0, yellow: 0 },
                winnerTeam: ""
            });
        }

        async function handleReturnToLobby() {
            if (!currentRoomData || currentRoomData.hostId !== localPlayerId) return;
            const resetPlayers = currentRoomData.players.map(p => ({ ...p, hand: [] }));
            await patchRoomData(activeRoomCode, {
                status: "lobby",
                players: resetPlayers,
                winnerTeam: ""
            });
        }

        // =========================================================================
        // 6. GAMEPLAY MOVES (STRICT TABLETOP RULES — NO BOARD HIGHLIGHTING)
        // =========================================================================
        function isMyTurn() {
            if (!currentRoomData || currentRoomData.status !== "playing") return false;
            const activeId = currentRoomData.turnOrder[currentRoomData.turnIndex];
            return activeId === localPlayerId;
        }

        function getMyPlayerObj() {
            if (!currentRoomData) return null;
            return currentRoomData.players.find(p => p.id === localPlayerId) || null;
        }

        function drawCardFromDeck(deck, discard) {
            if (deck.length === 0 && discard.length > 0) {
                // Reshuffle discard pile back into deck
                const reshuffled = [...discard];
                for (let i = reshuffled.length - 1; i > 0; i--) {
                    const j = Math.floor(Math.random() * (i + 1));
                    [reshuffled[i], reshuffled[j]] = [reshuffled[j], reshuffled[i]];
                }
                discard.length = 0;
                deck.push(...reshuffled);
            }
            return deck.length > 0 ? deck.shift() : null;
        }

        async function handleDeadCardSwap() {
            if (!isMyTurn()) {
                showToast("Wait for your turn!", "error");
                return;
            }
            if (usedDeadCardThisTurn) {
                showToast("You already swapped a dead card this turn.", "error");
                return;
            }
            const me = getMyPlayerObj();
            if (!me || selectedCardIndex === null || !me.hand[selectedCardIndex]) return;

            const card = me.hand[selectedCardIndex];
            const chipsGrid = JSON.parse(currentRoomData.chipsJson);

            if (!isCardDead(card, chipsGrid)) {
                showToast("That card still has an open space!", "error");
                return;
            }

            const deck = [...currentRoomData.deck];
            const discard = [...currentRoomData.discard, card];
            const drawn = drawCardFromDeck(deck, discard);

            const newHand = [...me.hand];
            if (drawn) {
                newHand[selectedCardIndex] = drawn;
            } else {
                newHand.splice(selectedCardIndex, 1);
            }

            usedDeadCardThisTurn = true;
            selectedCardIndex = null;

            const updatedPlayers = currentRoomData.players.map(p =>
                p.id === localPlayerId ? { ...p, hand: newHand } : p
            );

            await patchRoomData(activeRoomCode, {
                players: updatedPlayers,
                deck,
                discard,
                lastDiscard: card
            });
            showToast(`Swapped dead card ${formatCardText(card)}!`, "info");
        }

        async function handleBoardCellTap(r, c) {
            if (!currentRoomData || currentRoomData.status !== "playing") return;

            if (!isMyTurn()) {
                showToast("Not your turn!", "error");
                return;
            }

            const me = getMyPlayerObj();
            if (!me) return;

            if (selectedCardIndex === null || !me.hand[selectedCardIndex]) {
                showToast("Tap a card in your hand first!", "info");
                return;
            }

            if (isCorner(r, c)) {
                triggerShake();
                showToast("Corners are FREE wild spaces — cannot place chips there.", "error");
                return;
            }

            const selectedCard = me.hand[selectedCardIndex];
            const boardCard = BOARD_LAYOUT[r][c];
            const chipsGrid = JSON.parse(currentRoomData.chipsJson);
            const currentChip = chipsGrid[r][c];
            const lockedCells = currentRoomData.lockedCells || {};

            // Case A: One-Eyed Jack (JS or JH) -> Remove opponent's non-locked chip
            if (isOneEyedJack(selectedCard)) {
                if (!currentChip) {
                    triggerShake();
                    showToast("One-Eyed Jack removes an opponent's chip!", "error");
                    return;
                }
                if (currentChip === me.team) {
                    triggerShake();
                    showToast("You cannot remove your own team's chip!", "error");
                    return;
                }
                if (lockedCells[`${r},${c}`]) {
                    triggerShake();
                    showToast("Locked Sequence chip! Cannot be removed.", "error");
                    return;
                }
                chipsGrid[r][c] = null;
            }
            // Case B: Two-Eyed Jack (JC or JD) -> Place chip on ANY open space
            else if (isTwoEyedJack(selectedCard)) {
                if (currentChip) {
                    triggerShake();
                    showToast("That space is already occupied!", "error");
                    return;
                }
                chipsGrid[r][c] = me.team;
            }
            // Case C: Normal Card -> Must match board space & be unoccupied
            else {
                if (boardCard !== selectedCard) {
                    triggerShake();
                    showToast(`Mismatch! You selected ${formatCardText(selectedCard)}.`, "error");
                    return;
                }
                if (currentChip) {
                    triggerShake();
                    showToast("That space already has a chip!", "error");
                    return;
                }
                chipsGrid[r][c] = me.team;
            }

            // Play succeeded: Remove card from hand, add to discard, draw 1 replacement
            const deck = [...currentRoomData.deck];
            const discard = [...currentRoomData.discard, selectedCard];
            const drawn = drawCardFromDeck(deck, discard);

            const newHand = [...me.hand];
            newHand.splice(selectedCardIndex, 1);
            if (drawn) {
                newHand.push(drawn);
            }

            selectedCardIndex = null;

            // Evaluate sequences for all teams
            const modeCfg = GAME_MODES[currentRoomData.mode] || GAME_MODES["2v2"];
            const newScores = { ...currentRoomData.teamScores };
            let mergedLockedCells = {};

            for (const t of modeCfg.teams) {
                const evalRes = evaluateTeamSequences(t, chipsGrid, lockedCells);
                newScores[t] = evalRes.count;
                mergedLockedCells = { ...mergedLockedCells, ...evalRes.lockedCells };
            }

            // Check win condition
            let nextStatus = "playing";
            let winnerTeam = "";
            if (newScores[me.team] >= currentRoomData.winSequences) {
                nextStatus = "finished";
                winnerTeam = me.team;
            }

            const nextTurnIndex = (currentRoomData.turnIndex + 1) % currentRoomData.turnOrder.length;
            const updatedPlayers = currentRoomData.players.map(p =>
                p.id === localPlayerId ? { ...p, hand: newHand } : p
            );

            await patchRoomData(activeRoomCode, {
                status: nextStatus,
                winnerTeam,
                players: updatedPlayers,
                turnIndex: nextTurnIndex,
                deck,
                discard,
                lastDiscard: selectedCard,
                lastMoveCell: `${r},${c}`,
                chipsJson: JSON.stringify(chipsGrid),
                lockedCells: mergedLockedCells,
                teamScores: newScores
            });
        }

        // =========================================================================
        // 7. UI RENDERING (BOARD, PRIVATE HAND, HUD, LOBBY)
        // =========================================================================
        function formatCardText(code) {
            if (!code || code === "FREE") return code;
            const suit = code.slice(-1);
            const rank = code.slice(0, -1);
            return rank + (SUIT_SYMBOLS[suit] || suit);
        }

        function getCardParts(code) {
            const suit = code.slice(-1);
            const rank = code.slice(0, -1);
            return {
                rank,
                suit,
                symbol: SUIT_SYMBOLS[suit] || "",
                isRed: RED_SUITS.has(suit)
            };
        }

        function buildStaticBoardGrid() {
            const boardEl = document.getElementById("sequence-board");
            boardEl.innerHTML = "";

            for (let r = 0; r < 10; r++) {
                for (let c = 0; c < 10; c++) {
                    const code = BOARD_LAYOUT[r][c];
                    const cell = document.createElement("div");
                    cell.id = `cell-${r}-${c}`;
                    cell.className = "board-cell" + (code === "FREE" ? " corner-cell" : "");

                    if (code === "FREE") {
                        cell.innerHTML = `<span class="text-[10px] sm:text-xs font-black tracking-tighter">★</span>`;
                    } else {
                        const { rank, symbol, isRed } = getCardParts(code);
                        const colorClass = isRed ? "suit-red" : "suit-black";
                        cell.innerHTML = `
                            <span class="cell-rank ${colorClass}">${rank}</span>
                            <span class="cell-suit ${colorClass}">${symbol}</span>
                        `;
                    }

                    cell.addEventListener("click", () => handleBoardCellTap(r, c));
                    boardEl.appendChild(cell);
                }
            }
            resizeBoardSquare();
        }

        function resizeBoardSquare() {
            const wrapper = document.getElementById("board-wrapper");
            const board = document.getElementById("sequence-board");
            if (!wrapper || !board) return;
            const rect = wrapper.getBoundingClientRect();
            const size = Math.floor(Math.min(rect.width - 6, rect.height - 6));
            if (size > 80) {
                board.style.width = `${size}px`;
                board.style.height = `${size}px`;
            }
        }

        function handleRoomSnapshot(room) {
            currentRoomData = room;

            // Reset turn-specific dead card flag when turn changes
            const turnKey = `${room.turnIndex}_${room.lastDiscard}_${room.deck.length}`;
            if (turnKey !== lastTurnKey) {
                lastTurnKey = turnKey;
                usedDeadCardThisTurn = false;
            }

            if (room.status === "lobby") {
                switchView("lobby");
                renderLobby(room);
            } else {
                switchView("game");
                renderGame(room);
            }
        }

        function renderLobby(room) {
            document.getElementById("lobby-room-code").textContent = room.roomCode;
            const isHost = room.hostId === localPlayerId;
            const modeCfg = GAME_MODES[room.mode] || GAME_MODES["2v2"];

            document.getElementById("lobby-role-badge").textContent = isHost ? "You are Host" : "Guest";
            const modeSelect = document.getElementById("select-game-mode");
            const seqSelect = document.getElementById("select-win-seq");
            modeSelect.value = room.mode;
            seqSelect.value = String(room.winSequences);
            modeSelect.disabled = !isHost;
            seqSelect.disabled = !isHost;

            document.getElementById("btn-start-game").classList.toggle("hidden", !isHost);
            document.getElementById("waiting-host-msg").classList.toggle("hidden", isHost);

            // Team Picker Buttons
            const me = getMyPlayerObj();
            const pickerEl = document.getElementById("team-picker-buttons");
            pickerEl.innerHTML = "";
            modeCfg.teams.forEach(tKey => {
                const meta = TEAM_META[tKey];
                const isSelected = me && me.team === tKey;
                const btn = document.createElement("button");
                btn.className = `py-2 px-2 rounded-xl border text-xs font-extrabold flex items-center justify-center gap-1.5 transition ${
                    isSelected
                        ? "bg-slate-800 border-amber-400 text-white shadow"
                        : "bg-slate-950/60 border-slate-800 text-slate-400 hover:text-white"
                }`;
                btn.innerHTML = `<span class="w-2.5 h-2.5 rounded-full ${meta.dot}"></span><span>${meta.name}</span>`;
                btn.onclick = () => handleSelectTeam(tKey);
                pickerEl.appendChild(btn);
            });

            // Players List
            document.getElementById("lobby-player-count").textContent = `${room.players.length} / ${modeCfg.maxPlayers}`;
            const listEl = document.getElementById("lobby-players-list");
            listEl.innerHTML = "";
            room.players.forEach(p => {
                const meta = TEAM_META[p.team] || TEAM_META.blue;
                const row = document.createElement("div");
                row.className = "flex items-center justify-between bg-slate-800/70 px-3 py-2 rounded-xl border border-slate-700/60";
                row.innerHTML = `
                    <div class="flex items-center gap-2">
                        <span class="w-3 h-3 rounded-full ${meta.dot}"></span>
                        <span class="text-sm font-bold text-white">${escapeHtml(p.name)}</span>
                        ${p.id === room.hostId ? '<span class="text-[10px] px-1.5 py-0.5 rounded bg-amber-500/20 text-amber-300 font-bold">HOST</span>' : ""}
                        ${p.id === localPlayerId ? '<span class="text-[10px] text-slate-400">(You)</span>' : ""}
                    </div>
                    <span class="text-xs font-extrabold px-2 py-0.5 rounded border ${meta.badge}">${meta.name}</span>
                `;
                listEl.appendChild(row);
            });
        }

        function renderGame(room) {
            const modeCfg = GAME_MODES[room.mode] || GAME_MODES["2v2"];
            const chipsGrid = JSON.parse(room.chipsJson);
            const lockedCells = room.lockedCells || {};
            const me = getMyPlayerObj();

            // 1. Top Bar: Room Code & Team Sequence Scores
            document.getElementById("hud-room-code").textContent = room.roomCode;
            const scoresEl = document.getElementById("hud-team-scores");
            scoresEl.innerHTML = "";
            modeCfg.teams.forEach(tKey => {
                const meta = TEAM_META[tKey];
                const score = room.teamScores[tKey] || 0;
                const pill = document.createElement("div");
                pill.className = `flex items-center gap-1 px-2 py-0.5 rounded-md border text-[11px] sm:text-xs font-extrabold whitespace-nowrap ${meta.badge}`;
                pill.innerHTML = `<span class="w-2 h-2 rounded-full ${meta.dot}"></span><span>${score}/${room.winSequences}</span>`;
                scoresEl.appendChild(pill);
            });

            // Turn Indicator
            const activePlayerId = room.turnOrder[room.turnIndex];
            const activePlayer = room.players.find(p => p.id === activePlayerId);
            const turnPill = document.getElementById("hud-turn-pill");
            if (activePlayerId === localPlayerId) {
                turnPill.textContent = "YOUR TURN";
                turnPill.className = "px-2.5 py-1 rounded-full text-[11px] sm:text-xs font-black bg-amber-400 text-slate-950 shadow animate-pulse";
            } else {
                turnPill.textContent = activePlayer ? `${activePlayer.name}'s Turn` : "Playing";
                turnPill.className = "px-2.5 py-1 rounded-full text-[11px] sm:text-xs font-bold bg-slate-800 text-slate-300 border border-slate-700 truncate max-w-[130px]";
            }

            // 2. Opponents Strip (ONLY card counts shown — strictly NO opponent cards!)
            const stripEl = document.getElementById("hud-players-strip");
            stripEl.innerHTML = "";
            room.turnOrder.forEach(pid => {
                const p = room.players.find(pl => pl.id === pid);
                if (!p) return;
                const meta = TEAM_META[p.team] || TEAM_META.blue;
                const isTurn = pid === activePlayerId;
                const chipTag = document.createElement("div");
                chipTag.className = `flex items-center gap-1 px-2 py-0.5 rounded whitespace-nowrap ${
                    isTurn ? "bg-slate-800 text-white font-bold ring-1 ring-amber-400/70" : "text-slate-400"
                }`;
                chipTag.innerHTML = `
                    <span class="w-2 h-2 rounded-full ${meta.dot}"></span>
                    <span>${escapeHtml(p.name)}</span>
                    <span class="text-[10px] text-slate-500">(${p.hand.length}🂠)</span>
                `;
                stripEl.appendChild(chipTag);
            });

            // 3. Render Chips on Board (WITHOUT ANY CARD HIGHLIGHTING!)
            for (let r = 0; r < 10; r++) {
                for (let c = 0; c < 10; c++) {
                    const cellEl = document.getElementById(`cell-${r}-${c}`);
                    if (!cellEl) continue;

                    // Remove previous dynamic chip / dot
                    const oldChip = cellEl.querySelector(".chip");
                    if (oldChip) oldChip.remove();
                    const oldDot = cellEl.querySelector(".last-move-dot");
                    if (oldDot) oldDot.remove();

                    const teamColor = chipsGrid[r][c];
                    if (teamColor && TEAM_META[teamColor]) {
                        const chipDiv = document.createElement("div");
                        const isLocked = Boolean(lockedCells[`${r},${c}`]);
                        chipDiv.className = `chip ${TEAM_META[teamColor].chip}${isLocked ? " locked-seq" : ""}`;
                        cellEl.appendChild(chipDiv);
                    }

                    if (room.lastMoveCell === `${r},${c}`) {
                        const dot = document.createElement("span");
                        dot.className = "last-move-dot";
                        cellEl.appendChild(dot);
                    }
                }
            }

            // 4. Bottom Tray: My Team, Deck Count, Last Discard, & Private Hand
            if (me) {
                const myMeta = TEAM_META[me.team] || TEAM_META.blue;
                document.getElementById("my-team-dot").className = `w-2.5 h-2.5 rounded-full ${myMeta.dot}`;
                document.getElementById("my-team-label").textContent = `${myMeta.name} Team`;
            }
            document.getElementById("deck-count-label").textContent = `Deck: ${room.deck.length}`;
            document.getElementById("last-discard-card").textContent = room.lastDiscard ? formatCardText(room.lastDiscard) : "--";

            renderPrivateHand(me, chipsGrid);

            // 5. Victory Modal Check
            const victoryModal = document.getElementById("modal-victory");
            if (room.status === "finished" && room.winnerTeam) {
                const wMeta = TEAM_META[room.winnerTeam] || TEAM_META.blue;
                document.getElementById("victory-title").textContent = `${wMeta.name} Team Wins!`;
                document.getElementById("victory-subtitle").textContent = `Completed ${room.winSequences} Sequence${room.winSequences > 1 ? "s" : ""}`;
                document.getElementById("btn-play-again").classList.toggle("hidden", room.hostId !== localPlayerId);
                victoryModal.classList.remove("hidden");
            } else {
                victoryModal.classList.add("hidden");
            }
        }

        function renderPrivateHand(me, chipsGrid) {
            const handContainer = document.getElementById("my-hand-container");
            const deadBtn = document.getElementById("btn-dead-card");
            handContainer.innerHTML = "";
            deadBtn.classList.add("hidden");

            if (!me || !me.hand) return;

            if (selectedCardIndex !== null && selectedCardIndex >= me.hand.length) {
                selectedCardIndex = null;
            }

            me.hand.forEach((cardCode, idx) => {
                const { rank, symbol, isRed } = getCardParts(cardCode);
                const dead = isCardDead(cardCode, chipsGrid);
                const isSelected = selectedCardIndex === idx;

                if (isSelected && dead && isMyTurn() && !usedDeadCardThisTurn) {
                    deadBtn.classList.remove("hidden");
                }

                let jackTag = "";
                if (isTwoEyedJack(cardCode)) {
                    jackTag = `<span class="text-[8px] sm:text-[9px] font-black uppercase px-1 rounded bg-emerald-500 text-white self-center">WILD</span>`;
                } else if (isOneEyedJack(cardCode)) {
                    jackTag = `<span class="text-[8px] sm:text-[9px] font-black uppercase px-1 rounded bg-rose-600 text-white self-center">REMOVE</span>`;
                } else if (dead) {
                    jackTag = `<span class="text-[8px] sm:text-[9px] font-black uppercase px-1 rounded bg-slate-700 text-rose-300 self-center">DEAD</span>`;
                }

                const cardEl = document.createElement("div");
                cardEl.className = `hand-card${isSelected ? " selected" : ""}${dead ? " dead-card" : ""}`;
                const colorClass = isRed ? "text-red-600" : "text-slate-900";

                cardEl.innerHTML = `
                    <div class="flex items-center justify-between leading-none ${colorClass}">
                        <span class="font-black text-sm sm:text-base">${rank}</span>
                        <span class="text-xs sm:text-sm">${symbol}</span>
                    </div>
                    ${jackTag || `<div class="text-center text-lg sm:text-2xl leading-none ${colorClass}">${symbol}</div>`}
                    <div class="flex items-center justify-between leading-none rotate-180 ${colorClass}">
                        <span class="font-black text-sm sm:text-base">${rank}</span>
                        <span class="text-xs sm:text-sm">${symbol}</span>
                    </div>
                `;

                cardEl.onclick = () => {
                    // Notice: Selecting a card NEVER highlights spaces on the board!
                    selectedCardIndex = (selectedCardIndex === idx) ? null : idx;
                    renderPrivateHand(me, chipsGrid);
                };

                handContainer.appendChild(cardEl);
            });
        }

        // =========================================================================
        // 8. UTILITY FUNCTIONS, MODALS & OFFLINE FILE EXPORT
        // =========================================================================
        function switchView(viewName) {
            document.getElementById("view-menu").classList.toggle("hidden", viewName !== "menu");
            document.getElementById("view-lobby").classList.toggle("hidden", viewName !== "lobby");
            document.getElementById("view-game").classList.toggle("hidden", viewName !== "game");
            if (viewName === "game") {
                setTimeout(resizeBoardSquare, 20);
            }
        }

        function exitToMenu() {
            if (roomUnsubscribe) {
                roomUnsubscribe();
                roomUnsubscribe = null;
            }
            activeRoomCode = null;
            currentRoomData = null;
            selectedCardIndex = null;
            document.getElementById("modal-victory").classList.add("hidden");
            document.getElementById("modal-game-menu").classList.add("hidden");
            switchView("menu");
        }

        function showToast(msg, type = "info") {
            const container = document.getElementById("toast-container");
            const el = document.createElement("div");
            const bg = type === "error"
                ? "bg-rose-600/95 border-rose-400 text-white"
                : "bg-slate-800/95 border-amber-400/60 text-amber-200";
            el.className = `px-3.5 py-2 rounded-xl border text-xs font-bold shadow-xl text-center transition-opacity duration-200 ${bg}`;
            el.textContent = msg;
            container.appendChild(el);
            setTimeout(() => {
                el.style.opacity = "0";
                setTimeout(() => el.remove(), 220);
            }, 2200);
        }

        function triggerShake() {
            const board = document.getElementById("sequence-board");
            board.classList.remove("shake");
            void board.offsetWidth;
            board.classList.add("shake");
        }

        function copyTextFallback(text) {
            const ta = document.createElement("textarea");
            ta.value = text;
            ta.style.position = "fixed";
            ta.style.opacity = "0";
            document.body.appendChild(ta);
            ta.select();
            try {
                document.execCommand("copy");
                showToast(`Copied Room Code: ${text}`, "info");
            } catch (e) {
                showToast(`Room Code: ${text}`, "info");
            }
            document.body.removeChild(ta);
        }

        function downloadSelfContainedHtml() {
            const htmlContent = "<!DOCTYPE html>\n" + document.documentElement.outerHTML;
            const blob = new Blob([htmlContent], { type: "text/html" });
            const url = URL.createObjectURL(blob);
            const a = document.createElement("a");
            a.href = url;
            a.download = "Sequence-Multiplayer-Game.html";
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);
            showToast("Saved Sequence-Multiplayer-Game.html!", "info");
        }

        function escapeHtml(str) {
            return String(str)
                .replace(/&/g, "&amp;")
                .replace(/</g, "&lt;")
                .replace(/>/g, "&gt;")
                .replace(/"/g, "&quot;");
        }

        // =========================================================================
        // 9. EVENT LISTENERS & BOOTSTRAP
        // =========================================================================
        window.addEventListener("resize", resizeBoardSquare);
        window.addEventListener("orientationchange", () => setTimeout(resizeBoardSquare, 100));

        document.getElementById("btn-create-room").addEventListener("click", handleCreateRoom);
        document.getElementById("btn-join-room").addEventListener("click", handleJoinRoom);
        document.getElementById("btn-leave-lobby").addEventListener("click", exitToMenu);
        document.getElementById("btn-copy-code").addEventListener("click", () => {
            if (activeRoomCode) copyTextFallback(activeRoomCode);
        });

        document.getElementById("select-game-mode").addEventListener("change", handleHostSettingChange);
        document.getElementById("select-win-seq").addEventListener("change", handleHostSettingChange);
        document.getElementById("btn-start-game").addEventListener("click", handleStartGame);
        document.getElementById("btn-dead-card").addEventListener("click", handleDeadCardSwap);

        // Victory modal buttons
        document.getElementById("btn-play-again").addEventListener("click", handleReturnToLobby);
        document.getElementById("btn-victory-leave").addEventListener("click", exitToMenu);

        // In-game menu & rules modals
        document.getElementById("btn-game-menu").addEventListener("click", () => {
            document.getElementById("modal-game-menu").classList.remove("hidden");
        });
        document.getElementById("btn-close-game-menu").addEventListener("click", () => {
            document.getElementById("modal-game-menu").classList.add("hidden");
        });
        document.getElementById("btn-menu-rules").addEventListener("click", () => {
            document.getElementById("modal-game-menu").classList.add("hidden");
            document.getElementById("modal-rules").classList.remove("hidden");
        });
        document.getElementById("btn-menu-copy-code").addEventListener("click", () => {
            document.getElementById("modal-game-menu").classList.add("hidden");
            if (activeRoomCode) copyTextFallback(activeRoomCode);
        });
        document.getElementById("btn-menu-download").addEventListener("click", () => {
            document.getElementById("modal-game-menu").classList.add("hidden");
            downloadSelfContainedHtml();
        });
        document.getElementById("btn-menu-leave-game").addEventListener("click", exitToMenu);

        document.getElementById("btn-open-rules-menu").addEventListener("click", () => {
            document.getElementById("modal-rules").classList.remove("hidden");
        });
        document.getElementById("btn-close-rules").addEventListener("click", () => {
            document.getElementById("modal-rules").classList.add("hidden");
        });
        document.getElementById("btn-rules-got-it").addEventListener("click", () => {
            document.getElementById("modal-rules").classList.add("hidden");
        });
        document.getElementById("btn-download-html").addEventListener("click", downloadSelfContainedHtml);

        // Initialize board and backend on load
        window.onload = () => {
            buildStaticBoardGrid();
            initBackend();
        };
    </script>
</body>
</html>
