```dataviewjs
// CONFIG
const folder = "Journal";
const tag = "#daily_notes";
const vault = "Obsidian Vault";
const habitTags = [
  "#early_start", "#quality_sleep",
  "#fitness", "#upskill", "#movement", "#hydration"
];

const today = window.moment();
const todayStr = today.format("YYYY-MM-DD");

let selectedYear = today.year();
let selectedQuarter = Math.floor(today.month() / 3) + 1;

const cellSize = 28;
const dayNames = ["MON", "TUE", "WED", "THU", "FRI", "SAT", "SUN"];

const colorNone = "#f44336";
const colorPartial = "#FFAA33";
const colorComplete = "#30c750";
const colorTodayBorder = "#a259ff";

// Load files
const files = dv.pages(`"${folder}"`)
  .where(p => p.file.tags?.includes(tag))
  .where(p => p.file.name.match(/^\d{4}-\d{2}-\d{2}$/));

const habitData = {};
let trackingStartDate = null;

for (let p of files) {
  const date = p.file.name;
  const momentDate = window.moment(date, "YYYY-MM-DD");
  const isWeekday = !["Saturday", "Sunday"].includes(momentDate.format("dddd"));

  if (!trackingStartDate || date < trackingStartDate) {
    trackingStartDate = date;
  }

  const allTasks = p.file.tasks;

  let expectedHabits = 0;
  let completedHabits = 0;

  for (const tag of habitTags) {
    const task = allTasks.find(t => t.text.includes(tag));
    if (!task) continue;

    const isWeekdayOnly = task.text.includes("#weekdays_only");
    const shouldCount = !isWeekdayOnly || isWeekday;

    if (shouldCount) {
      expectedHabits += 1;
      if (task.completed) completedHabits += 1;
    }
  }

  habitData[date] = { completed: completedHabits, expected: expectedHabits };
}

const getCellColor = (dateStr, data) => {
  if (!trackingStartDate || dateStr < trackingStartDate) return "transparent";
  if (dateStr >= todayStr) return "transparent";
  if (!data || data.expected === 0) return "transparent";

  const ratio = data.completed / data.expected;
  if (ratio === 0) return colorNone;
  if (ratio < 1) return colorPartial;
  return colorComplete;
};

const isToday = (dateStr) => dateStr === todayStr;

const buildDayCell = (date, data) => {
  const dateStr = date.format("YYYY-MM-DD");
  const bgColor = getCellColor(dateStr, data);
  const highlight = isToday(dateStr) ? `border: 2px solid ${colorTodayBorder};` : "";

  const filePath = `${folder}/${dateStr}.md`;
  const encodedLink = `obsidian://open?vault=${encodeURIComponent(vault)}&file=${encodeURIComponent(filePath)}`;

  return `<a href="${encodedLink}" target="_blank" style="
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    width: ${cellSize}px;
    height: ${cellSize}px;
    margin: 1px;
    background: ${bgColor};
    color: ${bgColor === "transparent" ? '#888' : 'white'};
    font-size: 0.75em;
    text-decoration: none;
    border-radius: 6px;
    ${highlight}
  ">
    ${date.date()}
  </a>`;
};

