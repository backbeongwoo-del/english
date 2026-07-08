#english
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>OPIc Quest: Path to IH (Padlet Edition)</title>
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
        .padlet-column {
            background-color: #111827;
            border: 1px border #1f2937;
            border-radius: 1rem;
        }
        .padlet-card {
            background-color: #1f2937;
            border: 1px solid #374151;
            transition: all 0.2s ease-in-out;
        }
        .padlet-card:hover {
            border-color: #6366f1;
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(99, 102, 241, 0.25);
        }
        .active-padlet-card {
            border-color: #818cf8 !important;
            background-color: #312e81/40 !important;
            box-shadow: 0 0 15px rgba(99, 102, 241, 0.4);
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between overflow-x-hidden">

    <script>
        // Game state variables
        const gameState = {
            activeQuestionId: "desc-1",
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
            apiKey: "", 
            isProcessing: false
        };

        // Complete 12 Questions Categorized like Padlet columns
        const questionsDb = {
            "desc-1": {
                id: "desc-1",
                category: "description",
                title: "공원 묘사 (Favorite Park)",
                monsterName: "묘사 슬라임 (Description Slime)",
                avatar: "🦠",
                maxHp: 100,
                question: "I would like to know about your favorite park. What does it look like? Where is it located? Tell me everything about it in detail.",
                hint: "IH 공략: 단순히 'Good' 보다는 'scenic', 'vibrant', 'picturesque' 같은 다채로운 형용사로 전체적인 분위기를 묘사하세요."
            },
            "desc-2": {
                id: "desc-2",
                category: "description",
                title: "거주지/이웃 묘사 (Your Home & Neighborhood)",
                monsterName: "집 지키는 가고일 (Home Gargoyle)",
                avatar: "🦇",
                maxHp: 100,
                question: "Where do you live? Describe your home in detail. What is your favorite room and why do you like it?",
                hint: "IH 공략: 방의 구조뿐만 아니라 창밖 풍경, 내가 느끼는 정서(cozy, peaceful, spacious)에 맞춰 묘사 영역을 넓히세요."
            },
            "desc-3": {
                id: "desc-3",
                category: "description",
                title: "음악/뮤지션 선호 (Music Preferences)",
                monsterName: "소리굽쇠 사이렌 (Siren of Sound)",
                avatar: "🧜‍♀️",
                maxHp: 110,
                question: "What kind of music do you like? Who is your favorite singer or composer? Tell me about them and why you enjoy their music.",
                hint: "IH 공략: 장르의 전형적인 리듬감이나 보컬의 음색(melodious, soothing, energetic)을 풍성한 단어로 소개하세요."
            },
            "routine-1": {
                id: "routine-1",
                category: "routine",
                title: "공원 방문 루틴 (Park Routine)",
                monsterName: "루틴의 고블린 (Routine Goblin)",
                avatar: "👺",
                maxHp: 120,
                question: "Tell me about what you usually do when you visit your favorite park. What is your typical routine from the moment you arrive until you leave?",
                hint: "IH 공략: 시간 순서를 자연스럽게 정리하는 연결어(First of all, then, after that, finally)와 빈도 부사(frequently, occasionally)를 적극 활용하세요."
            },
            "routine-2": {
                id: "routine-2",
                category: "routine",
                title: "주말 일상 활동 (Weekend Habits)",
                monsterName: "나태의 슬로스 (Weekend Sloth)",
                avatar: "🦥",
                maxHp: 120,
                question: "What do you typically do on weekends? Walk me through your typical weekend activities from Saturday morning to Sunday evening.",
                hint: "IH 공략: 'I study, I eat'처럼 단순 나열하기보다, 'To blow off some steam, I usually...' 같은 관용적인 목적/이유 표현을 추가하세요."
            },
            "routine-3": {
                id: "routine-3",
                category: "routine",
                title: "재활용 습관 및 과정 (Recycling Routine)",
                monsterName: "분리수거 골렘 (Recycling Golem)",
                avatar: "🤖",
                maxHp: 130,
                question: "How do people in your country recycle? Explain the step-by-step process of sorting trash and garbage in your neighborhood.",
                hint: "IH 공략: 분리배출 항목(paper, plastic, glass)과 배출 요령(rinse thoroughly, flatten cardboard) 같은 전문성 있는 일상 어휘를 사용하세요."
            },
            "experience-1": {
                id: "experience-1",
                category: "experience",
                title: "공원 특별한 경험 (Unforgettable Memory at Park)",
                monsterName: "과거 경험의 스펙터 (Past Specter)",
                avatar: "👻",
                maxHp: 150,
                question: "Talk about a memorable or unexpected experience you had at a park. When did it happen, who were you with, and what made it so unforgettable?",
                hint: "IH 공략: 과거 경험은 시제 일치가 생명입니다! 모든 행동과 느낌을 부드럽게 과거 동사(went, saw, realized, was shocked)로 풀어내야 감점이 없습니다."
            },
            "experience-2": {
                id: "experience-2",
                category: "experience",
                title: "이전 집 vs 현재 집 비교 (Housing Comparison)",
                monsterName: "시공간의 웜 (Time-Space Worm)",
                avatar: "🐛",
                maxHp: 160,
                question: "Compare the home you lived in as a child with the home you live in now. What are the key differences between those two places?",
                hint: "IH 공략: 과거와 현재의 비교이므로 대조용 연결어(In contrast, on the other hand)와 비교급 패턴(much larger than, more convenient than)을 듬뿍 써야 합니다."
            },
            "experience-3": {
                id: "experience-3",
                category: "experience",
                title: "잊지 못할 국내/해외 여행 (Memorable Travel)",
                monsterName: "방랑의 미노타우르스 (Wandering Bull)",
                avatar: "🐂",
                maxHp: 160,
                question: "Describe a trip you took that was particularly memorable. Where did you go, what happened, and why did it stand out in your mind?",
                hint: "IH 공략: 사건의 기-승-전-결 구도를 세우고 느꼈던 생생한 감정 묘사(thrilled, exhausted, deeply moved)를 포함하세요."
            },
            "roleplay-1": {
                id: "roleplay-1",
                category: "roleplay",
                title: "[롤플레이] 자전거 대여 (Renting a Bicycle)",
                monsterName: "상황극의 화염 드래곤 (Roleplay Dragon)",
                avatar: "🐉",
                maxHp: 180,
                question: "You want to rent a bicycle at a park, but you have some questions. Call the rental shop and ask three to four questions about renting a bike.",
                hint: "IH 공략: 공손하고 정중한 통화 톤앤매너를 깔끔하게 살리세요 (I was wondering if I could..., Could you tell me if...). 자연스러운 리액션도 핵심입니다."
            },
            "roleplay-2": {
                id: "roleplay-2",
                category: "roleplay",
                title: "[롤플레이] 티켓 예매 돌발 상황 해결 (Booking Issue)",
                monsterName: "혼돈의 트롤 (Chaos Troll)",
                avatar: "👹",
                maxHp: 190,
                question: "You have booked tickets for a concert, but you cannot go due to an urgent situation. Call your friend, explain the problem, and suggest two alternatives.",
                hint: "IH 공략: 문제 발생에 따른 리얼한 아쉬움(Something urgent came up, I'm so sorry)과 대한 제시(Why don't we..., How about I give you...?)를 능숙하게 연출하세요."
            },
            "roleplay-3": {
                id: "roleplay-3",
                category: "roleplay",
                title: "[롤플레이] 여행사 문의 (Travel Agency Inquiry)",
                monsterName: "계약의 네크로맨서 (Agency Lich)",
                avatar: "💀",
                maxHp: 200,
                question: "You are planning a trip and want to book it through a travel agency. Call the travel agent and ask three or four detailed questions about the travel packages.",
                hint: "IH 공략: 구체적인 질문사항(price, itinerary, cancellation policy)을 전개하며 비즈니스 전화 통화 패턴을 깔끔히 유지하세요."
            }
        };
    </script>

    <!-- Navigation Bar -->
    <header class="border-b border-slate-800 bg-slate-900 px-4 py-3 sticky top-0 z-50 shadow-md">
        <div class="max-w-7xl mx-auto flex flex-col md:flex-row items-center justify-between gap-3">
            <div class="flex items-center space-x-3">
                <span class="text-3xl">⚔️</span>
                <div>
                    <h1 class="gaming-title text-xl md:text-2xl font-bold tracking-wider text-transparent bg-clip-text bg-gradient-to-r from-indigo-400 to-violet-500">
                        OPIC QUEST
                    </h1>
                    <p class="text-[10px] md:text-xs text-slate-400 tracking-widest uppercase">The Road to IH Master (Padlet Edition)</p>
                </div>
            </div>

            <!-- Gemini API Key Dynamic Input -->
            <div class="flex flex-wrap items-center gap-2 w-full md:w-auto justify-end">
                <div class="relative w-full max-w-xs sm:w-64">
                    <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-500">
                        <i class="fa-solid fa-key text-xs"></i>
                    </span>
                    <input type="password" id="api-key-input" onchange="updateApiKey(this.value)" placeholder="Gemini API Key를 입력하세요 (선택)" 
                           class="w-full pl-9 pr-3 py-1.5 bg-slate-950 border border-slate-800 rounded-lg text-xs text-slate-300 focus:outline-none focus:ring-1 focus:ring-indigo-500">
                </div>
                <div class="hidden sm:flex items-center space-x-2 text-xs bg-slate-800 px-3 py-1.5 rounded-full border border-slate-700 shrink-0">
                    <span class="w-2 h-2 rounded-full bg-emerald-500 animate-pulse"></span>
                    <span class="text-slate-300 font-medium">Auto-Evaluation Active</span>
                </div>
                <button onclick="openHelpModal()" class="text-slate-400 hover:text-white transition duration-200 p-1 shrink-0">
                    <i class="fa-regular fa-circle-question text-xl"></i>
                </button>
            </div>
        </div>
    </header>

    <!-- 1. Padlet Quest Board Section (Immediate Selection Board) -->
    <section class="max-w-7xl w-full mx-auto p-4 mt-2">
        <div class="bg-slate-900/60 border border-slate-800 rounded-3xl p-5 md:p-6 shadow-xl">
            <div class="flex flex-col md:flex-row md:items-center justify-between mb-5 gap-3">
                <div>
                    <h2 class="gaming-title text-lg md:text-xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-indigo-300 to-violet-400">
                        📋 OPIc QUEST BOARD (패들릿 스타일 질문 리스트)
                    </h2>
                    <p class="text-xs text-slate-400 mt-0.5">원하는 오픽 퀘스트 카드를 클릭하면, 곧바로 아래 전장으로 질문이 소환됩니다!</p>
                </div>
                <span class="text-xs font-mono text-indigo-400 bg-indigo-950/50 border border-indigo-900/60 px-3 py-1.5 rounded-full shrink-0 self-start md:self-auto">
                    <i class="fa-solid fa-square-poll-horizontal mr-1"></i> 총 12개 기출 퀘스트 대기 중
                </span>
            </div>

            <!-- Multi-Column Grid (Padlet Column Emulation) -->
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4">
                
                <!-- Col 1: Description -->
                <div class="padlet-column p-4 flex flex-col gap-3">
                    <div class="flex items-center justify-between border-b border-slate-800 pb-2 mb-1">
                        <span class="text-xs font-bold text-sky-400 tracking-wider uppercase"><i class="fa-solid fa-image mr-1"></i> 1. 인물/사물/공간 묘사</span>
                        <span class="text-[10px] bg-sky-950/50 text-sky-400 px-2 py-0.5 rounded-full border border-sky-900/40">Description</span>
                    </div>
                    <div class="flex flex-col gap-2.5" id="col-description">
                        <!-- Questions injected here dynamically -->
                    </div>
                </div>

                <!-- Col 2: Routine -->
                <div class="padlet-column p-4 flex flex-col gap-3">
                    <div class="flex items-center justify-between border-b border-slate-800 pb-2 mb-1">
                        <span class="text-xs font-bold text-amber-400 tracking-wider uppercase"><i class="fa-solid fa-rotate mr-1"></i> 2. 루틴 & 반복 일상</span>
                        <span class="text-[10px] bg-amber-950/50 text-amber-400 px-2 py-0.5 rounded-full border border-amber-900/40">Routine & Habits</span>
                    </div>
                    <div class="flex flex-col gap-2.5" id="col-routine">
                        <!-- Questions injected here dynamically -->
                    </div>
                </div>

                <!-- Col 3: Experience -->
                <div class="padlet-column p-4 flex flex-col gap-3">
                    <div class="flex items-center justify-between border-b border-slate-800 pb-2 mb-1">
                        <span class="text-xs font-bold text-rose-400 tracking-wider uppercase"><i class="fa-solid fa-clock-rotate-left mr-1"></i> 3. 특별한 과거 경험</span>
                        <span class="text-[10px] bg-rose-950/50 text-rose-400 px-2 py-0.5 rounded-full border border-rose-900/40">Experience</span>
                    </div>
                    <div class="flex flex-col gap-2.5" id="col-experience">
                        <!-- Questions injected here dynamically -->
                    </div>
                </div>

                <!-- Col 4: Roleplay -->
                <div class="padlet-column p-4 flex flex-col gap-3">
                    <div class="flex items-center justify-between border-b border-slate-800 pb-2 mb-1">
                        <span class="text-xs font-bold text-violet-400 tracking-wider uppercase"><i class="fa-solid fa-masks-theater mr-1"></i> 4. 롤플레이 & 돌발</span>
                        <span class="text-[10px] bg-violet-950/50 text-violet-400 px-2 py-0.5 rounded-full border border-violet-900/40">Roleplay & Crisis</span>
                    </div>
                    <div class="flex flex-col gap-2.5" id="col-roleplay">
                        <!-- Questions injected here dynamically -->
                    </div>
                </div>

            </div>
        </div>
    </section>

    <!-- Main Content Area -->
    <main id="battle-field" class="flex-grow max-w-7xl w-full mx-auto p-4 flex flex-col lg:flex-row gap-6 items-stretch my-2">
        
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
                    <span class="text-xs font-bold text-rose-500 tracking-wider uppercase"><i class="fa-solid fa-swords mr-1"></i> 현재 대결 몬스터</span>
                    <span class="text-xs font-bold text-indigo-400 bg-indigo-950/40 border border-indigo-900/50 px-3 py-1 rounded-full" id="stage-badge">
                        Category: Description
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
                            <span class="text-rose-400 font-bold"><i class="fa-solid fa-skull mr-1"></i> MONSTER HP</span>
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
                    <h3 class="text-xs font-bold text-indigo-400 tracking-wider uppercase">현재 오픽 질문 (Question Target)</h3>
                    <button onclick="readQuestion()" class="text-slate-400 hover:text-indigo-400 transition flex items-center gap-1.5 text-xs bg-slate-800 hover:bg-slate-700/60 px-2.5 py-1 rounded border border-slate-700" title="질문 음성으로 듣기">
                        <i class="fa-solid fa-volume-high"></i> 원어민 TTS 듣기
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
                        <textarea id="answer-textarea" placeholder="아래 마이크 버튼을 눌러 말하기 시작하거나, 영어 답변을 직접 타이핑해보세요! (OPIc IH 고득점을 위해서는 10문장 이상 꼼꼼하고 풍부하게 발화하는 연습을 추천합니다!)" 
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
    <section id="feedback-panel" class="hidden max-w-7xl w-full mx-auto p-4 mb-6 transition-all duration-500">
        <div class="bg-slate-900 border-2 border-indigo-500 rounded-2xl overflow-hidden shadow-2xl">
            <!-- Header -->
            <div class="bg-gradient-to-r from-indigo-900 via-indigo-950 to-slate-900 px-6 py-4 flex items-center justify-between border-b border-indigo-950">
                <div class="flex items-center gap-3">
                    <span class="text-2xl">🔮</span>
                    <div>
                        <h4 class="font-bold text-white text-base md:text-lg">AI 오픽 채점 피드백 (IH Assessment)</h4>
                        <p class="text-[10px] md:text-xs text-indigo-400">Gemini Real-Time Analysis</p>
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
                        <ul id="feedback-expressions" class="text-sm text-slate-300 space-y-2 list-none font-sans">
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
                            <span>계속 진행하기 (Continue Hunt)</span>
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
                <p class="text-xs text-slate-400 mt-1 uppercase tracking-widest">말하면서 공부하는 패들릿 오픽 IH 정복지</p>
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
                            <span class="text-[9px] text-slate-500">체력 특화</span>
                        </button>
                        <button type="button" onclick="selectClass('Mage', '🧙‍♂️')" id="class-Mage" class="class-card bg-slate-950 hover:bg-slate-800/50 border-2 border-indigo-500 rounded-xl p-3 text-center transition duration-200 focus:outline-none">
                            <span class="text-3xl block mb-1">🧙‍♂️</span>
                            <span class="text-xs font-bold block text-slate-300">마법사</span>
                            <span class="text-[9px] text-slate-500 font-medium text-indigo-400">마력 특화</span>
                        </button>
                        <button type="button" onclick="selectClass('Rogue', '🏹')" id="class-Rogue" class="class-card bg-slate-950 hover:bg-slate-800/50 border border-slate-800 rounded-xl p-3 text-center transition duration-200 focus:outline-none">
                            <span class="text-3xl block mb-1">🏹</span>
                            <span class="text-xs font-bold block text-slate-300">궁수</span>
                            <span class="text-[9px] text-slate-500">순발 특화</span>
                        </button>
                    </div>
                </div>

                <!-- Play Guide -->
                <div class="bg-indigo-950/20 border border-indigo-900/40 rounded-xl p-4 text-xs leading-relaxed text-slate-400 space-y-1.5">
                    <strong class="text-indigo-300 block mb-1">🛡️ 패들릿 모드 게임 플레이 규칙:</strong>
                    <p>• 상단의 **질문 카드** 중 풀고 싶은 문제를 클릭하면 언제든 곧바로 문제에 도전할 수 있습니다.</p>
                    <p>• 마이크를 켜서 영어로 유창하게 답변을 전송하여 몬스터를 격퇴하세요.</p>
                    <p>• 답변의 유창성, 문법 완성도, 필수 표현 탑재 여부에 따라 몬스터에게 강한 타격을 가할 수 있습니다.</p>
                </div>
            </div>

            <button onclick="startGame()" class="w-full py-4 mt-6 bg-gradient-to-r from-indigo-600 to-violet-600 hover:from-indigo-500 hover:to-violet-500 text-white font-bold rounded-xl shadow-lg transition duration-200">
                오픽 패들릿 사냥터 입장하기
            </button>
        </div>
    </div>

    <!-- Victory / Stage Completed Modal -->
    <div id="victory-modal" class="hidden fixed inset-0 bg-slate-950/90 backdrop-blur-md z-50 flex items-center justify-center p-4">
        <div class="bg-slate-900 border-2 border-yellow-500 max-w-md w-full rounded-2xl overflow-hidden shadow-2xl p-6 md:p-8 text-center animate-bounce-short">
            <span class="text-6xl mb-4 block">👑</span>
            <h2 class="gaming-title text-2xl md:text-3xl font-extrabold text-transparent bg-clip-text bg-gradient-to-r from-yellow-400 to-amber-500 tracking-wider mb-2">
                QUEST CLEARED!
            </h2>
            <p class="text-sm text-slate-300 mb-6" id="victory-desc">성공적으로 몬스터를 쓰러뜨리고 해당 오픽 문항을 마스터하셨습니다!</p>
            
            <div class="bg-slate-950 border border-slate-800 rounded-xl p-4 mb-6 text-left space-y-2">
                <h4 class="text-xs font-bold text-yellow-500 tracking-wider uppercase mb-1">전리품 획득 (Rewards)</h4>
                <div class="flex justify-between text-sm">
                    <span class="text-slate-400">경험치 획득</span>
                    <span class="text-emerald-400 font-bold" id="victory-xp">+50 XP</span>
                </div>
                <div class="flex justify-between text-sm">
                    <span class="text-slate-400">오픽 등급 완수율</span>
                    <span class="text-yellow-400 font-bold">IH Master +35%</span>
                </div>
            </div>

            <button onclick="closeVictoryModal()" class="w-full py-3.5 bg-gradient-to-r from-yellow-500 to-amber-500 hover:from-yellow-400 hover:to-amber-400 text-slate-950 font-bold rounded-xl shadow-lg transition duration-200">
                계속 다른 문항 도전하기
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
        <h3 class="gaming-title text-xl font-bold text-white tracking-widest uppercase mb-2">답변 평가하는 중...</h3>
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
        // Update API Key directly from header
        function updateApiKey(val) {
            gameState.apiKey = val.trim();
            if (gameState.apiKey) {
                showToast("🔑 Gemini API Key가 성공적으로 업데이트되었습니다. 실제 AI 채점 기능을 진행합니다!", "success");
            } else {
                showToast("🔑 API Key가 제거되었습니다. 시뮬레이션 엔진으로 자동 변경됩니다.", "info");
            }
        }

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
            
            // Render Padlet Questions
            renderPadletBoard();
            
            // Select default first question
            selectQuestion("desc-1");
            initSpeechRecognition();
        }

        // Render Padlet column-style interactive cards
        function renderPadletBoard() {
            const columns = {
                "description": document.getElementById('col-description'),
                "routine": document.getElementById('col-routine'),
                "experience": document.getElementById('col-experience'),
                "roleplay": document.getElementById('col-roleplay')
            };

            // Clear columns
            Object.values(columns).forEach(el => el.innerHTML = '');

            // Group questions
            Object.values(questionsDb).forEach(q => {
                const card = document.createElement('div');
                card.id = `card-${q.id}`;
                card.className = "padlet-card p-3 rounded-xl cursor-pointer text-xs flex flex-col justify-between hover:scale-[1.02]";
                card.onclick = () => selectQuestion(q.id);
                card.innerHTML = `
                    <div>
                        <div class="flex items-center gap-1.5 font-bold text-slate-100 mb-1 leading-tight">
                            <span class="text-base">${q.avatar}</span>
                            <span>${q.title}</span>
                        </div>
                        <p class="text-slate-400 line-clamp-2 text-[11px] italic mb-2">"${q.question}"</p>
                    </div>
                    <div class="flex items-center justify-between border-t border-slate-700/60 pt-1.5 mt-1 text-[10px] text-slate-500 font-mono">
                        <span>HP: ${q.maxHp}</span>
                        <span class="text-indigo-400 font-semibold flex items-center gap-1">시작하기 <i class="fa-solid fa-play"></i></span>
                    </div>
                `;
                columns[q.category].appendChild(card);
            });
        }

        // Select a specific question to set current battle targeting
        function selectQuestion(qId) {
            gameState.activeQuestionId = qId;
            const q = questionsDb[qId];

            // Highlight Active Card in Padlet UI
            document.querySelectorAll('.padlet-card').forEach(el => el.classList.remove('active-padlet-card'));
            const activeCard = document.getElementById(`card-${qId}`);
            if (activeCard) {
                activeCard.classList.add('active-padlet-card');
            }

            // Sync monster data with selected question
            gameState.monster.name = q.monsterName;
            gameState.monster.type = q.category;
            gameState.monster.avatar = q.avatar;
            gameState.monster.question = q.question;
            gameState.monster.hint = q.hint;
            gameState.monster.maxHp = q.maxHp;
            gameState.monster.hp = q.maxHp;

            // Sync Main HUD
            document.getElementById('stage-badge').innerText = `Category: ${q.category.toUpperCase()}`;
            document.getElementById('monster-avatar').innerText = q.avatar;
            document.getElementById('monster-name').innerText = q.monsterName;
            document.getElementById('monster-type').innerText = `TYPE: ${q.category.toUpperCase()}`;
            document.getElementById('monster-question-text').innerText = `"${q.question}"`;
            document.getElementById('monster-hint-text').innerText = q.hint;
            
            // Reset answer textarea
            document.getElementById('answer-textarea').value = '';
            updateWordCounter();

            updateMonsterHP();
            showToast(`🎯 '${q.title}' 퀘스트가 던전 전투 필드에 로드되었습니다!`, "info");
        }

        // Speech Recognition Setup
        function initSpeechRecognition() {
            window.SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            if (!window.SpeechRecognition) {
                showToast("⚠️ 이 브라우저는 마이크 입력을 통한 실시간 음성인식을 지원하지 않습니다. 답변을 타자로 타이핑하여 바로 도전해주세요!", "warning");
                return;
            }

            gameState.recognition = new window.SpeechRecognition();
            gameState.recognition.continuous = true;
            gameState.recognition.interimResults = true;
            gameState.recognition.lang = 'en-US';

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

        // Toggle microphone recording state
        function toggleSpeech() {
            if (!gameState.recognition) {
                showToast("마이크 음성 인식을 지원하지 않는 환경입니다. 직접 입력란에 타이핑하여 몬스터에게 대답해주세요!", "warning");
                return;
            }

            if (gameState.speechActive) {
                stopSpeech();
            } else {
                try {
                    gameState.recognition.start();
                } catch (e) {
                    console.error(e);
                    showToast("마이크 디바이스 연결 상태를 체크해주세요.", "error");
                }
            }
        }

        function stopSpeech() {
            if (gameState.recognition && gameState.speechActive) {
                gameState.recognition.stop();
            }
        }

        function updateWordCounter() {
            const text = document.getElementById('answer-textarea').value.trim();
            const words = text ? text.split(/\s+/).length : 0;
            document.getElementById('word-count').innerText = `${words} words`;
        }
        document.getElementById('answer-textarea').addEventListener('input', updateWordCounter);

        // TTS Reader
        function readQuestion() {
            const textToSpeak = document.getElementById('monster-question-text').innerText;
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
                const utterance = new SpeechSynthesisUtterance(textToSpeak);
                utterance.lang = 'en-US';
                utterance.rate = 0.9;
                window.speechSynthesis.speak(utterance);
                showToast("🔊 원어민 질문 오디오가 흐릅니다. 귀 기울여 듣고 답변을 구성해보세요.", "info");
            } else {
                showToast("현재 환경에서 음성 합성 장치를 활성화할 수 없습니다.", "warning");
            }
        }

        // Attack triggering evaluating routines
        async function attackMonster() {
            const answer = document.getElementById('answer-textarea').value.trim();
            if (!answer || answer.split(/\s+/).length < 5) {
                showToast("⚠️ 최소 5단어 이상의 영어 답변을 작성해야 몬스터에게 타격을 가할 수 있습니다!", "warning");
                return;
            }

            stopSpeech();
            gameState.isProcessing = true;
            toggleLoader(true);

            // Dynamically change loading quotes
            const loaders = [
                "답변의 전반적인 말하기 정밀도를 측정하고 있습니다...",
                "시제 사용 마법의 연쇄 계산 중...",
                "오픽 완벽 수집을 위한 Filler 다채로움 분석 중...",
                "Gemini 기반 오픽 채점 가중치 부여 연산 진행 중..."
            ];
            let index = 0;
            const loaderTimer = setInterval(() => {
                index = (index + 1) % loaders.length;
                document.getElementById('loader-quote').innerText = loaders[index];
            }, 2000);

            // API Prompt Configuration
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

            if (gameState.apiKey) {
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
                        throw new Error("Gemini API call failed.");
                    }

                    const result = await response.json();
                    const jsonText = result.candidates?.[0]?.content?.parts?.[0]?.text;
                    
                    if (jsonText) {
                        const parsedJson = JSON.parse(jsonText);
                        resolveCombat(parsedJson);
                    } else {
                        throw new Error("No parsed data.");
                    }
                } catch (error) {
                    console.error("Online API run error, falling back to simulator", error);
                    const fallbackData = simulateBattleFeedback(answer);
                    resolveCombat(fallbackData);
                } finally {
                    clearInterval(loaderTimer);
                    toggleLoader(false);
                    gameState.isProcessing = false;
                }
            } else {
                // If API key is empty, run offline simulator instantly
                setTimeout(() => {
                    const fallbackData = simulateBattleFeedback(answer);
                    resolveCombat(fallbackData);
                    clearInterval(loaderTimer);
                    toggleLoader(false);
                    gameState.isProcessing = false;
                }, 1500);
            }
        }

        // Handle battle system results
        function resolveCombat(evaluation) {
            const dmg = Math.round(evaluation.damage_dealt || 30);
            const score = evaluation.score || 70;

            // Deal Damage
            gameState.monster.hp = Math.max(0, gameState.monster.hp - dmg);
            updateMonsterHP();

            // Self damages
            let playerDmg = 0;
            if (score < 65) {
                playerDmg = 20;
                gameState.player.hp = Math.max(10, gameState.player.hp - playerDmg);
                showToast(`💥 전달력이 약해 보스의 강력한 리액션에 휘청거렸습니다! (-${playerDmg} HP)`, "warning");
            } else {
                gameState.player.hp = Math.min(gameState.player.maxHp, gameState.player.hp + 12);
            }
            updatePlayerHP();

            // Apply evaluation HUD display
            document.getElementById('feedback-score').innerText = score;
            document.getElementById('feedback-critique').innerText = evaluation.critique;

            // Grammars List Injections
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
                gList.innerHTML = `<li class="text-slate-400">문장 배치 및 시제가 완벽합니다! 군더더기 없는 타격이었네요.</li>`;
            }

            // High-end expressions
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
                eList.innerHTML = `<li class="text-slate-400">권장 표현이 지정되지 않았습니다.</li>`;
            }

            // Display Feedback Panel
            const fbPanel = document.getElementById('feedback-panel');
            fbPanel.classList.remove('hidden');
            fbPanel.scrollIntoView({ behavior: 'smooth' });

            showToast(`⚔️ 몬스터에게 성공적으로 ${dmg}의 데미지를 가했습니다!`, "success");

            if (gameState.monster.hp <= 0) {
                triggerVictory();
            }
        }

        // Local evaluation fallback simulator
        function simulateBattleFeedback(answer) {
            const wordCount = answer.split(/\s+/).length;
            let score = 55;
            let dmg = 15;
            let critique = "답변 길이가 상대적으로 다소 짧아 몬스터의 저항을 물리치기에 약합니다. 오픽 IH 기준 달성을 위해 일상 문맥을 더 보충해보세요!";
            const corrections = [];
            const expressions = [];

            // Simple heuristics parsing
            const hasPastKey = /went|visited|felt|saw|had|was|were|decided|started/i.test(answer);
            const hasFiller = /you know|well|i mean|anyway|actually|basically/i.test(answer);
            const hasConnective = /because|although|in addition|since|therefore|however/i.test(answer);

            if (wordCount > 70) {
                score = 83;
                dmg = 80;
                critique = "발화량이 아주 좋습니다! 오픽 채점에서 아주 높은 점수를 가져갈 수 있는 탄탄한 분량입니다. 풍성한 스토리가 전개되었습니다.";
                expressions.push("as clear as day (명명백백한)", "without a doubt (의심의 여지 없이)");
                if (!hasPastKey && (gameState.monster.type === 'experience')) {
                    score = 70;
                    dmg = 45;
                    critique = "분량은 좋았지만 과거 경험 문항임에도 과거 시제(went, played 등)가 보이지 않아 감점 처리되어 데미지가 감소했습니다. 시제 일치에 유의하세요!";
                    corrections.push("과거 행동 서술 시 동사를 과거형으로 고정할 것 (예: 'I go there' -> 'I went there')");
                }
            } else if (wordCount > 35) {
                score = 72;
                dmg = 45;
                critique = "중간 정도의 우수한 발화량입니다! 문맥이 부드럽지만, 'Well, you know' 같은 자연스러운 연결 필러(Filler)들을 추가한다면 확실히 완벽한 IH/AL로 올라설 수 있습니다.";
                corrections.push("발음 정렬 및 세부 묘사를 문맥에 맞게 보충할 것");
                expressions.push("vibrant atmosphere (활기 넘치는 분위기)", "stress relief (스트레스 해소)");
            } else {
                corrections.push("오픽 최소 권장 발화 단어 수 대비 부족 (최소 30단어 권장)");
                expressions.push("as much as possible (가능한 한 많이)", "get some fresh air (신선한 공기를 마시다)");
            }

            if (hasFiller) {
                score = Math.min(100, score + 5);
                dmg = Math.min(gameState.monster.maxHp, dmg + 10);
            }
            if (hasConnective) {
                score = Math.min(100, score + 4);
                dmg = Math.min(gameState.monster.maxHp, dmg + 8);
            }

            return {
                score: score,
                critique: critique,
                grammar_corrections: corrections,
                key_expressions: expressions,
                damage_dealt: dmg
            };
        }

        function triggerVictory() {
            setTimeout(() => {
                const victoryModal = document.getElementById('victory-modal');
                document.getElementById('victory-desc').innerText = `'${gameState.monster.name}'를 완벽히 정복하여 해당 문제를 마스터하셨습니다!`;
                victoryModal.classList.remove('hidden');
                
                // Add XP
                addXp(50);
            }, 1000);
        }

        function closeVictoryModal() {
            document.getElementById('victory-modal').classList.add('hidden');
            document.getElementById('feedback-panel').classList.add('hidden');
            
            // Highlight Next Suggestion Card dynamically or keep board interactive
            showToast("📋 퀘스트 보드에서 다음에 풀고 싶은 기출문제 카드를 자유롭게 선택해보세요!", "success");
            window.scrollTo({ top: 0, behavior: 'smooth' });
        }

        function closeFeedbackAndContinue() {
            document.getElementById('feedback-panel').classList.add('hidden');
            if (gameState.monster.hp > 0) {
                showToast("⚔️ 몬스터가 아직 쓰러지지 않았습니다! 추가 답변으로 연타 공격을 넣어보세요!", "warning");
                document.getElementById('answer-textarea').focus();
            }
        }

        // Stat display updates
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
                gameState.player.xp -= gameState.player.nextXp;
                gameState.player.level += 1;
                gameState.player.nextXp = Math.round(gameState.player.nextXp * 1.5);
                gameState.player.maxHp += 15;
                gameState.player.hp = gameState.player.maxHp;
                
                document.getElementById('player-level').innerText = `Lv ${gameState.player.level}`;
                updatePlayerHP();
                showToast(`🎉 축하합니다! 레벨업을 달성하여 오픽 대처력이 크게 상승했습니다. (Lv ${gameState.player.level})`, "success");
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

        function triggerMonsterHint() {
            showToast(`💡 보스의 방어력 약점 공략법: ${gameState.monster.hint}`, "info");
        }

        // Toast notifications
        function showToast(message, type = "info") {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            
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
            
            setTimeout(() => {
                toast.classList.remove('translate-y-2', 'opacity-0');
            }, 50);

            setTimeout(() => {
                toast.classList.add('translate-y-2', 'opacity-0');
                setTimeout(() => {
                    toast.remove();
                }, 300);
            }, 4500);
        }

        function openHelpModal() {
            document.getElementById('help-modal').classList.remove('hidden');
        }
        function closeHelpModal() {
            document.getElementById('help-modal').classList.add('hidden');
        }
    </script>
</body>
</html>
