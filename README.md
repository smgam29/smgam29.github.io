!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AIPU Meltdown</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    
    <!-- Tone.js for Chiptune Music (V6.4 Stealth Skip) --><script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.min.js"></script>
    <style>
        /* Retro Arcade Feel CSS */
        body {
            background-color: #1a1a2e; /* Deep dark purple/blue background */
            font-family: 'Press Start 2P', monospace; /* Retro arcade font */
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            overflow: hidden;
        }

        .game-container {
            border: 4px solid #4a00e0; /* Deep purple border */
            box-shadow: 
                0 0 15px #4a00e0, 
                0 0 30px #4a00e0 inset; /* Subtle outer glow */
            border-radius: 8px;
            padding: 1rem;
            background-color: #110e20; /* Even darker inner background */
            display: flex;
            flex-direction: column;
            align-items: center;
            width: 90%;
            max-width: 450px;
        }

        #gameCanvas {
            display: block;
            background-color: #0d0a1b; /* Inner dark space */
            border: 2px solid #00ffaa;
            /* Neon Glow Effect */
            filter: drop-shadow(0 0 5px #ff00ff);
        }

        /* The canvas container is used for the overall jitter effect */
        #canvasContainer {
            position: relative;
            transform-origin: center center;
            transition: transform 0.05s linear; /* Smooth visual shake */
        }


        .header-panel {
            color: #ffffff;
            text-align: center;
            margin-bottom: 1rem;
            font-size: 0.75rem;
            text-shadow: 0 0 5px #ffffff;
            width: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 0 1rem;
        }
        
        .info-panel {
            color: #00ffaa; /* Neon Green for instructions (will use L1 color for this static text) */
            text-align: center;
            margin-top: 1rem;
            font-size: 0.6rem;
            text-shadow: 0 0 5px #00ffaa;
        }

        h1 {
            font-size: 1.25rem;
            color: #ff00ff;
            text-shadow: 0 0 10px #ff00ff;
            margin: 0;
        }

        /* BUTTON STYLES */
        .execution-button {
            margin-top: 1.5rem;
            padding: 1rem 2rem;
            background-color: #ff00ff; /* Neon Pink */
            color: #1a1a2e; /* Dark text */
            border: 3px solid #ff00ff;
            border-radius: 4px;
            font-family: 'Press Start 2P', monospace;
            font-size: 1rem;
            cursor: pointer;
            text-shadow: none;
            box-shadow: 0 0 10px #ff00ff, 0 0 20px #ff00ff inset;
            transition: all 0.05s;
        }
        
        .execution-button:hover {
            box-shadow: 0 0 15px #ff00ff, 0 0 30px #ff00ff inset;
        }

        .execution-button:active {
            background-color: #ffffff;
            color: #1a1a2e;
            box-shadow: 0 0 5px #00ffff, 0 0 15px #00ffff inset;
            transform: scale(0.98);
        }

        /* Disabled state for lockout */
        .execution-button:disabled {
            background-color: #333333;
            color: #999999;
            border-color: #666666;
            box-shadow: none;
            cursor: not-allowed;
        }

        /* Flashing effect during meltdown lockout */
        @keyframes flash-red {
            0%, 100% {
                box-shadow: 0 0 10px #ff00ff, 0 0 20px #ff00ff inset;
            }
            50% {
                box-shadow: 0 0 25px #ff6600, 0 0 50px #ff6600 inset; /* Orange/Red pulse */
            }
        }

        .button-flash {
            animation: flash-red 0.5s infinite alternate;
        }
    </style>
