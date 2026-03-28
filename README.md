<!DOCTYPE html>
<html lang="en" class="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>GEMIGO - Personal AI Chat Bot</title>
    
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: { sans: ['Inter', 'sans-serif'] },
                    colors: {
                        brand: { purple: '#6B50FF', purpleHover: '#5A40E5' },
                        app: { light: '#F4F5F9', dark: '#080C16' }
                    }
                }
            }
        }
    </script>

    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/dompurify/3.0.6/purify.min.js"></script>
    <script src="https://unpkg.com/@phosphor-icons/web"></script>

    <style>
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        .dark ::-webkit-scrollbar-thumb { background: #334155; }

        .text-gradient-custom {
            background: linear-gradient(to right, #7148FC, #358BFF, #04E0E8);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
        }

        @keyframes slideUpMessage {
            from { opacity: 0; transform: translateY(15px); }
            to { opacity: 1; transform: translateY(0); }
        }
        .animate-message { animation: slideUpMessage 0.4s cubic-bezier(0.16, 1, 0.3, 1) forwards; }
        
        .prose pre { background-color: #1e293b; color: #f8fafc; padding: 1rem; border-radius: 0.75rem; margin: 0.5rem 0; overflow-x: auto; }
        .prose code { background: rgba(148, 163, 184, 0.15); padding: 0.2rem 0.4rem; border-radius: 0.25rem; font-family: monospace; }
        
        .typing-cursor::after {
            content: '▋'; display: inline-block; vertical-align: text-bottom;
            animation: blink 1s step-start infinite; color: #7148FC; margin-left: 2px;
        }
        @keyframes blink { 50% { opacity: 0; } }
    </style>
</head>
<body class="bg-app-light dark:bg-app-dark text-gray-900 dark:text-gray-100 transition-colors duration-300 antialiased overflow-hidden">

    <div id="loading-screen" class="fixed inset-0 z-[100] flex flex-col items-center justify-center bg-app-light dark:bg-app-dark transition-opacity duration-500">
        <i class="ph ph-spinner-gap animate-spin text-5xl text-brand-purple mb-4"></i>
        <div class="text-lg font-medium">Loading GEMIGO...</div>
    </div>

    <div id="app" class="flex h-[100dvh] w-full relative hidden opacity-0 transition-opacity duration-300">
        
        <div id="sidebar-overlay" class="fixed inset-0 bg-black/40 z-40 hidden lg:hidden backdrop-blur-sm transition-opacity" onclick="toggleSidebar()"></div>

        <aside id="sidebar" class="fixed lg:static inset-y-0 left-0 z-50 w-72 bg-[#EAECEF] dark:bg-[#0E1322] border-r border-gray-200 dark:border-gray-800 flex flex-col transition-transform duration-300 transform -translate-x-full lg:translate-x-0">
            <div class="p-4 flex items-center justify-between">
                <div class="flex items-center gap-2 font-bold text-xl tracking-tight">
                    <i class="ph-fill ph-sparkle text-brand-purple text-2xl"></i>
                    <span>GEMIGO</span>
                </div>
            </div>
            <div class="px-4 pb-4">
                <button onclick="startNewChat()" class="w-full flex items-center justify-center gap-2 py-2.5 px-4 bg-white dark:bg-[#1A2235] border border-gray-200 dark:border-gray-700 rounded-lg hover:bg-gray-50 transition shadow-sm font-medium">
                    <i class="ph ph-plus"></i> New Chat
                </button>
            </div>
            <div class="flex-1 overflow-y-auto px-3 space-y-1" id="chat-history-list"></div>
            <div class="p-4 border-t border-gray-200 dark:border-gray-800">
                <button onclick="openSettings()" class="w-full flex items-center gap-3 px-3 py-2 rounded-lg hover:bg-gray-200 dark:hover:bg-gray-800 transition text-sm">
                    <i class="ph ph-gear text-lg"></i> Settings
                </button>
            </div>
        </aside>

        <main class="flex-1 flex flex-col relative w-full h-full">
            
            <nav class="w-full px-4 py-3 flex items-center justify-between z-30 bg-app-light/80 dark:bg-app-dark/80 backdrop-blur-md absolute top-0 left-0 right-0">
                <div class="flex items-center">
                    <button id="sidebar-toggle-btn" onclick="toggleSidebar()" class="p-2 lg:hidden text-gray-600 dark:text-gray-400">
                        <i class="ph ph-list text-2xl"></i>
                    </button>
                </div>
                
                <div class="flex items-center gap-2">
                    <button onclick="toggleTheme()" class="p-2 rounded-full bg-white dark:bg-[#1A2235] border border-gray-200 dark:border-gray-700 shadow-sm transition-all active:scale-90">
                        <i class="ph ph-sun dark:hidden text-yellow-500 text-xl"></i>
                        <i class="ph ph-moon hidden dark:block text-blue-400 text-xl"></i>
                    </button>
                </div>
            </nav>

            <div id="scroll-container" class="flex-1 overflow-y-auto w-full pt-16">
                <div id="hero-section" class="flex flex-col items-center justify-center min-h-[65vh] px-4 w-full max-w-4xl mx-auto text-center">
                    <div class="w-14 h-14 rounded-full bg-white dark:bg-[#1A2235] shadow-sm border border-gray-200 dark:border-gray-800 flex items-center justify-center mb-6">
                        <i class="ph-fill ph-sparkle text-3xl text-gradient-custom"></i>
                    </div>
                    <h1 class="text-3xl md:text-4xl font-semibold mb-2 text-gray-800 dark:text-gray-100">Hello, I am <span class="text-gradient-custom">GEMIGO</span></h1>
                    <p class="text-gray-500 dark:text-gray-400 mb-10">How can I assist your coding journey today?</p>
                    
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 w-full max-w-2xl text-left">
                        <div onclick="setPrompt('build')" class="p-5 rounded-2xl border border-gray-200 dark:border-gray-800 bg-white dark:bg-[#111827] hover:border-brand-purple/50 cursor-pointer transition shadow-sm group">
                            <i class="ph-bold ph-lightning text-brand-purple text-xl mb-2"></i>
                            <p class="text-base font-bold">Build Code with Prompt Power</p>
                            <p class="text-xs text-gray-500 mt-1">Craft flawless websites & features with autonomous enhancements.</p>
                        </div>
                        <div onclick="setPrompt('review')" class="p-5 rounded-2xl border border-gray-200 dark:border-gray-800 bg-white dark:bg-[#111827] hover:border-blue-500/50 cursor-pointer transition shadow-sm">
                            <i class="ph-bold ph-shield-check text-blue-500 text-xl mb-2"></i>
                            <p class="text-base font-bold">Review Code for Quality</p>
                            <p class="text-xs text-gray-500 mt-1">Detect logic bugs and optimize performance.</p>
                        </div>
                    </div>
                </div>

                <div id="chat-messages" class="hidden flex-col w-full max-w-3xl mx-auto pb-10 pt-4 px-4 sm:px-6"></div>
                <div class="h-32 w-full"></div>
            </div>

            <div class="absolute bottom-0 left-0 right-0 p-4 bg-gradient-to-t from-app-light via-app-light dark:from-app-dark dark:via-app-dark pt-10">
                <div class="max-w-3xl mx-auto">
                    <form id="chat-form" class="relative flex items-end bg-white dark:bg-[#111827] border border-gray-300 dark:border-gray-700 shadow-lg rounded-2xl overflow-hidden focus-within:ring-2 focus-within:ring-brand-purple/30 transition-all">
                        <textarea id="chat-input" rows="1" class="w-full max-h-[180px] bg-transparent outline-none resize-none py-4 px-5 text-gray-900 dark:text-gray-100 text-base" placeholder="Message GEMIGO..." style="min-height: 56px;"></textarea>
                        <div class="p-2 flex gap-1">
                            <button type="submit" id="send-btn" class="w-11 h-11 flex items-center justify-center rounded-xl bg-brand-purple text-white shadow-md transition-transform active:scale-95">
                                <i class="ph-fill ph-paper-plane-right text-lg"></i>
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </main>
    </div>

    <div id="settings-modal" class="fixed inset-0 z-[60] bg-black/60 backdrop-blur-sm hidden items-center justify-center p-4">
        <div class="bg-white dark:bg-[#0E1322] w-full max-w-md rounded-2xl shadow-xl border border-gray-200 dark:border-gray-800">
            <div class="p-5 border-b border-gray-200 dark:border-gray-800 flex justify-between items-center">
                <h3 class="font-bold">Settings</h3>
                <button onclick="closeSettings()"><i class="ph ph-x"></i></button>
            </div>
            <div class="p-5">
                <label class="block text-xs font-bold uppercase mb-1">OpenRouter API Key</label>
                <input type="password" id="setting-apikey" class="w-full bg-gray-50 dark:bg-[#1A2235] border border-gray-300 dark:border-gray-700 rounded-lg p-2.5 outline-none focus:ring-2 focus:ring-brand-purple/50">
            </div>
            <div class="p-4 flex justify-end">
                <button onclick="saveSettings()" class="px-5 py-2 bg-brand-purple text-white rounded-lg text-sm font-bold">Save</button>
            </div>
        </div>
    </div>

    <script>
        const state = {
            apiKey: localStorage.getItem('gemigo_apikey') || '',
            model: 'deepseek/deepseek-r1:free',
            chats: JSON.parse(localStorage.getItem('gemigo_chats')) || [],
            currentChatId: null
        };

        const DOM = {
            app: document.getElementById('app'),
            loading: document.getElementById('loading-screen'),
            sidebar: document.getElementById('sidebar'),
            sidebarOverlay: document.getElementById('sidebar-overlay'),
            heroSection: document.getElementById('hero-section'),
            chatMessages: document.getElementById('chat-messages'),
            chatForm: document.getElementById('chat-form'),
            chatInput: document.getElementById('chat-input'),
            scrollContainer: document.getElementById('scroll-container'),
            chatHistoryList: document.getElementById('chat-history-list'),
            settingsModal: document.getElementById('settings-modal'),
            settingApiKey: document.getElementById('setting-apikey')
        };

        document.addEventListener('DOMContentLoaded', () => {
            if (localStorage.getItem('gemigo_theme') === 'dark') document.documentElement.classList.add('dark');
            DOM.settingApiKey.value = state.apiKey;
            renderHistory();
            
            DOM.chatInput.addEventListener('input', function() {
                this.style.height = 'auto';
                this.style.height = (this.scrollHeight) + 'px';
            });

            // CLICK OUTSIDE TO CLOSE SIDEBAR
            document.addEventListener('mousedown', (e) => {
                if (window.innerWidth < 1024) { 
                    const sidebar = DOM.sidebar;
                    const toggleBtn = document.getElementById('sidebar-toggle-btn');
                    if (!sidebar.classList.contains('-translate-x-full') && 
                        !sidebar.contains(e.target) && 
                        !toggleBtn.contains(e.target)) {
                        toggleSidebar();
                    }
                }
            });

            DOM.chatForm.addEventListener('submit', handleSendMessage);
            
            setTimeout(() => {
                DOM.loading.style.opacity = '0';
                setTimeout(() => {
                    DOM.loading.style.display = 'none';
                    DOM.app.classList.remove('hidden');
                    DOM.app.style.opacity = '1';
                }, 500);
            }, 800);
        });

        function setPrompt(type) {
            let text = "";
            let selectionStart = 0;
            let selectionEnd = 0;

            if (type === 'build') {
                const prefix = "Build Code with Prompt Power: Craft flawless code for the following feature/website: ";
                const selectable = "Paste your prompt here.";
                text = prefix + selectable;
                selectionStart = prefix.length;
                selectionEnd = text.length;
            } else if (type === 'review') {
                const prefix = "Review Code for Quality: Please analyze the following code snippet for improvements: ";
                const selectable = "[Paste code here]";
                text = prefix + selectable;
                selectionStart = prefix.length;
                selectionEnd = text.length;
            }
            
            DOM.chatInput.value = text;
            DOM.chatInput.focus();
            DOM.chatInput.style.height = 'auto';
            DOM.chatInput.style.height = (DOM.chatInput.scrollHeight) + 'px';

            setTimeout(() => {
                DOM.chatInput.setSelectionRange(selectionStart, selectionEnd);
            }, 10);
        }

        function toggleTheme() {
            document.documentElement.classList.toggle('dark');
            localStorage.setItem('gemigo_theme', document.documentElement.classList.contains('dark') ? 'dark' : 'light');
        }

        function toggleSidebar() { 
            DOM.sidebar.classList.toggle('-translate-x-full');
            DOM.sidebarOverlay.classList.toggle('hidden');
        }

        function openSettings() { DOM.settingsModal.classList.remove('hidden'); DOM.settingsModal.classList.add('flex'); }
        function closeSettings() { DOM.settingsModal.classList.add('hidden'); }
        function saveSettings() {
            state.apiKey = DOM.settingApiKey.value.trim();
            localStorage.setItem('gemigo_apikey', state.apiKey);
            closeSettings();
        }

        function startNewChat() {
            state.currentChatId = null;
            DOM.heroSection.classList.remove('hidden');
            DOM.chatMessages.classList.add('hidden');
            DOM.chatMessages.innerHTML = '';
            DOM.chatInput.value = '';
            DOM.chatInput.style.height = '56px';
        }

        function renderHistory() {
            DOM.chatHistoryList.innerHTML = '';
            [...state.chats].reverse().forEach(chat => {
                const div = document.createElement('div');
                div.className = `p-2.5 rounded-lg cursor-pointer text-sm truncate hover:bg-gray-100 dark:hover:bg-gray-800 transition`;
                div.textContent = chat.title;
                div.onclick = () => loadChat(chat.id);
                DOM.chatHistoryList.appendChild(div);
            });
        }

        function loadChat(chatId) {
            const chat = state.chats.find(c => c.id === chatId);
            if (!chat) return;
            state.currentChatId = chatId;
            DOM.heroSection.classList.add('hidden');
            DOM.chatMessages.classList.remove('hidden');
            DOM.chatMessages.classList.add('flex');
            DOM.chatMessages.innerHTML = '';
            chat.messages.forEach(msg => {
                if(msg.role !== 'system') appendMessage(msg.role, msg.content, false);
            });
            if(window.innerWidth < 1024) toggleSidebar();
        }

        function appendMessage(role, content, animate = true) {
            const div = document.createElement('div');
            div.className = `flex w-full mb-6 ${animate ? 'animate-message' : ''} ${role === 'user' ? 'justify-end' : 'justify-start'}`;
            const inner = document.createElement('div');
            inner.className = `max-w-[85%] rounded-2xl px-5 py-3.5 shadow-sm ${role === 'user' ? 'bg-gray-200 dark:bg-[#1A2235]' : 'bg-white dark:bg-[#111827] border border-gray-200 dark:border-gray-800 prose dark:prose-invert'}`;
            if (role === 'user') inner.textContent = content;
            else inner.innerHTML = DOMPurify.sanitize(marked.parse(content || ''));
            div.appendChild(inner);
            DOM.chatMessages.appendChild(div);
            return inner;
        }

        async function handleSendMessage(e) {
            e.preventDefault();
            const text = DOM.chatInput.value.trim();
            if (!text || !state.apiKey) { if(!state.apiKey) openSettings(); return; }

            DOM.chatInput.value = '';
            DOM.chatInput.style.height = '56px';
            DOM.heroSection.classList.add('hidden');
            DOM.chatMessages.classList.remove('hidden');
            DOM.chatMessages.classList.add('flex');

            if (!state.currentChatId) {
                state.currentChatId = Date.now().toString();
                state.chats.push({ id: state.currentChatId, title: text.slice(0, 30), messages: [{role: 'system', content: 'You are GEMIGO, a world-class coding AI.'}]});
            }
            
            const chat = state.chats.find(c => c.id === state.currentChatId);
            chat.messages.push({ role: 'user', content: text });
            appendMessage('user', text);
            const aiDiv = appendMessage('assistant', '');
            aiDiv.classList.add('typing-cursor');
            
            try {
                const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json', 'Authorization': `Bearer ${state.apiKey}` },
                    body: JSON.stringify({ model: state.model, messages: chat.messages, stream: true })
                });

                const reader = response.body.getReader();
                let fullText = "";
                while (true) {
                    const { done, value } = await reader.read();
                    if (done) break;
                    const chunk = new TextDecoder().decode(value);
                    const lines = chunk.split('\n');
                    for (const line of lines) {
                        if (line.startsWith('data: ') && line !== 'data: [DONE]') {
                            try {
                                const data = JSON.parse(line.slice(6));
                                fullText += data.choices[0]?.delta?.content || '';
                                aiDiv.innerHTML = DOMPurify.sanitize(marked.parse(fullText));
                                DOM.scrollContainer.scrollTop = DOM.scrollContainer.scrollHeight;
                            } catch(e){}
                        }
                    }
                }
                chat.messages.push({ role: 'assistant', content: fullText });
                localStorage.setItem('gemigo_chats', JSON.stringify(state.chats));
            } catch (err) { aiDiv.textContent = "Error: Check API Key."; }
            aiDiv.classList.remove('typing-cursor');
        }
    </script>
</body>
</html>
