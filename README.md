<!DOCTYPE html>
<html lang="ms">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Simulasi Pemula Pengubah Auto (Auto-Trans Starter)</title>
    <style>
        :root {
            --wire-off: #333;
            --wire-live: #ff3333;
            --wire-neutral: #3333ff;
            --bg-panel: #2c3e50;
            --component-bg: #ecf0f1;
        }
        body {
            font-family: 'Segoe UI', sans-serif;
            background-color: #1a1a1a;
            color: white;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 { color: #f1c40f; margin-bottom: 10px; text-align: center;}
        .container {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            justify-content: center;
            width: 100%;
            max-width: 1200px;
        }
        
        /* PANEL STYLES */
        .panel-box {
            background: var(--bg-panel);
            border: 4px solid #95a5a6;
            border-radius: 10px;
            padding: 20px;
            width: 300px;
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
            box-shadow: 0 10px 20px rgba(0,0,0,0.5);
            height: fit-content;
        }
        .panel-title { border-bottom: 2px solid #7f8c8d; width: 100%; text-align: center; padding-bottom: 5px; margin-bottom: 10px; font-weight: bold; color: #bdc3c7; }
        
        /* Buttons & Switches */
        .row { display: flex; gap: 15px; justify-content: center; align-items: center; width: 100%; }
        
        button {
            padding: 10px 15px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            box-shadow: 0 4px 0 rgba(0,0,0,0.3);
            transition: transform 0.1s;
        }
        button:active { transform: translateY(4px); box-shadow: none; }

        .btn-start { background: #27ae60; color: white; }
        .btn-stop { background: #c0392b; color: white; }
        .btn-emg { 
            background: #e74c3c; color: white; 
            border-radius: 50%; width: 60px; height: 60px; 
            border: 4px solid #c0392b; font-size: 10px;
        }
        .btn-emg.active { background: #731810; border-color: #fff; box-shadow: inset 0 0 10px #000; }
        
        .selector-switch {
            display: flex;
            flex-direction: column;
            align-items: center;
            background: #34495e;
            padding: 10px;
            border-radius: 5px;
        }
        .switch-knob {
            width: 40px;
            height: 40px;
            background: #fff;
            border-radius: 50%;
            border: 2px solid #7f8c8d;
            position: relative;
            cursor: pointer;
            transition: transform 0.3s;
        }
        .switch-knob::after {
            content: ''; position: absolute; top: 5px; left: 18px; width: 4px; height: 15px; background: #333;
        }
        .sel-manual { transform: rotate(-45deg); }
        .sel-off { transform: rotate(0deg); }
        .sel-auto { transform: rotate(45deg); }

        /* Lamps */
        .lamp { width: 30px; height: 30px; border-radius: 50%; border: 2px solid #555; background: #333; transition: background 0.2s; box-shadow: inset 0 0 5px #000;}
        .lamp.red.on { background: #ff0000; box-shadow: 0 0 15px #ff0000; }
        .lamp.yellow.on { background: #f1c40f; box-shadow: 0 0 15px #f1c40f; }
        .lamp.blue.on { background: #3498db; box-shadow: 0 0 15px #3498db; }
        .lamp.green.on { background: #2ecc71; box-shadow: 0 0 15px #2ecc71; }
        .lamp-label { font-size: 10px; text-align: center; margin-top: 2px; color: #aaa; }

        /* SCHEMATIC BOX & TABS */
        .schematic-container {
            flex-grow: 1;
            min-width: 600px;
            display: flex;
            flex-direction: column;
        }

        .tabs { display: flex; gap: 5px; margin-bottom: 0; }
        .tab-btn {
            padding: 10px 20px;
            background: #444;
            color: #aaa;
            border: none;
            border-radius: 5px 5px 0 0;
            cursor: pointer;
            font-weight: bold;
        }
        .tab-btn.active { background: white; color: black; }

        .schematic-box {
            background: white;
            color: black;
            padding: 10px;
            border-radius: 0 5px 5px 5px;
            overflow-x: auto;
            position: relative;
            height: 520px;
        }
        
        .schematic-view { display: none; }
        .schematic-view.active { display: block; }

        /* SVG Styles */
        svg { width: 100%; height: 500px; min-width: 750px; }
        
        .wire { fill: none; stroke: var(--wire-off); stroke-width: 2; stroke-linecap: round; transition: stroke 0.3s; }
        .wire.live { stroke: var(--wire-live); stroke-width: 3; }
        .wire.neutral { stroke: var(--wire-neutral); }
        
        /* Power Circuit Wire Colors */
        .wire.live-r { stroke: #e74c3c; stroke-width: 3; } /* Red Phase */
        .wire.live-y { stroke: #f1c40f; stroke-width: 3; } /* Yellow Phase */
        .wire.live-b { stroke: #3498db; stroke-width: 3; } /* Blue Phase */

        .symbol-box { fill: none; stroke: black; stroke-width: 2; }
        .symbol-coil { fill: none; stroke: black; stroke-width: 2; }
        .symbol-contact { stroke: black; stroke-width: 2; cursor: pointer; }
        .contact-label { font-size: 10px; fill: #666; font-family: monospace; cursor: pointer; }
        .contact-label:hover { fill: blue; font-weight: bold; text-decoration: underline; }

        .active-component { fill: rgba(46, 204, 113, 0.3); stroke: green; }
        .component-text { font-size: 12px; font-weight: bold; fill: #333; }

        /* CONFIGURATION MODAL */
        .config-panel {
            width: 100%;
            background: #333;
            padding: 10px;
            margin-top: 10px;
            border-radius: 5px;
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 10px;
        }
        .config-item { display: flex; flex-direction: column; }
        .config-item label { font-size: 12px; color: #aaa; }
        .config-item input { background: #222; border: 1px solid #555; color: white; padding: 2px 5px; font-size: 12px; }

        /* MOTOR ANIMATION */
        .motor-box {
            width: 100px; height: 100px; background: #444; border-radius: 10px; 
            display: flex; justify-content: center; align-items: center; flex-direction: column;
            border: 2px solid #777;
        }
        .fan { width: 60px; height: 60px; border-radius: 50%; border: 5px solid #ccc; border-top-color: #3498db; transition: animation 0.5s; }
        .spin-slow { animation: spin 1s linear infinite; }
        .spin-fast { animation: spin 0.3s linear infinite; }
        @keyframes spin { 100% { transform: rotate(360deg); } }

    </style>
</head>
<body>

    <h1>Simulasi Pendawian Auto-Trans (Manual & Auto)</h1>

    <div class="container">
        <!-- PANEL KAWALAN -->
        <div class="panel-box">
            <div class="panel-title">PANEL KAWALAN</div>
            
            <!-- Lamps -->
            <div class="row">
                <div style="display:flex; flex-direction:column; align-items:center;">
                    <div id="lampEmg" class="lamp red"></div>
                    <div class="lamp-label">Emergency</div>
                </div>
                <div style="display:flex; flex-direction:column; align-items:center;">
                    <div id="lampTrip" class="lamp yellow"></div>
                    <div class="lamp-label">Trip</div>
                </div>
                <div style="display:flex; flex-direction:column; align-items:center;">
                    <div id="lampStep1" class="lamp blue"></div>
                    <div class="lamp-label">Langkah 1</div>
                </div>
                <div style="display:flex; flex-direction:column; align-items:center;">
                    <div id="lampStep2" class="lamp green"></div>
                    <div class="lamp-label">Langkah 2</div>
                </div>
            </div>

            <!-- Selector Switch -->
            <div class="row" style="margin-top: 15px;">
                <label style="font-size: 0.8em; margin-right: 10px;">MAN</label>
                <div class="selector-switch" onclick="toggleSelector()">
                    <div id="knobSelector" class="switch-knob sel-off"></div>
                    <div style="font-size: 0.7em; margin-top:5px;">OFF</div>
                </div>
                <label style="font-size: 0.8em; margin-left: 10px;">AUTO</label>
            </div>

            <!-- Buttons -->
            <div class="row">
                <button class="btn-start" onmousedown="pressStart()" onmouseup="releaseStart()">START (ON)</button>
                <button class="btn-stop" onmousedown="pressStop()" onmouseup="releaseStop()">STOP (OFF)</button>
            </div>
            
            <div class="row">
                 <button id="btnEmg" class="btn-emg" onclick="toggleEmg()">EMG STOP</button>
            </div>

            <div class="row" style="border-top: 1px solid #555; padding-top: 10px; width: 100%;">
                <button style="background: #f39c12; font-size: 0.8em;" onclick="toggleTrip()">Test Trip (TOR)</button>
                <button style="background: #7f8c8d; font-size: 0.8em;" onclick="toggleMCB()" id="btnMcb">MCB: OFF</button>
            </div>

            <!-- Motor Visual -->
            <div class="motor-box">
                <div id="motorFan" class="fan"></div>
                <div id="motorText" style="font-size: 10px; margin-top: 5px;">OFF</div>
            </div>
            <div style="font-size: 0.8em; text-align: center; color: #ccc;">
                Auto Timer: <span id="timer1Display">0</span>s<br>
                Step Timer: <span id="timer2Display">0</span>s
            </div>
        </div>

        <!-- SCHEMATIC VIEW -->
        <div class="schematic-container">
            <!-- Tabs -->
            <div class="tabs">
                <button class="tab-btn active" onclick="setTab('control')">Litar Kawalan</button>
                <button class="tab-btn" onclick="setTab('power')">Litar Utama</button>
            </div>

            <div class="schematic-box">
                
                <!-- VIEW 1: LITAR KAWALAN -->
                <div id="viewControl" class="schematic-view active">
                    <div style="font-weight: bold; border-bottom: 2px solid #000; margin-bottom: 5px;">LITAR KAWALAN (CONTROL)</div>
                    <div style="font-size: 0.8em; color: #555; margin-bottom: 10px;">Klik pada nombor pin untuk mengubah nilainya.</div>
                    
                    <svg viewBox="0 0 800 500" id="schematicSvg">
                        <defs>
                            <circle id="node" r="3" fill="black" />
                        </defs>

                        <!-- LIVE RAIL -->
                        <line x1="50" y1="20" x2="50" y2="450" stroke="red" stroke-width="2" />
                        <text x="40" y="15" fill="red" font-weight="bold">L</text>

                        <!-- NEUTRAL RAIL -->
                        <line x1="750" y1="20" x2="750" y2="450" stroke="blue" stroke-width="2" />
                        <text x="740" y="15" fill="blue" font-weight="bold">N</text>

                        <!-- CONTROL COMPONENTS (SAME AS BEFORE) -->
                        <!-- RUNG 1: MCB & TOR & EMG -->
                        <line id="w1" x1="50" y1="50" x2="100" y2="50" class="wire" /> 
                        <rect x="100" y="40" width="20" height="20" fill="none" stroke="black" />
                        <text x="100" y="35" font-size="10">MCB</text>
                        <line id="w2" x1="120" y1="50" x2="150" y2="50" class="wire" />
                        
                        <!-- TOR (NC) -->
                        <g id="sym_tor" transform="translate(150, 50)">
                            <path d="M0 0 L10 -5 L20 0" stroke="black" fill="none" />
                            <text id="lbl_tor_nc" x="0" y="-10" class="contact-label" onclick="editPin('lbl_tor_nc', '95-96')">95-96</text>
                        </g>
                        <line id="w3" x1="170" y1="50" x2="200" y2="50" class="wire" />

                        <!-- EMG (NC) -->
                        <g id="sym_emg" transform="translate(200, 50)">
                            <circle cx="10" cy="0" r="3" fill="red" />
                            <path d="M0 0 L20 0" stroke="black" stroke-width="2" />
                            <text id="lbl_emg" x="0" y="-10" class="contact-label" onclick="editPin('lbl_emg', '1-2')">1-2</text>
                        </g>
                        <line id="w4" x1="220" y1="50" x2="250" y2="50" class="wire" />
                        <use href="#node" x="250" y="50" />

                        <!-- BRANCH: AUTO (RUNG 2) -->
                        <line id="w_auto_feed" x1="250" y1="50" x2="250" y2="100" class="wire" />
                        <g id="sw_auto" transform="translate(250, 100)">
                            <path d="M0 0 L20 10" stroke="black" stroke-width="2" id="vis_sw_auto"/>
                            <text x="25" y="5" font-size="10">AUTO</text>
                        </g>
                        <line id="w_auto_post" x1="270" y1="110" x2="350" y2="110" class="wire" />
                        <rect id="coil_tdr1" x="650" y="95" width="40" height="30" class="symbol-coil" />
                        <text x="655" y="115" font-size="12">TDR1</text>
                        <line id="w_tdr1_live" x1="350" y1="110" x2="650" y2="110" class="wire" />
                        <line id="w_tdr1_neu" x1="690" y1="110" x2="750" y2="110" class="wire neutral" />
                        <text x="650" y="90" font-size="10" fill="blue">(5s)</text>

                        <!-- RUNG 3: TDR1 Contact -> Relay R1 -->
                        <line id="w_r1_feed" x1="350" y1="110" x2="350" y2="150" class="wire" />
                        <g id="con_tdr1" transform="translate(350, 150)">
                            <path d="M0 0 L10 -10 L20 0" stroke="black" fill="none" />
                            <text id="lbl_tdr1_no" x="0" y="-15" class="contact-label" onclick="editPin('lbl_tdr1_no', '8-6')">8-6</text>
                        </g>
                        <line id="w_r1_mid" x1="370" y1="150" x2="650" y2="150" class="wire" />
                        <rect id="coil_r1" x="650" y="135" width="40" height="30" class="symbol-coil" />
                        <text x="660" y="155" font-size="12">R1</text>
                        <line id="w_r1_neu" x1="690" y1="150" x2="750" y2="150" class="wire neutral" />

                        <!-- RUNG 4: R1 Holding -->
                        <line id="w_r1_hold_feed" x1="350" y1="150" x2="350" y2="180" class="wire" />
                        <g id="con_r1_hold" transform="translate(350, 180)">
                            <path d="M0 0 L10 -10 L20 0" stroke="black" fill="none" />
                            <text id="lbl_r1_hold" x="0" y="-15" class="contact-label" onclick="editPin('lbl_r1_hold', '13-14')">13-14</text>
                        </g>
                        <line id="w_r1_hold_out" x1="370" y1="180" x2="400" y2="180" class="wire" />
                        <line id="w_r1_hold_join" x1="400" y1="180" x2="400" y2="150" class="wire" />
                        <use href="#node" x="400" y="150" />

                        <!-- BRANCH: MANUAL / COMMON START (RUNG 5) -->
                        <line id="w_man_feed" x1="250" y1="50" x2="250" y2="250" class="wire" />
                        <g id="sw_man" transform="translate(250, 250)">
                            <path d="M0 0 L20 10" stroke="black" stroke-width="2" id="vis_sw_man" />
                            <text x="25" y="5" font-size="10">MAN</text>
                        </g>
                        
                        <line id="w_stop_feed" x1="270" y1="260" x2="300" y2="260" class="wire" />
                        <g id="pb_stop" transform="translate(300, 260)">
                            <rect x="0" y="-5" width="20" height="10" fill="none" stroke="black" />
                            <text x="0" y="-10" font-size="10">STOP</text>
                        </g>
                        <line id="w_start_feed" x1="320" y1="260" x2="350" y2="260" class="wire" />
                        <g id="pb_start" transform="translate(350, 260)">
                            <line x1="0" y1="0" x2="20" y2="0" stroke="black" stroke-width="2" id="vis_pb_start"/>
                            <text x="0" y="-10" font-size="10">START</text>
                        </g>
                        
                        <line id="w_auto_inj_feed" x1="670" y1="165" x2="670" y2="220" class="wire" />
                        <line id="w_auto_inj_run" x1="670" y1="220" x2="380" y2="220" class="wire" />
                        <g id="con_r1_start" transform="translate(380, 220)">
                            <path d="M0 0 L10 -10 L20 0" stroke="black" fill="none" />
                            <text id="lbl_r1_start" x="0" y="-15" class="contact-label" onclick="editPin('lbl_r1_start', '43-44')">R1(43-44)</text>
                        </g>
                        <line id="w_auto_inj_join" x1="400" y1="220" x2="400" y2="260" class="wire" />

                        <!-- Main Bus Logic -->
                        <line id="w_main_bus" x1="370" y1="260" x2="450" y2="260" class="wire" />

                        <!-- SC & TC Logic -->
                        <line id="w_lc_nc_feed" x1="450" y1="260" x2="450" y2="300" class="wire" />
                        <g id="con_lc_nc" transform="translate(450, 300)">
                            <path d="M0 0 L20 0" stroke="black" />
                            <line x1="5" y1="5" x2="15" y2="-5" stroke="black" />
                            <text id="lbl_lc_nc" x="-20" y="5" class="contact-label" onclick="editPin('lbl_lc_nc', 'LC(21-22)')">LC(21-22)</text>
                        </g>
                        <line id="w_tdr2_nc_feed" x1="470" y1="300" x2="500" y2="300" class="wire" />
                        <g id="con_tdr2_nc" transform="translate(500, 300)">
                            <path d="M0 0 L20 0" stroke="black" />
                            <line x1="5" y1="5" x2="15" y2="-5" stroke="black" />
                            <text id="lbl_tdr2_nc" x="0" y="-15" class="contact-label" onclick="editPin('lbl_tdr2_nc', 'T2(8-5)')">T2(8-5)</text>
                        </g>
                        
                        <!-- Coils SC, TC, TDR2 -->
                        <line id="w_sc_feed" x1="520" y1="300" x2="650" y2="300" class="wire" />
                        <rect id="coil_sc" x="650" y="285" width="40" height="30" class="symbol-coil" />
                        <text x="660" y="305" font-size="12">SC</text>
                        <line id="w_sc_neu" x1="690" y1="300" x2="750" y2="300" class="wire neutral" />

                        <line id="w_tc_feed" x1="550" y1="300" x2="550" y2="340" class="wire" />
                        <line id="w_tc_run" x1="550" y1="340" x2="650" y2="340" class="wire" />
                        <rect id="coil_tc" x="650" y="325" width="40" height="30" class="symbol-coil" />
                        <text x="660" y="345" font-size="12">TC</text>
                        <line id="w_tc_neu" x1="690" y1="340" x2="750" y2="340" class="wire neutral" />
                        
                        <line id="w_tdr2_feed" x1="580" y1="300" x2="580" y2="380" class="wire" />
                        <line id="w_tdr2_run" x1="580" y1="380" x2="650" y2="380" class="wire" />
                        <rect id="coil_tdr2" x="650" y="365" width="40" height="30" class="symbol-coil" />
                        <text x="655" y="385" font-size="12">TDR2</text>
                        <line id="w_tdr2_neu" x1="690" y1="380" x2="750" y2="380" class="wire neutral" />

                        <!-- Holding SC -->
                        <line id="w_sc_hold_feed" x1="450" y1="260" x2="450" y2="230" class="wire" />
                        <g id="con_sc_hold" transform="translate(450, 230)">
                            <path d="M0 0 L10 -10 L20 0" stroke="black" fill="none" />
                            <text id="lbl_sc_hold" x="0" y="-15" class="contact-label" onclick="editPin('lbl_sc_hold', 'SC(13-14)')">SC(13-14)</text>
                        </g>
                        <line id="w_sc_hold_out" x1="470" y1="230" x2="530" y2="230" class="wire" />
                        <line id="w_sc_hold_join" x1="530" y1="230" x2="530" y2="300" class="wire" />

                        <!-- LC Logic -->
                        <line id="w_lc_path_feed" x1="420" y1="260" x2="420" y2="420" class="wire" />
                        <g id="con_tdr2_no" transform="translate(420, 420)">
                            <path d="M0 0 L10 -10 L20 0" stroke="black" fill="none" />
                            <text id="lbl_tdr2_no" x="0" y="-15" class="contact-label" onclick="editPin('lbl_tdr2_no', 'T2(8-6)')">T2(8-6)</text>
                        </g>
                        <line id="w_lc_mid" x1="440" y1="420" x2="480" y2="420" class="wire" />
                        <g id="con_sc_nc" transform="translate(480, 420)">
                            <path d="M0 0 L20 0" stroke="black" />
                            <line x1="5" y1="5" x2="15" y2="-5" stroke="black" />
                            <text id="lbl_sc_nc" x="-20" y="5" class="contact-label" onclick="editPin('lbl_sc_nc', 'SC(21-22)')">SC(21-22)</text>
                        </g>
                        <line id="w_lc_final" x1="500" y1="420" x2="650" y2="420" class="wire" />
                        <rect id="coil_lc" x="650" y="405" width="40" height="30" class="symbol-coil" />
                        <text x="660" y="425" font-size="12">LC</text>
                        <line id="w_lc_neu" x1="690" y1="420" x2="750" y2="420" class="wire neutral" />

                        <!-- LC Holding -->
                        <line id="w_lc_hold_feed" x1="420" y1="420" x2="420" y2="450" class="wire" />
                        <g id="con_lc_hold" transform="translate(420, 450)">
                            <path d="M0 0 L10 -10 L20 0" stroke="black" fill="none" />
                            <text id="lbl_lc_hold" x="0" y="20" class="contact-label" onclick="editPin('lbl_lc_hold', 'LC(13-14)')">LC(13-14)</text>
                        </g>
                        <line id="w_lc_hold_ret" x1="440" y1="450" x2="520" y2="450" class="wire" />
                        <line id="w_lc_hold_up" x1="520" y1="450" x2="520" y2="420" class="wire" />
                    </svg>
                </div>

                <!-- VIEW 2: LITAR UTAMA -->
                <div id="viewPower" class="schematic-view">
                    <div style="font-weight: bold; border-bottom: 2px solid #000; margin-bottom: 5px;">LITAR UTAMA (POWER)</div>
                    
                    <svg viewBox="0 0 800 500" id="powerSvg">
                        <!-- INPUT SUPPLY -->
                        <text x="30" y="30" fill="#e74c3c" font-weight="bold">R</text>
                        <text x="50" y="30" fill="#f1c40f" font-weight="bold">Y</text>
                        <text x="70" y="30" fill="#3498db" font-weight="bold">B</text>

                        <!-- WIRES TO MCB -->
                        <path id="p-mcb-in-r" d="M35 40 V80" class="wire" />
                        <path id="p-mcb-in-y" d="M55 40 V80" class="wire" />
                        <path id="p-mcb-in-b" d="M75 40 V80" class="wire" />

                        <!-- MCB 3 POLE -->
                        <rect x="30" y="80" width="50" height="40" class="component-rect" id="sym-mcb-p" style="stroke:black; fill:none; stroke-width:2;" />
                        <text x="85" y="105" class="component-text">MCB</text>

                        <!-- SPLIT POINT -->
                        <path id="p-mcb-out-r" d="M35 120 V150" class="wire" />
                        <path id="p-mcb-out-y" d="M55 120 V150" class="wire" />
                        <path id="p-mcb-out-b" d="M75 120 V150" class="wire" />

                        <!-- BRANCH LEFT: LC (RUN) -->
                        <path id="p-lc-in-r" d="M35 150 H150 V200" class="wire" />
                        <path id="p-lc-in-y" d="M55 150 H170 V200" class="wire" />
                        <path id="p-lc-in-b" d="M75 150 H190 V200" class="wire" />

                        <!-- LC CONTACTOR -->
                        <rect x="140" y="200" width="60" height="40" class="component-rect" id="sym-lc-p" style="stroke:black; fill:none; stroke-width:2;" />
                        <text x="210" y="225" class="component-text">LC (Run)</text>

                        <!-- BRANCH RIGHT: SC (START) -->
                        <path id="p-sc-in-r" d="M35 150 H350 V200" class="wire" />
                        <path id="p-sc-in-y" d="M55 150 H370 V200" class="wire" />
                        <path id="p-sc-in-b" d="M75 150 H390 V200" class="wire" />

                        <!-- SC CONTACTOR -->
                        <rect x="340" y="200" width="60" height="40" class="component-rect" id="sym-sc-p" style="stroke:black; fill:none; stroke-width:2;" />
                        <text x="410" y="225" class="component-text">SC (Start)</text>

                        <!-- AUTO TRANSFORMER -->
                        <path id="p-at-in-r" d="M350 240 V280" class="wire" />
                        <path id="p-at-in-y" d="M370 240 V280" class="wire" />
                        <path id="p-at-in-b" d="M390 240 V280" class="wire" />
                        
                        <rect x="340" y="280" width="60" height="60" class="component-rect" style="stroke:black; fill:#ddd; stroke-width:2;" />
                        <text x="345" y="315" font-size="10">Auto-Trans</text>

                        <!-- TC (STAR POINT) -->
                        <path id="p-tc-r" d="M350 340 V360 H450" class="wire" />
                        <path id="p-tc-y" d="M370 340 V370 H450" class="wire" />
                        <path id="p-tc-b" d="M390 340 V380 H450" class="wire" />

                        <rect x="450" y="350" width="40" height="40" class="component-rect" id="sym-tc-p" style="stroke:black; fill:none; stroke-width:2;" />
                        <text x="500" y="375" class="component-text">TC</text>
                        
                        <!-- Shorting Star -->
                        <path id="p-tc-short" d="M470 350 V340" class="wire" />

                        <!-- AUTO TRANS OUTPUT (TAPS) TO MOTOR LINE -->
                        <!-- Visual simplified: Taps join LC output -->
                        <path id="p-tap-r" d="M340 300 H320 V260 H220 V350" class="wire" style="stroke-dasharray: 5,5;" />
                        <path id="p-tap-y" d="M340 310 H310 V270 H230 V350" class="wire" style="stroke-dasharray: 5,5;" />
                        <path id="p-tap-b" d="M340 320 H300 V280 H240 V350" class="wire" style="stroke-dasharray: 5,5;" />
                        
                        <!-- LC OUTPUT -->
                        <path id="p-lc-out-r" d="M150 240 V350" class="wire" />
                        <path id="p-lc-out-y" d="M170 240 V350" class="wire" />
                        <path id="p-lc-out-b" d="M190 240 V350" class="wire" />

                        <!-- JUNCTION TO TOR -->
                        <path id="p-tor-in-r" d="M150 350 V380" class="wire" />
                        <path id="p-tor-in-y" d="M170 350 V380" class="wire" />
                        <path id="p-tor-in-b" d="M190 350 V380" class="wire" />

                        <!-- TOR -->
                        <rect x="140" y="380" width="60" height="30" class="component-rect" style="stroke:black; fill:none; stroke-width:2;" />
                        <text x="210" y="400" class="component-text">TOR</text>

                        <!-- MOTOR -->
                        <path id="p-motor-in-r" d="M150 410 V440" class="wire" />
                        <path id="p-motor-in-y" d="M170 410 V440" class="wire" />
                        <path id="p-motor-in-b" d="M190 410 V440" class="wire" />

                        <circle cx="170" cy="460" r="25" style="fill:none; stroke:black; stroke-width:3;" />
                        <text x="155" y="465" font-weight="bold">M</text>

                    </svg>
                </div>
            </div>
        </div>
    </div>

    <!-- Config Section -->
    <div style="width: 100%; max-width: 1200px;">
        <div style="color: #ccc; font-size: 0.9em; margin-top: 10px;">Tetapan Nombor Terminal (Edit Label):</div>
        <div class="config-panel" id="configPanel">
            <!-- Generated by JS -->
        </div>
    </div>

    <script>
        // STATE
        const state = {
            mcb: false,
            trip: false,
            emergency: false,
            selector: 'OFF', // OFF, MAN, AUTO
            startPressed: false,
            stopPressed: false,
            
            // Coils
            r1: false,
            tdr1: false,
            tdr2: false,
            sc: false,
            tc: false,
            lc: false,

            // Timers
            timer1Val: 0,
            timer2Val: 0,
            timer1Done: false,
            timer2Done: false,

            // Logic
            latchR1: false,
            latchStep1: false,
            latchStep2: false
        };

        // Settings for Pin Labels
        const pinLabels = {
            'lbl_tor_nc': '95-96',
            'lbl_emg': '1-2',
            'lbl_tdr1_no': '8-6',
            'lbl_r1_hold': '13-14',
            'lbl_r1_start': '43-44',
            'lbl_lc_nc': 'LC(21-22)',
            'lbl_tdr2_nc': 'T2(8-5)',
            'lbl_sc_hold': 'SC(13-14)',
            'lbl_tdr2_no': 'T2(8-6)',
            'lbl_sc_nc': 'SC(21-22)',
            'lbl_lc_hold': 'LC(13-14)'
        };

        // INIT
        function init() {
            renderConfig();
            gameLoop();
        }

        function setTab(tab) {
            document.querySelectorAll('.tab-btn').forEach(b => b.classList.remove('active'));
            document.querySelectorAll('.schematic-view').forEach(v => v.classList.remove('active'));
            
            if(tab === 'control') {
                document.querySelectorAll('.tab-btn')[0].classList.add('active');
                document.getElementById('viewControl').classList.add('active');
            } else {
                document.querySelectorAll('.tab-btn')[1].classList.add('active');
                document.getElementById('viewPower').classList.add('active');
            }
        }

        // INPUT HANDLERS
        function toggleMCB() {
            state.mcb = !state.mcb;
            document.getElementById('btnMcb').innerText = state.mcb ? "MCB: ON" : "MCB: OFF";
            document.getElementById('btnMcb').style.backgroundColor = state.mcb ? "#27ae60" : "#7f8c8d";
        }

        function toggleTrip() {
            state.trip = !state.trip;
        }

        function toggleEmg() {
            state.emergency = !state.emergency;
            if(state.emergency) {
                document.getElementById('btnEmg').classList.add('active');
            } else {
                document.getElementById('btnEmg').classList.remove('active');
            }
        }

        function toggleSelector() {
            const knob = document.getElementById('knobSelector');
            if(state.selector === 'OFF') {
                state.selector = 'AUTO';
                knob.className = 'switch-knob sel-auto';
            } else if (state.selector === 'AUTO') {
                state.selector = 'MAN';
                knob.className = 'switch-knob sel-manual';
            } else {
                state.selector = 'OFF';
                knob.className = 'switch-knob sel-off';
            }
            // Reset timers on switch change
            state.timer1Val = 0; state.timer1Done = false;
            state.timer2Val = 0; state.timer2Done = false;
            state.latchR1 = false;
            state.latchStep1 = false;
            state.latchStep2 = false;
        }

        function pressStart() { state.startPressed = true; }
        function releaseStart() { state.startPressed = false; }
        function pressStop() { state.stopPressed = true; }
        function releaseStop() { state.stopPressed = false; }

        function editPin(id, defaultVal) {
            const newVal = prompt("Masukkan nombor terminal:", pinLabels[id]);
            if(newVal) {
                pinLabels[id] = newVal;
                document.getElementById(id).textContent = newVal;
                renderConfig(); // Update input boxes
            }
        }

        function updateLabelFromInput(id, val) {
            pinLabels[id] = val;
            document.getElementById(id).textContent = val;
        }

        function renderConfig() {
            const panel = document.getElementById('configPanel');
            panel.innerHTML = '';
            for(let key in pinLabels) {
                const div = document.createElement('div');
                div.className = 'config-item';
                div.innerHTML = `
                    <label>${key.replace('lbl_', '').toUpperCase()}</label>
                    <input type="text" value="${pinLabels[key]}" onchange="updateLabelFromInput('${key}', this.value)">
                `;
                panel.appendChild(div);
            }
        }

        // SIMULATION LOOP (Logic & Visuals)
        function gameLoop() {
            // 1. RESET COIL STATES
            state.r1 = false;
            state.tdr1 = false;
            state.sc = false;
            state.tc = false;
            state.tdr2 = false;
            state.lc = false;

            // 2. CHECK POWER RAILS
            let liveAvailable = state.mcb && !state.trip && !state.emergency;

            // 3. LOGIC FLOW
            if(liveAvailable) {
                // AUTO BRANCH
                if (state.selector === 'AUTO') {
                    state.tdr1 = true; // Timer 1 energized
                    if(state.timer1Val < 5000) state.timer1Val += 20; 
                    else state.timer1Done = true;

                    if (state.timer1Done || state.latchR1) {
                        state.r1 = true;
                        state.latchR1 = true; 
                    }
                } else {
                    state.timer1Val = 0;
                    state.timer1Done = false;
                    state.latchR1 = false;
                }

                // MANUAL / START BRANCH
                let startSignal = false;
                if (state.selector === 'MAN' && !state.stopPressed && state.startPressed) startSignal = true;
                if (state.r1 && !state.stopPressed) startSignal = true;

                // Holding Logic for SC (Step 1)
                if ((startSignal || state.latchStep1) && !state.stopPressed && !state.latchStep2) {
                    if (!state.lc && !state.timer2Done) {
                        state.sc = true;
                        state.tc = true;
                        state.tdr2 = true;
                        state.latchStep1 = true;
                    }
                } else {
                    if(state.latchStep2) state.latchStep1 = false;
                }

                // Timer 2 Logic (Step 1 -> Step 2 Delay)
                if (state.tdr2) {
                    if (state.timer2Val < 5000) state.timer2Val += 20;
                    else state.timer2Done = true;
                } else {
                    if(!state.latchStep2) {
                        state.timer2Val = 0;
                        state.timer2Done = false;
                    }
                }

                // LC (Step 2) Logic
                let triggerStep2 = state.latchStep1 && state.timer2Done;
                if ((triggerStep2 || state.latchStep2) && !state.stopPressed) {
                    state.sc = false;
                    state.tc = false;
                    state.tdr2 = false; 
                    state.lc = true;
                    state.latchStep2 = true;
                    state.latchStep1 = false; 
                } else {
                    state.latchStep2 = false;
                }
            }

            // 4. UPDATE VISUALS (Wires & Components)
            updateVisuals();

            requestAnimationFrame(gameLoop);
        }

        function updateVisuals() {
            // Wires (Control)
            setWire('w1', state.mcb);
            setWire('w2', state.mcb);
            setWire('w3', state.mcb && !state.trip);
            setWire('w4', state.mcb && !state.trip && !state.emergency);

            let common = state.mcb && !state.trip && !state.emergency;
            
            // Auto Branch
            setWire('w_auto_feed', common);
            let autoSelected = common && state.selector === 'AUTO';
            setWire('w_auto_post', autoSelected);
            setWire('w_tdr1_live', autoSelected);
            
            // R1 Branch
            setWire('w_r1_feed', autoSelected);
            let tdr1Closed = autoSelected && state.timer1Done;
            let r1Hold = common && state.latchR1; 
            setWire('w_r1_mid', tdr1Closed || r1Hold);

            // Manual/Start Branch
            setWire('w_man_feed', common);
            let manSelected = common && state.selector === 'MAN';
            let r1ContactClosed = common && state.r1;
            
            setWire('w_stop_feed', manSelected || r1ContactClosed);
            setWire('w_auto_inj_run', r1ContactClosed);

            let afterStop = (manSelected || r1ContactClosed) && !state.stopPressed;
            setWire('w_start_feed', afterStop);

            let mainBusLive = afterStop && (state.startPressed || state.latchStep1 || state.latchStep2);
            setWire('w_main_bus', mainBusLive);

            // SC/TC/T2 Branch
            setWire('w_lc_nc_feed', mainBusLive);
            setWire('w_tdr2_nc_feed', mainBusLive && !state.lc);
            setWire('w_sc_feed', mainBusLive && !state.lc && !state.timer2Done);
            setWire('w_tc_feed', mainBusLive && !state.lc && !state.timer2Done);
            setWire('w_tdr2_feed', mainBusLive && !state.lc && !state.timer2Done);
            setWire('w_tc_run', mainBusLive && !state.lc && !state.timer2Done);
            setWire('w_tdr2_run', mainBusLive && !state.lc && !state.timer2Done);

            // LC Branch
            setWire('w_lc_path_feed', mainBusLive);
            let tdr2ContactClosed = state.timer2Done; 
            let lcHoldClosed = state.latchStep2;
            setWire('w_lc_mid', mainBusLive && (tdr2ContactClosed || lcHoldClosed));
            setWire('w_lc_final', mainBusLive && (tdr2ContactClosed || lcHoldClosed) && !state.sc);

            // --- POWER CIRCUIT VISUALS ---
            let pIn = state.mcb; // Power into MCB
            let pOut = pIn; // Power out of MCB (if MCB is just a switch here)
            
            setPowerWire('p-mcb-in', true); // Always live from grid
            setPowerWire('p-mcb-out', state.mcb);
            
            // Branch LC (Run)
            setPowerWire('p-lc-in', state.mcb);
            let lcLive = state.mcb && state.lc;
            setPowerWire('p-lc-out', lcLive);

            // Branch SC (Start)
            setPowerWire('p-sc-in', state.mcb);
            let scLive = state.mcb && state.sc;
            setPowerWire('p-at-in', scLive); // Into AutoTrans
            
            // AutoTrans Taps (active if SC is on AND TC is on)
            let tapsLive = scLive && state.tc;
            setPowerWire('p-tap', tapsLive);
            setPowerWire('p-tc', tapsLive); // Power to Star Point

            // Motor Input (Either LC Live OR Taps Live)
            let motorLive = (lcLive || tapsLive) && !state.trip;
            setPowerWire('p-tor-in', lcLive || tapsLive);
            setPowerWire('p-tor-out', motorLive); // Assuming TOR passes unless tripped, but visual 'trip' stops before here usually? 
            // In this logic, if trip, state.mcb is still on? No, toggleTrip sets state.trip.
            // If trip, the control circuit cuts off, so lc/sc becomes false.
            // So motorLive will naturally be false.
            setPowerWire('p-motor-in', motorLive);

            // Highlights for Power Components
            setComponent('sym-mcb-p', state.mcb);
            setComponent('sym-lc-p', state.lc);
            setComponent('sym-sc-p', state.sc);
            setComponent('sym-tc-p', state.tc);

            // Component Highlights (Control)
            setComponent('coil_tdr1', state.tdr1);
            setComponent('coil_r1', state.r1);
            setComponent('coil_sc', state.sc);
            setComponent('coil_tc', state.tc);
            setComponent('coil_tdr2', state.tdr2);
            setComponent('coil_lc', state.lc);

            // Lamps
            toggleLamp('lampEmg', state.emergency);
            toggleLamp('lampTrip', state.trip);
            toggleLamp('lampStep1', state.sc); // Blue
            toggleLamp('lampStep2', state.lc); // Green

            // Motor
            const fan = document.getElementById('motorFan');
            const motorText = document.getElementById('motorText');
            fan.className = 'fan';
            if (state.sc) {
                fan.classList.add('spin-slow');
                motorText.innerText = "Langkah 1 (Reduced)";
            } else if (state.lc) {
                fan.classList.add('spin-fast');
                motorText.innerText = "Langkah 2 (Full)";
            } else {
                motorText.innerText = "OFF";
            }

            // Displays
            document.getElementById('timer1Display').innerText = (state.timer1Val/1000).toFixed(1);
            document.getElementById('timer2Display').innerText = (state.timer2Val/1000).toFixed(1);

            // Switch Visuals
            const visStart = document.getElementById('vis_pb_start');
            if(state.startPressed) {
                 visStart.setAttribute('y2', '10'); visStart.setAttribute('x2', '20');
            } else {
                 visStart.setAttribute('y2', '0'); visStart.setAttribute('x2', '20');
            }

            const visAuto = document.getElementById('vis_sw_auto');
            visAuto.setAttribute('d', state.selector === 'AUTO' ? "M0 0 L20 0" : "M0 0 L20 10");
            const visMan = document.getElementById('vis_sw_man');
            visMan.setAttribute('d', state.selector === 'MAN' ? "M0 0 L20 0" : "M0 0 L20 10");
        }

        function setWire(id, isLive) {
            const el = document.getElementById(id);
            if(el) {
                el.setAttribute('class', isLive ? 'wire live' : 'wire');
            }
        }

        function setPowerWire(baseId, isLive) {
            ['r', 'y', 'b'].forEach(phase => {
                const el = document.getElementById(baseId + '-' + phase);
                if(el) {
                    if(isLive) {
                        el.classList.add('live-' + phase);
                        el.style.opacity = '1';
                    } else {
                        el.classList.remove('live-' + phase);
                        el.style.opacity = '0.3';
                    }
                }
            });
            // Special handling for short/tap
            if(baseId === 'p-tc') { // Star point
                 const el = document.getElementById('p-tc-short');
                 if(el) el.style.opacity = isLive ? '1' : '0.3';
            }
        }

        function setComponent(id, isActive) {
             const el = document.getElementById(id);
             if(el) {
                 if(id.includes('-p')) { // Power components
                    el.style.stroke = isActive ? '#2ecc71' : 'black';
                    el.style.fill = isActive ? 'rgba(46, 204, 113, 0.1)' : 'none';
                 } else {
                    el.setAttribute('class', isActive ? 'symbol-coil active-component' : 'symbol-coil');
                 }
             }
        }

        function toggleLamp(id, on) {
             const el = document.getElementById(id);
             if(on) el.classList.add('on'); else el.classList.remove('on');
        }

        init();

    </script>
</body>
</html>
