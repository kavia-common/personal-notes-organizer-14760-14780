<script lang="ts">
    import '../app.css';

    // Define the color theme at the layout level for CSS isolation
    const themeStyle = `
        --header-height: 64px;
        --footer-height: 40px;
        --sidebar-width: 300px;
    `;
    let { children } = $props();
</script>

<div class="app" style={themeStyle}>
    <header class="app-header">
        <div class="header-left">
            <div class="logo">🗒️</div>
            <div class="title">
                <h1>Notes</h1>
                <p class="subtitle">Minimal personal notes</p>
            </div>
        </div>
        <div class="header-actions">
            <a class="link" href="https://svelte.dev" target="_blank" rel="noreferrer">Svelte</a>
        </div>
    </header>

    <main class="app-main">
        {@render children()}
    </main>

    <footer class="app-footer">
        <div class="container">
            <span>© {new Date().getFullYear()} Notes App</span>
            <span class="dot">•</span>
            <span>Built with Svelte</span>
        </div>
    </footer>
</div>

<style>
    .app {
        min-height: 100vh;
        display: grid;
        grid-template-rows: var(--header-height) 1fr var(--footer-height);
        background: var(--color-background);
        color: var(--color-text-primary);
    }

    .app-header {
        position: sticky;
        top: 0;
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 0 16px;
        border-bottom: 1px solid var(--color-border);
        background: linear-gradient(180deg, #ffffff, #fbfbfb);
        backdrop-filter: blur(6px);
        z-index: 10;
    }

    .header-left {
        display: flex;
        align-items: center;
        gap: 12px;
    }

    .logo {
        height: 36px;
        width: 36px;
        display: grid;
        place-items: center;
        border-radius: 10px;
        background: var(--color-primary);
        color: var(--color-text-inverse);
        box-shadow: var(--shadow-sm);
        font-size: 18px;
    }

    .title h1 {
        margin: 0;
        font-size: 18px;
        font-weight: 700;
        letter-spacing: 0.2px;
    }

    .subtitle {
        margin: 0;
        color: var(--color-text-secondary);
        font-size: 12px;
    }

    .header-actions .link {
        color: var(--color-primary);
        font-weight: 600;
        padding: 8px 10px;
        border-radius: 8px;
        transition: background var(--transition);
    }
    .header-actions .link:hover {
        background: rgba(25,118,210,0.08);
    }

    .app-main {
        min-height: calc(100vh - var(--header-height) - var(--footer-height));
    }

    .app-footer {
        display: flex;
        align-items: center;
        border-top: 1px solid var(--color-border);
        background: var(--color-surface);
        color: var(--color-text-secondary);
        font-size: 12px;
    }

    .app-footer .container {
        width: 100%;
        padding: 0 16px;
        display: flex;
        gap: 8px;
        align-items: center;
        justify-content: center;
    }

    .dot {
        color: #bdbdbd;
    }
</style>