const buildCalendar = (month) => {
  const first = month.clone().startOf("month");
  const last = month.clone().endOf("month");
  const days = [];
  const startWeekday = first.isoWeekday();

  for (let i = 1; i < startWeekday; i++) {
    days.push(`<div style="width: ${cellSize}px; height: ${cellSize}px; margin: 1px;"></div>`);
  }

  for (let d = first.clone(); d.isSameOrBefore(last); d.add(1, "day")) {
    const dateStr = d.format("YYYY-MM-DD");
    const data = habitData[dateStr] ?? { completed: 0, expected: 0 };
    days.push(buildDayCell(d, data));
  }

  while (days.length < 42) {
    days.push(`<div style="width: ${cellSize}px; height: ${cellSize}px; margin: 1px;"></div>`);
  }

  return `
    <div style="display: flex; flex-direction: column; align-items: center; margin: 4px;">
      <div style="font-weight: 600; font-size: 1em; margin: 6px 0; text-align: center;">
        ${month.format("MMM")} <span style="color: ${colorTodayBorder}; font-weight: 700;">${month.format("YYYY")}</span>
      </div>
      <div style="display: grid; grid-template-columns: repeat(7, ${cellSize}px); gap: 4px; font-size: 0.65em; color: #777;">
        ${dayNames.map(d => `<div style="text-align:center;">${d}</div>`).join("")}
      </div>
      <div style="
		display: grid;
		grid-template-columns: repeat(7, ${cellSize}px);
		grid-template-rows: repeat(6, ${cellSize}px);
		gap: 4px;
		height: ${cellSize * 6 + 5 * 4}px; /* 6 rows + 5 gaps of 4px */
	  ">
        ${days.join("")}
      </div>
    </div>
  `;
};

const renderCalendars = () => {
  wrapper.innerHTML = "";

  const quarterStart = (selectedQuarter - 1) * 3;
  const months = [0, 1, 2].map(i =>
    window.moment().year(selectedYear).month(quarterStart + i)
  );

  months.forEach(month => {
    const cal = buildCalendar(month);
    const div = document.createElement("div");
	div.innerHTML = cal;
	div.style.transform = "scale(0.97)";
	div.style.transformOrigin = "top left";
	div.style.flex = "0 0 auto";  // Ensures flexbox respects width
	wrapper.appendChild(div);
  });

  title.textContent = `Q${selectedQuarter} ${selectedYear}`;
};

// UI elements
const controlRow = document.createElement("div");
controlRow.style.display = "flex";
controlRow.style.justifyContent = "center";
controlRow.style.alignItems = "center";
controlRow.style.gap = "12px";
controlRow.style.marginBottom = "7px";

const prevBtn = document.createElement("button");
prevBtn.textContent = "←";
prevBtn.onclick = () => {
  if (selectedQuarter === 1) {
    selectedQuarter = 4;
    selectedYear -= 1;
  } else {
    selectedQuarter -= 1;
  }
  renderCalendars();
};

const nextBtn = document.createElement("button");
nextBtn.textContent = "→";
nextBtn.onclick = () => {
  if (selectedQuarter === 4) {
    selectedQuarter = 1;
    selectedYear += 1;
  } else {
    selectedQuarter += 1;
  }
  renderCalendars();
};

const title = document.createElement("span");
title.style.fontWeight = "bold";
title.style.fontSize = "1em";
title.style.minWidth = "100px";
title.style.textAlign = "center";

controlRow.appendChild(prevBtn);
controlRow.appendChild(title);
controlRow.appendChild(nextBtn);

dv.container.style.padding = "4px";

dv.container.appendChild(controlRow);

// Wrapper for calendars
const wrapper = document.createElement("div");
wrapper.style.display = "grid";
wrapper.style.gridTemplateColumns = "repeat(3, 1fr)";
wrapper.style.gap = "10px";
wrapper.style.maxWidth = "100%";
wrapper.style.overflowX = "hidden";

dv.container.appendChild(wrapper);
renderCalendars();

// LEGEND
const legend = document.createElement("div");
legend.innerHTML = `
  <div style="margin-top: 12px; text-align: center; font-size: 0.75em; color: #666;">
    <div style="display: flex; justify-content: center; align-items: center; gap: 14px; flex-wrap: wrap;">
      <div style="display: flex; align-items: center; gap: 4px;">
        <div style="width: 16px; height: 16px; background: ${colorNone}; border-radius: 4px;"></div>
        <span>No habits completed</span>
      </div>
      <div style="display: flex; align-items: center; gap: 4px;">
        <div style="width: 16px; height: 16px; background: ${colorPartial}; border-radius: 4px;"></div>
        <span>Some habits completed</span>
      </div>
      <div style="display: flex; align-items: center; gap: 4px;">
        <div style="width: 16px; height: 16px; background: ${colorComplete}; border-radius: 4px;"></div>
        <span>All habits completed</span>
      </div>
      <div style="display: flex; align-items: center; gap: 4px;">
        <div style="width: 16px; height: 16px; border: 2px solid ${colorTodayBorder}; border-radius: 4px;"></div>
        <span>Today</span>
      </div>
    </div>
  </div>
`;

