<script lang="ts">
    import config, { saveConfig } from "./config.svelte";
    import { bareProxyUrls, wispProxyUrls, themes } from "./verdis";
    import proxyManager, { ServiceWorkerConfig } from "./proxy.svelte";
    import { Eye, Zap, Palette, Globe, Megaphone, User, Info, Newspaper, HelpCircle, X, ChevronRight, Monitor } from "@lucide/svelte";

    let { isConfigOpen = $bindable() }: { isConfigOpen: boolean } = $props();
    let modalElement: HTMLDialogElement = $state();
    let activeTab = $state("Cloaking");

    const tabs = [
        { id: "Cloaking", icon: Eye, label: "Cloaking" },
        { id: "Performance", icon: Zap, label: "Performance" },
        { id: "Themes", icon: Palette, label: "Themes" },
        { id: "Proxy", icon: Globe, label: "Proxy & Browser" },
        { id: "Advertising", icon: Megaphone, label: "Advertising" },
        { id: "Account", icon: User, label: "Account" },
        { id: "About", icon: Info, label: "About & Statistics" },
        { id: "News", icon: Newspaper, label: "News & Updates" },
        { id: "FAQ", icon: HelpCircle, label: "FAQ" },
    ];

    $effect(() => {
        saveConfig(config);
    });

    $effect(() => {
        if (isConfigOpen) {
            modalElement.showModal();
        } else {
            modalElement.close();
        }
    });

    $effect(() => {
        proxyManager.updateSWConfig(new ServiceWorkerConfig(config.adblock));
    });

    const faviconElement = document.querySelector<HTMLLinkElement>('link[rel="icon"]');

    function openAboutBlank() {
        var win = window.open();
        const url = window.location.href || "https://etherealproxy.netlify.app";
        win.document.body.style.margin = "0";
        win.document.body.style.height = "100vh";
        var iframe = win.document.createElement('iframe');
        iframe.style.border = "none";
        iframe.style.width = "100%";
        iframe.style.height = "100vh";
        iframe.style.margin = "0";
        iframe.src = url;
        win.document.body.appendChild(iframe);
    }
</script>

<dialog
    class="modal z-[100] backdrop-blur-md"
    bind:this={modalElement}
    onclose={() => (isConfigOpen = false)}
