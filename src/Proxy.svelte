<script lang="ts">
    import { splashText } from "./verdis";
    import proxyManager from "./proxy.svelte";

    let iframe: HTMLIFrameElement = $state();
    let iframeHasLoaded = $state(false);

    const iframeAllow =
        "accelerometer ambient-light-sensor attribution-reporting autoplay bluetooth browsing-topics camera compute-pressure " +
        "cross-origin-isolated display-capture document-domain encrypted-media fullscreen gamepad geolocation gyroscope hid " +
        "identity-credentials-get idle-detection local-fonts magnetometer microphone midi otp-credentials payment " +
        "picture-in-picture publickey-credentials-create publickey-credentials-get screen-wake-lock serial speaker-selection " +
        "storage-access usb web-share window-management xr-spatial-tracking";

    const iframeSandbox =
        "allow-popups allow-popups-to-escape-sandbox allow-downloads allow-forms allow-modals allow-orientation-lock " +
        "allow-pointer-lock allow-presentation allow-same-origin allow-scripts allow-storage-access-by-user-activation";

    function onIframeLoad() {
        if (!iframe) return;

        // Try to suppress alerts (only works for same-origin)
        try {
            const win = iframe.contentWindow;
            if (win) {
                win.alert = (msg) => console.log("Suppressed alert:", msg);
                win.confirm = (msg) => { console.log("Suppressed confirm:", msg); return true; };
            }
        } catch (e) {
            // Ignore cross-origin blocking
        }

        try {
            const src = iframe.contentWindow.location.pathname;
            
            // For UV proxy, check if the path includes prefix
            if (src.includes(proxyManager.uvConfig.prefix)) {
                iframeHasLoaded = true;
                proxyManager.url = proxyManager.uvConfig.decodeUrl(
                    src.slice(proxyManager.uvConfig.prefix.length),
                );
            } else {
                // If it's a direct load (same-origin but not proxy path), consider it loaded
                iframeHasLoaded = true;
            }
        } catch (e) {
            // Cross-origin access failed (Scramjet or No Proxy to external site)
            // This means the iframe has loaded successfully
            iframeHasLoaded = true;
        }
    }
</script>

<div class="w-screen h-screen fixed top-0 left-0">
    {#if !iframeHasLoaded}
        <div
            class="bg-base-200 w-full h-full flex flex-col items-center justify-center gap-5"
        >
            <span class="loading loading-spinner loading-xl mb-15"></span>
            <p>{splashText[Math.floor(Math.random() * splashText.length)]}</p>
            <button
                class="btn"
                onclick={() => (proxyManager.isProxyOpen = false)}
                >Not working? Click here to go back</button
            >
        </div>
    {/if}
    <iframe
        bind:this={iframe}
        title="Proxy"
        class="w-full h-full"
        src={proxyManager.iframeUrl}
        onload={onIframeLoad}
        allow={iframeAllow}
        sandbox={iframeSandbox}
    ></iframe>
</div>