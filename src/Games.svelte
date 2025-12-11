<script lang="ts">
    import proxyManager from "./proxy.svelte";
    import { Home, Search, Settings, X, Play } from "@lucide/svelte";
    import { games, type Game } from "./gamesData";
    import { fade, scale } from "svelte/transition";

    let { view = $bindable() }: { view: string } = $props();
    let searchQuery = $state("");
    
    // Check local storage for preference
    let proxyChoice = $state(localStorage.getItem("gameProxyChoice"));
    
    // Track which game is being launched if we need to ask for proxy
    let pendingGame: Game | null = $state(null);

    let filteredGames = $derived.by(() => {
        if (!searchQuery) return games;
        const lowerCaseQuery = searchQuery.toLowerCase();
        return games.filter(game => 
            game.name.toLowerCase().includes(lowerCaseQuery)
        );
    });

    function setProxyChoice(choice: string) {
        proxyChoice = choice;
        if (choice) {
            localStorage.setItem("gameProxyChoice", choice);
        } else {
            localStorage.removeItem("gameProxyChoice");
        }
    }

    function handleGameClick(game: Game) {
        if (proxyChoice) {
            launchGame(game.url, proxyChoice);
        } else {
            pendingGame = game;
        }
    }

    function launchGame(url: string, method: string) {
        // Save the choice if made from modal
        if (!proxyChoice) {
            setProxyChoice(method);
        }

        // Cloaking logic
        document.title = "Classroom";
        const link = document.querySelector("link[rel~='icon']") as HTMLLinkElement;
        if (link) {
            link.href = "/Google_Classroom_Logo.svg.png";
        } else {
            const newLink = document.createElement("link");
            newLink.rel = "icon";
            newLink.href = "/Google_Classroom_Logo.svg.png";
            document.head.appendChild(newLink);
        }

        if (method === 'scramjet') {
            proxyManager.iframeUrl = `https://google-i39l.onrender.com/share/scramjet/${url}`;
            proxyManager.isProxyOpen = true;
        } else if (method === 'noproxy') {
            proxyManager.iframeUrl = url;
            proxyManager.isProxyOpen = true;
        } else {
            // verdis (default)
            if (!proxyManager.startProxy(url)) {
                console.error("Proxy service not ready.");
                return;
            }
        }
        
        pendingGame = null;
    }
</script>

