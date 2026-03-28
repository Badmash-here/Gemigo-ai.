<!DOCTYPE html>
<html lang="en" class="light">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>GEMIGO AI - Pro Coding Assistant</title>
    
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
        ::-webkit-scrollbar { width: 5px; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        .dark ::-webkit-scrollbar-thumb { background: #334155; }
        .text-gradient-custom { background: linear-gradient(to right, #7148FC, #358BFF, #04E0E8); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .prose pre { background: #1e293b; color: #f8fafc; padding: 1rem; border-radius: 0.75rem; overflow-x: auto; margin: 10px 0; }
        .dropdown-menu { display: none; position: absolute; right: 0; top: 100%; z-index: 100; min-width: 140px; }
        .dropdown-menu.show { display: block; }
        .chat-item-active { background: rgba(107, 80, 255, 0.1); border: 1px solid rgba(107, 80, 255, 0.2); }
    </style>
</head>
<body class="bg-app-light dark:bg-app-dark text-gray-900 dark:text-gray-100 transition-colors duration-300 antialiased overflow-hidden">

    <div id="app" class="flex h-[100dvh] w-full relative">
        <div id="sidebar-overlay" class="fixed inset-0 bg-black/40 z-40 hidden lg:hidden" onclick="toggleSidebar()"></div>

        <aside id="sidebar" class="fixed lg:static inset-y-0 left-0 z-50 w-72 bg-[#EAECEF] dark:bg-[#0E1322] border-r border-gray-200 dark:border-gray-800 flex flex-col transition-transform duration-300 transform -translate-x-full lg:translate-x-0">
            <div class="p-4 flex items-center justify-between">
                <div class="flex items-center gap-2 font-bold text-xl tracking-tight cursor-pointer" onclick="location.reload()">
                    <i class="ph-fill ph-sparkle text-brand-purple text-2xl"></i>
                    <span>GEMIGO</span>
                </div>
            </div>
            
            <div class="px-4 pb-4">
                <button onclick="startNewChat()" class="w-full flex items-center justify-center gap-2 py-2.5 px-4 bg-white dark:bg-[#1A2235] border border-gray-200 dark:border-gray-700 rounded-lg hover:shadow-md transition font-medium">
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

        <main class="flex-1 flex flex-col relative w-full h-full overflow-hidden">
            <nav class="w-full px-4 py-3 flex items-center justify-between z-30 bg-app-light/80 dark:bg-app-dark/80 backdrop-blur-md">
                <button onclick="toggleSidebar()" class="p-2 lg:hidden"><i class="ph ph-list text-2xl"></i></button>
                <button onclick="toggleTheme()" class="p-2 rounded-full bg-white dark:bg-[#1A2235] shadow-sm ml-auto">
                    <i class="ph ph-sun dark:hidden text-yellow-500 text-xl"></i>
                    <i class="ph ph-moon hidden dark:block text-blue-400 text-xl"></i>
                </button>
            </nav>

            <div id="scroll-container" class="flex-1 overflow-y-auto w-full">
                <div id="hero-section" class="flex flex-col items-center justify-center min-h-[70vh] px-4 w-full max-w-4xl mx-auto text-center">
                    <div class="w-16 h-16 rounded-full bg-white dark:bg-[#1A2235] border border-gray-200 dark:border-gray-800 flex items-center justify-center mb-6 shadow-md">
                        <i class="ph-fill ph-sparkle text-4xl text-gradient-custom"></i>
                    </div>
                    <h1 class="text-3xl md:text-4xl font-bold mb-3 tracking-tight">Hello, I am <span class="text-gradient-custom">GEMIGO</span></h1>
                    <p class="text-gray-500 dark:text-gray-400 mb-10 text-lg">Ready to build something amazing today?</p>
                    
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-4 w-full max-w-2xl text-left">
                        <div onclick="setPrompt('build')" class="p-6 rounded-2xl border border-gray-200 dark:border-gray-800 bg-white dark:bg-[#111827] hover:border-brand-purple cursor-pointer transition shadow-sm group">
                            <i class="ph-bold ph-lightning text-brand-purple text-2xl mb-2"></i>
                            <p class="font-bold text-lg">Build Code with Prompt Power</p>
                            <p class="text-sm text-gray-500 mt-1">Create production-ready code with auto-enhancements.</p>
                        </div>
                        <div onclick="setPrompt('review')" class="p-6 rounded-2xl border border-gray-200 dark:border-gray-800 bg-white dark:bg-[#111827] hover:border-blue-500 cursor-pointer transition shadow-sm">
                            <i class="ph-bold ph-shield-check text-blue-500 text-2xl mb-2"></i>
                            <p class="font-bold text-lg">Review Code for Quality</p>
                            <p class="text-sm text-gray-500 mt-1">Optimization and bug detection for your snippets.</p>
                        </div>
                    </div>
                </div>

                <div id="chat-messages" class="hidden flex-col w-full max-w-3xl mx-auto pb-20 pt-10 px-4 sm:px-6"></div>
                <div id="bottom-anchor" class="h-24"></div>
            </div>

            <div class="p-4 bg-gradient-to-t from-app-light dark:from-app-dark pt-10">
                <div class="max-w-3xl mx-auto">
                    <form id="chat-form" class="relative flex items-end bg-white dark:bg-[#111827] border border-gray-300 dark:border-gray-700 shadow-xl rounded-2xl overflow-hidden focus-within:ring-2 focus-within:ring-brand-purple/30 transition-all">
                        <textarea id="chat-input" rows="1" class="w-full max-h-[200px] bg-transparent outline-none resize-none py-4 px-5 text-base" placeholder="Message GEMIGO..."></textarea>
                        <div class="p-2">
                            <button type="submit" class="w-12 h-12 flex items-center justify-center rounded-xl bg-brand-purple text-white shadow-lg active:scale-90 transition">
                                <i class="ph-fill ph-paper-plane-right text-xl"></i>
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </main>
    </div>

    <div id="settings-modal" class="fixed inset-0 z-[60] bg-black/60 backdrop-blur-sm hidden items-center justify-center p-4">
        <div class="bg-white dark:bg-[#0E1322] w-full max-w-md rounded-2xl shadow-2xl border border-gray-200 dark:border-gray-800 p-6">
            <h3 class="text-xl font-bold mb-4">Settings</h3>
            <label class="block text-xs font-bold text-gray-500 uppercase mb-1">OpenRouter API Key</label>
            <input type="password" id="setting-apikey" class="w-full bg-gray-50 dark:bg-[#1A2235] border border-gray-300 dark:border-gray-700 rounded-lg p-3 outline-none mb-6 focus:ring-2 focus:ring-brand-purple">
            <div class="flex justify-end gap-3">
                <button onclick="closeSettings()" class="px-4 py-2 text-sm font-medium">Cancel</button>
                <button onclick="saveSettings()" class="px-6 py-2 bg-brand-purple text-white rounded-lg text-sm font-bold shadow-md">Save Key</button>
            </div>
        </div>
    </div>

    <script>
        // --- State Management ---
        const state = {
            apiKey: localStorage.getItem('gemigo_apikey') || '',
            model: 'deepseek/deepseek-r1:free',
            chats: JSON.parse(localStorage.getItem('gemigo_chats')) || [],
            currentChatId: null
        };

        const DOM = {
            sidebar: document.getElementById('sidebar'),
            sidebarOverlay: document.getElementById('sidebar-overlay'),
            chatHistoryList: document.getElementById('chat-history-list'),
            heroSection: document.getElementById('hero-section'),
            chatMessages: document.getElementById('chat-messages'),
            chatForm: document.getElementById('chat-form'),
            chatInput: document.getElementById('chat-input'),
            scrollContainer: document.getElementById('scroll-container'),
            settingsModal: document.getElementById('settings-modal'),
            settingApiKey: document.getElementById('setting-apikey')
        };

        // --- Core Logic ---
        document.addEventListener('DOMContentLoaded', () => {
            if (localStorage.getItem('gemigo_theme') === 'dark') document.documentElement.classList.add('dark');
            DOM.settingApiKey.value = state.apiKey;
            renderHistory();
            
            DOM.chatInput.addEventListener('input', function() {
                this.style.height = 'auto';
                this.style.height = (this.scrollHeight) + 'px';
            });

            DOM.chatForm.addEventListener('submit', handleSendMessage);
        });

        function startNewChat() {
            state.currentChatId = null;
            DOM.heroSection.classList.remove('hidden');
            DOM.chatMessages.classList.add('hidden');
            DOM.chatMessages.innerHTML = '';
            DOM.chatInput.value = '';
            DOM.chatInput.style.height = '56px';
            renderHistory();
            if(window.innerWidth < 1024) toggleSidebar();
        }

        function setPrompt(type) {
            let text = "";
            let prefix = "";
            if (type === 'build') {
                prefix = "Build Code with Prompt Power: Craft flawless code for the following feature/website: ";
                text = prefix + "Paste your prompt here.";
            } else {
                prefix = "Review Code for Quality: Analyze this code for improvements: ";
                text = prefix + "[Paste code here]";
            }
            DOM.chatInput.value = text;
            DOM.chatInput.focus();
            DOM.chatInput.style.height = (DOM.chatInput.scrollHeight) + 'px';
            setTimeout(() => DOM.chatInput.setSelectionRange(prefix.length, text.length), 10);
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
                state.chats.push({
                    id: state.currentChatId,
                    title: text.substring(0, 30),
                    messages: [{role: 'system', content: 'You are GEMIGO AI.'}]
                });
            }
            
            const chat = state.chats.find(c => c.id === state.currentChatId);
            chat.messages.push({ role: 'user', content: text });
            appendMessage('user', text);
            
            const aiDiv = appendMessage('assistant', '...');
            
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
                saveAndRender();
            } catch (err) { aiDiv.textContent = "Error: Check API Key Settings."; }
        }

        function appendMessage(role, content) {
            const div = document.createElement('div');
            div.className = `flex w-full mb-6 ${role === 'user' ? 'justify-end' : 'justify-start'}`;
            const inner = document.createElement('div');
            inner.className = `max-w-[85%] rounded-2xl px-5 py-3.5 shadow-sm ${role === 'user' ? 'bg-gray-200 dark:bg-[#1A2235]' : 'bg-white dark:bg-[#111827] border border-gray-200 dark:border-gray-800 prose dark:prose-invert'}`;
            if (role === 'user') inner.textContent = content;
            else inner.innerHTML = DOMPurify.sanitize(marked.parse(content));
            div.appendChild(inner);
            DOM.chatMessages.appendChild(div);
            return inner;
        }

        // --- History & Dropdown Logic ---
        function renderHistory() {
            DOM.chatHistoryList.innerHTML = '';
            [...state.chats].reverse().forEach(chat => {
                const isActive = chat.id === state.currentChatId;
                const div = document.createElement('div');
                div.className = `group relative flex items-center justify-between p-2.5 rounded-xl cursor-pointer transition ${isActive ? 'chat-item-active' : 'hover:bg-gray-200 dark:hover:bg-gray-800'}`;
                
                div.innerHTML = `
                    <div class="flex items-center gap-3 truncate flex-1 pr-2" onclick="loadChat('${chat.id}')">
                        <i class="ph ph-chat-centered-text text-brand-purple"></i>
                        <span class="truncate text-sm font-medium">${chat.title}</span>
                    </div>
                    <div class="relative">
                        <button onclick="toggleDropdown(event, '${chat.id}')" class="p-1 hover:bg-gray-300 dark:hover:bg-gray-700 rounded-lg transition opacity-0 group-hover:opacity-100">
                            <i class="ph ph-dots-three-vertical-bold"></i>
                        </button>
                        <div id="dropdown-${chat.id}" class="dropdown-menu bg-white dark:bg-[#1A2235] border border-gray-200 dark:border-gray-700 rounded-xl shadow-2xl overflow-hidden py-1">
                            <button onclick="renameChat('${chat.id}')" class="w-full text-left px-4 py-2.5 text-xs hover:bg-gray-100 dark:hover:bg-gray-800 flex items-center gap-2 font-medium">
                                <i class="ph ph-pencil-simple"></i> Rename
                            </button>
                            <button onclick="shareChat('${chat.id}')" class="w-full text-left px-4 py-2.5 text-xs hover:bg-gray-100 dark:hover:bg-gray-800 flex items-center gap-2 font-medium">
                                <i class="ph ph-share-network"></i> Share
                            </button>
                            <button onclick="deleteChat('${chat.id}')" class="w-full text-left px-4 py-2.5 text-xs hover:bg-red-50 text-red-500 flex items-center gap-2 font-medium">
                                <i class="ph ph-trash"></i> Delete
                            </button>
                        </div>
                    </div>
                `;
                DOM.chatHistoryList.appendChild(div);
            });
        }

        function toggleDropdown(e, id) {
            e.stopPropagation();
            const allMenus = document.querySelectorAll('.dropdown-menu');
            const targetMenu = document.getElementById(`dropdown-${id}`);
            const isShowing = targetMenu.classList.contains('show');
            allMenus.forEach(m => m.classList.remove('show'));
            if (!isShowing) targetMenu.classList.add('show');
        }

        function loadChat(chatId) {
            const chat = state.chats.find(c => c.id === chatId);
            if (!chat) return;
            state.currentChatId = chatId;
            DOM.heroSection.classList.add('hidden');
            DOM.chatMessages.classList.remove('hidden');
            DOM.chatMessages.classList.add('flex');
            DOM.chatMessages.innerHTML = '';
            chat.messages.forEach(msg => { if(msg.role !== 'system') appendMessage(msg.role, msg.content); });
            renderHistory();
            if(window.innerWidth < 1024) toggleSidebar();
        }

        function renameChat(id) {
            const chat = state.chats.find(c => c.id === id);
            const newTitle = prompt("Update Chat Name:", chat.title);
            if (newTitle && newTitle.trim()) {
                chat.title = newTitle.trim();
                saveAndRender();
            }
        }

        function deleteChat(id) {
            if (confirm("Permanently delete this chat?")) {
                state.chats = state.chats.filter(c => c.id !== id);
                if (state.currentChatId === id) startNewChat();
                saveAndRender();
            }
        }

        function shareChat(id) {
            const chat = state.chats.find(c => c.id === id);
            const text = chat.messages.filter(m => m.role !== 'system').map(m => `[${m.role.toUpperCase()}]: ${m.content}`).join('\n\n');
            navigator.clipboard.writeText(text).then(() => alert("Chat copied to clipboard!"));
        }

        function saveAndRender() {
            localStorage.setItem('gemigo_chats', JSON.stringify(state.chats));
            renderHistory();
        }

        function toggleSidebar() { 
            DOM.sidebar.classList.toggle('-translate-x-full'); 
            DOM.sidebarOverlay.classList.toggle('hidden');
        }

        function toggleTheme() {
            document.documentElement.classList.toggle('dark');
            localStorage.setItem('gemigo_theme', document.documentElement.classList.contains('dark') ? 'dark' : 'light');
        }

        function openSettings() { DOM.settingsModal.classList.remove('hidden'); DOM.settingsModal.classList.add('flex'); }
        function closeSettings() { DOM.settingsModal.classList.add('hidden'); }
        function saveSettings() {
            state.apiKey = DOM.settingApiKey.value.trim();
            localStorage.setItem('gemigo_apikey', state.apiKey);
            closeSettings();
        }

        // Global click to close menus
        window.addEventListener('click', () => {
            document.querySelectorAll('.dropdown-menu').forEach(m => m.classList.remove('show'));
        });
    </script>
</body>
</html>