dv.container.appendChild(legend);
```
---
```dataviewjs
// === HABIT SETTINGS ===
const habits = [
  { name: "⏰", tag: "#early_start" },
  { name: "😴", tag: "#quality_sleep" },
  { name: "🏋️‍♂️", tag: "#fitness", weekdaysOnly: true },
  { name: "📚", tag: "#upskill", weekdaysOnly: true },
  { name: "🚶‍♂️", tag: "#movement" },
  { name: "💧", tag: "#hydration" }
];

// === COLOR VARIABLES ===
const colorGreen = "#22c55e";
const colorRed = "#ef4444";
const colorPurple = "#a259ff";
const colorLightGray = "#e5e7eb";
const colorDarkGray = "#374151";

// === STATE ===
let weekOffset = 0;

const journalFiles = dv.pages('"Journal"').where(p => p.file?.name?.match(/\d{4}-\d{2}-\d{2}/));
const sortedFiles = journalFiles.sort(p => p.file.name);
const earliestDate = sortedFiles[0]?.file.name;
const trackingStart = earliestDate ? window.moment(earliestDate, "YYYY-MM-DD") : null;

function prefersDark() {
  return window.matchMedia('(prefers-color-scheme: dark)').matches;
}

function renderNavigation(container) {
  const nav = document.createElement("div");
  nav.style.display = "flex";
  nav.style.justifyContent = "center";
  nav.style.alignItems = "center";
  nav.style.gap = "12px";
  nav.style.marginBottom = "14px";

  const weekStart = window.moment().startOf("isoWeek").add(weekOffset, "weeks");
  const weekEnd = weekStart.clone().endOf("isoWeek");
  const weekRangeText = `${weekStart.format("MMM D")} - ${weekEnd.format("MMM D")}`;

  const leftBtn = document.createElement("button");
  leftBtn.textContent = "←";
  leftBtn.onclick = () => {
    weekOffset--;
    renderTiles(container);
  };

  const centerText = document.createElement("div");
  centerText.textContent = weekRangeText;
  centerText.style.fontSize = "16px";
  centerText.style.fontWeight = "600";

  const rightBtn = document.createElement("button");
  rightBtn.textContent = "→";
  rightBtn.onclick = () => {
    weekOffset++;
    renderTiles(container);
  };

  nav.appendChild(leftBtn);
  nav.appendChild(centerText);
  nav.appendChild(rightBtn);

  return nav;
}

