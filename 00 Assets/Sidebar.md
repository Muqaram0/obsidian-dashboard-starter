---
cssclasses:
  - dashboard
  - dashboard-side
---

```dataviewjs
const DAILY = "Daily";
const DAYS_AHEAD = 45;

const css = await app.vault.adapter.read(".obsidian/snippets/dashboard.css");
let styleEl = document.getElementById("dh-style");
if (!styleEl) { styleEl = document.head.createEl("style"); styleEl.id = "dh-style"; }
styleEl.textContent = css;

// Always open in the main editor area, never inside this sidebar pane
const open = async (target) => {
  const leaf = app.workspace.getMostRecentLeaf(app.workspace.rootSplit) || app.workspace.getLeaf("tab");
  app.workspace.setActiveLeaf(leaf, { focus: true });
  await app.workspace.openLinkText(target, "", false);
};
const root = dv.el("div", "", { cls: "dh dh-sb" });

// Tasks with a due date (📅 2026-09-30 or [due:: 2026-09-30]); tasks in a daily note are due that day
const dailyDate = t => {
  const m = t.path.match(new RegExp(`^${DAILY}/(\\d{4}-\\d{2}-\\d{2})\\.md$`));
  return m ? dv.date(m[1]) : null;
};
const allTasks = dv.pages().file.tasks
  .where(t => !t.completed)
  .map(t => { if (!t.due) t.due = dailyDate(t); return t; })
  .where(t => t.due);
const dueOn = {};
allTasks.forEach(t => { const k = t.due.toISODate(); (dueOn[k] ||= []).push(t); });

// ---------- Calendar ----------
const calWrap = root.createDiv();
let cur = moment().startOf("month");
const renderCal = () => {
  calWrap.empty();
  const head = calWrap.createDiv({ cls: "dh-cal-head" });
  head.createSpan({ cls: "dh-cal-title", text: cur.format("MMMM YYYY") });
  const nav = head.createDiv({ cls: "nav" });
  nav.createEl("button", { cls: "dh-btn", text: "‹" }).onclick = () => { cur.subtract(1, "month"); renderCal(); };
  nav.createEl("button", { cls: "dh-btn", text: "Today" }).onclick = () => { cur = moment().startOf("month"); renderCal(); };
  nav.createEl("button", { cls: "dh-btn", text: "›" }).onclick = () => { cur.add(1, "month"); renderCal(); };
  const g = calWrap.createDiv({ cls: "dh-cal" });
  ["W", "SUN", "MON", "TUE", "WED", "THU", "FRI", "SAT"].forEach(t => g.createDiv({ cls: "dow", text: t }));
  const start = cur.clone().startOf("month").startOf("week");
  for (let w = 0; w < 6; w++) {
    const ws = start.clone().add(w, "weeks");
    g.createDiv({ cls: "wk", text: ws.format("w") });
    for (let i = 0; i < 7; i++) {
      const day = ws.clone().add(i, "days");
      const key = day.format("YYYY-MM-DD");
      const path = `${DAILY}/${key}`;
      let cls = "day";
      if (!day.isSame(cur, "month")) cls += " out";
      if (day.isSame(moment(), "day")) cls += " today";
      if (dv.page(path) || dueOn[key]) cls += " note";
      const cell = g.createDiv({ cls, text: day.format("D") });
      if (dueOn[key]) cell.setAttr("title", dueOn[key].map(t => t.text).join("\n"));
      cell.onclick = () => open(path);
    }
  }
};
renderCal();

// ---------- Focus timer (state in localStorage so it survives re-renders) ----------
const store = (k, def) => JSON.parse(localStorage.getItem(k) || JSON.stringify(def));
const put = (k, v) => localStorage.setItem(k, JSON.stringify(v));
const today = moment().format("YYYY-MM-DD");
const focus = root.createDiv({ cls: "dh-sb-focus" });
focus.createDiv({ cls: "dh-h", text: "FOCUS" });
const clock = focus.createDiv({ cls: "dh-pomo-clock" });
const prow = focus.createDiv({ cls: "dh-pomo-row" });
const pinfo = focus.createDiv({ cls: "dh-pomo-info" });
const T = () => store("dh-timer", { mode: "focus", left: 1500, endAt: null });
const remaining = s => s.endAt ? Math.max(0, Math.ceil((s.endAt - Date.now()) / 1000)) : s.left;
const startBtn = prow.createEl("button", { cls: "dh-btn" });
const tick = () => {
  let s = T();
  if (s.endAt && remaining(s) === 0) {
    const ps = store("mochi-pomo", {});
    if (s.mode === "focus") { ps[today] = (ps[today] || 0) + 1; put("mochi-pomo", ps); new Notice("🍅 Focus done! Mochi is proud. Take 5."); s = { mode: "break", left: 300, endAt: null }; }
    else { new Notice("☕ Break over, back to work!"); s = { mode: "focus", left: 1500, endAt: null }; }
    put("dh-timer", s);
  }
  const r = remaining(s);
  clock.setText(`${String(Math.floor(r / 60)).padStart(2, "0")}:${String(r % 60).padStart(2, "0")}`);
  startBtn.setText(s.endAt ? "❚❚ Pause" : "▶ Start");
  const n = store("mochi-pomo", {})[today] || 0;
  pinfo.setText(`${n} session${n === 1 ? "" : "s"} today · ${s.mode}`);
};
startBtn.onclick = () => {
  const s = T();
  put("dh-timer", s.endAt ? { ...s, left: remaining(s), endAt: null } : { ...s, endAt: Date.now() + s.left * 1000 });
  tick();
};
prow.createEl("button", { cls: "dh-btn", text: "↺ Reset" }).onclick = () => { put("dh-timer", { mode: "focus", left: 1500, endAt: null }); tick(); };
tick();
const iv = setInterval(() => { if (!clock.isConnected) return clearInterval(iv); tick(); }, 1000);

// ---------- Toolbar ----------
const bar = root.createDiv({ cls: "dh-sb-bar" });
const tool = (icon, tip, fn) => { const b = bar.createEl("button", { cls: "dh-btn", text: icon, attr: { "aria-label": tip } }); b.onclick = fn; };
tool("＋", "Today's note", async () => {
  const path = `${DAILY}/${moment().format("YYYY-MM-DD")}.md`;
  if (!app.vault.getAbstractFileByPath(path)) await app.vault.create(path, "- [ ] ");
  open(path);
});
tool("⌂", "Homepage", () => open("Homepage"));
tool("↻", "Refresh", () => app.workspace.trigger("dataview:refresh-views"));

// ---------- Timeline ----------
const tl = root.createDiv({ cls: "dh-tl" });
const upcoming = allTasks
  .where(t => t.due <= dv.date("today").plus({ days: DAYS_AHEAD }))
  .sort(t => t.due, "asc");
if (!upcoming.length) {
  tl.createDiv({ cls: "dh-empty", text: "Nothing due. Add a task in any note like:" });
  tl.createEl("code", { text: "- [ ] Root Jeeves 📅 2026-09-30" });
}
let lastDate = "";
upcoming.forEach(t => {
  const dd = moment(t.due.toISODate());
  const label = dd.format("MMMM D");
  if (label !== lastDate) { tl.createDiv({ cls: "dh-tl-date" + (dd.isBefore(moment(), "day") ? " late" : ""), text: label }); lastDate = label; }
  const it = tl.createDiv({ cls: "dh-tl-item" });
  it.createDiv({ cls: "ic", text: "⏳" });
  const body = it.createDiv();
  body.createEl("a", { cls: "dh-tl-text", text: t.text.replace(/📅\s*\d{4}-\d{2}-\d{2}|\[due::.*?\]/g, "").trim() }).onclick = () => open(t.path);
  const meta = body.createDiv({ cls: "meta" });
  meta.appendText(`✏️📅 ${dd.isSame(moment(), "day") ? "today" : dd.fromNow()}  `);
  const src = meta.createEl("a", { text: `📄 ${t.path.split("/").pop().replace(".md", "")}` });
  src.onclick = () => open(t.path);
});
```
