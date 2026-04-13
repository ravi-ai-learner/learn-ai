```dataviewjs
// === HABIT SETTINGS ===
const habits = [
  { name: "😴", tag: "#quality_sleep" },
  { name: "🚶‍♂️", tag: "#movement" },
  { name: "💧", tag: "#hydration" },
  { name: "⏰", tag: "#early_start", weekdaysOnly: true },
  { name: "🏋️‍♂️", tag: "#fitness", weekdaysOnly: true },
  { name: "📚", tag: "#upskill", weekdaysOnly: true }
];

// === COLOR VARIABLES ===
const colorGreen = "#22c55e";
const colorRed = "#ef4444";
const colorPurple = "#a259ff";
const colorLightGray = "#e5e7eb";
const colorDarkGray = "#374151";

// === STATE ===
let weekOffset = 0;
let showDates = true;

const journalFiles = dv.pages('"Journal"').where(p => p.file?.name?.match(/\d{4}-\d{2}-\d{2}/));
const sortedFiles = journalFiles.sort(p => p.file.name);
const earliestDate = sortedFiles[0]?.file.name;
const trackingStart = earliestDate ? window.moment(earliestDate, "YYYY-MM-DD") : null;

function prefersDark() {
  return window.matchMedia('(prefers-color-scheme: dark)').matches;
}

function renderNavigation(container) {
  const navWrapper = document.createElement("div");
  navWrapper.style.position = "relative";
  navWrapper.style.marginBottom = "12.6px";
  navWrapper.style.height = "30px"; // Adjust as needed to fit both elements cleanly

  // === Show Dates Toggle (Top Right) ===
  const toggleContainer = document.createElement("div");
  toggleContainer.style.display = "flex";
  toggleContainer.style.alignItems = "center";
  toggleContainer.style.gap = "5.4px";
  toggleContainer.style.position = "absolute";
  toggleContainer.style.right = "0";
  toggleContainer.style.top = "0";

  const toggleLabel = document.createElement("span");
  toggleLabel.textContent = "Show Dates";
  toggleLabel.style.fontSize = "10.8px";
  toggleLabel.style.color = "var(--text-muted)";

  const switchWrapper = document.createElement("label");
  switchWrapper.style.position = "relative";
  switchWrapper.style.display = "inline-block";
  switchWrapper.style.width = "30.6px";
  switchWrapper.style.height = "16.2px";

  const switchInput = document.createElement("input");
  switchInput.type = "checkbox";
  switchInput.checked = showDates;
  switchInput.style.opacity = "0";
  switchInput.style.width = "0";
  switchInput.style.height = "0";

  const slider = document.createElement("span");
  slider.style.position = "absolute";
  slider.style.top = "0";
  slider.style.left = "0";
  slider.style.right = "0";
  slider.style.bottom = "0";
  slider.style.borderRadius = "30.6px";
  slider.style.backgroundColor = showDates ? colorGreen : prefersDark() ? "#555" : "#ccc";
  slider.style.transition = ".4s";

  const knob = document.createElement("span");
  knob.style.position = "absolute";
  knob.style.left = "1.8px";
  knob.style.bottom = "1.8px";
  knob.style.width = "12.6px";
  knob.style.height = "12.6px";
  knob.style.backgroundColor = "#fff";
  knob.style.borderRadius = "50%";
  knob.style.transition = ".4s";
  knob.style.transform = showDates ? "translateX(14.4px)" : "translateX(0)";

  slider.appendChild(knob);
  switchWrapper.appendChild(switchInput);
  switchWrapper.appendChild(slider);

  switchInput.addEventListener("change", () => {
    showDates = switchInput.checked;
    slider.style.backgroundColor = showDates ? colorGreen : prefersDark() ? "#555" : "#ccc";
    knob.style.transform = showDates ? "translateX(14.4px)" : "translateX(0)";
    renderTiles(container);
  });

  toggleContainer.appendChild(toggleLabel);
  toggleContainer.appendChild(switchWrapper);


  // === Week Navigation Controls (Centered) ===
  const controls = document.createElement("div");
  controls.style.display = "flex";
  controls.style.alignItems = "center";
  controls.style.justifyContent = "center";
  controls.style.gap = "7.2px";
  controls.style.position = "absolute";
  controls.style.left = "50%";
  controls.style.transform = "translateX(-50%)";
  controls.style.top = "0";

  const weekStart = window.moment().startOf("isoWeek").add(weekOffset, "weeks");
  const weekEnd = weekStart.clone().endOf("isoWeek");
  const weekRangeText = `${weekStart.format("MMM D")} – ${weekEnd.format("MMM D")}`;

  const btnStyle = btn => {
    btn.style.fontSize = "12.6px";
    btn.style.padding = "1.8px 5.4px";
    btn.style.border = "none";
    btn.style.background = "transparent";
    btn.style.cursor = "pointer";
    btn.style.color = "var(--text-muted)";
  };

  const leftBtn = document.createElement("button");
  leftBtn.textContent = "←";
  btnStyle(leftBtn);
  leftBtn.onclick = () => { weekOffset--; renderTiles(container); };

  const centerText = document.createElement("span");
  centerText.textContent = weekRangeText;
  centerText.style.fontSize = "12.6px";
  centerText.style.fontWeight = "500";

  const rightBtn = document.createElement("button");
  rightBtn.textContent = "→";
  btnStyle(rightBtn);
  rightBtn.onclick = () => { weekOffset++; renderTiles(container); };

  controls.appendChild(leftBtn);
  controls.appendChild(centerText);
  controls.appendChild(rightBtn);

  // Append both to wrapper
  navWrapper.appendChild(controls);
  navWrapper.appendChild(toggleContainer);

  return navWrapper;
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

    if (isBeforeTracking) {
	  border = `2px solid var(--divider-color, #ccc)`;
	} else if (completed) {
	  background = colorGreen;
	  content = `<svg width="10.8" height="10.8" viewBox="0 0 14 14" fill="none" stroke="#fff" stroke-width="2.25" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 7 6 10 11 4" /></svg>`;
	  textColor = colorGreen;
	} else if (skip) {
	  background = colorLightGray;
	} else if (!isFuture && trackingStart && !isToday) {
	  background = colorRed;
	  content = `<svg width="9" height="9" viewBox="0 0 12 12" fill="none" stroke="#fff" stroke-width="1.8" stroke-linecap="round"><line x1="2" y1="2" x2="10" y2="10"/><line x1="10" y1="2" x2="2" y2="10"/></svg>`;
	  textColor = colorRed;
	} else {
	  border = `2px solid var(--divider-color, #ccc)`;
	  content = showDates ? `<span style="font-size: 9px; color: ${textColor};">${date.date()}</span>` : '';
	}
	
	if (isToday) {
	  border = `2px solid ${colorPurple}`;
	}

    html += `
      <div style="display: flex; flex-direction: column; align-items: center; width: 21.6px;">
        <div style="width: 24px; height: 24px; border-radius: 50%; background: ${background}; border: ${border}; display: flex; justify-content: center; align-items: center;">
          ${content}
        </div>
        <div style="font-size: 10.8px; font-weight: 500; color: ${textColor}; margin-top: 3.6px;">${dayLabel}</div>
      </div>`;
  }

  return `<div style="display: flex; justify-content: space-between; padding: 0 1.8px; margin: 9px 0;">${html}</div>`;
}

