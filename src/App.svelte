<script lang="ts">
    import Config from "./Config.svelte";
    import config from "./config.svelte";
    import proxyManager from "./proxy.svelte";
    import { onEnterKeyPressed } from "./util";
    import autoProxyProber from "./prober.svelte";
    import { History } from "./history";
    import Games from "./Games.svelte";
    import Classroom from "./classroom/Classroom.svelte";
    import { brandIcons } from "./icons";
    import {
        Search,
        Settings,
        RotateCw,
        ArrowRight,
        ArrowLeft,
        X,
        Gamepad2,
        Maximize2,
        Code,
        Home
    } from "@lucide/svelte";
    import { onMount } from "svelte";
    import { fade, fly } from "svelte/transition";

    $effect(() => {
        if (config.useBare && config.bareSelectedProxy === "auto") {
            autoProxyProber.probeBare();
        } else if (config.wispSelectedProxy === "auto") {
            autoProxyProber.probeWisp();
        }
    });

    onMount(() => {
        if (document) {
            document.title = localStorage.getItem("tabTitle") || "Classroom";
            const faviconElement = document.querySelector<HTMLLinkElement>("link[rel~='icon']");
            if (faviconElement) {
                faviconElement.href = localStorage.getItem("faviconUrl") || "/Google_Classroom_Logo.svg.png";
            } else {
                const newLink = document.createElement("link");
                newLink.rel = "icon";
                newLink.href = localStorage.getItem("faviconUrl") || "/Google_Classroom_Logo.svg.png";
                document.head.appendChild(newLink);
            }
            window.document.documentElement.style.setProperty("--color-blue-500", localStorage.getItem("theme") || "#2b7fff")
            window.document.documentElement.style.setProperty("--color-slate-950", localStorage.getItem("bgColor") || "#0d1117")
        }
    });

    let urlBar: HTMLInputElement = $state();
    let searchbar: HTMLInputElement = $state();

    $effect(() => {
        if (view === 'home' && !proxyManager.isProxyOpen && urlBar) {
            urlBar.focus();
        }
    });

    $effect(() => {
        proxyManager.setProxyServer(proxyManager.proxyUrl);
    });

    let destinationInput = $state("");
    let view = $state("classroom"); 
    let isConfigOpen = $state(false);
    let isWarping = $state(false);

    function startProxy(url?: string) {
        const dest = url || destinationInput;
        if (proxyManager.startProxy(dest)) {
            console.log("proxy started for", dest);
        }
    }

    let iframeHasLoaded = $state(false);
    let iframe: HTMLIFrameElement = $state();

    const iframeAllow =
        "accelerometer ambient-light-sensor attribution-reporting autoplay bluetooth browsing-topics camera compute-pressure " +
        "cross-origin-isolated display-capture document-domain encrypted-media fullscreen gamepad geolocation gyroscope hid " +
        "identity-credentials-get idle-detection local-fonts magnetometer microphone midi otp-credentials payment " +
        "picture-in-picture publickey-credentials-create publickey-credentials-get screen-wake-lock serial speaker-selection " +
        "storage-access usb web-share window-management xr-spatial-tracking";

    const iframeSandbox =
        "allow-popups allow-downloads allow-forms allow-modals allow-orientation-lock " +
        "allow-pointer-lock allow-presentation allow-same-origin allow-scripts allow-storage-access-by-user-activation";

    function onIframeLoad() {
        if (iframe == undefined) return;
        const src = iframe.contentWindow.location.pathname;
        if (!src.includes(proxyManager.uvConfig.prefix)) return;

        iframeHasLoaded = true;
                    
        if (searchbar) {
            searchbar.value = proxyManager.url;
        }
        destinationInput = proxyManager.url;

        proxyManager.url = proxyManager.uvConfig.decodeUrl(
            src.slice(proxyManager.uvConfig.prefix.length),
        );
    }

    let proxyHistory = new History();
    $effect(() => {
        proxyHistory.addToHistory(proxyManager.url);
    });

    function setUrl(url: string | null) {
        if (url == null) return;
        proxyManager.url = url;
        proxyManager.reloadIframe();
    }

    function triggerWarp() {
        isWarping = true;
        setTimeout(() => {
            view = 'home';
            isWarping = false;
        }, 1500);
    }

    const shortcuts = [
        { name: "Google", url: "https://google.com", style: "bg-white text-black", logo: brandIcons.Google },
        { name: "YouTube", url: "https://www.youtube.com", style: "bg-white text-black", logo: brandIcons.YouTube },
        { name: "Spotify", url: "https://open.spotify.com", style: "bg-[#1DB954] text-white", logo: brandIcons.Spotify },
        { name: "Discord", url: "https://discord.com", style: "bg-[#5865F2] text-white", logo: brandIcons.Discord },
        { name: "ChatGPT", url: "https://chatgpt.com", style: "bg-[#10A37F] text-white", textLogo: "ChatGPT", logo: null },
        { name: "GeForce Now", url: "https://play.geforcenow.com", style: "bg-[#76B900] text-black", textLogo: "GEFORCE NOW", logo: null },
        { name: "GitHub", url: "https://github.com", style: "bg-[#181717] text-white", logo: brandIcons.GitHub },
        { name: "Twitch", url: "https://twitch.tv", style: "bg-[#9146FF] text-white", logo: brandIcons.Twitch },
        { name: "ESPN", url: "https://espn.com", style: "bg-[#CC0000] text-white", textLogo: "ESPN", logo: null },
        { name: "TikTok", url: "https://tiktok.com", style: "bg-black text-white", logo: brandIcons.TikTok },
    ];

    function toggleFullscreen() {
        if (!document.fullscreenElement) {
            document.documentElement.requestFullscreen();
        } else {
            if (document.exitFullscreen) {
                document.exitFullscreen();
            }
        }
    }

