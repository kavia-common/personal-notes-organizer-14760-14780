<script lang="ts">
    import { browser } from '$app/environment';

    type Note = {
        id: string;
        title: string;
        content: string;
        createdAt: number;
        updatedAt: number;
        pinned?: boolean;
        color?: string | null;
    };

    // Keys and helpers for storage
    const STORAGE_KEY = 'notes-app-v1';

    function loadNotes(): Note[] {
        if (!browser) return [];
        try {
            const raw = localStorage.getItem(STORAGE_KEY);
            if (!raw) return [];
            const parsed = JSON.parse(raw);
            return Array.isArray(parsed) ? parsed : [];
        } catch {
            return [];
        }
    }

    function saveNotes(notes: Note[]) {
        if (!browser) return;
        localStorage.setItem(STORAGE_KEY, JSON.stringify(notes));
    }

    // App state
    let notes: Note[] = loadNotes();
    let query = '';
    let selectedId: string | null = notes.length ? notes[0].id : null;
    let sortBy: 'updated' | 'created' | 'alpha' = 'updated';
    let showPinnedOnly = false;

    // Persist notes to localStorage whenever they change (only in browser)
    $: if (browser) saveNotes(notes);

    function createNote() {
        const ts = Date.now();
        const note: Note = {
            id: crypto.randomUUID(),
            title: 'Untitled',
            content: '',
            createdAt: ts,
            updatedAt: ts,
            pinned: false,
            color: null
        };
        notes = [note, ...notes];
        selectedId = note.id;
        saveNotes(notes);
    }

    function updateSelected(partial: Partial<Note>) {
        if (!selectedId) return;
        notes = notes.map(n => n.id === selectedId ? { ...n, ...partial, updatedAt: Date.now() } : n);
        saveNotes(notes);
    }

    function deleteSelected() {
        if (!selectedId) return;
        const idx = notes.findIndex(n => n.id === selectedId);
        if (idx === -1) return;
        const next = [...notes];
        next.splice(idx, 1);
        notes = next;
        selectedId = notes[idx]?.id ?? notes[idx - 1]?.id ?? null;
        saveNotes(notes);
    }

    function selectNote(id: string) {
        selectedId = id;
    }

    function togglePin(id?: string) {
        const targetId = id ?? selectedId;
        if (!targetId) return;
        notes = notes.map(n => n.id === targetId ? { ...n, pinned: !n.pinned, updatedAt: Date.now() } : n);
        saveNotes(notes);
    }

    function setColor(color: string | null) {
        if (!selectedId) return;
        notes = notes.map(n => n.id === selectedId ? { ...n, color, updatedAt: Date.now() } : n);
        saveNotes(notes);
    }

    // Filtering and sorting utilities
    function normalize(s: string) {
        return s.toLowerCase().trim();
    }
    function matches(n: Note, q: string) {
        if (!q) return true;
        const t = normalize(n.title);
        const c = normalize(n.content);
        const qq = normalize(q);
        return t.includes(qq) || c.includes(qq);
    }

    function sorted(list: Note[]): Note[] {
        const collator = new Intl.Collator(undefined, { sensitivity: 'base' });
        const data = [...list];
        if (sortBy === 'updated') {
            data.sort((a, b) => Number(b.pinned) - Number(a.pinned) || b.updatedAt - a.updatedAt);
        } else if (sortBy === 'created') {
            data.sort((a, b) => Number(b.pinned) - Number(a.pinned) || b.createdAt - a.createdAt);
        } else {
            data.sort((a, b) => Number(b.pinned) - Number(a.pinned) || collator.compare(a.title, b.title));
        }
        return data;
    }

    $: filtered = sorted(notes.filter((n) => matches(n, query) && (!showPinnedOnly || n.pinned)));

    $: selected = notes.find((n) => n.id === selectedId) ?? null;

    // UI helpers
    function fmtDate(ts: number) {
        const d = new Date(ts);
        return d.toLocaleString();
    }

    function onTitleInput(e: Event) {
        const value = (e.target as HTMLInputElement).value;
        updateSelected({ title: value });
    }

    function onContentInput(e: Event) {
        const value = (e.target as HTMLTextAreaElement).value;
        updateSelected({ content: value });
    }
</script>

<svelte:head>
    <title>Notes — Minimal</title>
    <meta name="description" content="Personal notes app with create, edit, delete and search features." />
</svelte:head>

