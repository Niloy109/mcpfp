<script>
    import { page } from "$app/stores";
    import { generateQuery } from "$lib/scripts/utils";

    const overlays = ["normal", "warn", "mute", "jail", "ban", "kick"];
    const propList = [
        { id: "rose", label: "Rose" },
        { id: "crown", label: "Crown" },
        { id: "labcoat", label: "Labcoat" },
        { id: "batman", label: "Batman" },
    ];

    let user = "Dinnerbone";
    let overlay = "normal";
    let pfpType = "normal";
    let url = "";
    let no_bg = false;
    let namemc = false;
    let color1 = "";
    let color2 = "";
    let props = [];

    let copied = false;
    let loading = true;

    let query = "";
    $: query = generateQuery([
        { url },
        { "pfp-type": pfpType == "normal" ? "" : pfpType },
        { namemc: namemc ? "true" : "" },
        { "no-background": no_bg ? "true" : "" },
        { props: props.length ? props.join(",") : "" },
        {
            gradient:
                !no_bg && color1 && color2
                    ? `${color1.replace("#", "")}-${color2.replace("#", "")}`
                    : "",
        },
    ]);

    $: base = $page.url.origin + "/";
    $: path = `api/pfp/${overlay}/${user || "Steve"}.png${query}`;
    $: fullUrl = base + path;
    // show spinner whenever the preview URL changes
    $: fullUrl, (loading = true);

    async function copy() {
        try {
            await navigator.clipboard.writeText(fullUrl);
            copied = true;
            setTimeout(() => (copied = false), 1500);
        } catch (e) {
            console.log(e);
        }
    }

    function resetGradient() {
        color1 = "";
        color2 = "";
    }
</script>

<svelte:head>
    <title>mcpfp - Minecraft profile picture generator</title>
</svelte:head>

<div class="bg"></div>