</script>

{#if view !== 'classroom'}
    <div class="fixed inset-0 z-0 pointer-events-none bg-black overflow-hidden">
        <div class="star-container {isWarping ? 'warp' : ''}">
            <div class="stars"></div>
        </div>
        <!-- Purple Glow at Bottom Center -->
        <div class="absolute bottom-[-20%] left-1/2 -translate-x-1/2 w-[80vw] h-[60vh] bg-[#7c3aed] opacity-20 blur-[120px] rounded-full pointer-events-none"></div>
    </div>
{/if}

<div class="min-h-screen text-white font-sans selection:bg-purple-500/30 overflow-hidden relative">
    <Config bind:isConfigOpen></Config>

    {#if proxyManager.isProxyOpen}
        <!-- Proxy View -->
        <div class="fixed inset-0 z-10 bg-black" transition:fade={{ duration: 300 }}>
            <iframe
                bind:this={iframe}
                title="Proxy"
                class="w-full h-full border-none"
                src={proxyManager.iframeUrl}
                onload={onIframeLoad}
                allow={iframeAllow}
                sandbox={iframeSandbox}
            ></iframe>
        </div>

        <!-- Floating Control Dock -->
        <div class="fixed bottom-8 left-1/2 -translate-x-1/2 z-50 flex items-center gap-2 p-2 bg-zinc-900/90 backdrop-blur-xl border border-zinc-800/50 rounded-full shadow-2xl shadow-black/50 transition-all hover:scale-[1.02] hover:bg-zinc-900" transition:fly={{ y: 50, duration: 400 }}>
            <div class="flex items-center gap-1 px-2 border-r border-zinc-800">
                <button class="btn btn-ghost btn-circle btn-sm text-zinc-400 hover:text-white hover:bg-zinc-800" onclick={() => setUrl(proxyHistory.goBackward())}><ArrowLeft size={18} /></button>
                <button class="btn btn-ghost btn-circle btn-sm text-zinc-400 hover:text-white hover:bg-zinc-800" onclick={() => setUrl(proxyHistory.goForward())}><ArrowRight size={18} /></button>
                <button class="btn btn-ghost btn-circle btn-sm text-zinc-400 hover:text-white hover:bg-zinc-800" onclick={() => iframe.contentWindow.location.reload()}><RotateCw size={16} /></button>
            </div>
            <div class="relative w-64 group">
                <input
                    type="text"
                    class="w-full h-9 pl-3 pr-10 bg-zinc-950/50 border border-zinc-800 rounded-full text-xs text-zinc-300 focus:outline-none focus:border-[#8b5cf6]/50 focus:text-white transition-all text-center group-hover:text-left"
                    value={proxyManager.url}
                    bind:this={searchbar}
                    onkeydown={onEnterKeyPressed(() => { proxyManager.setDestination(searchbar.value); proxyManager.reloadIframe(); })}
                    placeholder="Search or enter URL"
                />
            </div>
            <div class="flex items-center gap-1 px-2 border-l border-zinc-800">
                <button class="btn btn-ghost btn-circle btn-sm text-zinc-400 hover:text-white hover:bg-zinc-800" onclick={() => (isConfigOpen = true)}><Settings size={18} /></button>
                <button class="btn btn-ghost btn-circle btn-sm text-red-400 hover:text-red-300 hover:bg-red-500/10" onclick={() => (proxyManager.isProxyOpen = false)}><X size={18} /></button>
            </div>
        </div>

    {:else if view === 'classroom'}
        <!-- Google Classroom Clone -->
        <div class="relative z-10" transition:fade={{ duration: 300 }}>
            <Classroom enterApp={() => view = 'welcome'} />
        </div>

    {:else if view === 'games'}
        <div class="relative z-10">
            <Games bind:view />
        </div>
    {:else if view === 'home'}
        <!-- Dashboard Home -->
        <div class="relative z-10 min-h-screen flex flex-col p-4" transition:fade={{ duration: 800 }}>
            <!-- Top Navigation -->
            <div class="absolute top-0 left-0 right-0 h-14 flex items-center justify-between px-6 z-20 border-b border-white/5 bg-black/20 backdrop-blur-sm">
                <div class="flex items-center gap-2">
                    <button class="p-2 text-zinc-400 hover:text-white transition-colors"><ArrowLeft size={18} /></button>
                    <button class="p-2 text-zinc-400 hover:text-white transition-colors"><RotateCw size={16} /></button>
                    <button class="p-2 text-zinc-400 hover:text-white transition-colors"><ArrowRight size={18} /></button>
                </div>
                
                <div class="flex-1 max-w-2xl mx-4">
                    <div class="relative group">
                        <Search class="absolute left-3 top-1/2 -translate-y-1/2 text-zinc-500 w-4 h-4" />
                        <input 
                            type="text" 
                            placeholder="Enter search or web address" 
                            class="w-full bg-zinc-900/50 border border-zinc-800 rounded-full py-2 pl-10 pr-4 text-sm text-zinc-300 focus:outline-none focus:border-zinc-700 transition-colors"
                            onkeydown={onEnterKeyPressed(() => startProxy(urlBar.value))}
                            bind:this={urlBar}
                        />
                    </div>
                </div>

                <div class="flex items-center gap-2">
                    <button class="p-2 text-zinc-400 hover:text-white transition-colors" onclick={() => view = 'home'}><Home size={18} /></button>
                    <button class="p-2 text-zinc-400 hover:text-white transition-colors" onclick={() => window.open('https://github.com/tenclips/verdis', '_blank')}><Code size={18} /></button>
                    <button class="p-2 text-zinc-400 hover:text-white transition-colors" onclick={toggleFullscreen}><Maximize2 size={18} /></button>
                    <button class="p-2 text-zinc-400 hover:text-white transition-colors" onclick={() => isConfigOpen = true}><Settings size={18} /></button>
                </div>
            </div>

            <!-- Main Content -->
            <div class="flex-1 flex flex-col items-center justify-center -mt-16 w-full max-w-5xl mx-auto">
                
                <!-- Center Search -->
                <div class="w-full max-w-lg mb-12 relative group">
                    <div class="relative flex items-center bg-[#0a0a0a] border border-zinc-800 rounded-full h-12 px-4 shadow-lg transition-colors group-hover:border-zinc-700">
                        <img src="/Google_Classroom_Logo.svg.png" alt="Icon" class="w-5 h-5 mr-3 opacity-80" />
                        <input
                            type="text"
                            class="w-full bg-transparent border-none text-zinc-300 placeholder-zinc-600 focus:outline-none focus:ring-0 text-sm"
                            placeholder="Search Space..."
                            bind:value={destinationInput}
                            onkeydown={onEnterKeyPressed(() => startProxy())}
                        />
                        <Search size={16} class="text-zinc-600 ml-2" />
                    </div>
                </div>

                <!-- Shortcuts Grid -->
                <div class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-3 w-full">
                    {#each shortcuts as shortcut}
                        <button 
                            class="h-16 rounded-xl flex items-center justify-center relative overflow-hidden transition-transform hover:scale-[1.02] shadow-lg {shortcut.style}"
                            onclick={() => startProxy(shortcut.url)}
                        >
                            {#if shortcut.logo}
                                <svg viewBox="0 0 24 24" class="h-6 w-auto fill-current" xmlns="http://www.w3.org/2000/svg">
                                    {@html shortcut.logo}
                                </svg>
                                {#if shortcut.name === "Google" || shortcut.name === "YouTube" || shortcut.name === "Discord" || shortcut.name === "GitHub" || shortcut.name === "Twitch" || shortcut.name === "TikTok"}
                                    <span class="ml-2 font-bold text-lg tracking-tight {shortcut.name === 'YouTube' ? 'tracking-tighter' : ''}">{shortcut.name}</span>
                                {/if}
                            {:else if shortcut.textLogo}
                                <span class="font-black text-lg italic tracking-tighter">{shortcut.textLogo}</span>
                            {:else}
                                <span class="font-bold">{shortcut.name}</span>
                            {/if}
                        </button>
                    {/each}
                </div>
            </div>
        </div>

    {:else}
        <!-- Welcome Screen -->
        <div class="relative z-10 min-h-screen flex flex-col items-center justify-center text-center p-4">
            {#if !isWarping}
                <div class="flex flex-col items-center justify-center" transition:fade={{ duration: 500 }}>
                    <h1 class="text-5xl md:text-7xl font-bold mb-4 tracking-tight text-white">
                        Welcome to <span class="text-transparent bg-clip-text bg-gradient-to-r from-purple-500 to-indigo-500">amazing.</span><span class="animate-pulse text-purple-500">|</span>
                    </h1>
                    <p class="text-zinc-400 text-lg mb-10 font-light tracking-wide">Start your unblocking journey today!</p>
                    
                    <button 
                        class="bg-gradient-to-r from-[#7c3aed] to-[#9333ea] text-white px-8 py-3.5 rounded-full font-bold text-sm transition-all duration-300 flex items-center gap-2 hover:shadow-[0_0_20px_rgba(124,58,237,0.5)] hover:scale-105 active:scale-95"
                        onclick={triggerWarp}
                    >
                        Get Started
                        <span class="text-lg">✨</span>
                    </button>
                </div>
            {/if}
        </div>
    {/if}
</div>

<style>
    /* Optimized Star Animation - GPU Accelerated */
    .star-container {
        position: absolute;
        width: 100%;
        height: 100%;
        perspective: 1000px;
        transform-style: preserve-3d;
        will-change: transform; /* Hint to browser to promote to layer */
    }

    .stars {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        height: 400%; /* Quadruple height for even smoother looping at high speeds */
        background-image: 
            radial-gradient(1.5px 1.5px at 10% 10%, #fff, rgba(0,0,0,0)),
            radial-gradient(1.5px 1.5px at 20% 30%, #fff, rgba(0,0,0,0)),
            radial-gradient(1px 1px at 30% 70%, #fff, rgba(0,0,0,0)),
            radial-gradient(2px 2px at 40% 40%, #fff, rgba(0,0,0,0)),
            radial-gradient(1.5px 1.5px at 50% 90%, #fff, rgba(0,0,0,0)),
            radial-gradient(1.5px 1.5px at 60% 20%, #fff, rgba(0,0,0,0)),
            radial-gradient(1px 1px at 70% 50%, #fff, rgba(0,0,0,0)),
            radial-gradient(2px 2px at 80% 80%, #fff, rgba(0,0,0,0)),
            radial-gradient(1px 1px at 90% 10%, #fff, rgba(0,0,0,0)),
            radial-gradient(1.5px 1.5px at 15% 15%, #fff, rgba(0,0,0,0)),
            radial-gradient(1.5px 1.5px at 25% 35%, #fff, rgba(0,0,0,0)),
            radial-gradient(2px 2px at 35% 75%, #fff, rgba(0,0,0,0)),
            radial-gradient(2px 2px at 45% 45%, #fff, rgba(0,0,0,0));
        background-size: 800px 800px; /* Larger pattern */
        background-repeat: repeat;
        animation: star-scroll 120s linear infinite;
        opacity: 0.7;
    }

    /* Warp Effect */
    .star-container.warp .stars {
        animation: star-scroll 0.2s linear infinite; /* Very fast */
        opacity: 0.8;
        transform: scaleY(15); /* Stretch vertically for trail effect */
        transform-origin: center;
        filter: blur(1px); /* Slight blur for speed sensation */
    }
    
    .star-container.warp {
        transition: all 2s ease-in;
    }

    @keyframes star-scroll {
        from { transform: translateY(0) translateZ(0); } /* Hardware accel */
        to { transform: translateY(-25%) translateZ(0); } /* Move up by 1/4 since height is 400% */
    }
</style>