<div class="layout">
    <aside class="sidebar">
        <div class="sidebar-header">
            <button class="btn primary" on:click={createNote} aria-label="Create new note">
                <span class="plus">+</span>
                New Note
            </button>
        </div>

        <div class="search">
            <input
                type="search"
                placeholder="Search notes..."
                bind:value={query}
                aria-label="Search notes"
            />
            <div class="filters">
                <label class="checkbox">
                    <input type="checkbox" bind:checked={showPinnedOnly} />
                    <span>Show pinned</span>
                </label>

                <div class="segmented">
                    <button class:active={sortBy === 'updated'} on:click={() => (sortBy = 'updated')} title="Sort by last updated">Updated</button>
                    <button class:active={sortBy === 'created'} on:click={() => (sortBy = 'created')} title="Sort by created date">Created</button>
                    <button class:active={sortBy === 'alpha'} on:click={() => (sortBy = 'alpha')} title="Sort alphabetically">A–Z</button>
                </div>
            </div>
        </div>

        <ul class="list">
            {#if filtered.length === 0}
                <li class="empty">No notes found.</li>
            {/if}
            {#each filtered as n (n.id)}
                <li
                    class="list-item {n.id === selectedId ? 'selected' : ''}"
                    style={`--note-color:${n.color ?? 'transparent'}`}
                    aria-current={n.id === selectedId ? 'true' : 'false'}
                >
                    <button class="item-click" on:click={() => selectNote(n.id)} aria-label={`Select note ${n.title || 'Untitled'}`}>
                        <div class="left">
                            <div class="swatch" title="Note color"></div>
                            <div class="meta">
                                <div class="title">
                                    {n.title || '(Untitled)'}
                                    {#if n.pinned}
                                        <span class="pin" title="Pinned">📌</span>
                                    {/if}
                                </div>
                                <div class="date">Updated {fmtDate(n.updatedAt)}</div>
                            </div>
                        </div>
                        <div class="right">
                            <div class="preview">{n.content ? n.content.slice(0, 60) : 'No content'}</div>
                        </div>
                    </button>
                </li>
            {/each}
        </ul>
    </aside>

    <section class="editor">
        {#if selected}
            <div class="editor-header">
                <input
                    class="title-input"
                    placeholder="Note title"
                    value={selected.title}
                    on:input={onTitleInput}
                />
                <div class="actions">
                    <div class="color-palette" title="Set note color">
                        <button
                            class="color none"
                            on:click={() => setColor(null)}
                            aria-label="No color"
                            title="No color"
                        ></button>
                        {#each ['#FFF9C4', '#E3F2FD', '#E8F5E9', '#FCE4EC', '#FFF3E0'] as c (c)}
                            <button
                                class="color"
                                style={`--c:${c}`}
                                aria-label={`Color ${c}`}
                                title={c}
                                on:click={() => setColor(c)}
                            ></button>
                        {/each}
                    </div>
                    <button class="btn ghost" on:click={() => togglePin()} aria-label="Toggle pin">{selected.pinned ? 'Unpin' : 'Pin'}</button>
                    <button class="btn danger" on:click={deleteSelected} aria-label="Delete note">Delete</button>
                </div>
            </div>

            <textarea
                class="content"
                placeholder="Start typing your note..."
                value={selected.content}
                on:input={onContentInput}
            ></textarea>
        {:else}
            <div class="empty-editor">
                <p>No note selected.</p>
                <button class="btn primary" on:click={createNote}>Create your first note</button>
            </div>
        {/if}
    </section>
</div>

<style>
    .layout {
        height: 100%;
        display: grid;
        grid-template-columns: 320px 1fr;
        overflow: hidden;
    }

    /* Sidebar */
    .sidebar {
        border-right: 1px solid var(--color-border);
        background: var(--color-surface);
        display: grid;
        grid-template-rows: auto auto 1fr;
        min-height: calc(100vh - var(--header-height) - var(--footer-height));
    }

    .sidebar-header {
        padding: 12px;
        border-bottom: 1px solid var(--color-border);
        display: flex;
        gap: 8px;
    }

    .btn {
        display: inline-flex;
        align-items: center;
        gap: 8px;
        padding: 8px 12px;
        border-radius: var(--radius-md);
        border: 1px solid var(--color-border);
        background: white;
        color: var(--color-text-primary);
        cursor: pointer;
        transition: transform var(--transition), background var(--transition), color var(--transition), border-color var(--transition), box-shadow var(--transition);
        box-shadow: var(--shadow-sm);
    }
    .btn:hover { transform: translateY(-1px); }
    .btn:active { transform: translateY(0); }

    .btn.primary {
        background: var(--color-primary);
        border-color: var(--color-primary);
        color: var(--color-text-inverse);
        box-shadow: 0 6px 16px rgba(25,118,210,0.25);
    }

    .btn.danger {
        background: #E53935;
        border-color: #E53935;
        color: #fff;
        box-shadow: 0 6px 16px rgba(229,57,53,0.18);
    }

    .btn.ghost {
        background: transparent;
        color: var(--color-secondary);
    }

    .plus {
        height: 22px;
        width: 22px;
        display: grid;
        place-items: center;
        background: rgba(255,255,255,0.18);
        border-radius: 6px;
        font-weight: 700;
    }

    .search {
        padding: 12px;
        border-bottom: 1px solid var(--color-border);
        display: grid;
        gap: 10px;
        background: white;
    }

    .search input[type="search"] {
        width: 100%;
        padding: 10px 12px;
        border-radius: var(--radius-md);
        border: 1px solid var(--color-border);
        outline: none;
        background: #fff;
        color: var(--color-text-primary);
        transition: border-color var(--transition), box-shadow var(--transition);
    }

    .search input[type="search"]:focus {
        border-color: var(--color-primary);
        box-shadow: 0 0 0 3px rgba(25,118,210,0.12);
    }

    .filters {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 8px;
    }

    .checkbox {
        display: inline-flex;
        align-items: center;
        gap: 8px;
        color: var(--color-secondary);
        font-size: 13px;
    }

    .segmented {
        display: inline-flex;
        border: 1px solid var(--color-border);
        border-radius: 10px;
        overflow: hidden;
        background: #fff;
    }
    .segmented button {
        padding: 6px 10px;
        border: none;
        background: transparent;
        color: var(--color-secondary);
        cursor: pointer;
        font-size: 12px;
    }
    .segmented button.active {
        background: rgba(25,118,210,0.08);
        color: var(--color-primary);
        font-weight: 600;
    }

    .list {
        overflow: auto;
        padding: 6px;
        display: grid;
        gap: 6px;
        margin: 0;
        list-style: none;
    }

    .list-item {
        width: 100%;
        border: 1px solid var(--color-border);
        background: #fff;
        border-radius: var(--radius-md);
        box-shadow: var(--shadow-sm);
    }

    .item-click {
        width: 100%;
        padding: 10px;
        display: grid;
        grid-template-columns: 1fr auto;
        gap: 8px;
        text-align: left;
        cursor: pointer;
        background: transparent;
        border: none;
        border-radius: var(--radius-md);
        transition: transform var(--transition), box-shadow var(--transition), border-color var(--transition), background var(--transition);
    }

    .item-click:hover { transform: translateY(-1px); }
    .list-item:hover { box-shadow: var(--shadow-md); }
    .list-item.selected { border-color: var(--color-primary); box-shadow: 0 0 0 3px rgba(25,118,210,0.12); }

    .left { display: flex; gap: 10px; align-items: center; }
    .meta { display: grid; gap: 2px; }

    .title {
        font-weight: 600;
        color: var(--color-text-primary);
        display: inline-flex;
        align-items: center;
        gap: 6px;
    }

    .pin {
        color: var(--color-accent);
        font-size: 14px;
    }

    .date {
        color: var(--color-text-secondary);
        font-size: 12px;
    }

    .right .preview {
        color: var(--color-text-secondary);
        font-size: 12px;
        max-width: 180px;
        text-overflow: ellipsis;
        white-space: nowrap;
        overflow: hidden;
    }

    .swatch {
        height: 14px;
        width: 14px;
        border-radius: 4px;
        background: var(--note-color);
        border: 1px solid var(--color-border);
    }

    /* Editor Area */
    .editor {
        min-width: 0;
        display: grid;
        grid-template-rows: auto 1fr;
        background: linear-gradient(180deg, #ffffff, #fdfdfd);
    }

    .editor-header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 12px;
        padding: 16px;
        border-bottom: 1px solid var(--color-border);
        background: #fff;
    }

    .title-input {
        width: 100%;
        padding: 10px 12px;
        border-radius: var(--radius-md);
        border: 1px solid var(--color-border);
        outline: none;
        background: #fff;
        color: var(--color-text-primary);
        font-weight: 700;
        font-size: 18px;
        transition: border-color var(--transition), box-shadow var(--transition);
    }
    .title-input:focus {
        border-color: var(--color-primary);
        box-shadow: 0 0 0 3px rgba(25,118,210,0.12);
    }

    .actions {
        display: inline-flex;
        align-items: center;
        gap: 8px;
    }

    .color-palette {
        display: inline-flex;
        gap: 6px;
        align-items: center;
        padding-right: 4px;
        margin-right: 8px;
        border-right: 1px dashed var(--color-border);
    }

    .color {
        height: 24px;
        width: 24px;
        border-radius: 50%;
        border: 1px solid var(--color-border);
        background: var(--c);
        cursor: pointer;
    }
    .color.none {
        background:
          linear-gradient(135deg, transparent 45%, #e57373 45% 55%, transparent 55%),
          #fff;
    }

    .content {
        width: 100%;
        height: 100%;
        padding: 18px 20px 80px;
        border: none;
        resize: none;
        outline: none;
        background: #fff;
        color: var(--color-text-primary);
        line-height: 1.6;
        font-size: 16px;
        caret-color: var(--color-primary);
    }

    .empty-editor {
        height: 100%;
        display: grid;
        place-items: center;
        gap: 12px;
        color: var(--color-text-secondary);
    }

    /* Responsive */
    @media (max-width: 900px) {
        .layout {
            grid-template-columns: 1fr;
        }
        .sidebar {
            min-height: auto;
        }
        .editor {
            min-height: 60vh;
        }
    }
</style>