</head>
<body>

    <div class="game-container">
        
        <div class="header-panel">
            <!-- Wrap title and score/level in a column flexbox to stack them -->
            <div class="flex flex-col items-center">
                <h1>AIPU MELTDOWN</h1>
                <div class="flex space-x-4 mt-1">
                    <!-- Meltdown Counter -->
                    <div id="meltdownCounter" class="text-xs" style="color:#00ffaa; text-shadow: 0 0 5px #00ffaa;">
                        SUCCESSFUL MELTDOWNS: 0
                    </div>
                    <!-- Level Counter -->
                    <div id="levelDisplay" class="text-xs" style="color:#00ffff; text-shadow: 0 0 5px #00ffff;">
                        LEVEL: 1
                    </div>
                </div>
            </div>
        </div>
        
        <!-- Game Canvas Container (Used for Jitter Effect) -->
        <div id="canvasContainer">
             <canvas id="gameCanvas" width="400" height="400"></canvas>
        </div>

        <!-- Execution Button (Active text is PROMPT ME) --><button id="executionButton" class="execution-button">START GAME</button>
        
        <!-- Reset Button (Visible after game finished) -->
        <button id="resetGameButton" 
                onclick="initializeGame(true)"
                class="execution-button hidden mt-4" 
                style="background-color: #00ffff; color: #1a1a2e; border-color: #00ffff; box-shadow: 0 0 10px #00ffff, 0 0 20px #00ffff inset;">
            RESTART SYSTEM
        </button>


        <!-- Lockout Message Display -->
        <div id="lockoutMessage" class="text-xs mt-2 mb-2 text-center" style="color:#ff6600; text-shadow: 0 0 5px #ff6600; min-height: 12px; font-size: 0.8rem;">
            <!-- Message content will be set by JS -->
        </div>

        <!-- Instructions (Level skip instruction removed here) -->
        <div class="info-panel">
            <p>Mash the **PROMPT ME BUTTON** to run Computational Tasks!</p>
            <p>Overheat the GPU to **100%** before it cools!</p>
        </div>
    </div>

    <script type="module">
        // Firebase Imports (Mandatory Setup)
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, setDoc, doc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- Firebase Initialization ---
        let app;
        let db;
        let auth;
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        if (firebaseConfig) {
            try {
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                setLogLevel('Debug');
                console.log("Firebase initialized successfully.");

                // Authentication
                if (initialAuthToken) {
                    signInWithCustomToken(auth, initialAuthToken).then(userCredential => {
                        console.log("Signed in with custom token. User ID:", userCredential.user.uid);
                    }).catch(error => {
                        console.error("Error signing in with custom token:", error);
                        signInAnonymously(auth).then(() => console.log("Signed in anonymously after token error.")).catch(e => console.error("Anonymous sign-in failed:", e));
                    });
                } else {
                    signInAnonymously(auth).then(() => console.log("Signed in anonymously.")).catch(e => console.error("Anonymous sign-in failed:", e));
                }

            } catch (error) {
                console.error("Error initializing Firebase:", error);
            }
        }
        
        // --- TONE.JS MUSIC SETUP (V6.4 Update) ---
        let mainSynth = null;
        let crashSynth = null;
        let bassSynth = null;      
        let counterSynth = null;   
        let drumSynth = null;      
        let hihatSynth = null; 
        
        let mainSequence = null;
        let drumSequence = null;   
        let bassSequence = null;   
        let counterSequence = null;
        let hihatSequence = null; 
        
        const BASE_BPM = 120;
        
        // Extended 16-step melody pattern (C-Minor feel, 8n duration)
        const NOTE_PATTERN = [
            "C5", "G4", "A4", "F4", "D5", "A4", "G4", "C4",
            "E5", "B4", "C5", "G4", "A4", "D5", "E5", "F5"
        ];
        // Simple bass progression (8 steps, played as quarter notes, covering 2 bars of 8th notes)
        const BASS_PATTERN = ["C3", "-", "G2", "-", "A2", "-", "F2", "-"];
        // Sparce counter-melody (Higher octave, 16 steps, played as half notes, covering 4 bars of 8th notes)
        const COUNTER_PATTERN = ["-", "-", "-", "G5", "-", "-", "-", "A5", "-", "-", "-", "B5", "-", "-", "-", "C6"];
        // Drum pattern: Kick (C2) on 1 & 3, Snare (G3) on 2 & 4 (quarter notes)
        const DRUM_PATTERN = [
            { note: "C2", velocity: 1.0 }, { note: "G3", velocity: 0.8 }, 
            { note: "C2", velocity: 1.0 }, { note: "G3", velocity: 0.8 }
        ];

        // Syncopated Hi-Hat Pattern (16 steps of 8th notes covering 2 bars)
        const HIHAT_PATTERN = [
            "-", "C4", "C4", "-", 
            "-", "C4", "-", "C4", 
            "C4", "-", "C4", "-", 
            "-", "C4", "-", "C4"
        ];
        
        // Triumphant Coda Sequence: designed to sound final and sustained.
        const CODA_NOTES = [ 
            { note: "C6", duration: "8n", velocity: 1.0 },
            { note: "A5", duration: "8n", velocity: 1.0 },
            { note: "F5", duration: "8n", velocity: 1.0 },
            { note: "C5", duration: "4n", velocity: 0.8 },
            { note: "G5", duration: "1n", velocity: 0.8 } 
        ];


        /** Initializes all Tone.js synths and sequences. */
        function initMusic() {
            // Main Chiptune Synth (Square wave for 8-bit sound)
            mainSynth = new Tone.MonoSynth({
                oscillator: { type: "square" },
                envelope: { attack: 0.005, decay: 0.1, sustain: 0.05, release: 0.8 }, 
                filter: { frequency: 1000, Q: 1 }
            }).toDestination();
            
            // Bass Synth (Sawtooth for fuller bass)
            bassSynth = new Tone.MonoSynth({
                oscillator: { type: "sawtooth" },
                envelope: { attack: 0.01, decay: 0.2, sustain: 0.5, release: 0.2 },
                filter: { frequency: 500, Q: 2 }
            }).toDestination();
            bassSynth.volume.value = -10; 
            
            // Counter Synth (Triangle for softer, contrasting lead)
            counterSynth = new Tone.MonoSynth({
                oscillator: { type: "triangle" },
                envelope: { attack: 0.01, decay: 0.1, sustain: 0.5, release: 0.1 },
                filter: { frequency: 1500, Q: 1 }
            }).toDestination();
            counterSynth.volume.value = -8; 
            
            // Drum Synth (Membrane for Kick/Snare)
            drumSynth = new Tone.MembraneSynth({
                pitchDecay: 0.05,
                octaves: 10,
                envelope: { attack: 0.001, decay: 0.3, sustain: 0.01 }
            }).toDestination();
            drumSynth.volume.value = -5;
            
            // Crash/Meltdown Sound Synth (White noise with rapid decay)
            crashSynth = new Tone.NoiseSynth({
                noise: { type: "white" },
                envelope: { attack: 0.001, decay: 0.2, sustain: 0, release: 0.01, releaseCurve: "exponential" }
            }).toDestination();

            // Hi-Hat Synth (Metal Synth for sharp, metallic sound)
            hihatSynth = new Tone.MetalSynth({
                frequency: 200,
                envelope: { attack: 0.001, decay: 0.1, release: 0.05 },
                harmonicity: 5.1,
                modulationIndex: 32,
                resonance: 4000,
                octaves: 1.5,
            }).toDestination();
            // Increased volume from -12 to -6 for audibility (Hi-Hat Fix)
            hihatSynth.volume.value = -6; 

            // Setup Sequences
            Tone.Transport.bpm.value = BASE_BPM; 
            
            // Main Melody (8th notes, playing 16 steps total)
            mainSequence = new Tone.Sequence((time, note) => {
                if (note !== "-") mainSynth.triggerAttackRelease(note, "8n", time);
            }, NOTE_PATTERN, "8n");
            mainSequence.start(0);
            mainSequence.mute = true;
            
            // Drums (Quarter notes, 4 steps)
            drumSequence = new Tone.Sequence((time, event) => {
                if (event.note !== "-") drumSynth.triggerAttackRelease(event.note, event.note === "C2" ? "8n" : "16n", time, event.velocity); 
            }, DRUM_PATTERN, "4n");
            drumSequence.start(0);
            drumSequence.mute = true;
            
            // Bassline (Quarter notes, 8 steps)
            bassSequence = new Tone.Sequence((time, note) => {
                if (note !== "-") bassSynth.triggerAttackRelease(note, "4n", time);
            }, BASS_PATTERN, "4n");
            bassSequence.start(0);
            bassSequence.mute = true;
            
            // Counter-Melody (8th notes, 16 steps total, 4 bars)
            counterSequence = new Tone.Sequence((time, note) => {
                if (note !== "-") counterSynth.triggerAttackRelease(note, "8n", time);
            }, COUNTER_PATTERN, "8n");
            counterSequence.start(0);
            counterSequence.mute = true;

            // Hi-Hat Sequence (8th notes, 16 steps total, syncopated)
            hihatSequence = new Tone.Sequence((time, note) => {
                // Correctly pass the pitch (note) and duration ("16n")
                if (note !== "-") hihatSynth.triggerAttackRelease(note, "16n", time, 0.7); 
            }, HIHAT_PATTERN, "8n");
            hihatSequence.start(0);
            hihatSequence.mute = true; 
        }
        
        /** Plays the chiptune crash sound effect. */
        function playCrashSound() {
            // Plays a sharp noise burst
            crashSynth.triggerAttackRelease("8n", Tone.now()); 
        }

        /** Updates the BPM and layer muting based on the level. */
        function updateMusicStatus(level) {
            // 1. Update BPM (increases by 10 BPM per level)
            const newBPM = BASE_BPM + (level - 1) * 10;
            Tone.Transport.bpm.rampTo(newBPM, "0.5"); 
            
            // 2. Update Layers (L1: Main, L3: Drums, L5: Bass, L8: Counter, L9: HiHat)
            if (mainSequence) mainSequence.mute = false; 

            if (drumSequence) drumSequence.mute = level < 3;
            if (bassSequence) bassSequence.mute = level < 5;
            if (counterSequence) counterSequence.mute = level < 8;
            if (hihatSequence) hihatSequence.mute = level < 9; // UNMUTE AT LEVEL 9!
        }

        /** Plays the coda sequence for the Level 10 victory. (V6.0 Hard Fix) */
        function playCodaSequence() {
            if (!mainSynth) return;

            // 1. HARD STOP AND DISPOSE ALL LOOPING SEQUENCES
            if (mainSequence) mainSequence.stop();
            if (drumSequence) drumSequence.stop();
            if (bassSequence) bassSequence.stop();
            if (counterSequence) counterSequence.stop();
            if (hihatSequence) hihatSequence.stop(); 

            Tone.Transport.cancel(Tone.now()); 
            Tone.Transport.start();

            // 2. MANUALLY TRIGGER CODA NOTES
            let time = Tone.now();
            let totalDuration = 0;

            CODA_NOTES.forEach(item => {
                const durationSeconds = Tone.Time(item.duration).toSeconds();
                
                mainSynth.triggerAttackRelease(
                    item.note, 
                    item.duration, 
                    time + totalDuration, 
                    item.velocity
                );
                
                totalDuration += durationSeconds;
            });
            
            // 3. SCHEDULE FINAL CLEANUP (Ramp down and stop transport)
            Tone.Transport.scheduleOnce(() => {
                mainSynth.volume.rampTo(-40, 0.8); 
                
                Tone.Transport.scheduleOnce(() => {
                    Tone.Transport.stop();
                }, "+0.8");
                
            }, time + totalDuration - Tone.Time("4n").toSeconds()); 

            console.log("Meltdown Coda Sequence Triggered Manually.");
        }


        /** Starts the Tone.Transport and unmutes the main sequence (called after Tone.start()). */
        function startGameMusic() {
            if (Tone.context.state === 'running') {
                Tone.Transport.start();
                updateMusicStatus(currentLevel); 
            }
        }

        /** Stops the music transport and sequences (used for full shutdown). */
        function stopMusic() {
            if (typeof Tone === 'undefined' || !Tone.Transport) return;
            // A safer stop for non-coda shutdowns
            if (mainSequence) mainSequence.mute = true;
            if (drumSequence) drumSequence.mute = true;
            if (bassSequence) bassSequence.mute = true;
            if (counterSequence) counterSequence.mute = true;
            if (hihatSequence) hihatSequence.mute = true; 
            
            Tone.Transport.stop();
        }


        // --- GAME CONSTANTS ---
        const CANVAS_WIDTH = 400;
        const CANVAS_HEIGHT = 400;
        const GAME_LOGIC_TICK = 50; // Milliseconds for heat decay logic
        const OVERLAY_DURATION = 2000; // 2 seconds for level display overlay
        const FINAL_FLOURISH_DURATION = 3000; // 3 seconds delay for L10 meltdown animation before static screen
        
        // --- TIMING MECHANICS ---
        const BASE_HEAT_LOCKOUT_MS = 5000; // Total time button is disabled (5s, constant)
        const MIN_DRAIN_DURATION_MS = 1000; // Min time for heat bar movement (L1)
        const MAX_DRAIN_DURATION_MS = 5000; // Max time for heat bar movement (L10)
        
        // Game Logic 
        const HEAT_PER_PRESS = 3; // % increase per click 
        const JITTER_START_TEMP = 75; // NEW: Start jittering at 75%

        // --- UPDATED DIFFICULTY SETTINGS ---
        const BASE_DECAY_RATE = 0.05; 
        const DIFFICULTY_MULTIPLIER = 1.30; 
        let currentBaseDecayRate = BASE_DECAY_RATE; 
        
        const MAX_LEVEL = 10; 
        const SCORE_TEXT_PREFIX = 'SUCCESSFUL MELTDOWNS: '; 
        const AIPU_VERSION = 'V6.4'; // Version tracking (Updated)

        // Neon Colors (TIER-BASED)
        const COLOR_OUTLINE_T1 = '#00ffaa';    // Neon Green (L1-3 outline/fan)
        const COLOR_OUTLINE_T2 = '#ffff00';    // Electric Yellow (L4-9 outline/fan)
        const COLOR_TEXT_T1 = '#ff00ff';       // Neon Pink (L1-3 text/chip)
        const COLOR_TEXT_T2 = '#ff6600';       // Deep Orange (L4-9 text/chip/sparks)
        const COLOR_TEMP_COLD = '#00ffff';     // Cyan (Low Temp)
        const COLOR_TEMP_HOT = '#ff0000';      // Red (High Temp)

        // --- GAME STATE ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const executionButton = document.getElementById('executionButton'); 
        const meltdownCounter = document.getElementById('meltdownCounter'); 
        const levelDisplay = document.getElementById('levelDisplay'); 
        const lockoutMessage = document.getElementById('lockoutMessage'); 
        const resetGameButton = document.getElementById('resetGameButton'); 
        const canvasContainer = document.getElementById('canvasContainer'); 

        let gameLoopInterval;
        let gameRunning = false;
        let fanRotation = 0;
        let meltdownCount = 0; 
        let currentLevel = 1; 
        let levelStartTime = 0; 
        let gpu = {
            temperature: 0, 
            maxTemp: 100,
            fanSpeed: 5 
        };
        let lastPressTime = 0;
        let pressDelay = 75; 
        let animationFrameId;
        let lockoutActive = false; 
        let meltdownActive = false; 
        let isMaxLevelMeltdown = false; 
        let ambientSparks = []; 
        let gameFinished = false; 
        let showFinalGameOverOverlay = false; 
        
        // DYNAMIC STATE
        let visualTemperature = 0; 
        let drainStartTime = 0; 
        let currentDrainDurationMS = MIN_DRAIN_DURATION_MS; 
        
        // --- UTILITY FUNCTIONS ---
        
        /** Calculates the Heat Bar Drain Time using linear interpolation. */
        function getDrainDurationMS(level) {
            if (level <= 1) return MIN_DRAIN_DURATION_MS;
            if (level >= MAX_LEVEL) return MAX_DRAIN_DURATION_MS;
            
            const range = MAX_DRAIN_DURATION_MS - MIN_DRAIN_DURATION_MS;
            const multiplier = (level - 1) / (MAX_LEVEL - 1);
            
            return MIN_DRAIN_DURATION_MS + (multiplier * range);
        }

        /** Calculates the decay rate for the current level. */
        function getCurrentDecayRate() {
            const decayRate = BASE_DECAY_RATE * Math.pow(DIFFICULTY_MULTIPLIER, currentLevel - 1);
            return decayRate;
        }

        /** Updates the score and level display elements. */
        function updateDisplays() {
            meltdownCounter.textContent = SCORE_TEXT_PREFIX + meltdownCount;
            if (gameFinished) {
                levelDisplay.textContent = 'LEVEL: MAXED OUT';
                levelDisplay.style.color = '#ff0000'; 
                levelDisplay.style.textShadow = '0 0 5px #ff0000';
            } else {
                levelDisplay.textContent = 'LEVEL: ' + currentLevel;
                levelDisplay.style.color = '#00ffff'; 
                levelDisplay.style.textShadow = '0 0 5px #00ffff';
            }
        }

        /** Determines the main outline color based on the current level. */
        function getLevelColor() {
            if (currentLevel >= 4 && !gameFinished) {
                return COLOR_OUTLINE_T2; 
            }
            if (gameFinished) {
                return COLOR_TEMP_HOT; 
            }
            return COLOR_OUTLINE_T1; 
        }
        
        /** Determines the main accent text color based on the current level. */
        function getTextColor() {
            if (showFinalGameOverOverlay) {
                return '#00ffff'; 
            }
            if (currentLevel >= 4) {
                return COLOR_TEXT_T2; 
            }
            return COLOR_TEXT_T1; 
        }

        // --- DRAWING FUNCTIONS ---

        /** Draws a static heat sink pattern (L4-9) */
        function drawHeatSink(x, y, width, height) {
            if (gameFinished || currentLevel < 4 || currentLevel >= MAX_LEVEL) return;
            
            const currentOutlineColor = getLevelColor();
            ctx.strokeStyle = currentOutlineColor;
            ctx.lineWidth = 1;
            ctx.globalAlpha = 0.4;
            
            for (let i = 1; i < 10; i++) {
                const px = x + (i * width / 10);
                ctx.beginPath();
                ctx.moveTo(px, y);
                ctx.lineTo(px, y + height);
                ctx.stroke();
            }
            
            for (let j = 1; j < 6; j++) {
                const py = y + (j * height / 6);
                ctx.beginPath();
                ctx.moveTo(x, py);
                ctx.lineTo(x + width, py);
                ctx.stroke();
            }
            
            ctx.globalAlpha = 1.0;
        }

        /** Draws the main GPU card outline. */
        function drawGPUBox() {
            const x = 50;
            const y = 50;
            const width = 300;
            const height = 250;
            
            const currentOutlineColor = getLevelColor();
            const currentTextColor = getTextColor();
            
            let flickerAlpha = 1.0;
            if (currentLevel >= 7 && currentLevel < MAX_LEVEL && !gameFinished) {
                 flickerAlpha = Math.random() < 0.1 ? 0.3 : 1.0; 
            }
            
            if (gameFinished) {
                ctx.globalAlpha = 0.2; 
                ctx.strokeStyle = '#444';
                ctx.shadowColor = '#000';
            } else {
                ctx.globalAlpha = flickerAlpha;
                ctx.strokeStyle = currentOutlineColor;
                ctx.shadowColor = currentOutlineColor;
            }


            // --- OUTER BOX ---
            ctx.lineWidth = 4;
            ctx.shadowBlur = 15;
            
            ctx.strokeRect(x, y, width, height);

            // --- INNER CHIP ---
            ctx.strokeStyle = currentTextColor;
            ctx.shadowColor = currentTextColor;

            if (currentLevel >= 7 && currentLevel < MAX_LEVEL && !gameFinished) {
                // Tier 3: Jagged/Damaged Chip (L7-9)
                ctx.beginPath();
                ctx.moveTo(x + 50, y + 50 + (Math.random() * 2 - 1)); 
                ctx.lineTo(x + 250 + (Math.random() * 2 - 1), y + 50 + (Math.random() * 2 - 1));
                ctx.lineTo(x + 250 + (Math.random() * 2 - 1), y + 200 + (Math.random() * 2 - 1));
                ctx.lineTo(x + 50 + (Math.random() * 2 - 1), y + 200 + (Math.random() * 2 - 1));
                ctx.closePath();
                ctx.stroke();
            } else if (!gameFinished) {
                // Tiers 1 & 2: Normal Chip (L1-6)
                ctx.strokeRect(x + 50, y + 50, 200, 150);
            }
            
            ctx.globalAlpha = 1.0; 

            // GPU Title
            ctx.font = '10px "Press Start 2P"';
            ctx.fillStyle = currentTextColor;
            ctx.textAlign = 'center';
            if (gameFinished) {
                ctx.fillText(`AIPU ${AIPU_VERSION} (OFFLINE)`, x + width / 2, y + 30);
            } else {
                ctx.fillText(`AIPU ${AIPU_VERSION} (L${currentLevel})`, x + width / 2, y + 30);
            }
            
            drawHeatSink(x, y, width, height);

            // Pulsing glow when meltdown is active (Level 10)
            if (meltdownActive) {
                const pulse = Math.sin(Date.now() / 150) * 0.5 + 0.5; 
                ctx.shadowBlur = 20 + (pulse * 10); 
                ctx.shadowColor = `rgba(255, 0, 0, ${0.5 + pulse * 0.5})`; 
                ctx.strokeStyle = `rgba(255, 0, 0, ${0.8 + pulse * 0.2})`;
                ctx.strokeRect(x - 5, y - 5, width + 10, height + 10); 
            }

            ctx.shadowBlur = 0;
        }
        
        /** Draws a basic, stylized fire effect over the GPU chip (Meltdown phase only) */
        function drawFireEffect() {
            if (!meltdownActive) return; 

            const xStart = 70;
            const yStart = 200;
            const width = 260;
            const height = 120; 
            const fireParticles = 50; 

            ctx.globalAlpha = 0.8;
            ctx.shadowBlur = 15;

            for (let i = 0; i < fireParticles; i++) {
                const tempFactor = i / fireParticles; 
                const size = 8 + Math.random() * 20; 
                const dx = Math.random() * width;
                const dy = Math.random() * height * (1 - tempFactor);

                const fireX = xStart + dx - size / 2;
                const fireY = yStart - dy;

                if (tempFactor < 0.4) { 
                    ctx.fillStyle = '#ffcc00'; 
                    ctx.shadowColor = '#ffcc00';
                } else if (tempFactor < 0.7) {
                    ctx.fillStyle = '#ff6600'; 
                    ctx.shadowColor = '#ff6600';
                } else {
                    ctx.fillStyle = '#ff0000'; 
                    ctx.shadowColor = '#ff0000';
                }

                ctx.beginPath();
                ctx.arc(fireX, fireY, size / 3, 0, Math.PI * 2);
                ctx.fill();
            }
            ctx.globalAlpha = 1.0;
            ctx.shadowBlur = 0;
        }

        /** Draws the fan animation */
        function drawFan() {
            if (meltdownActive && !isMaxLevelMeltdown || gameFinished) return; 

            const fanConfigs = [];
            
            if (currentLevel <= 4) {
                fanConfigs.push({ centerX: CANVAS_WIDTH / 2, centerY: 175, radius: 60 });
            } else {
                fanConfigs.push({ centerX: 150, centerY: 175, radius: 40 }); 
                fanConfigs.push({ centerX: 250, centerY: 175, radius: 40 }); 
            }

            const currentOutlineColor = getLevelColor();
            const isBrokenLevel = (currentLevel === MAX_LEVEL); 
            
            fanConfigs.forEach((config, index) => {
                const { centerX, centerY, radius } = config;
                
                ctx.save();
                ctx.translate(centerX, centerY);
                
                let rotateFan = true; 
                
                if (isMaxLevelMeltdown) { 
                    if (index === 1) { 
                        rotateFan = false;
                    }
                }
                
                if (rotateFan) { 
                    ctx.rotate(fanRotation);
                }

                ctx.strokeStyle = currentOutlineColor;
                ctx.lineWidth = 3;
                ctx.shadowBlur = 10;
                ctx.shadowColor = currentOutlineColor;

                ctx.beginPath();
                ctx.arc(0, 0, 5, 0, Math.PI * 2);
                ctx.stroke();

                for (let i = 0; i < 4; i++) {
                    ctx.beginPath();
                    
                    if (isBrokenLevel) { 
                        if (i < 3) {
                            ctx.moveTo(0, 0);
                            ctx.lineTo(0, -radius); 
                        } else {
                            ctx.moveTo(0, 0);
                            ctx.lineTo(0, -radius / 3); 
                        }
                    } else {
                        ctx.moveTo(0, 0);
                        ctx.lineTo(0, -radius);
                    }

                    ctx.stroke();
                    ctx.rotate(Math.PI / 2); 
                }

                ctx.restore();
            });
            ctx.shadowBlur = 0;
        }

        /** Draws the vertical temperature bar */
        function drawTemperatureBar() {
            const x = 360;
            const y = 50;
            const width = 20;
            const height = 250;
            
            if (gameFinished) {
                 // Static gray bar for game over
                ctx.strokeStyle = '#444';
                ctx.lineWidth = 3;
                ctx.shadowBlur = 0;
                ctx.strokeRect(x, y, width, height);

                ctx.fillStyle = '#111';
                ctx.fillRect(x + 1, y + 1, width - 2, height - 2);

                ctx.font = '12px "Press Start 2P"';
                ctx.fillStyle = '#444';
                ctx.textAlign = 'center';
                ctx.fillText('N/A', x + width / 2, y + height + 25);
                return;
            }

            const tempPercent = visualTemperature / gpu.maxTemp;
            
            const gradient = ctx.createLinearGradient(0, y + height, 0, y);
            gradient.addColorStop(0, COLOR_TEMP_COLD);
            gradient.addColorStop(0.5, '#ffff00'); 
            gradient.addColorStop(1, COLOR_TEMP_HOT);

            // Outer Frame
            ctx.strokeStyle = getLevelColor();
            ctx.lineWidth = 3;
            ctx.shadowBlur = 10;
            ctx.shadowColor = getLevelColor();
            ctx.strokeRect(x, y, width, height);

            // Temperature Fill
            const fillHeight = height * tempPercent;
            const fillY = y + height - fillHeight;
            
            ctx.fillStyle = gradient;
            ctx.shadowBlur = 15;
            ctx.shadowColor = COLOR_TEMP_HOT; 
            ctx.fillRect(x + 1, fillY + 1, width - 2, fillHeight - 2);

            ctx.shadowBlur = 0;

            // Temp Text
            ctx.font = '12px "Press Start 2P"';
            ctx.fillStyle = getTextColor();
            ctx.textAlign = 'center';
            ctx.fillText(Math.round(visualTemperature) + '%', x + width / 2, y + height + 25);
        }
        
        /** Draws the ambient spark effects. */
        function drawAmbientSparks() {
            if (currentLevel < 7 || meltdownActive || gameFinished) {
                ambientSparks = [];
                return;
            }
            
            if (ambientSparks.length < 30 && Math.random() < 0.2) {
                ambientSparks.push({ 
                    x: 50 + Math.random() * 300, 
                    y: 50 + Math.random() * 250, 
                    size: 1 + Math.random() * 2,
                    vx: (Math.random() - 0.5) * 0.5, 
                    vy: -1 - Math.random() * 1.5, 
                    alpha: 1.0
                });
            }
            
            ambientSparks = ambientSparks.map(spark => {
                spark.x += spark.vx;
                spark.y += spark.vy;
                spark.alpha -= 0.05; 
                return spark;
            }).filter(spark => spark.alpha > 0 && spark.y > 50); 
            
            ctx.fillStyle = COLOR_TEXT_T2;
            ctx.shadowBlur = 5;
            ctx.shadowColor = COLOR_TEXT_T2;
            ambientSparks.forEach(spark => {
                ctx.globalAlpha = spark.alpha;
                ctx.fillRect(spark.x, spark.y, spark.size, spark.size);
            });
            ctx.globalAlpha = 1.0;
            ctx.shadowBlur = 0;
        }

        /** Draws a temporary overlay showing the current level. */
        function drawLevelOverlay() {
            const elapsed = Date.now() - levelStartTime;
            
            if (elapsed < OVERLAY_DURATION) {
                const fadeStart = OVERLAY_DURATION - 1000;
                let alpha = 1;
                if (elapsed > fadeStart) {
                    alpha = 1 - (elapsed - fadeStart) / 1000;
                }

                alpha = Math.max(0, Math.min(1, alpha)); 
                const pulse = Math.sin(Date.now() / 100) * 0.1; 

                ctx.globalAlpha = alpha;
                
                ctx.fillStyle = getTextColor();
                ctx.textAlign = 'center';
                ctx.font = '30px "Press Start 2P"';
                ctx.shadowBlur = 30 + (pulse * 10);
                ctx.shadowColor = getTextColor();
                
                ctx.fillText(`LEVEL ${currentLevel}`, CANVAS_WIDTH / 2, CANVAS_HEIGHT / 2 + 10);

                ctx.globalAlpha = 1.0;
                ctx.shadowBlur = 0;
            }
        }
        
        /** Updates the visual temperature during the dynamic lockout drain. */
        function updateDrainEffect() {
            if (lockoutActive) {
                const elapsed = Date.now() - drainStartTime;
                
                if (elapsed >= currentDrainDurationMS) { 
                    visualTemperature = 0;
                } else {
                    const drainRatio = elapsed / currentDrainDurationMS; 
                    visualTemperature = Math.max(0, 100 * (1 - drainRatio));
                }
            } else {
                visualTemperature = gpu.temperature;
            }
        }

        /** Main drawing function */
        function draw() {
            ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
            
            // 1. Update the visual temperature state
            updateDrainEffect();

            // --- JITTER EFFECT ---
            let screenJitterX = 0;
            let screenJitterY = 0;
            
            // Apply jitter effect to the canvas container (whole screen effect)
            if (!gameFinished && visualTemperature >= JITTER_START_TEMP) {
                const jitterScale = (visualTemperature - JITTER_START_TEMP) / (100 - JITTER_START_TEMP); // 0 to 1
                
                // Max jitter up to 5px
                screenJitterX = (Math.random() * 2 - 1) * 5 * jitterScale; 
                screenJitterY = (Math.random() * 2 - 1) * 5 * jitterScale;

                canvasContainer.style.transform = `translate(${screenJitterX}px, ${screenJitterY}px)`;
            } else if (isMaxLevelMeltdown && !showFinalGameOverOverlay) {
                // Extreme jitter for L10 flourish
                canvasContainer.style.transform = `translate(${(Math.random() * 8 - 4)}px, ${(Math.random() * 8 - 4)}px)`;
            } else {
                // No jitter or reset jitter
                canvasContainer.style.transform = 'translate(0, 0)';
            }
            // ---------------------------------------
            
            drawGPUBox();
            drawFan(); 
            drawTemperatureBar();
            drawFireEffect(); 
            drawAmbientSparks(); 
            drawLevelOverlay(); 

            // --- MAX LEVEL: Extreme Flashing Overlay ---
            if (isMaxLevelMeltdown && !showFinalGameOverOverlay) {
                const pulse = Math.sin(Date.now() / 50) * 0.5 + 0.5; 
                ctx.globalAlpha = 0.3 + (pulse * 0.2); 
                
                if (Math.floor(Date.now() / 100) % 2 === 0) {
                    ctx.fillStyle = '#ff6600'; 
                } else {
                    ctx.fillStyle = '#ffff00'; 
                }
                
                ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
                ctx.globalAlpha = 1.0;
            }
            // -------------------------------------------
            
            // --- FINAL GAME OVER Screen (Fades in after flourish) ---
            if (showFinalGameOverOverlay) {
                ctx.fillStyle = 'rgba(17, 14, 32, 0.9)'; 
                ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
                
                ctx.fillStyle = '#00ffff'; 
                ctx.textAlign = 'center';
                ctx.font = '28px "Press Start 2P"';
                ctx.shadowBlur = 30;
                ctx.shadowColor = '#00ffff';
                
                ctx.fillText("SYSTEM OFFLINE", CANVAS_WIDTH / 2, CANVAS_HEIGHT / 2 - 30);
                
                ctx.font = '16px "Press Start 2P"';
                ctx.fillText("VICTORY ACHIEVED", CANVAS_WIDTH / 2, CANVAS_HEIGHT / 2 + 10);
                
                ctx.font = '10px "Press Start 2P"';
                ctx.fillText(`Final Meltdowns: ${meltdownCount}`, CANVAS_WIDTH / 2, CANVAS_HEIGHT / 2 + 50);
                
                ctx.shadowBlur = 0;
            }
        }

        // --- GAME LOGIC ---
        
        /** Updates the temperature (decay) and checks win condition */
        function updateGameLogic() {
            if (!gameRunning || gameFinished) return;

            // 1. Heat Decay (Temperature decreases over time, based on current level)
            const decayRate = getCurrentDecayRate(); 
            gpu.temperature = Math.max(0, gpu.temperature - decayRate);
            
            if (gpu.temperature >= gpu.maxTemp) {
                 endGame("SUCCESS: GPU MELTDOWN! YOU WIN!", true); // Pass true to indicate a natural win
            }
        }
        
        /** Animation loop for the fan rotation and drawing */
        function animateFan() {
            if (!meltdownActive) {
                let tempFactor = visualTemperature / 100; 
                gpu.fanSpeed = 5 + (tempFactor * 25); 
                fanRotation += 0.05 * (gpu.fanSpeed / 5);
            } else if (isMaxLevelMeltdown && !showFinalGameOverOverlay) {
                fanRotation += 0.08; 
            } else {
                fanRotation = 0; 
            }

            draw();

            if (!showFinalGameOverOverlay) {
                animationFrameId = requestAnimationFrame(animateFan);
            }
        }

        /** Ends the game and displays the final message and screen (Win/Loss) 
         * @param {string} message - The text message to display.
         * @param {boolean} isNaturalWin - True if triggered by 100% heat, false if triggered by test skip.
        */
        function endGame(message, isNaturalWin = false) {
            gameRunning = false;
            clearInterval(gameLoopInterval);
            
            const isWin = message.includes("MELTDOWN");
            meltdownActive = isWin; 
            
            if (isWin) {
                playCrashSound(); 
            }

            if (isWin && currentLevel === MAX_LEVEL) {
                isMaxLevelMeltdown = true;
                gameFinished = true; 
                playCodaSequence();
            }
            
            if (isWin) {
                meltdownCount++;

                if (!gameFinished) {
                    currentLevel++;
                    updateMusicStatus(currentLevel); 
                }

                updateDisplays(); 

                // Set the duration for the visual bar drain
                currentDrainDurationMS = getDrainDurationMS(currentLevel); 
                drainStartTime = Date.now();
                
                // --- LOCKOUT LOGIC (ONLY for natural win) ---
                if (isNaturalWin) {
                    lockoutActive = true;
                    executionButton.disabled = true;
                    
                    executionButton.classList.add('button-flash'); 
                    executionButton.textContent = "MELTDOWN"; 
                    resetGameButton.classList.add('hidden'); 

                    const lockoutDuration = BASE_HEAT_LOCKOUT_MS; 
                    let timer = Math.ceil(lockoutDuration / 1000); 
                    
                    lockoutMessage.textContent = `MELTDOWN: ${timer} SECONDS`; 

                    const lockoutTimerInterval = setInterval(() => {
                        timer--;
                        if (timer > 0) {
                            lockoutMessage.textContent = `MELTDOWN: ${timer} SECONDS`;
                        }
                    }, 1000);
                    
                    // 1. TOTAL LOCKOUT TIMEOUT (Constant 5 seconds)
                    setTimeout(() => {
                        clearInterval(lockoutTimerInterval);
                        lockoutActive = false;
                        
                        // Proceed to final screen or reset for next level
                        handlePostMeltdownReset();
                    }, lockoutDuration);
                } else {
                    // Test Skip or immediate level reset (no lockout)
                    handlePostMeltdownReset();
                }

            } else {
                // Loss/Game Start state
                executionButton.textContent = "START GAME";
                executionButton.disabled = false;
                lockoutMessage.textContent = '';
                resetGameButton.classList.add('hidden'); 
            }

            if (!gameFinished) {
                // Display message over canvas for non-final outcomes
                ctx.fillStyle = 'rgba(17, 14, 32, 0.8)'; 
                ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
                
                ctx.fillStyle = getTextColor();
                ctx.textAlign = 'center';
                ctx.font = '14px "Press Start 2P"';
                ctx.shadowBlur = 20;
                ctx.shadowColor = getTextColor();
                
                const lines = message.split(': ');
                lines.forEach((line, index) => {
                    ctx.fillText(line, CANVAS_WIDTH / 2, CANVAS_HEIGHT / 2 - 20 + (index * 25));
                });
                ctx.shadowBlur = 0;
            }
            
            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            animateFan();
        }
        
        /** Handles the logic after a meltdown, either showing the final screen or resetting for the next level. */
        function handlePostMeltdownReset() {
            if (gameFinished) {
                lockoutMessage.textContent = ''; 
                executionButton.classList.remove('button-flash'); 
                
                // --- 3-SECOND VISUAL FLOURISH TIMEOUT (After Coda Audio) ---
                setTimeout(() => {
                    showFinalGameOverOverlay = true; 
                    
                    executionButton.textContent = "GAME OVER";
                    executionButton.disabled = true; 
                    resetGameButton.classList.remove('hidden'); 
                    
                    draw();
                }, FINAL_FLOURISH_DURATION); 

            } else { // Levels 1-9 win (continue playing)
                lockoutMessage.textContent = ''; 
                executionButton.classList.remove('button-flash'); 
                
                isMaxLevelMeltdown = false;
                gpu.temperature = 0; 
                
                resetGame(); 
            }
        }


        // --- NON-DESTRUCTIVE TEST SKIP FUNCTION (UPDATED) ---
        /** Non-destructive function to instantly skip to the next level for testing. */
        function skipLevelForTesting() {
            if (!gameRunning || lockoutActive || gameFinished) return;

            // Trigger max level win if 'N' is pressed on MAX_LEVEL
            if (currentLevel >= MAX_LEVEL) {
                // If max level, trigger the win state (without lockout)
                endGame("SUCCESS: GPU MELTDOWN! YOU WIN!", false); 
                return;
            }

            // Standard level skip (L1 -> L2, etc.)
            currentLevel++;
            meltdownCount++; // Count it as a successful meltdown
            
            // NOTE: The UI message is intentionally removed here to keep the skip hidden.
            
            // Reset GPU and game state
            gpu.temperature = 0;
            visualTemperature = 0; 
            
            // Update game visuals and audio
            updateDisplays();
            updateMusicStatus(currentLevel);
            
            // Trigger the Level Overlay (since a new level started)
            levelStartTime = Date.now(); 

            // Ensure we update canvas quickly
            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            animateFan();

            console.log(`[TEST MODE] Skipped to Level ${currentLevel}`);
        }
        // --- END NON-DESTRUCTIVE TEST SKIP FUNCTION ---


        // --- INITIALIZATION / RESET ---

        /** Sets up the initial game state on load, or resets the game to L1 if 'fullReset' is true. */
        function initializeGame(fullReset = false) {
            gameRunning = false;
            lockoutActive = false;
            meltdownActive = false; 
            isMaxLevelMeltdown = false; 
            showFinalGameOverOverlay = false; 
            
            stopMusic(); 
            // Re-initialize music to reset all Tone sequences/state for a clean restart
            initMusic(); 

            if (fullReset) {
                meltdownCount = 0;
                currentLevel = 1;
            }
            gameFinished = false; 
            ambientSparks = []; 
            visualTemperature = 0; 
            currentDrainDurationMS = getDrainDurationMS(currentLevel); 
            gpu.temperature = 0;
            gpu.fanSpeed = 5;

            levelStartTime = Date.now(); 

            updateDisplays();
            lockoutMessage.textContent = ''; 

            executionButton.textContent = "START GAME";
            executionButton.disabled = false;
            executionButton.classList.remove('button-flash'); 
            resetGameButton.classList.add('hidden'); 

            // Display welcome message on canvas
            ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
            ctx.fillStyle = 'rgba(17, 14, 32, 0.8)'; 
            ctx.fillRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
            
            ctx.fillStyle = COLOR_TEXT_T1;
            ctx.textAlign = 'center';
            ctx.font = '16px "Press Start 2P"';
            ctx.shadowBlur = 20;
            ctx.shadowColor = COLOR_TEXT_T1;
            
            ctx.fillText(`AIPU ${AIPU_VERSION}`, CANVAS_WIDTH / 2, CANVAS_HEIGHT / 2 - 20); 

            ctx.font = '12px "Press Start 2P"';
            ctx.fillText("CLICK THE BUTTON TO START", CANVAS_WIDTH / 2, CANVAS_HEIGHT / 2 + 50);
            ctx.shadowBlur = 0;
            
            if (gameLoopInterval) clearInterval(gameLoopInterval);
            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            animateFan();
        }

        function resetGame() {
            gpu.temperature = 0;
            visualTemperature = 0; 
            gpu.fanSpeed = 20; 
            gameRunning = true;
            lockoutActive = false; 
            meltdownActive = false; 
            isMaxLevelMeltdown = false; 
            ambientSparks = []; 
            executionButton.textContent = "PROMPT ME"; 
            executionButton.disabled = false; 
            lastPressTime = 0;
            lockoutMessage.textContent = ''; 
            resetGameButton.classList.add('hidden'); 
            
            levelStartTime = Date.now(); 
            
            updateDisplays(); 

            if (gameLoopInterval) clearInterval(gameLoopInterval);
            gameLoopInterval = setInterval(updateGameLogic, GAME_LOGIC_TICK);
            
            // Music status update is crucial here for the new level
            updateMusicStatus(currentLevel); 

            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            animateFan(); 
        }

        // --- INPUT HANDLERS ---
        
        executionButton.addEventListener('click', (e) => {
            
            if (lockoutActive || gameFinished) {
                e.preventDefault();
                return;
            }

            if (!gameRunning) {
                // Start the audio context and music here, tied directly to the click
                Tone.start().then(() => {
                    startGameMusic(); // Start Transport and update music status/layers
                    resetGame();      // Set game state to L1 and start logic loop
                }).catch(err => console.error("Tone.js failed to start:", err));
                return;
            }
            
            const currentTime = Date.now();
            
            // Prevent spamming too fast 
            if (currentTime - lastPressTime < pressDelay) {
                e.preventDefault();
                return;
            }
            
            lastPressTime = currentTime;

            // Increase temperature
            gpu.temperature = Math.min(gpu.maxTemp, gpu.temperature + HEAT_PER_PRESS);
            
            // Check win condition immediately after press
            if (gpu.temperature >= gpu.maxTemp) {
                 endGame("SUCCESS: GPU MELTDOWN! YOU WIN!", true); // Natural win
            }
        });

        // --- KEYBOARD LISTENER FOR LEVEL SKIP ---
        document.addEventListener('keydown', (e) => {
            // Check for 'N' key press (case insensitive)
            if (e.key.toUpperCase() === 'N') {
                e.preventDefault(); 
                skipLevelForTesting();
            }
        });


        // Initialize the music setup immediately on script load
        initMusic();
        
        // --- WINDOW LOAD ---
        window.onload = function() {
            initializeGame(true);
            
            function resizeCanvas() {
                const container = document.querySelector('.game-container');
                const containerWidth = container.clientWidth - 32; 
                
                const size = Math.min(containerWidth, CANVAS_WIDTH);
                canvas.width = size;
                canvas.height = size;
                
                draw();
            }

            window.addEventListener('resize', resizeCanvas);
            resizeCanvas(); 
        };

    </script>

</body>
</html>
