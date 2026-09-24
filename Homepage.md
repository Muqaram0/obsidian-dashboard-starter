---
cssclasses:
  - dashboard
greeting: Hey there, let's get to work
tagline: one focused hour beats a distracted day.
exam_date: 2026-12-15
links:
  - title: STUDY
    items:
      - label: Example Note
        target: Example Note
  - title: LIFE
    items:
      - label: Daily Note
        target: "@today"
projects:
  - name: Example Project
    status: In progress
    comments: edit me on the page with the Edit button
---

```dataviewjs
// ================= SETTINGS =================
// Titles, links, projects and exam date are edited on the page (✎ Edit) and saved above.
const BANNER   = "00 Assets/spider-lily.png";
const DAILY    = "Daily";
const MACHINES = '"01 Hackbox/Machine Tracking"';
// =============================================

const css = await app.vault.adapter.read(".obsidian/snippets/dashboard.css");
let styleEl = document.getElementById("dh-style");
if (!styleEl) { styleEl = document.head.createEl("style"); styleEl.id = "dh-style"; }
styleEl.textContent = css;

const file = app.vault.getAbstractFileByPath(dv.current().file.path);

// Force reading view whenever this note is opened in Live Preview
for (const leaf of app.workspace.getLeavesOfType("markdown")) {
  if (leaf.view.file?.path === file.path && leaf.view.getMode() === "source") {
    const st = leaf.getViewState();
    await leaf.setViewState({ ...st, state: { ...st.state, mode: "preview" } });
  }
}
const fm = app.metadataCache.getFileCache(file)?.frontmatter || {};
const data = JSON.parse(JSON.stringify({
  greeting: fm.greeting || "Hey there, let's get to work",
  tagline: fm.tagline || "",
  exam_date: fm.exam_date || moment().add(90, "days").format("YYYY-MM-DD"),
  links: fm.links || [],
  projects: fm.projects || [],
}));
const persist = () => app.fileManager.processFrontMatter(file, f => Object.assign(f, JSON.parse(JSON.stringify(data))));

const open = (target) => {
  if (target === "@today") target = `${DAILY}/${moment().format("YYYY-MM-DD")}`;
  const af = app.vault.getAbstractFileByPath(target);
  if (af && af.children) {
    const fe = app.workspace.getLeavesOfType("file-explorer")[0];
    if (fe) { app.workspace.revealLeaf(fe); fe.view.revealInFolder(af); }
    return;
  }
  app.workspace.openLinkText(target, "", false);
};
const store = (k, def) => JSON.parse(localStorage.getItem(k) || JSON.stringify(def));
const put = (k, v) => localStorage.setItem(k, JSON.stringify(v));
const today = moment().format("YYYY-MM-DD");
const bar = (parent, pct, color = "") => {
  const b = parent.createDiv({ cls: "dh-bar " + color });
  b.createEl("i").style.width = `${Math.max(0, Math.min(100, pct))}%`;
};
const editable = (el, on, onSave) => {
  if (!on) return;
  el.contentEditable = "true";
  el.spellcheck = false;
  el.onkeydown = e => { if (e.key === "Enter") { e.preventDefault(); el.blur(); } };
  el.onblur = () => onSave(el.innerText.trim());
};
let drag = null;
const dropTarget = (el, onDrop) => {
  el.ondragover = e => { if (drag) { e.preventDefault(); el.classList.add("over"); } };
  el.ondragleave = () => el.classList.remove("over");
  el.ondrop = e => { e.preventDefault(); el.classList.remove("over"); if (drag) onDrop(drag); drag = null; };
};
const handle = (parent, payload) => {
  const hd = parent.createSpan({ cls: "dh-handle", text: "⠿" });
  hd.draggable = true;
  hd.ondragstart = e => { drag = payload; e.dataTransfer.effectAllowed = "move"; e.dataTransfer.setData("text/plain", ""); };
  return hd;
};
const xBtn = (parent, fn) => { const x = parent.createSpan({ cls: "dh-x", text: "×" }); x.onclick = e => { e.stopPropagation(); fn(); }; };

const root = dv.el("div", "", { cls: "dh" });
if (Date.now() - (window.__dhIntroAt || 0) > 60000) root.addClass("intro");
window.__dhIntroAt = Date.now();

// ---------- Banner + date badge ----------
const banner = root.createDiv({ cls: "dh-banner" });
banner.createDiv({ cls: "dh-banner-img" }).createEl("img", { attr: { src: app.vault.adapter.getResourcePath(BANNER) } });
const badge = banner.createDiv({ cls: "dh-datebadge" });
badge.createDiv({ cls: "m", text: moment().format("MMM").toUpperCase() });
badge.createDiv({ cls: "d", text: moment().format("D") });
badge.createDiv({ cls: "w", text: moment().format("dddd") });

const grid = root.createDiv({ cls: "dh-grid" });
const main = grid.createDiv({ cls: "dh-main" });
// Keep the Sidebar note (calendar + upcoming) open in the right sidebar
const sideFile = app.vault.getMarkdownFiles().find(f => f.basename === "Sidebar");
if (sideFile) {
  const shown = app.workspace.getLeavesOfType("markdown").some(l => l.view?.file?.path === sideFile.path && l.getRoot() === app.workspace.rightSplit);
  if (!shown) {
    const leaf = app.workspace.getRightLeaf(false);
    await leaf.openFile(sideFile, { state: { mode: "preview" }, active: false });
    app.workspace.revealLeaf(leaf);
  }
}

// ================= MAIN (re-renderable) =================
const renderMain = () => {
  const edit = store("dh-edit", false);
  main.empty();
  main.toggleClass("editing", edit);

  const top = main.createDiv({ cls: "dh-top" });
  const tbtn = top.createEl("button", { cls: "dh-btn dh-edit-btn", text: edit ? "✓ Done" : "✎ Edit" });
  tbtn.onclick = () => { put("dh-edit", !edit); renderMain(); };

  // Greeting
  const h = new Date().getHours();
  const part = h < 5 ? "Burning the midnight oil" : h < 12 ? "Good morning" : h < 17 ? "Good afternoon" : h < 21 ? "Good evening" : "Late night grind";
  const title = main.createDiv({ cls: "dh-title", text: data.greeting });
  editable(title, edit, v => { if (v && v !== data.greeting) { data.greeting = v; persist(); } });
  const sub = main.createDiv({ cls: "dh-sub" });
  sub.createSpan({ text: `${part} — ` });
  const tag = sub.createSpan({ text: data.tagline || (edit ? "add a tagline…" : "") });
  editable(tag, edit, v => { if (v !== data.tagline) { data.tagline = v; persist(); } });

  // Quote of the day
  const quotes = [
    ["Try harder.", "OffSec"],
    ["Enumeration is the key.", "Every OSCP ever"],
    ["Discipline is choosing what you want most over what you want now.", "Abraham Lincoln"],
    ["Small daily improvements lead to staggering long-term results.", "Unknown"],
    ["The quieter you become, the more you are able to hear.", "Rumi"],
    ["It always seems impossible until it's done.", "Nelson Mandela"],
    ["Amateurs hack systems, professionals hack people.", "Bruce Schneier"],
    ["Slow is smooth, smooth is fast.", "Navy SEALs"],
    ["We suffer more in imagination than in reality.", "Seneca"],
    ["You don't rise to the level of your goals, you fall to the level of your systems.", "James Clear"],
  ];
  const qd = main.createDiv({ cls: "dh-quote" });
  const showQuote = (q, who) => {
    qd.empty();
    qd.createSpan({ text: `“${q}”` });
    qd.createEl("small", { text: `— ${who}` });
    const nb = qd.createEl("button", { cls: "dh-btn dh-quote-new", text: "↻", attr: { "aria-label": "New quote" } });
    nb.onclick = () => { put("dh-quote", {}); renderMain(); };
  };
  // One quote per day from dummyjson.com, cached; built-in list if offline
  const cached = store("dh-quote", {});
  if (cached.day === today) showQuote(cached.q, cached.who);
  else {
    showQuote(...quotes[moment().dayOfYear() % quotes.length]);
    fetch("https://dummyjson.com/quotes/random")
      .then(r => r.json())
      .then(j => {
        if (!j?.quote) return;
        put("dh-quote", { day: today, q: j.quote, who: j.author });
        showQuote(j.quote, j.author);
      })
      .catch(() => {});
  }
  // Link pills
  const links = main.createDiv({ cls: "dh-links" });
  data.links.forEach((grp, g) => {
    const col = links.createDiv({ cls: "dh-col" });
    const head = col.createDiv({ cls: "dh-h", text: grp.title });
    editable(head, edit, v => { if (v && v !== grp.title) { grp.title = v; persist(); } });
    if (edit) {
      dropTarget(head, src => { if (src.type !== "pill") return; const [it] = data.links[src.g].items.splice(src.i, 1); grp.items.push(it); persist(); renderMain(); });
      xBtn(head, () => { data.links.splice(g, 1); persist(); renderMain(); });
    }
    (grp.items || []).forEach((it, i) => {
      const pill = col.createEl("a", { cls: "dh-pill" });
      if (edit) {
        handle(pill, { type: "pill", g, i });
        const lab = pill.createSpan({ text: it.label });
        editable(lab, true, v => { if (v && v !== it.label) { it.label = v; persist(); } });
        pill.setAttr("title", `→ ${it.target}`);
        xBtn(pill, () => { grp.items.splice(i, 1); persist(); renderMain(); });
        dropTarget(pill, src => {
          if (src.type !== "pill") return;
          const [moved] = data.links[src.g].items.splice(src.i, 1);
          let ti = i; if (src.g === g && src.i < i) ti--;
          grp.items.splice(ti, 0, moved); persist(); renderMain();
        });
      } else {
        pill.setText(it.label);
        pill.onclick = () => open(it.target);
      }
    });
    if (edit) {
      const wrap = col.createDiv({ cls: "dh-add-wrap" });
      const add = wrap.createEl("input", { cls: "dh-add", attr: { placeholder: "+ type a note name or path…" } });
      const list = wrap.createDiv({ cls: "dh-suggest" });
      let matches = [], sel = 0;
      const pick = (target) => {
        if (!target) return;
        (grp.items ||= []).push({ label: target.split("/").pop().replace(/\.(md|base)$/, ""), target });
        persist(); renderMain();
      };
      const draw = () => {
        list.empty();
        matches.forEach((m, k) => {
          const row = list.createDiv({ cls: "dh-sug" + (k === sel ? " sel" : "") });
          row.createDiv({ text: m.basename });
          row.createDiv({ cls: "p", text: m.parent?.path === "/" ? "/" : m.parent?.path });
          row.onmousedown = e => { e.preventDefault(); pick(m.extension === "md" ? m.path.replace(/\.md$/, "") : m.path); };
        });
      };
      add.oninput = () => {
        const q = add.value.trim().toLowerCase();
        sel = 0;
        if (!q) { matches = []; return draw(); }
        const score = f => {
          const n = (f.basename ?? f.name).toLowerCase(), p = f.path.toLowerCase();
          if (n === q) return 0; if (n.startsWith(q)) return 1; if (n.includes(q)) return 2; if (p.includes(q)) return 3; return 9;
        };
        const items = [...app.vault.getFiles().filter(f => ["md", "base"].includes(f.extension)), ...app.vault.getAllLoadedFiles().filter(f => f.children && f.path !== "/")];
        matches = items.map(f => [score(f), f]).filter(([s]) => s < 9)
          .sort((a, b) => a[0] - b[0] || a[1].path.length - b[1].path.length).slice(0, 8)
          .map(([, f]) => ({ path: f.path, parent: f.parent, basename: f.basename ?? f.name, extension: f.extension ?? "folder" }));
        draw();
      };
      add.onkeydown = e => {
        if (e.key === "ArrowDown") { e.preventDefault(); sel = Math.min(sel + 1, matches.length - 1); draw(); }
        else if (e.key === "ArrowUp") { e.preventDefault(); sel = Math.max(sel - 1, 0); draw(); }
        else if (e.key === "Escape") { add.value = ""; matches = []; draw(); }
        else if (e.key === "Enter") {
          e.preventDefault();
          const m = matches[sel];
          pick(m ? (m.extension === "md" ? m.path.replace(/\.md$/, "") : m.path) : add.value.trim());
        }
      };
      add.onblur = () => setTimeout(() => { matches = []; draw(); }, 100);
    }
  });
  if (edit) {
    const addCol = links.createDiv({ cls: "dh-col" });
    addCol.createEl("button", { cls: "dh-btn dh-ghost", text: "+ column" }).onclick = () => { data.links.push({ title: "NEW", items: [] }); persist(); renderMain(); };
  }

  // Pet + countdowns
  const row = main.createDiv({ cls: "dh-row" });
  const petCol = row.createDiv({ cls: "dh-center" });
  petCol.createDiv({ cls: "dh-h", text: "PET OF THE MONTH" });
  const petState = store("mochi-pet", { pets: 0, fed: "" });
  const hungry = petState.fed !== today;
  const card = petCol.createDiv({ cls: "dh-pet" });
  const art = card.createDiv({ cls: "dh-pet-art" + (hungry ? " hungry" : "") });
  const face = art.createEl("span", { text: hungry ? "🙀" : "🐱" });
  const pbody = card.createDiv({ cls: "dh-pet-body" });
  pbody.createDiv({ cls: "dh-pet-name" }).innerHTML = "<b>♥</b> Mochi";
  const say = pbody.createDiv({ cls: "dh-pet-say" });
  const lines = hungry
    ? ["I'm hungry… feed me?", "Feed me and I'll guard your notes."]
    : ["Purr… you got this.", "Drink some water.", "Try harder.", "Enumerate everything.", "Proud of you!"];
  say.setText(lines[Math.floor(Math.random() * lines.length)]);
  const love = pbody.createDiv();
  const loveBar = () => { love.empty(); bar(love, (petState.pets % 50) * 2); };
  loveBar();
  const pbtns = pbody.createDiv({ cls: "dh-pet-btns" });
  const petCount = pbtns.createSpan({ text: `♥ ${petState.pets}` });
  pbtns.createEl("button", { cls: "dh-btn", text: "Pet" }).onclick = () => {
    petState.pets++; put("mochi-pet", petState);
    petCount.setText(`♥ ${petState.pets}`); loveBar();
    face.setText("😻"); say.setText("Purrrr ♥");
    art.classList.remove("pop"); void art.offsetWidth; art.classList.add("pop");
    setTimeout(() => face.setText("🐱"), 1200);
  };
  pbtns.createEl("button", { cls: "dh-btn", text: "Feed 🐟" }).onclick = () => {
    petState.fed = today; put("mochi-pet", petState);
    art.classList.remove("hungry"); face.setText("😸"); say.setText("Yum! Thank you!");
  };

  const cdCol = row.createDiv({ cls: "dh-center" });
  cdCol.createDiv({ cls: "dh-h", text: "UP NEXT" });
  cdCol.createDiv({ cls: "dh-cd-head", text: "Coming up on your calendar:" });
  const nextList = cdCol.createDiv();
  (async () => {
    const start = moment().startOf("day");
    const entries = [];
    // Calendar days (daily notes dated today or later)
    for (const f of app.vault.getMarkdownFiles()) {
      if (!f.path.startsWith(DAILY + "/")) continue;
      const day = moment(f.basename, "YYYY-MM-DD", true);
      if (!day.isValid() || day.isBefore(start)) continue;
      const text = (await app.vault.cachedRead(f)).replace(/^---[\s\S]*?---/, "")
        .split("\n").map(l => l.replace(/^[#>\-*\s]+|\[[ x]\]\s*/g, "").trim()).find(Boolean);
      if (text) entries.push({ day, text, path: f.path });
    }
    // Tasks with due dates
    dv.pages().file.tasks.where(t => !t.completed && t.due && t.due >= dv.date("today")).forEach(t =>
      entries.push({ day: moment(t.due.toISODate()), text: t.text.replace(/📅\s*\d{4}-\d{2}-\d{2}|\[due::.*?\]/g, "").trim(), path: t.path }));
    entries.sort((a, b) => a.day - b.day);
    if (!entries.length) { nextList.createDiv({ cls: "dh-empty", text: "Nothing planned — click a day in the calendar to add something." }); return; }
    entries.slice(0, 4).forEach(e => {
      const c = nextList.createDiv({ cls: "dh-cd" });
      const l = c.createEl("a", { cls: "l", text: e.text.length > 34 ? e.text.slice(0, 33) + "…" : e.text });
      l.onclick = () => open(e.path);
      const n = e.day.diff(start, "days");
      c.createSpan({ cls: "r", text: n === 0 ? "today" : n === 1 ? "tomorrow" : `${n} days` });
    });
  })();

  // Project tracking
  main.createDiv({ cls: "dh-h", text: "PROJECT TRACKING" });
  const STATUSES = ["Not started", "In progress", "On hold", "Done"];
  const table = main.createEl("table", { cls: "dh-table" });
  const hr = table.createEl("tr");
  [`Project (${data.projects.length})`, "Status", "Comments"].forEach(t => hr.createEl("th", { text: t }));
  data.projects.forEach((p, i) => {
    const tr = table.createEl("tr");
    const nameTd = tr.createEl("td");
    if (edit) {
      handle(nameTd, { type: "proj", i });
      dropTarget(tr, src => {
        if (src.type !== "proj") return;
        const [m] = data.projects.splice(src.i, 1);
        data.projects.splice(src.i < i ? i - 1 : i, 0, m); persist(); renderMain();
      });
    }
    const nm = nameTd.createSpan({ text: p.name });
    const st = tr.createEl("td").createSpan({
      cls: "dh-status s-" + (p.status || STATUSES[0]).toLowerCase().replace(/\s+/g, "-"),
      text: p.status || STATUSES[0],
      attr: { "aria-label": "Click to change status" },
    });
    st.onclick = () => { p.status = STATUSES[(STATUSES.indexOf(p.status) + 1) % STATUSES.length]; persist(); renderMain(); };
    const cmTd = tr.createEl("td", { cls: "dh-muted" });
    const cm = cmTd.createSpan({ text: p.comments || (edit ? "add a comment…" : "") });
    if (edit) {
      editable(nm, true, v => { if (v && v !== p.name) { p.name = v; persist(); } });
      editable(cm, true, v => { if (v !== (p.comments || "")) { p.comments = v; persist(); } });
      xBtn(cmTd, () => { data.projects.splice(i, 1); persist(); renderMain(); });
    }
  });
  if (edit) {
    main.createEl("button", { cls: "dh-btn dh-ghost", text: "+ project" }).onclick = () => {
      data.projects.push({ name: "New project", status: "Not started", comments: "" });
      persist(); renderMain();
    };
  }

  // Recently edited (chips)
  main.createDiv({ cls: "dh-h", text: "RECENTLY EDITED" });
  const rc = main.createDiv({ cls: "dh-recent-chips" });
  dv.pages().where(p => !["Homepage", "Sidebar"].includes(p.file.name)).sort(p => p.file.mtime, "desc").limit(6).forEach(p => {
    const c = rc.createEl("a", { cls: "dh-chip" });
    c.createSpan({ text: p.file.name });
    c.createEl("small", { text: moment(p.file.mtime.toMillis()).fromNow() });
    c.onclick = () => open(p.file.path);
  });
};
renderMain();
```