function renderWeekCircles(tag, weekdaysOnly) {
  const today = window.moment().format("YYYY-MM-DD");
  const weekStart = window.moment().startOf('isoWeek').add(weekOffset, 'weeks');

  let html = '';
  for (let i = 0; i < 7; i++) {
    const date = weekStart.clone().add(i, 'days');
    const dateStr = date.format("YYYY-MM-DD");
    const isToday = dateStr === today;
    const isWeekend = ["0", "6"].includes(date.format("d"));
    const dayLabel = date.format("dd")[0];
    const isBeforeTracking = !trackingStart || date.isBefore(trackingStart, 'day');
    const isFuture = date.isAfter(today, 'day');

    const skip = weekdaysOnly && isWeekend;

    let background = "var(--background-primary)";
    let content = '';
    let border = '';
    let textColor = "var(--text-muted)";

    const page = dv.page(dateStr);
    const hasPage = !!page;
    const completed = hasPage && page.file?.tasks?.some(t => t.text.includes(tag) && t.completed);

    if (skip) {
      background = colorLightGray;
    } else if (isBeforeTracking) {
      border = `1px solid var(--divider-color, #ccc)`;
    } else if (completed) {
      background = colorGreen;
      content = `<svg width="12" height="12" viewBox="0 0 14 14" fill="none" stroke="#fff" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 7 6 10 11 4" /></svg>`;
      textColor = colorGreen;
    } else if (!isFuture && trackingStart && !isToday) {
      background = colorRed;
      content = `<svg width="10" height="10" viewBox="0 0 12 12" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round"><line x1="2" y1="2" x2="10" y2="10"/><line x1="10" y1="2" x2="2" y2="10"/></svg>`;
      textColor = colorRed;
    } else {
      border = `1px solid var(--divider-color, #ccc)`;
      content = `<span style="font-size: 10px; color: ${textColor};">${date.date()}</span>`;
    }

    // Always apply purple border for today
    if (isToday) {
      border = `2px solid ${colorPurple}`;
    }

    html += `
      <div style="display: flex; flex-direction: column; align-items: center; width: 24px;">
        <div style="width: 21px; height: 21px; border-radius: 50%; background: ${background}; border: ${border}; display: flex; justify-content: center; align-items: center;">
          ${content}
        </div>
        <div style="font-size: 12px; font-weight: 500; color: ${textColor}; margin-top: 4px;">${dayLabel}</div>
      </div>`;
  }

  return `<div style="display: flex; justify-content: space-between; padding: 0 2px; margin: 10px 0;">${html}</div>`;
}

function renderTiles(container) {
  container.innerHTML = '';
  container.appendChild(renderNavigation(container));

  const grid = document.createElement("div");
  grid.style.display = "grid";
  grid.style.gridTemplateColumns = "repeat(auto-fit, minmax(220px, 1fr))";
  grid.style.gap = "12px";

  for (let habit of habits) {
    const streak = getStreakData(habit.tag, habit.weekdaysOnly);
    const tile = document.createElement("div");
    tile.style.padding = "8px";
    tile.style.borderRadius = "12px";
    tile.style.boxShadow = prefersDark()
	    ? "0 2px 10px rgba(0,0,0,0.25)"
	    : "0 2px 10px rgba(0,0,0,0.08)";
    tile.style.background = "var(--background-primary)";

    tile.innerHTML = `
	  <div style="display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 4px;">
	    <div style="display: flex; align-items: baseline; gap: 4px;">
	      <span style="font-size: 15px; font-weight: 600;">${streak.consistency}</span>
	      <span style="font-size: 12px; margin-left: -2px">%</span>
	      <span style="font-size: 12px; color: var(--text-muted);">Consistency</span>
	    </div>
	    <div style="font-size: 20px; margin-right: 3px;">${habit.name}</div>
	  </div>
	  ${renderWeekCircles(habit.tag, habit.weekdaysOnly)}
	  <div style="display: flex; justify-content: space-between; margin-top: 12px;">
	    <div style="flex: 1; display: flex; flex-direction: column; align-items: center;">
	      <div style="font-size: 12px; color: var(--text-muted);">Current Streak</div>
	      <div style="font-size: 15px;"><b>${streak.current}</b> <span style="font-size: 12px;">days</span></div>
	    </div>
	    <div style="flex: 1; display: flex; flex-direction: column; align-items: center;">
	      <div style="font-size: 12px; color: var(--text-muted);">Longest Streak</div>
	      <div style="font-size: 15px;"><b>${streak.longest}</b> <span style="font-size: 12px;">days</span></div>
	    </div>
	  </div>
	`;

    grid.appendChild(tile);
  }

  const legend = document.createElement("div");
  legend.style.display = "flex";
  legend.style.justifyContent = "center";
  legend.style.gap = "16px";
  legend.style.marginTop = "20px";
  legend.style.fontSize = "12px";
  legend.style.color = "var(--text-muted)";

  function getIconSVG(type) {
    if (type === "check") {
      return `<svg width="12" height="12" viewBox="0 0 14 14" fill="none" stroke="#fff" stroke-width="2.5" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 7 6 10 11 4" /></svg>`;
    }
    if (type === "cross") {
      return `<svg width="10" height="10" viewBox="0 0 12 12" fill="none" stroke="#fff" stroke-width="2" stroke-linecap="round"><line x1="2" y1="2" x2="10" y2="10"/><line x1="10" y1="2" x2="2" y2="10"/></svg>`;
    }
    return "";
  }

  const legendItems = [
    { color: colorRed, label: "Missed", icon: getIconSVG("cross") },
    { color: colorLightGray, label: "Not Tracked", icon: "" },
    { color: colorGreen, label: "Completed", icon: getIconSVG("check") },
    {
      color: "transparent",
      label: "Today",
      icon: "",
      border: `2px solid ${colorPurple}`
    }
  ];

  for (let item of legendItems) {
    const itemDiv = document.createElement("div");
    itemDiv.style.display = "flex";
    itemDiv.style.alignItems = "center";
    itemDiv.style.gap = "6px";

    const dot = document.createElement("div");
    dot.style.width = "21px";
    dot.style.height = "21px";
    dot.style.borderRadius = "50%";
    dot.style.background = item.color;
    dot.style.display = "flex";
    dot.style.alignItems = "center";
    dot.style.justifyContent = "center";

    if (item.border) {
      dot.style.border = item.border;
    }

    dot.innerHTML = item.icon;
    itemDiv.appendChild(dot);
    itemDiv.appendChild(document.createTextNode(item.label));
    legend.appendChild(itemDiv);
  }

  container.appendChild(grid);
  container.appendChild(legend);
}