function renderTiles(container) {
  container.innerHTML = '';
  container.appendChild(renderNavigation(container));

  const grid = document.createElement("div");
  grid.style.display = "grid";
  grid.style.gridTemplateColumns = "repeat(auto-fit, minmax(198px, 1fr))";
  grid.style.gap = "10.8px";

  for (let habit of habits) {
    const streak = getStreakData(habit.tag, habit.weekdaysOnly);
    const tile = document.createElement("div");
    tile.style.padding = "7.2px";
    tile.style.borderRadius = "10.8px";
    tile.style.boxShadow = prefersDark()
      ? "0 1.8px 9px rgba(0,0,0,0.25)"
      : "0 1.8px 9px rgba(0,0,0,0.08)";
    tile.style.background = "var(--background-primary)";

    tile.innerHTML = `
      <div style="display: flex; align-items: baseline; justify-content: space-between; margin-bottom: 3.6px;">
        <div style="display: flex; align-items: baseline; gap: 3.6px;">
          <span style="font-size: 13.5px; font-weight: 600;">${streak.consistency}</span>
          <span style="font-size: 10.8px; margin-left: -1.8px">%</span>
          <span style="font-size: 10.8px; color: var(--text-muted);">Consistency</span>
        </div>
        <div style="font-size: 21px; margin-right: 2.7px;">${habit.name}</div>
      </div>
      ${renderWeekCircles(habit.tag, habit.weekdaysOnly)}
      <div style="display: flex; justify-content: space-between; margin-top: 10.8px;">
        <div style="flex: 1; display: flex; flex-direction: column; align-items: center;">
          <div style="font-size: 10.8px; color: var(--text-muted);">Current Streak</div>
          <div style="font-size: 13.5px;"><b>${streak.current}</b> <span style="font-size: 10.8px;">days</span></div>
        </div>
        <div style="flex: 1; display: flex; flex-direction: column; align-items: center;">
          <div style="font-size: 10.8px; color: var(--text-muted);">Longest Streak</div>
          <div style="font-size: 13.5px;"><b>${streak.longest}</b> <span style="font-size: 10.8px;">days</span></div>
        </div>
      </div>
    `;

    grid.appendChild(tile);
  }

  const legend = document.createElement("div");
  legend.style.display = "flex";
  legend.style.justifyContent = "center";
  legend.style.gap = "14.4px";
  legend.style.marginTop = "18px";
  legend.style.fontSize = "10.8px";
  legend.style.color = "var(--text-muted)";

  function getIconSVG(type) {
    if (type === "check") {
      return `<svg width="10.8" height="10.8" viewBox="0 0 14 14" fill="none" stroke="#fff" stroke-width="2.25" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 7 6 10 11 4" /></svg>`;
    }
    if (type === "cross") {
      return `<svg width="9" height="9" viewBox="0 0 12 12" fill="none" stroke="#fff" stroke-width="1.8" stroke-linecap="round"><line x1="2" y1="2" x2="10" y2="10"/><line x1="10" y1="2" x2="2" y2="10"/></svg>`;
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
    itemDiv.style.gap = "5.4px";

    const dot = document.createElement("div");
    dot.style.width = "16.2px";
    dot.style.height = "16.2px";
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

  // === Add Note Below Legend ===
    if (trackingStart) {
	  const note = document.createElement("div");
	  note.style.marginTop = "9px";
	  note.style.fontSize = "10.8px";
	  note.style.color = "var(--text-muted)";
	  note.style.textAlign = "center";
	  const formattedDate = trackingStart.format("MMMM D, YYYY"); // e.g., August 1, 2025
	  note.textContent = `Note: All streak and consistency stats are measured from ${formattedDate}.`;
	  container.appendChild(note);
	}
}

function getStreakData(tag, weekdaysOnly) {
  const journalNotes = dv.pages('"Journal"')
    .where(p => p.file.name.match(/^\d{4}-\d{2}-\d{2}$/))
    .sort(p => p.file.name, 'asc');

  if (journalNotes.length === 0) {
    return { current: 0, longest: 0, completed: 0, tracked: 0, consistency: 0 };
  }

  const start = window.moment(journalNotes[0].file.name);
  const end = window.moment();
  const range = end.diff(start, 'days') + 1;

  let longest = 0;
  let currentStreak = 0;
  let previousDayWasStreak = false;
  let completed = 0;
  let tracked = 0;

  for (let i = 0; i < range; i++) {
    const date = start.clone().add(i, 'days');
    const dateStr = date.format("YYYY-MM-DD");
    const isToday = date.isSame(end, 'day');
    const isWeekend = ["0", "6"].includes(date.format("d"));
    const isTrackedDay = !weekdaysOnly || !isWeekend;

    const page = dv.page(dateStr);
    const hasPage = !!page;
    const done = hasPage && page.file?.tasks?.some(t => t.text.includes(tag) && t.completed);

    // For consistency %: count tracked days (required or voluntarily done)
    if (isTrackedDay || done) {
      if (!isToday || done) tracked++;
    }

    if (done) completed++;

    // For streaks: only include tracked days
    if (isTrackedDay || done) {
      if (done) {
        currentStreak++;
        longest = Math.max(longest, currentStreak);
      } else {
        if (!isToday) {
          currentStreak = 0;
        }
      }
    }
  }

  // Check if today breaks the streak
  const todayStr = end.format("YYYY-MM-DD");
  const todayPage = dv.page(todayStr);
  const todayDone = todayPage?.file?.tasks?.some(t => t.text.includes(tag) && t.completed);
  const current = todayDone ? currentStreak : currentStreak;

  const consistency = tracked ? Math.round((completed / tracked) * 100) : 0;

  return { current, longest, completed, tracked, consistency };
}

const container = dv.container;
container.style.padding = "3.6px";
renderTiles(container);
```
---
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

const cellSize = 24;
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

// 🔧 FIX: Backfill missing days with zero-completed habits
if (trackingStartDate) {
  const start = window.moment(trackingStartDate, "YYYY-MM-DD").clone();
  const end = today.clone().subtract(1, "day");

  for (let d = start.clone(); d.isSameOrBefore(end); d.add(1, "day")) {
    const dateStr = d.format("YYYY-MM-DD");
    if (habitData[dateStr]) continue;

    const isWeekday = !["Saturday", "Sunday"].includes(d.format("dddd"));

    let expectedHabits = 0;
    for (const tag of habitTags) {
      const isWeekdayOnly = tag.includes("#weekdays_only");
      const shouldCount = !isWeekdayOnly || isWeekday;
      if (shouldCount) expectedHabits += 1;
    }

    habitData[dateStr] = { completed: 0, expected: expectedHabits };
  }
}

const getCellColor = (dateStr, data) => {
  if (!trackingStartDate || dateStr < trackingStartDate) return "transparent";
  if (!data || data.expected === 0) return "transparent";
  const ratio = data.completed / data.expected;
  if (isToday(dateStr)) return null;
  if (dateStr >= todayStr) return "transparent"; // future
  if (ratio === 0) return colorNone;
  if (ratio < 1) return colorPartial;
  return colorComplete;
};

const isToday = (dateStr) => dateStr === todayStr;

const buildDayCell = (date, data) => {
  const dateStr = date.format("YYYY-MM-DD");
  const isTodayDate = isToday(dateStr);

  let bgColor = getCellColor(dateStr, data);
  let highlight = "";

  if (isTodayDate) {
    highlight = `border: 2px solid ${colorTodayBorder};`;
    const ratio = data?.expected > 0 ? (data.completed / data.expected) : 0;
    if (ratio === 0) {
      bgColor = "transparent";
    } else if (ratio < 1) {
      bgColor = colorPartial;
    } else {
      bgColor = colorComplete;
    }
  }

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
    color: ${bgColor === 'transparent' ? '#888' : 'white'};
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
      <div style="
		display: grid;
		grid-template-columns: repeat(7, ${cellSize}px);
		gap: 4px;
		font-size: 0.65em;
		color: #777;
		line-height: 1;
		text-align: center;
	  ">
        ${dayNames.map(d => `
		  <div style="
		    white-space: nowrap;
		    overflow: hidden;
		    text-overflow: ellipsis;
		    width: ${cellSize}px;
		    height: ${cellSize}px;
		    display: flex;
		    align-items: center;
		    justify-content: center;
		  ">${d}</div>
		`).join("")}
      </div>
      <div style="
		display: grid;
		grid-template-columns: repeat(7, ${cellSize}px);
		grid-template-rows: repeat(6, ${cellSize}px);
		gap: 4px;
		height: ${cellSize * 6 + 5 * 4}px;
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
	div.style.flex = "0 0 auto";
	wrapper.appendChild(div);
  });

  title.textContent = `Q${selectedQuarter} ${selectedYear}`;
};

// === Quarter Navigation Styled Like Week Buttons ===
const controlRow = document.createElement("div");
controlRow.style.position = "relative";
controlRow.style.height = "28px";
controlRow.style.marginBottom = "14px";

const controls = document.createElement("div");
controls.style.display = "flex";
controls.style.alignItems = "center";
controls.style.gap = "8px";

const styleNavButton = (btn) => {
  btn.style.fontSize = "12.6px";
  btn.style.padding = "1.8px 5.4px";
  btn.style.border = "none";
  btn.style.background = "transparent";
  btn.style.cursor = "pointer";
  btn.style.color = "var(--text-muted)";
};

const prevBtn = document.createElement("button");
prevBtn.textContent = "←";
styleNavButton(prevBtn);
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
styleNavButton(nextBtn);
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
title.textContent = `Q${selectedQuarter} ${selectedYear}`;
title.style.fontSize = "12.6px";
title.style.fontWeight = "500";

controls.appendChild(prevBtn);
controls.appendChild(title);
controls.appendChild(nextBtn);
controls.style.position = "absolute";
controls.style.left = "50%";
controls.style.top = "0";
controls.style.transform = "translateX(-50%)";

controlRow.appendChild(controls);

dv.container.style.padding = "4px";
dv.container.appendChild(controlRow);

// Calendar Grid Container
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