>
    <div class="fixed inset-0 bg-black/60" onclick={() => isConfigOpen = false}></div>
    
    <div class="fixed inset-0 flex items-center justify-center p-4 sm:p-8 pointer-events-none">
        <div class="bg-[#050505] border border-white/10 w-full max-w-5xl h-[80vh] rounded-3xl shadow-2xl flex overflow-hidden pointer-events-auto relative">
            
            <!-- Sidebar -->
            <div class="w-72 border-r border-white/5 bg-black/40 flex flex-col">
                <div class="p-8">
                    <h2 class="text-3xl font-bold text-white tracking-tight">Settings</h2>
                </div>
                
                <div class="flex-1 overflow-y-auto px-4 space-y-1">
                    {#each tabs as tab}
                        <button
                            class="w-full flex items-center gap-4 px-4 py-3.5 rounded-xl transition-all duration-200 text-left group relative {activeTab === tab.id ? 'bg-white/5 text-white' : 'text-zinc-500 hover:text-zinc-300 hover:bg-white/5'}"
                            onclick={() => activeTab = tab.id}
                        >
                            {#if activeTab === tab.id}
                                <div class="absolute left-0 top-1/2 -translate-y-1/2 h-6 w-1 bg-[#8b5cf6] rounded-r-full shadow-[0_0_10px_#8b5cf6]"></div>
                            {/if}
                            <svelte:component this={tab.icon} size={20} class={activeTab === tab.id ? 'text-[#a78bfa]' : 'group-hover:text-zinc-300'} />
                            <span class="font-medium text-sm">{tab.label}</span>
                        </button>
                    {/each}
                    
                    <div class="px-4 py-3 mt-4">
                        <span class="inline-flex items-center gap-2 px-3 py-1 rounded-full bg-zinc-900 border border-zinc-800 text-xs text-zinc-500">
                            <span class="w-1.5 h-1.5 rounded-full bg-zinc-600"></span>
                            Coming Soon
                        </span>
                        <div class="flex items-center gap-4 px-4 py-3.5 text-zinc-600 cursor-not-allowed">
                            <Monitor size={20} />
                            <span class="font-medium text-sm">Plugins</span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Content Area -->
            <div class="flex-1 bg-black/20 flex flex-col min-w-0 relative">
                <!-- Close Button -->
                <button 
                    class="absolute top-6 right-6 p-2 rounded-full bg-zinc-900/50 hover:bg-zinc-800 text-zinc-400 hover:text-white transition-all border border-white/5 z-10"
                    onclick={() => isConfigOpen = false}
                >
                    <X size={20} />
                </button>

                <div class="flex-1 overflow-y-auto p-8 sm:p-12">
                    <h3 class="text-2xl font-bold text-white mb-8">{activeTab}</h3>

                    {#if activeTab === "Cloaking"}
                        <div class="space-y-6">
                            <!-- About Blank Card -->
                            <div class="bg-zinc-900/30 border border-white/5 rounded-2xl p-6">
                                <h4 class="text-lg font-semibold text-white mb-2">About:Blank & Blob Cloaking</h4>
                                <p class="text-sm text-zinc-400 leading-relaxed mb-6 max-w-2xl">
                                    About:Blank allows you to hide your tab history, and blockers such as GoGuardian by appearing that you are on a blank tab. If About:Blank doesn't work, then you can try using the blob cloaking which uses temporary data.
                                </p>
                                <div class="flex gap-4">
                                    <button 
                                        class="flex-1 bg-zinc-800 hover:bg-zinc-700 text-white font-medium py-3 px-4 rounded-xl transition-colors border border-white/5"
                                        onclick={openAboutBlank}
                                    >
                                        Launch About:Blank
                                    </button>
                                    <button class="flex-1 bg-zinc-800 hover:bg-zinc-700 text-white font-medium py-3 px-4 rounded-xl transition-colors border border-white/5">
                                        Launch Blob
                                    </button>
                                </div>
                            </div>

                            <!-- Auto Cloaking -->
                            <div class="bg-zinc-900/30 border border-white/5 rounded-2xl p-6">
                                <h4 class="text-lg font-semibold text-white mb-2">Automatic cloaking</h4>
                                <p class="text-sm text-zinc-400 leading-relaxed mb-2">
                                    This toggles automatic cloaking when the site first loads which hides the site from entering your history.
                                </p>
                                <p class="text-xs text-zinc-500 mb-6">Note: Only one automatic cloaking toggle is allowed at a time.</p>
                                
                                <div class="space-y-4 divide-y divide-white/5">
                                    <div class="flex items-center justify-between pt-2">
                                        <span class="text-sm text-zinc-300">Auto-Launch About:Blank</span>
                                        <input type="checkbox" class="toggle toggle-secondary" />
                                    </div>
                                    <div class="flex items-center justify-between pt-4">
                                        <span class="text-sm text-zinc-300">Auto-Launch Blob</span>
                                        <input type="checkbox" class="toggle toggle-secondary" />
                                    </div>
                                </div>
                            </div>

                            <!-- Tab Customization -->
                            <div class="bg-zinc-900/30 border border-white/5 rounded-2xl p-6">
                                <h4 class="text-lg font-semibold text-white mb-4">Tab Customization</h4>
                                <div class="grid gap-4">
                                    <div>
                                        <label class="text-xs text-zinc-500 uppercase font-bold tracking-wider mb-2 block">Tab Title</label>
                                        <input
                                            class="w-full bg-zinc-950 border border-zinc-800 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-[#8b5cf6] transition-colors"
                                            placeholder="Classroom"
                                            value={config.tabTitle}
                                            oninput={(e) => {
                                                const val = (e.target as HTMLInputElement).value;
                                                document.title = val;
                                                config.tabTitle = val;
                                                localStorage.setItem("tabTitle", val);
                                            }}
                                        />
                                    </div>
                                    <div>
                                        <label class="text-xs text-zinc-500 uppercase font-bold tracking-wider mb-2 block">Favicon URL</label>
                                        <input
                                            class="w-full bg-zinc-950 border border-zinc-800 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-[#8b5cf6] transition-colors"
                                            placeholder="https://..."
                                            oninput={(e) => {
                                                const val = (e.target as HTMLInputElement).value;
                                                if (faviconElement) faviconElement.href = val;
                                                localStorage.setItem("faviconUrl", val);
                                            }}
                                        />
                                    </div>
                                </div>
                            </div>
                        </div>
                    {:else if activeTab === "Proxy"}
                        <div class="space-y-6">
                            <div class="bg-zinc-900/30 border border-white/5 rounded-2xl p-6">
                                <h4 class="text-lg font-semibold text-white mb-4">Connection Protocol</h4>
                                <div class="grid grid-cols-2 gap-4">
                                    <button 
                                        class="flex items-center justify-between p-4 rounded-xl border transition-all {config.useBare ? 'bg-[#8b5cf6]/10 border-[#8b5cf6] text-white' : 'bg-zinc-950 border-zinc-800 text-zinc-400 hover:bg-zinc-900'}"
                                        onclick={() => config.useBare = true}
                                    >
                                        <span class="font-medium">Bare</span>
                                        {#if config.useBare}<div class="w-2 h-2 rounded-full bg-[#8b5cf6] shadow-[0_0_10px_#8b5cf6]"></div>{/if}
                                    </button>
                                    <button 
                                        class="flex items-center justify-between p-4 rounded-xl border transition-all {!config.useBare ? 'bg-[#8b5cf6]/10 border-[#8b5cf6] text-white' : 'bg-zinc-950 border-zinc-800 text-zinc-400 hover:bg-zinc-900'}"
                                        onclick={() => config.useBare = false}
                                    >
                                        <span class="font-medium">Wisp</span>
                                        {#if !config.useBare}<div class="w-2 h-2 rounded-full bg-[#8b5cf6] shadow-[0_0_10px_#8b5cf6]"></div>{/if}
                                    </button>
                                </div>
                            </div>

                            <div class="bg-zinc-900/30 border border-white/5 rounded-2xl p-6">
                                <h4 class="text-lg font-semibold text-white mb-4">Proxy Server</h4>
                                <div class="space-y-4">
                                    <div class="relative">
                                        <select 
                                            class="w-full bg-zinc-950 border border-zinc-800 rounded-xl px-4 py-3 text-white appearance-none focus:outline-none focus:border-[#8b5cf6]"
                                            value={config.useBare ? config.bareSelectedProxy : config.wispSelectedProxy}
                                            onchange={(e) => {
                                                const val = (e.target as HTMLSelectElement).value;
                                                if (config.useBare) config.bareSelectedProxy = val;
                                                else config.wispSelectedProxy = val;
                                            }}
                                        >
                                            {#each Object.entries(config.useBare ? bareProxyUrls : wispProxyUrls) as [url, name]}
                                                <option value={url}>{name}</option>
                                            {/each}
                                        </select>
                                        <ChevronRight class="absolute right-4 top-1/2 -translate-y-1/2 text-zinc-500 rotate-90" size={16} />
                                    </div>
                                    
                                    {#if (config.useBare ? config.bareSelectedProxy : config.wispSelectedProxy) === "custom"}
                                        <input
                                            class="w-full bg-zinc-950 border border-zinc-800 rounded-xl px-4 py-3 text-white focus:outline-none focus:border-[#8b5cf6] transition-colors"
                                            placeholder="Enter custom URL..."
                                            value={config.useBare ? config.bareCustomProxy : config.wispCustomProxy}
                                            oninput={(e) => {
                                                const val = (e.target as HTMLInputElement).value;
                                                if (config.useBare) config.bareCustomProxy = val;
                                                else config.wispCustomProxy = val;
                                            }}
                                        />
                                    {/if}
                                </div>
                            </div>
                        </div>
                    {:else if activeTab === "Themes"}
                        <div class="grid grid-cols-2 sm:grid-cols-3 gap-4">
                            {#each Object.entries(themes) as [name, [hex, bg]]}
                                <button
                                    class="p-4 rounded-xl border border-white/10 hover:border-white/20 transition-all text-left flex flex-col gap-2 relative overflow-hidden group"
                                    style="background-color: {bg};"
                                    onclick={() => { 
                                        window.document.documentElement.style.setProperty("--color-blue-500", hex);
                                        window.document.documentElement.style.setProperty("--color-slate-950", bg);
                                        localStorage.setItem("theme", hex)
                                        localStorage.setItem("themeName", name)
                                        localStorage.setItem("bgColor", bg)
                                        window.location.reload()
                                    }}
                                >
                                    <div class="w-full h-20 rounded-lg mb-2" style="background: linear-gradient(135deg, {bg} 0%, {hex} 100%)"></div>
                                    <span class="font-medium text-white capitalize">{name}</span>
                                </button>
                            {/each}
                        </div>
                    {:else if activeTab === "Advertising"}
                         <div class="bg-zinc-900/30 border border-white/5 rounded-2xl p-6">
                            <h4 class="text-lg font-semibold text-white mb-2">Adblocker</h4>
                            <p class="text-sm text-zinc-400 mb-6">Block intrusive ads and trackers while browsing.</p>
                            <div class="flex items-center justify-between">
                                <span class="text-sm text-zinc-300">Enable Adblock</span>
                                <input type="checkbox" class="toggle toggle-primary" bind:checked={config.adblock} />
                            </div>
                        </div>
                    {:else}
                        <div class="flex flex-col items-center justify-center h-64 text-zinc-500">
                            <div class="w-16 h-16 rounded-full bg-zinc-900 flex items-center justify-center mb-4">
                                <span class="text-2xl">🚧</span>
                            </div>
                            <p>This section is under construction.</p>
                        </div>
                    {/if}
                </div>
            </div>
        </div>
    </div>
</dialog>