<main>
    <header>
        <img class="logo" src="https://minotar.net/avatar/Suss/64" alt="" />
        <div>
            <h1>mcpfp <span class="tag">modified</span></h1>
            <p class="sub">
                Cute profile pictures from Minecraft skins. A modified version
                of
                <a href="https://github.com/MauritsWilke/mcpfp" target="_blank"
                    >MauritsWilke/mcpfp</a
                > with custom image generation.
            </p>
        </div>
    </header>

    <section class="grid">
        <!-- PLAYGROUND -->
        <div class="card playground">
            <h2>Playground</h2>

            <div class="preview-wrap">
                <div class="preview">
                    {#if loading}<span class="spinner"></span>{/if}
                    <img
                        src={fullUrl}
                        alt="Generated avatar for {user}"
                        class:hidden={loading}
                        on:load={() => (loading = false)}
                        on:error={() => (loading = false)}
                    />
                </div>
                <a class="btn ghost" href={fullUrl} download="{user || 'avatar'}.png" target="_blank">
                    Open image
                </a>
            </div>

            <div class="controls">
                <label class="field">
                    <span>Username</span>
                    <input type="text" bind:value={user} placeholder="Dinnerbone" />
                </label>

                <div class="field">
                    <span>Overlay</span>
                    <div class="chips">
                        {#each overlays as o}
                            <label class="chip" class:active={overlay === o}>
                                <input type="radio" bind:group={overlay} value={o} />
                                {o}
                            </label>
                        {/each}
                    </div>
                </div>

                <div class="field">
                    <span>Props</span>
                    <div class="chips">
                        {#each propList as p}
                            <label class="chip" class:active={props.includes(p.id)}>
                                <input type="checkbox" bind:group={props} value={p.id} />
                                {p.label}
                            </label>
                        {/each}
                    </div>
                </div>

                <div class="field">
                    <span>Avatar type</span>
                    <div class="types">
                        <button
                            type="button"
                            class="type"
                            class:active={pfpType === "normal"}
                            on:click={() => (pfpType = "normal")}
                            aria-label="Normal avatar"
                            style="background-image:url('/normal.png')"
                        ></button>
                        <button
                            type="button"
                            class="type"
                            class:active={pfpType === "chibi"}
                            on:click={() => (pfpType = "chibi")}
                            aria-label="Chibi avatar"
                            style="background-image:url('/chibi.png')"
                        ></button>
                    </div>
                </div>

                <div class="field" class:disabled={no_bg}>
                    <span>Gradient</span>
                    <div class="row">
                        <input type="color" bind:value={color1} disabled={no_bg} />
                        <input type="color" bind:value={color2} disabled={no_bg} />
                        <button type="button" class="btn ghost small" on:click={resetGradient}>Reset</button>
                    </div>
                </div>

                <label class="field">
                    <span>Skin URL <em>(optional)</em></span>
                    <input type="text" bind:value={url} placeholder="https://textures.minecraft.net/..." />
                </label>

                <div class="switches">
                    <label class="switch">
                        <input type="checkbox" bind:checked={no_bg} />
                        <i></i> No background
                    </label>
                    <label class="switch">
                        <input type="checkbox" bind:checked={namemc} />
                        <i></i> NameMC skin
                    </label>
                </div>
            </div>
        </div>

        <!-- DOCS -->
        <div class="card docs">
            <h2>API</h2>

            <h3>Base URL</h3>
            <code class="url">{base}</code>

            <h3>Endpoint</h3>
            <p>Full body skin image with types for custom image generation.</p>
            <code class="get"><b>GET</b> /api/pfp/&#123;type&#125;/&#123;username&#125;.png</code>

            <p class="types-list">
                Available types:
                {#each overlays as o}<code>{o}</code>{/each}
            </p>

            <h3>Query options</h3>
            <ul class="opts">
                <li><code>pfp-type=chibi</code> chibi avatar</li>
                <li><code>props=rose,crown</code> add cosmetics</li>
                <li><code>gradient=ff0000-0000ff</code> background colors</li>
                <li><code>no-background=true</code> transparent background</li>
                <li><code>namemc=true</code> treat username as a NameMC skin id</li>
                <li><code>url=...</code> use a skin from an allowed host</li>
            </ul>

            <h3>Your URL</h3>
            <div class="copybox">
                <code class="link">{fullUrl}</code>
                <button type="button" class="btn" on:click={copy}>
                    {copied ? "Copied!" : "Copy"}
                </button>
            </div>
        </div>
    </section>

    <footer>
        Source code:
        <a target="_blank" href="https://github.com/towsifkafi/mcpfp"
            >github.com/towsifkafi/mcpfp</a
        >
    </footer>
</main>

<style lang="scss">
    @import url("https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap");

    :global(:root) {
        --bg: #0b0b10;
        --card: rgba(255, 255, 255, 0.04);
        --card-border: rgba(255, 255, 255, 0.09);
        --field: #14141c;
        --field-border: #272734;
        --text: #ececf3;
        --muted: #9a9ab0;
        --accent: #ff9795;
        --accent2: #95ffe3;
        --gold: #ffd595;
    }
    :global(*) {
        box-sizing: border-box;
    }
    :global(body) {
        margin: 0;
        background: var(--bg);
        color: var(--text);
        font-family: "Inter", system-ui, sans-serif;
        line-height: 1.5;
        -webkit-font-smoothing: antialiased;
    }
    :global(code) {
        font-family: "JetBrains Mono", ui-monospace, monospace;
        font-size: 0.82rem;
    }

    .bg {
        position: fixed;
        inset: 0;
        z-index: -1;
        background:
            radial-gradient(600px 400px at 15% -5%, rgba(255, 151, 149, 0.18), transparent 70%),
            radial-gradient(600px 400px at 90% 10%, rgba(149, 255, 227, 0.12), transparent 70%),
            var(--bg);
    }

    main {
        max-width: 1100px;
        margin: 0 auto;
        padding: 40px 16px 32px;
    }

    header {
        display: flex;
        align-items: center;
        gap: 16px;
        margin-bottom: 28px;
    }
    .logo {
        width: 56px;
        height: 56px;
        flex-shrink: 0;
        border-radius: 12px;
        image-rendering: pixelated;
        border: 1px solid var(--card-border);
        box-shadow: 0 0 0 4px rgba(255, 151, 149, 0.12);
    }
    h1 {
        margin: 0;
        font-size: 1.8rem;
        letter-spacing: -0.02em;
    }
    .tag {
        font-size: 0.7rem;
        font-weight: 600;
        vertical-align: middle;
        color: var(--bg);
        background: var(--accent);
        padding: 2px 8px;
        border-radius: 999px;
        margin-left: 6px;
    }
    .sub {
        margin: 4px 0 0;
        color: var(--muted);
        font-size: 0.95rem;
    }

    a {
        color: var(--accent);
        text-decoration: none;
        &:hover {
            text-decoration: underline;
        }
    }

    .grid {
        display: grid;
        grid-template-columns: minmax(0, 1.05fr) minmax(0, 1fr);
        gap: 20px;
        align-items: start;
    }

    .card {
        background: var(--card);
        border: 1px solid var(--card-border);
        border-radius: 16px;
        padding: 22px;
        backdrop-filter: blur(6px);
    }
    h2 {
        margin: 0 0 16px;
        font-size: 1.05rem;
        text-transform: uppercase;
        letter-spacing: 0.08em;
        color: var(--muted);
    }
    h3 {
        margin: 18px 0 6px;
        font-size: 0.95rem;
    }
    p {
        margin: 6px 0;
        color: var(--muted);
        font-size: 0.92rem;
    }

    /* ---------- playground ---------- */
    .playground {
        display: grid;
        grid-template-columns: 300px 1fr;
        grid-template-rows: auto auto;
        column-gap: 24px;
        .controls {
            grid-column: 2;
            grid-row: 2;
        }
        h2 {
            grid-column: 1 / -1;
        }
    }
    .preview-wrap {
        grid-column: 1;
        grid-row: 2;
        display: flex;
        flex-direction: column;
        gap: 12px;
        align-items: stretch;
    }
    .preview {
        position: relative;
        width: 300px;
        height: 300px;
        border-radius: 14px;
        overflow: hidden;
        border: 1px solid var(--card-border);
        background:
            conic-gradient(#1b1b24 25%, #14141c 0 50%, #1b1b24 0 75%, #14141c 0) 0 0 / 20px 20px;
        img {
            width: 100%;
            height: 100%;
            display: block;
            image-rendering: auto;
            &.hidden {
                opacity: 0;
            }
        }
    }
    .spinner {
        position: absolute;
        top: 50%;
        left: 50%;
        width: 28px;
        height: 28px;
        margin: -14px 0 0 -14px;
        border: 3px solid rgba(255, 255, 255, 0.15);
        border-top-color: var(--accent);
        border-radius: 50%;
        animation: spin 0.7s linear infinite;
    }
    @keyframes spin {
        to {
            transform: rotate(360deg);
        }
    }

    .controls {
        display: flex;
        flex-direction: column;
        gap: 16px;
        min-width: 0;
    }
    .field {
        display: flex;
        flex-direction: column;
        gap: 6px;
        > span {
            font-size: 0.8rem;
            font-weight: 600;
            color: var(--muted);
            em {
                font-weight: 400;
                font-style: normal;
                opacity: 0.7;
            }
        }
        &.disabled {
            opacity: 0.45;
        }
    }
    .row {
        display: flex;
        gap: 8px;
        align-items: center;
    }

    input[type="text"] {
        width: 100%;
        padding: 9px 12px;
        color: var(--text);
        font: inherit;
        font-size: 0.9rem;
        background: var(--field);
        border: 1px solid var(--field-border);
        border-radius: 10px;
        outline: none;
        transition: border-color 0.15s, box-shadow 0.15s;
        &:focus {
            border-color: var(--accent);
            box-shadow: 0 0 0 3px rgba(255, 151, 149, 0.15);
        }
    }
    input[type="color"] {
        width: 44px;
        height: 36px;
        padding: 2px;
        background: var(--field);
        border: 1px solid var(--field-border);
        border-radius: 10px;
        cursor: pointer;
    }

    .chips {
        display: flex;
        flex-wrap: wrap;
        gap: 6px;
    }
    .chip {
        position: relative;
        padding: 6px 12px;
        font-size: 0.85rem;
        text-transform: capitalize;
        background: var(--field);
        border: 1px solid var(--field-border);
        border-radius: 999px;
        cursor: pointer;
        user-select: none;
        transition: all 0.15s;
        input {
            position: absolute;
            opacity: 0;
            pointer-events: none;
        }
        &:hover {
            border-color: #3a3a4d;
        }
        &.active {
            color: var(--bg);
            font-weight: 600;
            background: var(--accent);
            border-color: var(--accent);
        }
    }

    .types {
        display: flex;
        gap: 10px;
    }
    .type {
        width: 56px;
        height: 56px;
        padding: 0;
        background-size: cover;
        background-position: center;
        border: 2px solid var(--field-border);
        border-radius: 12px;
        cursor: pointer;
        transition: transform 0.15s, border-color 0.15s;
        &:hover {
            transform: translateY(-2px);
        }
        &.active {
            border-color: var(--accent);
            box-shadow: 0 0 0 3px rgba(255, 151, 149, 0.18);
        }
    }

    .switches {
        display: flex;
        flex-wrap: wrap;
        gap: 10px 22px;
    }
    .switch {
        display: flex;
        align-items: center;
        gap: 8px;
        font-size: 0.88rem;
        cursor: pointer;
        input {
            display: none;
        }
        i {
            position: relative;
            width: 36px;
            height: 20px;
            background: var(--field-border);
            border-radius: 999px;
            transition: background 0.15s;
            &::after {
                content: "";
                position: absolute;
                top: 2px;
                left: 2px;
                width: 16px;
                height: 16px;
                background: #fff;
                border-radius: 50%;
                transition: transform 0.15s;
            }
        }
        input:checked + i {
            background: var(--accent);
            &::after {
                transform: translateX(16px);
            }
        }
    }

    .btn {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        padding: 9px 16px;
        font: inherit;
        font-size: 0.88rem;
        font-weight: 600;
        color: var(--bg);
        background: var(--accent);
        border: 1px solid var(--accent);
        border-radius: 10px;
        cursor: pointer;
        white-space: nowrap;
        transition: filter 0.15s;
        &:hover {
            filter: brightness(1.1);
            text-decoration: none;
        }
        &.ghost {
            color: var(--text);
            background: transparent;
            border-color: var(--field-border);
            &:hover {
                border-color: var(--accent);
            }
        }
        &.small {
            padding: 7px 12px;
        }
    }

    /* ---------- docs ---------- */
    code {
        padding: 3px 7px;
        border-radius: 6px;
        background: var(--field);
    }
    .url {
        color: var(--gold);
        display: inline-block;
        max-width: 100%;
        overflow-wrap: anywhere;
    }
    .get {
        display: inline-block;
        max-width: 100%;
        overflow-wrap: anywhere;
        color: var(--accent);
        b {
            color: var(--accent2);
            margin-right: 4px;
        }
    }
    .types-list code {
        margin-right: 4px;
        color: var(--text);
    }
    .opts {
        margin: 0;
        padding: 0;
        list-style: none;
        display: flex;
        flex-direction: column;
        gap: 6px;
        font-size: 0.88rem;
        color: var(--muted);
        code {
            margin-right: 6px;
            color: var(--accent2);
        }
    }
    .copybox {
        display: flex;
        gap: 8px;
        align-items: stretch;
        .link {
            flex: 1;
            min-width: 0;
            padding: 10px 12px;
            color: var(--accent2);
            overflow-wrap: anywhere;
            word-break: break-word;
            border: 1px solid var(--field-border);
            border-radius: 10px;
        }
    }

    footer {
        margin-top: 28px;
        text-align: center;
        color: var(--muted);
        font-size: 0.88rem;
    }

    /* ---------- responsive ---------- */
    @media (max-width: 960px) {
        .grid {
            grid-template-columns: minmax(0, 1fr);
        }
    }
    @media (max-width: 620px) {
        main {
            padding-top: 24px;
        }
        h1 {
            font-size: 1.45rem;
        }
        .card {
            padding: 16px;
        }
        .playground {
            display: flex;
            flex-direction: column;
            gap: 18px;
        }
        .preview-wrap {
            align-items: center;
        }
        .preview {
            width: min(300px, 100%);
            height: auto;
            aspect-ratio: 1;
        }
        .copybox {
            flex-direction: column;
        }
    }
</style>
