<script lang="ts">
    import proxyManager from "./proxy.svelte";
    import { Home, Search, Settings } from "@lucide/svelte";
    import { games, type Game } from "./gamesData";

    let { view = $bindable() }: { view: string } = $props();
    let searchQuery = $state("");
    
    // Check local storage for preference
    let proxyChoice = $state(localStorage.getItem("gameProxyChoice"));

    let filteredGames = $derived.by(() => {
        if (!searchQuery) return games;
        const lowerCaseQuery = searchQuery.toLowerCase();
        return games.filter(game => 
            game.name.toLowerCase().includes(lowerCaseQuery)
        );
    });

    function setProxyChoice(choice: string) {
        proxyChoice = choice;
        localStorage.setItem("gameProxyChoice", choice);
    }

    function playGame(url: string) {
        // Cloaking
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

        if (proxyChoice === 'scramjet') {
            // Scramjet Logic
            proxyManager.iframeUrl = `https://google-i39l.onrender.com/share/scramjet/${url}`;
            proxyManager.isProxyOpen = true;
        } else if (proxyChoice === 'noproxy') {
            // No Proxy Logic
            proxyManager.iframeUrl = url;
            proxyManager.isProxyOpen = true;
        } else {
            // verdis. Logic (Default)
            if (!proxyManager.startProxy(url)) {
                console.error("The proxy service is not ready yet. Please wait a few seconds for the connection to establish and try again.");
                return;
            }
        }
    }
</script>

<div class="p-8 text-white bg-slate-950 min-h-screen relative">
    {#if !proxyChoice}
        <div class="fixed inset-0 z-50 flex items-center justify-center bg-black/80 backdrop-blur-sm">
            <div class="bg-zinc-900 border border-zinc-800 p-8 rounded-2xl max-w-4xl w-full text-center shadow-2xl">
                <h2 class="text-3xl font-bold mb-2">Select Proxy</h2>
                <p class="text-zinc-400 mb-8">Choose how you want to load games.</p>
                
                <div class="grid grid-cols-1 md:grid-cols-3 gap-4">
                    <button class="group relative flex flex-col items-center justify-center p-6 bg-zinc-800 hover:bg-zinc-700 border-2 border-zinc-700 hover:border-blue-500 rounded-xl transition-all duration-300 cursor-pointer" onclick={() => setProxyChoice('scramjet')}>
                        <span class="text-2xl font-bold mb-2 text-blue-400">Scramjet</span>
                        <p class="text-sm text-zinc-400 group-hover:text-zinc-200">scramjet works for 60% of schools</p>
                    </button>
                    
                    <button class="group relative flex flex-col items-center justify-center p-6 bg-zinc-800 hover:bg-zinc-700 border-2 border-zinc-700 hover:border-purple-500 rounded-xl transition-all duration-300 cursor-pointer" onclick={() => setProxyChoice('verdis')}>
                        <span class="text-2xl font-bold mb-2 text-purple-400">verdis.</span>
                        <p class="text-sm text-zinc-400 group-hover:text-zinc-200">70% of games work, works for everyone</p>
                    </button>

                    <button class="group relative flex flex-col items-center justify-center p-6 bg-zinc-800 hover:bg-zinc-700 border-2 border-zinc-700 hover:border-green-500 rounded-xl transition-all duration-300 cursor-pointer" onclick={() => setProxyChoice('noproxy')}>
                        <span class="text-2xl font-bold mb-2 text-green-400">No Proxy</span>
                        <p class="text-sm text-zinc-400 group-hover:text-zinc-200">works with all games 100% and it will work if ur at my school or u can try it</p>
                    </button>
                </div>
            </div>
        </div>
    {/if}

    <div class="flex justify-between items-center mb-8 gap-4 { !proxyChoice ? 'blur-sm pointer-events-none' : '' }">
        <h1 class="text-4xl font-bold text-transparent bg-clip-text bg-gradient-to-r from-blue-500 to-purple-300 whitespace-nowrap">Games</h1>
        
        <div class="flex items-center gap-4 w-full justify-end">
            {#if proxyChoice}
                <button class="btn btn-ghost btn-sm text-zinc-500" onclick={() => setProxyChoice("")} title="Change Proxy Selection">
                    {proxyChoice === 'scramjet' ? 'Scramjet' : proxyChoice === 'noproxy' ? 'No Proxy' : 'verdis.'}
                </button>
            {/if}
            <div class="relative w-full max-w-md">
                <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
                    <Search class="h-5 w-5 text-gray-400" />
                </div>
                <input 
                    type="text" 
                    placeholder="Search games..." 
                    class="input input-bordered w-full pl-10 bg-zinc-900 border-zinc-700 text-white placeholder-gray-400 focus:outline-none focus:border-blue-500"
                    bind:value={searchQuery}
                />
            </div>
            
            <button class="btn btn-ghost btn-circle" onclick={() => view = 'home'} title="Go Home">
                <Home />
            </button>
        </div>
    </div>

    <div class={ !proxyChoice ? 'blur-sm pointer-events-none' : '' }>
        {#if filteredGames.length === 0}
            <div class="text-center text-gray-400 mt-10">
                <p>No games found matching "{searchQuery}"</p>
            </div>
        {:else}
            <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-6 gap-4">
                {#each filteredGames as game}
                    <div class="card bg-zinc-900 shadow-xl cursor-pointer transform hover:scale-105 transition-transform duration-300 group" onclick={() => playGame(game.url)}>
                        <figure class="relative h-32 w-full overflow-hidden rounded-t-xl">
                            <img src={game.imageUrl} alt={game.name} class="h-full w-full object-cover" loading="lazy" />
                            <div class="absolute inset-0 bg-black/0 group-hover:bg-black/20 transition-colors duration-300"></div>
                        </figure>
                        <div class="card-body p-3 items-center text-center">
                            <h2 class="card-title text-sm font-medium line-clamp-2">{game.name}</h2>
                        </div>
                    </div>
                {/each}
            </div>
        {/if}
    </div>
</div>