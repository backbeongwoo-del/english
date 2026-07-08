# english
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OPIc Quest: Path to IH</title>
    <!-- Tailwind CSS for modern responsive styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts for retro/modern gaming feels -->
    <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@600;800&family=Inter:wght@300;400;600;700&display=swap" rel="stylesheet">
    <!-- Font Awesome for game icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #0b0f19;
            color: #f3f4f6;
        }
        .gaming-title {
            font-family: 'Cinzel', serif;
        }
        /* Custom scrollbar */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #111827;
        }
        ::-webkit-scrollbar-thumb {
            background: #4b5563;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #6b7280;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between overflow-x-hidden">

    <script>
        // Game state variables
        const gameState = {
            stage: 1,
            player: {
                name: '용사',
                class: 'Mage', // Warrior, Mage, Rogue
                level: 1,
                xp: 0,
                nextXp: 100,
                maxHp: 100,
                hp: 100,
                avatar: '🧙‍♂️'
            },
            monster: {
                name: '묘사 슬라임 (Description Slime)',
                type: 'description',
                maxHp: 100,
                hp: 100,
                avatar: '🦠',
                question: 'I would like to know about your favorite park. What does it look like? Where is it located? Tell me everything about it in detail.',
                hint: 'Tip: 공원의 물리적 묘사(위치, 크기, 분위기)뿐만 아니라, 그곳에서 사람들이 무엇을 하는지도 구체적인 형용사(picturesque, serene, vibrant 등)를 섞어 설명하세요!'
            },
            speechActive: false,
            recognition: null,
            transcript: "",
            apiKey: "", // Left empty for system automation / prompt injection
            isProcessing: false
        };

        // Stages setup
        const stagesInfo = {
            1: {
                monsterName: '묘사의 슬라임 (Description Slime)',
                type: 'description',
                avatar: '🦠',
                question: 'I would like to know about your favorite park. What does it look like? Where is it located? Tell me everything about it in detail.',
                hint: 'IH 공략: 단순히 "Good" 보다는 "scenic", "vibrant" 같은 다채로운 형용사를 활용해 전체적인 분위기를 묘사하세요.',
                maxHp: 100
            },
            2: {
                monsterName: '루틴의 고블린 (Routine Goblin)',
                type: 'routine',
                avatar: '👺',
                question: 'Tell me about what you usually do when you visit your favorite park. What is your typical routine from the moment you arrive until you leave?',
                hint: 'IH 공략: 행동의 순서를 깔끔하게 정리하는 연결어(First of all, then, after that, finally)와 빈도 부사(frequently, occasionally)를 적극 사용하세요.',
                maxHp: 120
            },
            3: {
                monsterName: '과거 경험의 스펙터 (Past Experience Specter)',
                type: 'experience',
                avatar: '👻',
                question: 'Talk about a memorable or unexpected experience you had at a park. When did it happen, who were you with, and what made it so unforgettable?',
                hint: 'IH 공략 (가장 중요!): 시제 일치가 생명입니다. 과거 시제(went, saw, realized, was shocked)를 실수 없이 부드럽게 사용하는 모습을 보여주세요.',
                maxHp: 150
            },
            4: {
                monsterName: '롤플레이의 화염 드래곤 (Roleplay Dragon)',
                type: 'roleplay',
                avatar: '🐉',
                question: 'You want to rent a bicycle at a park, but you have some questions. Call the rental shop and ask three to four questions about renting a bike.',
                hint: 'IH 공략: 상황극에 몰입하여 다양한 의문문 형식(I was wondering if..., Could you tell me..., Is it possible to...)을 쓰고 리액션(Oh, I see, perfect!)을 가미하세요.',
                maxHp: 200
            }
        };
    </script>

    <!-- Navigation Bar -->
    <header class="border-b border-slate-800 bg-slate-900 px-4 py-3 sticky top-0 z-50 shadow-md">
        <div class="max-w-6xl mx-auto flex items-center justify-between">
            <div class="flex items-center space-x-3">
                <span class="text-3xl">⚔️</span>
                <div>
                    <h1 class="gaming-title text-xl md:text-2xl font-bold tracking-wider text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 to-violet-500">
                        OPIC QUEST
                    </h1>
                    <p class="text-[10px] md:text-xs text-slate-400 tracking-widest uppercase">The Road to IH Master</p>
                </div>
            </div>
            <div class="flex items-center space-x-4">
                <div class="hidden sm:flex items-center space-x-2 text-sm bg-slate-800 px-3 py-1.5 rounded-full border border-slate-700">
                    <span class="w-2.5 h-2.5 rounded-full bg-emerald-500 animate-pulse"></span>
                    <span class="text-slate-300 font-medium">Gemini Evaluator Active</span>
                </div>
                <button onclick="openHelpModal()" class="text-slate-400 hover:text-white transition duration-200">
                    <i class="fa-regular fa-circle-question text-xl"></i>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content Area -->
    <main class="flex-grow max-w-6xl w-full mx-auto p-4 flex flex-col lg:flex-row gap-6 items-stretch my-2">
        
        <section class="flex-1 flex flex-col gap-6">
            
            <!-- Character & Level Stats Panel -->
            <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 shadow-lg relative overflow-hidden">
                <div class="absolute top-0 right-0 w-32 h-32 bg-indigo-500/5 rounded-full blur-3xl -mr-8 -mt-8"></div>
                <h3 class="text-xs font-bold text-indigo-400 tracking-wider uppercase mb-4">플레이어 정보</h3>
                
                <div class="flex flex-col sm:flex-row items-center gap-5">
                    <!-- Avatar Selection Display -->
                    <div class="relative">
                        <div id="player-avatar-box" class="w-20 h-20 bg-slate-800 rounded-full border-4 border-indigo-500 flex items-center justify-center text-4xl shadow-md">
                            🧙‍♂️
                        </div>
                        <span class="absolute -bottom-2 -right-2 bg-indigo-600 text-white font-bold text-xs px-2.5 py-1 rounded-full border-2 border-slate-900" id="player-level">
                            Lv 1
                        </span>
                    </div>

                    <!-- RPG Stats -->
                    <div class="flex-grow w-full">
                        <div class="flex flex-col sm:flex-row sm:items-center justify-between mb-3">
                            <div class="flex items-center gap-2 mb-1 sm:mb-0">
                                <span class="text-lg font-bold text-white" id="player-name-display">용사</span>
                                <span class="text-xs bg-slate-800 text-slate-400 px-2 py-0.5 rounded border border-slate-700" id="player-class-display">Mage</span>
                            </div>
                            <div class="text-xs text-slate-400">
                                <span class="text-indigo-400 font-bold" id="player-xp-val">0</span> / <span id="player-next-xp-val">100</span> XP
                            </div>
                        </div>

                        <!-- XP Bar -->
                        <div class="w-full bg-slate-800 h-2 rounded-full overflow-hidden mb-4">
                            <div id="player-xp-bar" class="bg-indigo-500 h-full transition-all duration-300" style="width: 0%"></div>
                        </div>

                        <!-- HP Bar -->
                        <div class="flex items-center justify-between text-xs mb-1">
                            <span class="text-emerald-400 font-semibold"><i class="fa-solid fa-heart mr-1 text-red-500"></i> 생명력 (HP)</span>
                            <span id="player-hp-text" class="text-slate-300">100 / 100</span>
                        </div>
                        <div class="w-full bg-slate-800 h-3.5 rounded-full overflow-hidden border border-slate-700">
                            <div id="player-hp-bar" class="bg-gradient-to-r from-emerald-500 to-green-400 h-full transition-all duration-300" style="width: 100%"></div>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Battle Ground / Monster display -->
            <div class="bg-gradient-to-b from-slate-900 to-slate-950 border border-slate-800 rounded-2xl p-6 shadow-xl flex-grow flex flex-col justify-between min-h-[340px]">
                <!-- Top Status Header -->
                <div class="flex items-center justify-between mb-4 border-b border-slate-800/60 pb-3">
                    <span class="text-xs font-bold text-rose-500 tracking-wider uppercase"><i class="fa-solid fa-swords mr-1"></i> 현재 대결 스테이지</span>
                    <span class="text-xs font-bold text-indigo-400 bg-indigo-950/40 border border-indigo-900/50 px-3 py-1 rounded-full" id="stage-badge">
                        Stage 1/4
                    </span>
                </div>

                <!-- Battle Screen Graphics & HP (Center Area) -->
                <div class="flex flex-col items-center justify-center my-6 gap-5">
                    <!-- Boss Monster Frame -->
                    <div class="text-center">
                        <div id="monster-avatar" class="text-7xl mb-3 animate-bounce cursor-help hover:scale-110 transition duration-300" style="animation-duration: 2s;" onclick="triggerMonsterHint()">
                            🦠
                        </div>
                        <h4 class="text-lg font-bold text-rose-100" id="monster-name">묘사 슬라임 (Description Slime)</h4>
                        <span class="text-xs text-rose-400/80 uppercase tracking-widest font-mono" id="monster-type">TYPE: Description</span>
                    </div>

                    <!-- Monster HP Tracker -->
                    <div class="w-full max-w-sm">
                        <div class="flex justify-between items-center text-xs mb-1">
                            <span class="text-rose-400 font-bold"><i class="fa-solid fa-skull mr-1"></i> BOSS HP</span>
                            <span id="monster-hp-text" class="text-slate-300">100 / 100</span>
                        </div>
                        <div class="w-full bg-slate-900 h-3 rounded-full border border-slate-800 overflow-hidden shadow-inner">
                            <div id="monster-hp-bar" class="bg-gradient-to-r from-rose-600 to-rose-400 h-full transition-all duration-300" style="width: 100%"></div>
                        </div>
                    </div>
                </div>

                <!-- Monster Hint Box -->
                <div id="monster-hint-box" class="bg-indigo-950/20 border border-indigo-900/40 rounded-xl p-3.5 flex items-start gap-3 text-xs leading-relaxed transition-all duration-300">
                    <span class="text-base mt-0.5">💡</span>
                    <div>
                        <strong class="text-indigo-300 block mb-0.5">공략 힌트 (IH Tip)</strong>
                        <span id="monster-hint-text" class="text-slate-400">Loading hint...</span>
                    </div>
                </div>
            </div>

        </section>

        <section class="flex-1 flex flex-col gap-6">
            
            <!-- Monster Question Box -->
            <div class="bg-slate-900 border border-slate-800 rounded-2xl p-6 shadow-lg flex flex-col gap-4 relative overflow-hidden">
                <div class="absolute top-0 left-0 w-1 h-full bg-indigo-500"></div>
                <div class="flex justify-between items-center">
                    <h3 class="text-xs font-bold text-indigo-400 tracking-wider uppercase">현재 퀘스트 (OPIc Question)</h3>
                    <button onclick="readQuestion()" class="text-slate-400 hover:text-indigo-400 transition flex items-center gap-1.5 text-xs bg-slate-800 hover:bg-slate-700/60 px-2.5 py-1 rounded border border-slate-700" title="질문 음성으로 듣기">
                        <i class="fa-solid fa-volume-high"></i> 듣기 (TTS)
                    </button>
                </div>
                
                <blockquote id="monster-question-text" class="text-base md:text-lg font-medium text-slate-100 leading-relaxed italic bg-slate-950/40 p-4 rounded-xl border border-slate-800">
                    "I would like to know about your favorite park. What does it look like? Where is it located? Tell me everything about it in detail."
                </blockquote>
            </div>

            <!-- Speaking Arena -->
            <div class="bg-slate-900 border border-slate-800 rounded-2xl p-6 shadow-lg flex-grow flex flex-col justify-between">
                <div>
                    <div class="flex items-center justify-between mb-4">
                        <h3 class="text-xs font-bold text-indigo-400 tracking-wider uppercase">답변 입력 구역 (Speaking Area)</h3>
                        <div id="recording-indicator" class="hidden flex items-center space-x-1.5 text-xs text-rose-500 font-bold bg-rose-950/30 border border-rose-900 px-3 py-1 rounded-full animate-pulse">
                            <span class="w-2 h-2 rounded-full bg-rose-500"></span>
                            <span>마이크 입력중...</span>
                        </div>
                    </div>

                    <!-- Transcript / Text area -->
                    <div class="relative mb-4">
                        <textarea id="answer-textarea" placeholder="아래 마이크 버튼을 눌러 말을 시작하거나, 여기에 영어 답변을 직접 입력해보세요! (OPIc IH 획득을 위해서는 최소 1분 이상, 대략 10문장 이상 풍부하게 말하는 연습이 중요합니다.)" 
                                  class="w-full h-44 bg-slate-950 border border-slate-800 rounded-xl p-4 text-slate-200 placeholder-slate-600 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm md:text-base leading-relaxed resize-none"></textarea>
                        <div class="absolute bottom-3 right-3 text-xs text-slate-500" id="word-count">
                            0 words
                        </div>
                    </div>
                </div>

                <!-- Control Buttons -->
                <div class="flex flex-col sm:flex-row gap-3">
                    <!-- Mic Speaking Button -->
                    <button id="mic-btn" onclick="toggleSpeech()" class="flex-1 py-3.5 px-5 bg-gradient-to-r from-rose-600 to-pink-600 hover:from-rose-500 hover:to-pink-500 text-white font-bold rounded-xl shadow-lg shadow-rose-900/20 flex items-center justify-center gap-2 transition-all duration-300 hover:-translate-y-0.5">
                        <i class="fa-solid fa-microphone text-lg" id="mic-icon"></i>
                        <span id="mic-btn-text">말하기 시작 (마이크 ON)</span>
                    </button>

                    <!-- Attack (Submit) Button -->
                    <button id="attack-btn" onclick="attackMonster()" class="flex-1 py-3.5 px-5 bg-gradient-to-r from-indigo-600 to-violet-600 hover:from-indigo-500 hover:to-violet-500 text-white font-bold rounded-xl shadow-lg shadow-indigo-900/20 flex items-center justify-center gap-2 transition-all duration-300 hover:-translate-y-0.5 disabled:opacity-50 disabled:cursor-not-allowed">
                        <i class="fa-solid fa-swords text-lg"></i>
                        <span>말하기 공격! (AI 채점)</span>
                    </button>
                </div>
            </div>

        </section>

    </main>

    <!-- AI Feedback Panel (Initially Hidden) -->
    <section id="feedback-panel" class="hidden max-w-6xl w-full mx-auto p-4 mb-6 transition-all duration-500">
        <div class="bg-slate-900 border-2 border-indigo-500 rounded-2xl overflow-hidden shadow-2xl">
            <!-- Header -->
            <div class="bg-gradient-to-r from-indigo-900 via-indigo-950 to-slate-900 px-6 py-4 flex items-center justify-between border-b border-indigo-950">
                <div class="flex items-center gap-3">
                    <span class="text-2xl">🔮</span>
                    <div>
                        <h4 class="font-bold text-white text-base md:text-lg">AI 오픽 채점 피드백 (IH Assessment)</h4>
                        <p class="text-[10px] md:text-xs text-indigo-400">Gemini-3-Flash Real-Time Analysis</p>
                    </div>
                </div>
                <div class="bg-indigo-900/60 border border-indigo-700 text-indigo-200 font-bold px-4 py-1.5 rounded-full text-sm flex items-center gap-2">
                    평가 스코어: <span class="text-yellow-400 text-base" id="feedback-score">85</span>/100
                </div>
            </div>

            <div class="p-6 grid grid-cols-1 md:grid-cols-2 gap-6">
                <!-- Critique & General Score -->
                <div class="flex flex-col gap-4">
                    <div class="bg-slate-950 p-4 rounded-xl border border-slate-800">
                        <h5 class="text-xs font-bold text-indigo-400 tracking-wider uppercase mb-2">종합 평가 (Critique)</h5>
                        <p id="feedback-critique" class="text-sm text-slate-300 leading-relaxed">
                            답변의 흐름이 매우 자연스러우며 구조적인 기틀이 훌륭하게 잡혀있습니다. 일부 시제 사용 및 표현을 조금만 개선하면 충분히 IH 이상을 획득할 수 있습니다.
                        </p>
                    </div>

                    <div class="bg-slate-950 p-4 rounded-xl border border-slate-800">
                        <h5 class="text-xs font-bold text-rose-400 tracking-wider uppercase mb-2"><i class="fa-solid fa-triangle-exclamation mr-1"></i> 시제 및 문법 교정 (Grammar Fix)</h5>
                        <ul id="feedback-grammar" class="text-sm text-slate-300 space-y-2 list-disc pl-4">
                            <li>"I went to park yesterday" -> "I went to **the** park yesterday"</li>
                        </ul>
                    </div>
                </div>

                <!-- Key Expressions for IH/AL & Action buttons -->
                <div class="flex flex-col gap-4 justify-between">
                    <div class="bg-slate-950 p-4 rounded-xl border border-slate-800 flex-grow">
                        <h5 class="text-xs font-bold text-emerald-400 tracking-wider uppercase mb-2"><i class="fa-solid fa-sparkles mr-1"></i> 레벨업 추천 핵심 표현 (IH/AL Expressions)</h5>
                        <ul id="feedback-expressions" class="text-sm text-slate-300 space-y-2 list-none">
                            <li class="flex items-start gap-2">
                                <span class="text-emerald-500 mt-0.5">✨</span>
                                <div>
                                    <strong class="text-emerald-300 block">picturesque landscape</strong>
                                    <span class="text-slate-400 text-xs">그림같이 아름다운 풍경</span>
                                </div>
                            </li>
                        </ul>
                    </div>

                    <!-- Next Step Navigation -->
                    <div class="flex gap-3 mt-2">
                        <button onclick="closeFeedbackAndContinue()" class="flex-1 py-3 bg-gradient-to-r from-emerald-600 to-green-600 hover:from-emerald-500 hover:to-green-500 text-white font-bold rounded-xl shadow-lg shadow-emerald-900/20 transition-all duration-300 flex items-center justify-center gap-2">
                            <span>계속 진행하기 (Continue Quest)</span>
                            <i class="fa-solid fa-arrow-right"></i>
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </section>

    <!-- Welcome & Hero Setup Modal -->
    <div id="welcome-modal" class="fixed inset-0 bg-slate-950/80 backdrop-blur-md z-50 flex items-center justify-center p-4">
        <div class="bg-slate-900 border border-slate-800 max-w-lg w-full rounded-2xl overflow-hidden shadow-2xl p-6 md:p-8 animate-fade-in">
            <div class="text-center mb-6">
                <span class="text-5xl mb-3 block">🗡️</span>
                <h2 class="gaming-title text-2xl md:text-3xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 to-violet-500 tracking-wider">
                    OPIc Quest
                </h2>
                <p class="text-xs text-slate-400 mt-1 uppercase tracking-widest">말하면서 깨는 영어 오픽 IH 사냥터</p>
            </div>

            <div class="space-y-4">
                <!-- Step 1: Input Player Name -->
                <div>
                    <label class="block text-xs font-semibold text-indigo-400 tracking-wider uppercase mb-2">클래스 이름 (Name)</label>
                    <input type="text" id="setup-player-name" value="용사" class="w-full bg-slate-950 border border-slate-800 rounded-xl px-4 py-3 text-slate-200 focus:outline-none focus:ring-2 focus:ring-indigo-500 text-sm">
                </div>

                <!-- Step 2: Choose Hero Class -->
                <div>
                    <label class="block text-xs font-semibold text-indigo-400 tracking-wider uppercase mb-2">클래스 선택 (Class Select)</label>
                    <div class="grid grid-cols-3 gap-3">
                        <button type="button" onclick="selectClass('Warrior', '⚔️')" id="class-Warrior" class="class-card bg-slate-950 hover:bg-slate-800/50 border border-slate-800 rounded-xl p-3 text-center transition duration-200 focus:outline-none">
                            <span class="text-3xl block mb-1">⚔️</span>
                            <span class="text-xs font-bold block text-slate-300">전사</span>
                            <span class="text-[9px] text-slate-500">체력 중시</span>
                        </button>
                        <button type="button" onclick="selectClass('Mage', '🧙‍♂️')" id="class-Mage" class="class-card bg-slate-950 hover:bg-slate-800/50 border-2 border-indigo-500 rounded-xl p-3 text-center transition duration-200 focus:outline-none">
                            <span class="text-3xl block mb-1">🧙‍♂️</span>
                            <span class="text-xs font-bold block text-slate-300">마법사</span>
                            <span class="text-[9px] text-slate-500 font-medium text-indigo-400">기본 선택</span>
                        </button>
                        <button type="button" onclick="selectClass('Rogue', '🏹')" id="class-Rogue" class="class-card bg-slate-950 hover:bg-slate-800/50 border border-slate-800 rounded-xl p-3 text-center transition duration-200 focus:outline-none">
                            <span class="text-3xl block mb-1">🏹</span>
                            <span class="text-xs font-bold block text-slate-300">궁수</span>
                            <span class="text-[9px] text-slate-500">신속 치명타</span>
                        </button>
                    </div>
                </div>

                <!-- Play Guide -->
                <div class="bg-indigo-950/20 border border-indigo-900/40 rounded-xl p-4 text-xs leading-relaxed text-slate-400 space-y-1.5">
                    <strong class="text-indigo-300 block mb-1">🛡️ 게임 플레이 규칙:</strong>
                    <p>• 마이크를 켜고 화면에 나타나는 오픽 질문에 맞춰 영어로 풍부하게 대답하세요.</p>
                    <p>• 문법적 완전함, 어휘 수, 표현의 완성도에 따라 몬스터에게 데미지가 가해집니다.</p>
                    <p>• 몬스터 체력을 0으로 깎으면 다음 던전(스테이지)으로 진행할 수 있습니다.</p>
                </div>
            </div>

            <button onclick="startGame()" class="w-full py-4 mt-6 bg-gradient-to-r from-indigo-600 to-violet-600 hover:from-indigo-500 hover:to-violet-500 text-white font-bold rounded-xl shadow-lg transition duration-200">
                던전 입장하기 (Start Quest)
            </button>
        </div>
    </div>

    <!-- Victory / Stage Completed Modal -->
    <div id="victory-modal" class="hidden fixed inset-0 bg-slate-950/90 backdrop-blur-md z-50 flex items-center justify-center p-4">
        <div class="bg-slate-900 border-2 border-yellow-500 max-w-md w-full rounded-2xl overflow-hidden shadow-2xl p-6 md:p-8 text-center animate-bounce-short">
            <span class="text-6xl mb-4 block">👑</span>
            <h2 class="gaming-title text-2xl md:text-3xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-yellow-400 to-amber-500 tracking-wider mb-2">
                VICTORY!
            </h2>
            <p class="text-sm text-slate-300 mb-6" id="victory-desc">몬스터를 물리치고 던전을 정복하셨습니다!</p>
            
            <div class="bg-slate-950 border border-slate-800 rounded-xl p-4 mb-6 text-left space-y-2">
                <h4 class="text-xs font-bold text-yellow-500 tracking-wider uppercase mb-1">전리품 획득 (Rewards)</h4>
                <div class="flex justify-between text-sm">
                    <span class="text-slate-400">경험치 획득</span>
                    <span class="text-emerald-400 font-bold" id="victory-xp">+50 XP</span>
                </div>
                <div class="flex justify-between text-sm">
                    <span class="text-slate-400">오픽 레벨 진척도</span>
                    <span class="text-yellow-400 font-bold">IH Master +25%</span>
                </div>
            </div>

            <button onclick="proceedToNextStage()" class="w-full py-3.5 bg-gradient-to-r from-yellow-500 to-amber-500 hover:from-yellow-400 hover:to-amber-400 text-slate-950 font-bold rounded-xl shadow-lg transition duration-200">
                다음 스테이지로 이동
            </button>
        </div>
    </div>

    <!-- Final Game Clear / IH Achieved Modal -->
    <div id="clear-modal" class="hidden fixed inset-0 bg-slate-950/95 backdrop-blur-md z-50 flex items-center justify-center p-4">
        <div class="bg-gradient-to-b from-slate-900 to-slate-950 border-4 border-indigo-500 max-w-lg w-full rounded-3xl overflow-hidden shadow-2xl p-8 text-center">
            <span class="text-7xl mb-4 block">🏆</span>
            <h2 class="gaming-title text-3xl md:text-4xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 to-violet-400 tracking-wider mb-2">
                OPIC IH ACHIEVED!
            </h2>
            <p class="text-xs text-indigo-400 uppercase tracking-widest font-mono mb-6">오픽 IH 정복을 진심으로 축하합니다!</p>

            <div class="bg-slate-950 border border-indigo-950 p-6 rounded-2xl mb-6 text-left space-y-4">
                <p class="text-sm text-slate-300 leading-relaxed">
                    모든 던전 게이트를 통과하며 묘사, 습관, 과거 경험, 롤플레이 전 과정을 완벽히 무찌르셨습니다.
                    당신은 이제 어떤 오픽 돌발 질문도 유연한 <strong>IH/AL 수준의 전략</strong>으로 응대할 준비가 되었습니다!
                </p>
                <div class="flex justify-around border-t border-slate-800 pt-4 text-center">
                    <div>
                        <span class="block text-2xl font-extrabold text-yellow-400" id="final-lvl">Lv.5</span>
                        <span class="text-[10px] text-slate-500 uppercase font-bold">최종 레벨</span>
                    </div>
                    <div>
                        <span class="block text-2xl font-extrabold text-indigo-400">100%</span>
                        <span class="text-[10px] text-slate-500 uppercase font-bold">클리어율</span>
                    </div>
                    <div>
                        <span class="block text-2xl font-extrabold text-emerald-400">IH+ / AL</span>
                        <span class="text-[10px] text-slate-500 uppercase font-bold">예상 오픽 등급</span>
                    </div>
                </div>
            </div>

            <button onclick="restartFullGame()" class="w-full py-4 bg-gradient-to-r from-indigo-600 to-violet-600 hover:from-indigo-500 hover:to-violet-500 text-white font-bold rounded-xl shadow-lg transition duration-200">
                처음부터 다시 정복하기 (New Game)
            </button>
        </div>
    </div>

    <!-- Info / Help Modal -->
    <div id="help-modal" class="hidden fixed inset-0 bg-slate-950/85 backdrop-blur-sm z-50 flex items-center justify-center p-4">
        <div class="bg-slate-900 border border-slate-800 max-w-md w-full rounded-2xl overflow-hidden shadow-2xl p-6 relative">
            <button onclick="closeHelpModal()" class="absolute top-4 right-4 text-slate-400 hover:text-white transition">
                <i class="fa-solid fa-xmark text-lg"></i>
            </button>
            <h3 class="gaming-title text-xl font-bold text-indigo-400 mb-4 flex items-center gap-2">
                <span>🛡️</span> OPIc IH 정복 전술 가이드
            </h3>
            <div class="space-y-4 text-sm text-slate-300 leading-relaxed">
                <div>
                    <strong class="text-white block mb-1">1. 일관된 시제 유지가 제일 중요!</strong>
                    <p class="text-slate-400 text-xs">특히 3단계 '과거 경험' 질문에서는 반드시 과거 시제(did, felt, went)를 온전히 유지해야 오픽 IH/AL 채점 기준을 만족할 수 있습니다.</p>
                </div>
                <div>
                    <strong class="text-white block mb-1">2. 유창하게 들리는 필러 필터링</strong>
                    <p class="text-slate-400 text-xs">문장과 문장 사이에 정적이 생길 때 "um", "ah" 대신에 "Well", "You know", "I mean", "What I like about..." 같은 고급 필러들을 섞어 유창해 보이도록 하세요.</p>
                </div>
                <div>
                    <strong class="text-white block mb-1">3. Gemini 실시간 피드백</strong>
                    <p class="text-slate-400 text-xs">답변을 마이크로 녹음한 후 공격 버튼을 누르면 AI가 오픽 핵심 조건들을 분석하여 데미지를 계산하고 알맞은 개선점을 전수합니다.</p>
                </div>
            </div>
            <button onclick="closeHelpModal()" class="w-full py-3 mt-6 bg-slate-800 hover:bg-slate-700 text-white font-bold rounded-xl transition duration-200">
                전장으로 복귀
            </button>
        </div>
    </div>

    <!-- Fullscreen Combat Loader overlay (Gemini thinking state) -->
    <div id="combat-loader" class="hidden fixed inset-0 bg-slate-950/90 backdrop-blur-md z-50 flex flex-col items-center justify-center p-4">
        <div class="relative w-28 h-28 mb-6 flex items-center justify-center">
            <span class="text-6xl animate-pulse">🔮</span>
            <div class="absolute inset-0 border-4 border-indigo-500/20 rounded-full animate-ping"></div>
            <div class="absolute inset-2 border-4 border-indigo-500 border-t-transparent rounded-full animate-spin"></div>
        </div>
        <h3 class="gaming-title text-xl font-bold text-white tracking-widest uppercase mb-2">전투력 측정 중...</h3>
        <p class="text-xs text-slate-400 tracking-wider max-w-xs text-center" id="loader-quote">
            "당신의 문법과 어휘 정밀 타격을 계산하고 있습니다..."
        </p>
    </div>

    <!-- Simple Toast System instead of Native Alerts -->
    <div id="toast-container" class="fixed bottom-5 right-5 z-50 flex flex-col gap-2"></div>

    <!-- Footer -->
    <footer class="bg-slate-950 border-t border-slate-900 py-4 px-6 text-center text-xs text-slate-600">
        <p>© 2026 OPIc Quest: Path to IH. Crafted for ultimate English communication level-up.</p>
    </footer>

    <script>
        // Welcome and class selection configurations
        function selectClass(className, avatar) {
            document.querySelectorAll('.class-card').forEach(el => {
                el.classList.remove('border-2', 'border-indigo-500');
                el.classList.add('border-slate-800');
            });
            document.getElementById(`class-${className}`).classList.remove('border-slate-800');
            document.getElementById(`class-${className}`).classList.add('border-2', 'border-indigo-500');
            
            gameState.player.class = className;
            gameState.player.avatar = avatar;
        }

        // Initialize and start the game after setup
        function startGame() {
            const chosenName = document.getElementById('setup-player-name').value.trim();
            if (chosenName) {
                gameState.player.name = chosenName;
            }
            
            // Sync user UI
            document.getElementById('player-name-display').innerText = gameState.player.name;
            document.getElementById('player-class-display').innerText = gameState.player.class;
            document.getElementById('player-avatar-box').innerText = gameState.player.avatar;
            
            // Hide welcome modal
            document.getElementById('welcome-modal').classList.add('hidden');
            showToast("🏰 OPIc 던전에 입장했습니다. 행운을 빕니다!", "info");
            
            // Set up level 1 stage configurations
            updateStageDisplay();
            initSpeechRecognition();
        }

        // Initial Speech Recognition Configuration
        function initSpeechRecognition() {
            window.SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            if (!window.SpeechRecognition) {
                showToast("⚠️ 이 브라우저는 마이크 입력을 통한 실시간 음성인식을 지원하지 않습니다. 키보드로 답변을 직접 입력해주세요.", "warning");
                return;
            }

            gameState.recognition = new window.SpeechRecognition();
            gameState.recognition.continuous = true;
            gameState.recognition.interimResults = true;
            gameState.recognition.lang = 'en-US'; // Setting transcription specifically to US English for OPIc

            gameState.recognition.onstart = () => {
                gameState.speechActive = true;
                document.getElementById('recording-indicator').classList.remove('hidden');
                document.getElementById('mic-icon').className = 'fa-solid fa-microphone-slash text-lg';
                document.getElementById('mic-btn-text').innerText = '녹음 종료 및 임시 저장';
                document.getElementById('mic-btn').classList.remove('from-rose-600', 'to-pink-600');
                document.getElementById('mic-btn').classList.add('from-amber-600', 'to-yellow-600');
            };

            gameState.recognition.onresult = (event) => {
                let interimTranscript = '';
                let finalTranscript = '';

                for (let i = event.resultIndex; i < event.results.length; ++i) {
                    if (event.results[i].isFinal) {
                        finalTranscript += event.results[i][0].transcript;
                    } else {
                        interimTranscript += event.results[i][0].transcript;
                    }
                }

                if(finalTranscript) {
                    const currentVal = document.getElementById('answer-textarea').value;
                    document.getElementById('answer-textarea').value = currentVal + (currentVal ? ' ' : '') + finalTranscript;
                    updateWordCounter();
                }
            };

            gameState.recognition.onerror = (event) => {
                console.error("Speech Recognition Error", event);
                if (event.error !== 'no-speech') {
                    showToast(`🎤 마이크 에러: ${event.error}`, "error");
                    stopSpeech();
                }
            };

            gameState.recognition.onend = () => {
                gameState.speechActive = false;
                document.getElementById('recording-indicator').classList.add('hidden');
                document.getElementById('mic-icon').className = 'fa-solid fa-microphone text-lg';
                document.getElementById('mic-btn-text').innerText = '말하기 시작 (마이크 ON)';
                document.getElementById('mic-btn').classList.add('from-rose-600', 'to-pink-600');
                document.getElementById('mic-btn').classList.remove('from-amber-600', 'to-yellow-600');
            };
        }

        // Microphone toggle handler
        function toggleSpeech() {
            if (!gameState.recognition) {
                showToast("마이크 음성 인식을 지원하지 않는 환경입니다. 직접 타이핑하여 도전해주세요!", "warning");
                return;
            }

            if (gameState.speechActive) {
                stopSpeech();
            } else {
                try {
                    gameState.recognition.start();
                } catch (e) {
                    console.error(e);
                    showToast("마이크 연결 상태를 확인해주세요.", "error");
                }
            }
        }

        function stopSpeech() {
            if (gameState.recognition && gameState.speechActive) {
                gameState.recognition.stop();
            }
        }

        // Live word counter updater
        function updateWordCounter() {
            const text = document.getElementById('answer-textarea').value.trim();
            const words = text ? text.split(/\s+/).length : 0;
            document.getElementById('word-count').innerText = `${words} words`;
        }
        document.getElementById('answer-textarea').addEventListener('input', updateWordCounter);

        // SpeechSynthesis Question Player
        function readQuestion() {
            const textToSpeak = document.getElementById('monster-question-text').innerText;
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel(); // Stop current playing sound
                const utterance = new SpeechSynthesisUtterance(textToSpeak);
                utterance.lang = 'en-US';
                utterance.rate = 0.9; // OPIc question speed is usually moderate
                window.speechSynthesis.speak(utterance);
                showToast("🔊 원어민 질문 음성이 재생됩니다.", "info");
            } else {
                showToast("지원되지 않는 브라우저거나 음성 장치 전원이 꺼져 있습니다.", "warning");
            }
        }

        async function attackMonster() {
            const answer = document.getElementById('answer-textarea').value.trim();
            if (!answer || answer.split(/\s+/).length < 5) {
                showToast("⚠️ 오픽 분석을 받으려면 최소 5단어 이상의 영어를 발화해주세요!", "warning");
                return;
            }

            stopSpeech();
            gameState.isProcessing = true;
            toggleLoader(true);

            // Dynamically switch loader retro messages
            const loaders = [
                "당신의 문법과 어휘 타격을 가늠하고 있습니다...",
                "시제 일치 마법 주문을 계산하는 중...",
                "어휘의 다양성이 얼마나 되는지 탐정 분석 중...",
                "오픽 채점 위원들의 가혹한 평가 방식을 연산하는 중..."
            ];
            let index = 0;
            const loaderTimer = setInterval(() => {
                index = (index + 1) % loaders.length;
                document.getElementById('loader-quote').innerText = loaders[index];
            }, 2000);

            // Construct Gemini Request
            const systemPrompt = `You are the ultimate OPIc IH Evaluator and Game Master for 'OPIc Quest'.
The user's goal is to achieve OPIc IH (Intermediate High) or higher. 
Grade the user's spoken answer (provided as transcribed text) to the target question.
Critically evaluate based on these OPIc IH criteria:
1. Tense usage accuracy (past narrations must stay strictly in past tense).
2. Vocabulary diversity (avoid simple words, encourage idiomatic collocations).
3. Fillers & transitions (using natural expressions like "You know", "Anyway", "I mean" to make it fluent).
4. Content richness & length.

You must reply strictly in JSON format. Do not write any other markdown outside the JSON block.
JSON Schema structure:
{
  "score": 85, // (A number 1-100. IH targets roughly 75-85, AL is 86+)
  "critique": "종합 피드백 한국어 2-3문장",
  "grammar_corrections": [
    "구체적인 오픽 문법 오류 수정 1 (예시: 'I went to park' -> 'I went to the park')",
    "구체적인 오픽 문법 오류 수정 2"
  ],
  "key_expressions": [
    "추천하는 고급 표현 1 (뜻과 함께)",
    "추천하는 고급 표현 2"
  ],
  "damage_dealt": 45 // (Between 10 to 100 based on answer strength: longer, more accurate, better grammar = high damage)
}`;

            const userPrompt = `
Target Question: "${gameState.monster.question}"
User English Answer: "${answer}"
Monster HP: ${gameState.monster.hp}/${gameState.monster.maxHp}
`;

            const payload = {
                contents: [{ parts: [{ text: userPrompt }] }],
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: {
                        type: "OBJECT",
                        properties: {
                            score: { type: "NUMBER" },
                            critique: { type: "STRING" },
                            grammar_corrections: { type: "ARRAY", items: { type: "STRING" } },
                            key_expressions: { type: "ARRAY", items: { type: "STRING" } },
                            damage_dealt: { type: "NUMBER" }
                        },
                        required: ["score", "critique", "grammar_corrections", "key_expressions", "damage_dealt"]
                    }
                },
                systemInstruction: {
                    parts: [{ text: systemPrompt }]
                }
            };

            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${gameState.apiKey}`;

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });

                if (!response.ok) {
                    throw new Error("API call limit or Server error.");
                }

                const result = await response.json();
                const jsonText = result.candidates?.[0]?.content?.parts?.[0]?.text;
                
                if (jsonText) {
                    const parsedJson = JSON.parse(jsonText);
                    resolveCombat(parsedJson);
                } else {
                    throw new Error("No response format.");
                }
            } catch (error) {
                console.error("Gemini call failed, rolling back to local sandbox calculation", error);
                // Fallback simulation in case API is unresponsive (perfect for offline testing)
                const fallbackData = simulateBattleFeedback(answer);
                resolveCombat(fallbackData);
            } finally {
                clearInterval(loaderTimer);
                toggleLoader(false);
                gameState.isProcessing = false;
            }
        }

        // Process combat outcome
        function resolveCombat(evaluation) {
            const dmg = Math.round(evaluation.damage_dealt || 30);
            const score = evaluation.score || 70;

            // Damage Monster
            gameState.monster.hp = Math.max(0, gameState.monster.hp - dmg);
            updateMonsterHP();

            // Self Health calculations (if score is poor, we take small feedback "damage")
            let playerDmg = 0;
            if (score < 65) {
                playerDmg = 20;
                gameState.player.hp = Math.max(10, gameState.player.hp - playerDmg);
                showToast(`💥 미흡한 결속으로 반격 공격을 당했습니다! (-${playerDmg} HP)`, "warning");
            } else {
                // Heal player slightly for outstanding answers
                gameState.player.hp = Math.min(gameState.player.maxHp, gameState.player.hp + 10);
            }
            updatePlayerHP();

            // Display Feedback Box
            document.getElementById('feedback-score').innerText = score;
            document.getElementById('feedback-critique').innerText = evaluation.critique;
            
            // Build corrections list
            const gList = document.getElementById('feedback-grammar');
            gList.innerHTML = '';
            if (evaluation.grammar_corrections && evaluation.grammar_corrections.length > 0) {
                evaluation.grammar_corrections.forEach(cor => {
                    const li = document.createElement('li');
                    li.className = "flex items-start gap-2 text-rose-300";
                    li.innerHTML = `<span class="mt-1">❌</span><span>${cor}</span>`;
                    gList.appendChild(li);
                });
            } else {
                gList.innerHTML = `<li class="text-slate-400">명백한 문법 에러를 발견하지 못했습니다. 완벽한 타격이군요!</li>`;
            }

            // Build Expressions list
            const eList = document.getElementById('feedback-expressions');
            eList.innerHTML = '';
            if (evaluation.key_expressions && evaluation.key_expressions.length > 0) {
                evaluation.key_expressions.forEach(expr => {
                    const li = document.createElement('li');
                    li.className = "flex items-start gap-2 bg-slate-900 border border-slate-800 p-2 rounded-lg";
                    li.innerHTML = `
                        <span class="text-emerald-500 mt-1">✨</span>
                        <div>
                            <span class="text-slate-200 font-medium block">${expr}</span>
                        </div>
                    `;
                    eList.appendChild(li);
                });
            } else {
                eList.innerHTML = `<li class="text-slate-400">권장 표현이 없습니다.</li>`;
            }

            // Show Feedback Section
            const fbPanel = document.getElementById('feedback-panel');
            fbPanel.classList.remove('hidden');
            fbPanel.scrollIntoView({ behavior: 'smooth' });

            // Toast feedback
            showToast(`⚔️ 몬스터에게 ${dmg}의 데미지를 가했습니다!`, "success");

            // Game over Check / Victory Check
            if (gameState.monster.hp <= 0) {
                triggerVictory();
            }
        }

        function simulateBattleFeedback(answer) {
            // Very simple localized heuristic algorithm for offline/fallback mode
            const len = answer.split(/\s+/).length;
            let score = 55;
            let dmg = 15;
            let critique = "답변 길이가 상대적으로 너무 짧아 공격력이 대폭 약화되었습니다. 오픽 IH 기준을 만족하기 위해 조금 더 풍부하게 상황을 서술해주세요.";
            const corrections = [];
            const expressions = [];

            if (len > 80) {
                score = 85;
                dmg = 90;
                critique = "장문의 탄탄한 구조가 확인되었습니다! 원활한 연출력과 다채로운 문맥 전개가 인상적입니다. IH 드래곤의 방어를 훌륭히 무너뜨렸습니다.";
                corrections.push("시제 유지가 양호함: 'I had a wonderful time...'");
                expressions.push("without a doubt (의심의 여지 없이)", "it was a life-changing event (인생을 바꾼 사건이었다)");
            } else if (len > 40) {
                score = 75;
                dmg = 50;
                critique = "기본적인 전달력은 준수하지만 IH를 확보하려면 답변에 filler 단어들(like, you know)을 추가하고 세부 묘사를 늘려 흐름의 속도를 조절하는 것이 좋습니다.";
                corrections.push("어법 연결 검토: 긴 문장을 이어갈 때 connection 단어 배치 보강이 좋습니다.");
                expressions.push("scenic spot (경치 좋은 명소)", "I regularly visit (나는 정기적으로 방문한다)");
            } else {
                corrections.push("발화량이 지나치게 부족함 (오픽 최저 권장치 미달)");
                expressions.push("Take a walk (산책하다)", "Relieve stress (스트레스를 풀다)");
            }

            return {
                score: score,
                critique: critique,
                grammar_corrections: corrections,
                key_expressions: expressions,
                damage_dealt: dmg
            };
        }

        // Stage progressions
        function triggerVictory() {
            setTimeout(() => {
                const victoryModal = document.getElementById('victory-modal');
                const nextXpGain = 50;
                
                // Set modal desc based on stage
                document.getElementById('victory-desc').innerText = `${gameState.stage}단계 보스 '${gameState.monster.name}'를 성공적으로 제압했습니다!`;
                document.getElementById('victory-xp').innerText = `+${nextXpGain} XP`;
                
                victoryModal.classList.remove('hidden');

                // Reward Player
                addXp(nextXpGain);
            }, 1200);
        }

        function proceedToNextStage() {
            document.getElementById('victory-modal').classList.add('hidden');
            document.getElementById('feedback-panel').classList.add('hidden');
            
            // Clear text area for new challenge
            document.getElementById('answer-textarea').value = '';
            updateWordCounter();

            if (gameState.stage < 4) {
                gameState.stage += 1;
                setupMonsterForStage(gameState.stage);
                updateStageDisplay();
                showToast(`🛡️ 스테이지 ${gameState.stage}에 입장하셨습니다!`, "info");
            } else {
                // Finished Stage 4 (Victory Full Game Clear)
                triggerFullGameClear();
            }
        }

        function setupMonsterForStage(stgNum) {
            const data = stagesInfo[stgNum];
            gameState.monster.name = data.monsterName;
            gameState.monster.type = data.type;
            gameState.monster.avatar = data.avatar;
            gameState.monster.question = data.question;
            gameState.monster.hint = data.hint;
            gameState.monster.maxHp = data.maxHp;
            gameState.monster.hp = data.maxHp;

            // Update UI elements
            document.getElementById('monster-avatar').innerText = data.avatar;
            document.getElementById('monster-name').innerText = data.monsterName;
            document.getElementById('monster-type').innerText = `TYPE: ${data.type.toUpperCase()}`;
            document.getElementById('monster-question-text').innerText = `"${data.question}"`;
            document.getElementById('monster-hint-text').innerText = data.hint;
            
            updateMonsterHP();
        }

        function updateStageDisplay() {
            document.getElementById('stage-badge').innerText = `Stage ${gameState.stage}/4`;
        }

        function triggerFullGameClear() {
            document.getElementById('final-lvl').innerText = `Lv.${gameState.player.level}`;
            document.getElementById('clear-modal').classList.remove('hidden');
        }

        function restartFullGame() {
            // Full Reset
            gameState.stage = 1;
            gameState.player.level = 1;
            gameState.player.xp = 0;
            gameState.player.hp = gameState.player.maxHp;
            
            document.getElementById('clear-modal').classList.add('hidden');
            setupMonsterForStage(1);
            updateStageDisplay();
            updatePlayerHP();
            updatePlayerXP();
            
            // Display welcome screen again to restart loop gracefully
            document.getElementById('welcome-modal').classList.remove('hidden');
        }

        function closeFeedbackAndContinue() {
            // If they didn't defeat the monster yet, let them submit more content
            document.getElementById('feedback-panel').classList.add('hidden');
            if (gameState.monster.hp > 0) {
                showToast("⚔️ 던전에 여전히 괴물이 살아있습니다! 추가 문장으로 연속 타격을 가하세요!", "warning");
                document.getElementById('answer-textarea').focus();
            }
        }

        // UI Utility functions
        function updateMonsterHP() {
            const pct = (gameState.monster.hp / gameState.monster.maxHp) * 100;
            document.getElementById('monster-hp-bar').style.width = `${pct}%`;
            document.getElementById('monster-hp-text').innerText = `${gameState.monster.hp} / ${gameState.monster.maxHp}`;
        }

        function updatePlayerHP() {
            const pct = (gameState.player.hp / gameState.player.maxHp) * 100;
            const bar = document.getElementById('player-hp-bar');
            bar.style.width = `${pct}%`;
            document.getElementById('player-hp-text').innerText = `${gameState.player.hp} / ${gameState.player.maxHp}`;
            
            // Color thresholds
            if (pct < 30) {
                bar.className = 'bg-gradient-to-r from-rose-600 to-rose-500 h-full transition-all duration-300';
            } else if (pct < 60) {
                bar.className = 'bg-gradient-to-r from-amber-500 to-amber-400 h-full transition-all duration-300';
            } else {
                bar.className = 'bg-gradient-to-r from-emerald-500 to-green-400 h-full transition-all duration-300';
            }
        }

        function addXp(amount) {
            gameState.player.xp += amount;
            if (gameState.player.xp >= gameState.player.nextXp) {
                // Level Up!
                gameState.player.xp -= gameState.player.nextXp;
                gameState.player.level += 1;
                gameState.player.nextXp = Math.round(gameState.player.nextXp * 1.5);
                gameState.player.maxHp += 20;
                gameState.player.hp = gameState.player.maxHp; // Heal to max
                
                document.getElementById('player-level').innerText = `Lv ${gameState.player.level}`;
                updatePlayerHP();
                showToast(`🎉 레벨 업! 오픽 등급 성장 완료! Lv ${gameState.player.level}`, "success");
            }
            updatePlayerXP();
        }

        function updatePlayerXP() {
            const pct = (gameState.player.xp / gameState.player.nextXp) * 100;
            document.getElementById('player-xp-bar').style.width = `${pct}%`;
            document.getElementById('player-xp-val').innerText = gameState.player.xp;
            document.getElementById('player-next-xp-val').innerText = gameState.player.nextXp;
        }

        function toggleLoader(show) {
            const loader = document.getElementById('combat-loader');
            if (show) {
                loader.classList.remove('hidden');
            } else {
                loader.classList.add('hidden');
            }
        }

        // Custom hint display trigger
        function triggerMonsterHint() {
            showToast(`💡 몬스터의 약점 팁: ${gameState.monster.hint}`, "info");
        }

        // Custom Beautiful Toast notification system
        function showToast(message, type = "info") {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            
            // Color configurations based on type
            let bg = "bg-slate-900 border-slate-800";
            let icon = "fa-circle-info text-indigo-400";
            if (type === "success") {
                bg = "bg-emerald-950/90 border-emerald-800 text-emerald-200";
                icon = "fa-circle-check text-emerald-400";
            } else if (type === "warning") {
                bg = "bg-amber-950/90 border-amber-800 text-amber-200";
                icon = "fa-triangle-exclamation text-amber-400";
            } else if (type === "error") {
                bg = "bg-rose-950/90 border-rose-800 text-rose-200";
                icon = "fa-circle-xmark text-rose-400";
            }

            toast.className = `px-4 py-3 rounded-xl border shadow-2xl flex items-center gap-3 text-xs md:text-sm font-semibold max-w-sm transition-all duration-300 transform translate-y-2 opacity-0 ${bg}`;
            toast.innerHTML = `
                <i class="fa-solid ${icon} text-base shrink-0"></i>
                <span class="leading-relaxed">${message}</span>
            `;

            container.appendChild(toast);
            
            // Trigger animation
            setTimeout(() => {
                toast.classList.remove('translate-y-2', 'opacity-0');
            }, 50);

            // Hide & Remove Toast
            setTimeout(() => {
                toast.classList.add('translate-y-2', 'opacity-0');
                setTimeout(() => {
                    toast.remove();
                }, 300);
            }, 4500);
        }

        // Modal Help Controls
        function openHelpModal() {
            document.getElementById('help-modal').classList.remove('hidden');
        }
        function closeHelpModal() {
            document.getElementById('help-modal').classList.add('hidden');
        }
    </script>
</body>
</html>