function getStreakData(tag, weekdaysOnly) {
  const journalNotes = dv.pages('"Journal"')
    .where(p => p.file.name.match(/^\d{4}-\d{2}-\d{2}$/))
    .sort(p => p.file.name, 'asc');

  if (journalNotes.length === 0) return { current: 0, longest: 0, completed: 0, tracked: 0, consistency: 0 };

  const start = window.moment(journalNotes[0].file.name);
  const end = window.moment(); // today
  const range = end.diff(start, 'days') + 1;

  let longest = 0;
  let streak = 0;
  let completed = 0;
  let tracked = 0;
  let streakUntilYesterday = 0;

  for (let i = 0; i < range; i++) {
    const date = start.clone().add(i, 'days');
    const dateStr = date.format("YYYY-MM-DD");
    const isToday = date.isSame(end, 'day');
    const isWeekend = ["0", "6"].includes(date.format("d"));
    const skip = weekdaysOnly && isWeekend;
    if (skip) continue;

    const page = dv.page(dateStr);
    const hasPage = !!page;
    const done = hasPage && page.file?.tasks?.some(t => t.text.includes(tag) && t.completed);

    if (!isToday || done) tracked++;

    if (done) {
      streak++;
      completed++;
      longest = Math.max(longest, streak);
    } else {
      if (!isToday) {
        streak = 0;
        streakUntilYesterday = 0;
      } else {
        streakUntilYesterday = streak;
      }
    }
  }

  const today = end.format("YYYY-MM-DD");
  const todayPage = dv.page(today);
  const todayDone = todayPage?.file?.tasks?.some(t => t.text.includes(tag) && t.completed);

  const current = todayDone ? streak : streakUntilYesterday;
  const consistency = tracked ? Math.round((completed / tracked) * 100) : 0;

  return { current, longest, completed, tracked, consistency };
}

const container = dv.container;
container.style.padding = "4px";
renderTiles(container);
```