<div class="p-8 text-white bg-slate-950 min-h-screen relative font-sans">
    <!-- Game Launch Modal -->
    {#if pendingGame}
        <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/60 backdrop-blur-md p-4" transition:fade={{ duration: 200 }}>
            <div class="bg-zinc-900 border border-zinc-800 p-0 rounded-2xl max-w-2xl w-full shadow-2xl overflow-hidden" transition:scale={{ start: 0.95, duration: 200 }}>
                <!-- Modal Header with Game Info -->
                <div class="relative h-48 w-full">
                    <img src={pendingGame.imageUrl} alt={pendingGame.name} class="w-full h-full object-cover opacity-60" />
                    <div class="absolute inset-0 bg-gradient-to-t from-zinc-900 to-transparent"></div>
                    <div class="absolute bottom-6 left-6 right-6 flex justify-between items-end">
                        <h2 class="text-4xl font-bold text-white shadow-black drop-shadow-lg">{pendingGame.name}</h2>
                    </div>
                    <button class="absolute top-4 right-4 p-2 bg-black/40 hover:bg-black/60 rounded-full transition-colors text-white" onclick={() => pendingGame = null}>
                        <X size={24} />
                    </button>
                </div>

                <!-- Modal Content -->
                <div class="p-8">
                    <p class="text-zinc-400 mb-6 text-lg">Select a launch method to play this game:</p>
                    
                    <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <button class="group relative flex flex-col items-center justify-center p-4 bg-zinc-800/50 hover:bg-zinc-800 border-2 border-zinc-700/50 hover:border-purple-500 rounded-xl transition-all duration-200 cursor-pointer" onclick={() => launchGame(pendingGame!.url, 'verdis')}>
                            <div class="mb-3 p-3 bg-purple-500/10 rounded-full text-purple-400 group-hover:scale-110 transition-transform">
                                <Play size={24} fill="currentColor" />
                            </div>
                            <span class="text-xl font-bold mb-1 text-white">verdis.</span>
                            <span class="text-xs text-zinc-500 font-mono uppercase tracking-wider mb-2">Recommended</span>
                            <p class="text-xs text-zinc-400 text-center leading-relaxed">Fast & reliable. Works for 70% of games.</p>
                        </button>

                        <button class="group relative flex flex-col items-center justify-center p-4 bg-zinc-800/50 hover:bg-zinc-800 border-2 border-zinc-700/50 hover:border-blue-500 rounded-xl transition-all duration-200 cursor-pointer" onclick={() => launchGame(pendingGame!.url, 'scramjet')}>
                            <div class="mb-3 p-3 bg-blue-500/10 rounded-full text-blue-400 group-hover:scale-110 transition-transform">
                                <Settings size={24} />
                            </div>
                            <span class="text-xl font-bold mb-1 text-white">Scramjet</span>
                            <span class="text-xs text-zinc-500 font-mono uppercase tracking-wider mb-2">Alternative</span>
                            <p class="text-xs text-zinc-400 text-center leading-relaxed">Works for 60% of schools. Good fallback.</p>
                        </button>

                        <button class="group relative flex flex-col items-center justify-center p-4 bg-zinc-800/50 hover:bg-zinc-800 border-2 border-zinc-700/50 hover:border-green-500 rounded-xl transition-all duration-200 cursor-pointer" onclick={() => launchGame(pendingGame!.url, 'noproxy')}>
                            <div class="mb-3 p-3 bg-green-500/10 rounded-full text-green-400 group-hover:scale-110 transition-transform">
                                <Home size={24} />
                            </div>
                            <span class="text-xl font-bold mb-1 text-white">No Proxy</span>
                            <span class="text-xs text-zinc-500 font-mono uppercase tracking-wider mb-2">Direct</span>
                            <p class="text-xs text-zinc-400 text-center leading-relaxed">100% compatibility. Best performance if unblocked.</p>
                        </button>
                    </div>
                    
                    <p class="text-zinc-600 text-xs text-center mt-6">Your choice will be saved for future games. You can change this anytime.</p>
                </div>
            </div>
        </div>
    {/if}

    <!-- Header -->
    <div class="flex flex-col md:flex-row justify-between items-center mb-8 gap-6">
        <div class="flex flex-col">
            <h1 class="text-5xl font-black text-transparent bg-clip-text bg-gradient-to-r from-blue-400 via-purple-400 to-blue-400 animate-gradient bg-300% tracking-tight">Games</h1>
            <p class="text-zinc-400 mt-1 font-medium">Browse and play your favorite titles</p>
        </div>
        
        <div class="flex flex-col sm:flex-row items-center gap-4 w-full md:w-auto">
            {#if proxyChoice}
                <button class="px-4 py-2 rounded-full bg-zinc-800 hover:bg-zinc-700 border border-zinc-700 text-xs text-zinc-400 transition-colors flex items-center gap-2" onclick={() => setProxyChoice("")} title="Current Mode">
                    <span class="w-2 h-2 rounded-full {proxyChoice === 'noproxy' ? 'bg-green-500' : proxyChoice === 'scramjet' ? 'bg-blue-500' : 'bg-purple-500'}"></span>
                    Using {proxyChoice === 'scramjet' ? 'Scramjet' : proxyChoice === 'noproxy' ? 'Direct' : 'verdis.'}
                </button>
            {/if}
            
            <div class="relative w-full sm:w-64 group">
                <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                    <Search class="h-4 w-4 text-zinc-500 group-focus-within:text-blue-400 transition-colors" />
                </div>
                <input 
                    type="text" 
                    placeholder="Search..." 
                    class="input w-full pl-10 bg-zinc-900/50 border-zinc-800 text-white placeholder-zinc-500 focus:bg-zinc-900 focus:outline-none focus:border-blue-500/50 focus:ring-1 focus:ring-blue-500/50 rounded-xl transition-all h-10 text-sm"
                    bind:value={searchQuery}
                />
            </div>
            
            <button class="btn btn-circle btn-sm bg-zinc-800 hover:bg-zinc-700 border-none text-zinc-400 hover:text-white transition-all" onclick={() => view = 'home'} title="Go Home">
                <Home size={18} />
            </button>
        </div>
    </div>

    <!-- Games Grid -->
    <div>
        {#if filteredGames.length === 0}
            <div class="flex flex-col items-center justify-center text-zinc-500 mt-20 gap-4">
                <Search size={48} class="opacity-20" />
                <p>No games found matching "{searchQuery}"</p>
            </div>
        {:else}
            <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 xl:grid-cols-6 gap-6">
                {#each filteredGames as game}
                    <button class="group relative flex flex-col bg-zinc-900 hover:bg-zinc-800 rounded-2xl overflow-hidden transition-all duration-300 hover:-translate-y-1 hover:shadow-xl hover:shadow-black/20 text-left" onclick={() => handleGameClick(game)}>
                        <div class="relative aspect-[4/3] overflow-hidden w-full">
                            <img src={game.imageUrl} alt={game.name} class="w-full h-full object-cover transition-transform duration-500 group-hover:scale-110" loading="lazy" />
                            <div class="absolute inset-0 bg-black/0 group-hover:bg-black/20 transition-colors duration-300"></div>
                            
                            <!-- Play Icon Overlay -->
                            <div class="absolute inset-0 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity duration-300">
                                <div class="w-12 h-12 bg-white/20 backdrop-blur-md rounded-full flex items-center justify-center text-white transform scale-50 group-hover:scale-100 transition-transform duration-300">
                                    <Play size={20} fill="currentColor" />
                                </div>
                            </div>
                        </div>
                        <div class="p-4">
                            <h3 class="font-bold text-zinc-100 text-sm leading-tight line-clamp-1 group-hover:text-blue-400 transition-colors">{game.name}</h3>
                            <p class="text-xs text-zinc-500 mt-1">Click to play</p>
                        </div>
                    </button>
                {/each}
            </div>
        {/if}
    </div>
</div>

<style>
    .bg-300\% {
        background-size: 300% 300%;
    }
    
    @keyframes gradient {
        0% { background-position: 0% 50%; }
        50% { background-position: 100% 50%; }
        100% { background-position: 0% 50%; }
    }
    
    .animate-gradient {
        animation: gradient 8s ease infinite;
    }
</style>