```dataviewjs
// ===== SHARED CONFIG =====
const folder = "Journal";
const habitTags = [
  { name: "😴", tag: "#quality_sleep" },
  { name: "🚶‍♂️", tag: "#movement" },
  { name: "💧", tag: "#hydration" },
  { name: "⏰", tag: "#early_start", weekdaysOnly: true },
  { name: "🏋️‍♂️", tag: "#fitness", weekdaysOnly: true },
  { name: "📚", tag: "#upskill", weekdaysOnly: true }
];
const colorGreen = "#22c55e";
const colorRed = "#ef4444";
const colorPurple = "#a259ff";
const colorLightGray = "#e5e7eb";
const colorDarkGray = "#374151";
const colorNone = "#f44336";
const colorPartial = "#FFAA33";
const colorComplete = "#30c750";
const colorTodayBorder = "#a259ff";
// ===== SHARED STATE =====
const today = window.moment();
const todayStr = today.format("YYYY-MM-DD");
let weekOffset = 0;
let monthOffset = 0;
let hideDates = true;
let showMonthly = false;
let selectedYear = today.year();
let selectedQuarter = Math.floor(today.month() / 3) + 1;
let showHeatmap = false;
let customTrackingStart = null; // New variable for custom tracking start date

// Load journal files once
const journalFiles = dv.pages('"Journal"').where(p => p.file?.name?.match(/\d{4}-\d{2}-\d{2}/));
const sortedFiles = journalFiles.sort(p => p.file.name);
const earliestDate = sortedFiles.length > 0 ? window.moment(sortedFiles[0].file.name, "YYYY-MM-DD") : null;
// const earliestDate = sortedFiles.length > 0 ? window.moment("2025-09-01", "YYYY-MM-DD") : null;
const latestDate = window.moment(sortedFiles[sortedFiles.length - 1]?.file.name, "YYYY-MM-DD") || today;

// Default tracking start is the earliest date or today if no files exist
let trackingStart = customTrackingStart || earliestDate || today;

// ===== STREAK VIEW CODE =====
function prefersDark() {
  return window.matchMedia('(prefers-color-scheme: dark)').matches;
}

function renderDatePicker(container) {
  const pickerContainer = document.createElement("div");
  pickerContainer.style.display = "flex";
  pickerContainer.style.alignItems = "center";
  pickerContainer.style.gap = "9px";
  pickerContainer.style.marginRight = "6px"; // Add some margin to separate from other toggles

  const pickerLabel = document.createElement("span");
  pickerLabel.textContent = "Since:";
  pickerLabel.style.fontSize = "10.8px";
  pickerLabel.style.color = "var(--text-muted)";

  // Create a wrapper for better styling control
  const pickerWrapper = document.createElement("div");
  pickerWrapper.style.position = "relative";
  pickerWrapper.style.display = "flex";
  pickerWrapper.style.alignItems = "center";

  const pickerInput = document.createElement("input");
  pickerInput.type = "date";
  pickerInput.style.padding = "3.6px 7.2px 3.6px 25px"; // Add left padding for the icon
  pickerInput.style.borderRadius = "4.5px";
  pickerInput.style.border = "1px solid var(--divider-color)";
  pickerInput.style.background = "var(--background-primary)";
  pickerInput.style.color = "var(--text-normal)";
  pickerInput.style.fontSize = "12px";
  pickerInput.style.width = "110px"; // Fixed width for consistency
  pickerInput.style.height = "24px"; // Fixed height
  pickerInput.style.appearance = "none"; // Remove default styling
  pickerInput.style.webkitAppearance = "none"; // For Safari

  // Set min/max dates based on available journal files
  if (earliestDate) {
    pickerInput.min = earliestDate.format("YYYY-MM-DD");
  }
  if (latestDate) {
    pickerInput.max = latestDate.format("YYYY-MM-DD");
  }

  // Set initial value
  pickerInput.value = trackingStart.format("YYYY-MM-DD");

  // Add calendar icon (since the native one might be hidden)
  const calendarIcon = document.createElement("div");
  calendarIcon.style.position = "absolute";
  calendarIcon.style.left = "8px";
  calendarIcon.style.color = "var(--text-muted)";
  calendarIcon.style.fontSize = "12px";
  calendarIcon.style.pointerEvents = "none";
  calendarIcon.textContent = "🗓️";

  pickerWrapper.appendChild(pickerInput);
  pickerWrapper.appendChild(calendarIcon);

  pickerInput.addEventListener("change", (e) => {
    const newDate = window.moment(e.target.value, "YYYY-MM-DD");
    if (newDate.isValid()) {
      customTrackingStart = newDate;
      trackingStart = newDate;
      renderStreakTiles(streakContainer);
      renderHeatmapCalendars();
    }
  });

  pickerContainer.appendChild(pickerLabel);
  pickerContainer.appendChild(pickerWrapper);
  return pickerContainer;
}

function renderToggleGroup(container) {
  const toggleContainer = document.createElement("div");
  toggleContainer.style.display = "flex";
  toggleContainer.style.alignItems = "center";
  toggleContainer.style.gap = "18px";
  toggleContainer.style.position = "absolute";
  toggleContainer.style.left = "0";
  toggleContainer.style.top = "0";

  // Date Picker (new addition)
  toggleContainer.appendChild(renderDatePicker(container));

  // Monthly View Toggle
  const monthlyToggle = document.createElement("div");
  monthlyToggle.style.display = "flex";
  monthlyToggle.style.alignItems = "center";
  monthlyToggle.style.gap = "5.4px";
  const monthlyLabel = document.createElement("span");
  monthlyLabel.textContent = "Month View";
  monthlyLabel.style.fontSize = "10.8px";
  monthlyLabel.style.color = "var(--text-muted)";
  const monthlySwitchWrapper = document.createElement("label");
  monthlySwitchWrapper.style.position = "relative";
  monthlySwitchWrapper.style.display = "inline-block";
  monthlySwitchWrapper.style.width = "30.6px";
  monthlySwitchWrapper.style.height = "16.2px";
  const monthlySwitchInput = document.createElement("input");
  monthlySwitchInput.type = "checkbox";
  monthlySwitchInput.checked = showMonthly;
  monthlySwitchInput.style.opacity = "0";
  monthlySwitchInput.style.width = "0";
  monthlySwitchInput.style.height = "0";
  const monthlySlider = document.createElement("span");
  monthlySlider.style.position = "absolute";
  monthlySlider.style.top = "0";
  monthlySlider.style.left = "0";
  monthlySlider.style.right = "0";
  monthlySlider.style.bottom = "0";
  monthlySlider.style.borderRadius = "30.6px";
  monthlySlider.style.backgroundColor = showMonthly ? colorGreen : prefersDark() ? "#555" : "#ccc";
  monthlySlider.style.transition = ".4s";
  const monthlyKnob = document.createElement("span");
  monthlyKnob.style.position = "absolute";
  monthlyKnob.style.left = "1.8px";
  monthlyKnob.style.bottom = "1.8px";
  monthlyKnob.style.width = "12.6px";
  monthlyKnob.style.height = "12.6px";
  monthlyKnob.style.backgroundColor = "#fff";
  monthlyKnob.style.borderRadius = "50%";
  monthlyKnob.style.transition = ".4s";
  monthlyKnob.style.transform = showMonthly ? "translateX(14.4px)" : "translateX(0)";
  monthlySlider.appendChild(monthlyKnob);
  monthlySwitchWrapper.appendChild(monthlySwitchInput);
  monthlySwitchWrapper.appendChild(monthlySlider);
  monthlySwitchInput.addEventListener("change", () => {
    showMonthly = monthlySwitchInput.checked;
    monthlySlider.style.backgroundColor = showMonthly ? colorGreen : prefersDark() ? "#555" : "#ccc";
    monthlyKnob.style.transform = showMonthly ? "translateX(14.4px)" : "translateX(0)";
    weekOffset = 0;
    monthOffset = 0;
    renderStreakTiles(streakContainer);
  });
  monthlyToggle.appendChild(monthlyLabel);
  monthlyToggle.appendChild(monthlySwitchWrapper);

  // Heatmap Toggle
  const heatmapToggle = document.createElement("div");
  heatmapToggle.style.display = "flex";
  heatmapToggle.style.alignItems = "center";
  heatmapToggle.style.gap = "5.4px";
  const heatmapLabel = document.createElement("span");
  heatmapLabel.textContent = "Heatmap";
  heatmapLabel.style.fontSize = "10.8px";
  heatmapLabel.style.color = "var(--text-muted)";
  const heatmapSwitchWrapper = document.createElement("label");
  heatmapSwitchWrapper.style.position = "relative";
  heatmapSwitchWrapper.style.display = "inline-block";
  heatmapSwitchWrapper.style.width = "30.6px";
  heatmapSwitchWrapper.style.height = "16.2px";
  const heatmapSwitchInput = document.createElement("input");
  heatmapSwitchInput.type = "checkbox";
  heatmapSwitchInput.checked = showHeatmap;
  heatmapSwitchInput.style.opacity = "0";
  heatmapSwitchInput.style.width = "0";
  heatmapSwitchInput.style.height = "0";
  const heatmapSlider = document.createElement("span");
  heatmapSlider.style.position = "absolute";
  heatmapSlider.style.top = "0";
  heatmapSlider.style.left = "0";
  heatmapSlider.style.right = "0";
  heatmapSlider.style.bottom = "0";
  heatmapSlider.style.borderRadius = "30.6px";
  heatmapSlider.style.backgroundColor = showHeatmap ? colorGreen : prefersDark() ? "#555" : "#ccc";
  heatmapSlider.style.transition = ".4s";
  const heatmapKnob = document.createElement("span");
  heatmapKnob.style.position = "absolute";
  heatmapKnob.style.left = "1.8px";
  heatmapKnob.style.bottom = "1.8px";
  heatmapKnob.style.width = "12.6px";
  heatmapKnob.style.height = "12.6px";
  heatmapKnob.style.backgroundColor = "#fff";
  heatmapKnob.style.borderRadius = "50%";
  heatmapKnob.style.transition = ".4s";
  heatmapKnob.style.transform = showHeatmap ? "translateX(14.4px)" : "translateX(0)";
  heatmapSlider.appendChild(heatmapKnob);
  heatmapSwitchWrapper.appendChild(heatmapSwitchInput);
  heatmapSwitchWrapper.appendChild(heatmapSlider);
  heatmapSwitchInput.addEventListener("change", () => {
    showHeatmap = heatmapSwitchInput.checked;
    heatmapSlider.style.backgroundColor = showHeatmap ? colorGreen : prefersDark() ? "#555" : "#ccc";
    heatmapKnob.style.transform = showHeatmap ? "translateX(14.4px)" : "translateX(0)";
    updateHeatmapVisibility();
  });
  heatmapToggle.appendChild(heatmapLabel);
  heatmapToggle.appendChild(heatmapSwitchWrapper);

  // Show Dates Toggle
  const datesToggle = document.createElement("div");
  datesToggle.style.display = "flex";
  datesToggle.style.alignItems = "center";
  datesToggle.style.gap = "5.4px";
  const datesLabel = document.createElement("span");
  datesLabel.textContent = "Dates";
  datesLabel.style.fontSize = "10.8px";
  datesLabel.style.color = "var(--text-muted)";
  const datesSwitchWrapper = document.createElement("label");
  datesSwitchWrapper.style.position = "relative";
  datesSwitchWrapper.style.display = "inline-block";
  datesSwitchWrapper.style.width = "30.6px";
  datesSwitchWrapper.style.height = "16.2px";
  const datesSwitchInput = document.createElement("input");
  datesSwitchInput.type = "checkbox";
  datesSwitchInput.checked = hideDates;
  datesSwitchInput.style.opacity = "0";
  datesSwitchInput.style.width = "0";
  datesSwitchInput.style.height = "0";
  const datesSlider = document.createElement("span");
  datesSlider.style.position = "absolute";
  datesSlider.style.top = "0";
  datesSlider.style.left = "0";
  datesSlider.style.right = "0";
  datesSlider.style.bottom = "0";
  datesSlider.style.borderRadius = "30.6px";
  datesSlider.style.backgroundColor = hideDates ? colorGreen : prefersDark() ? "#555" : "#ccc";
  datesSlider.style.transition = ".4s";
  const datesKnob = document.createElement("span");
  datesKnob.style.position = "absolute";
  datesKnob.style.left = "1.8px";
  datesKnob.style.bottom = "1.8px";
  datesKnob.style.width = "12.6px";
  datesKnob.style.height = "12.6px";
  datesKnob.style.backgroundColor = "#fff";
  datesKnob.style.borderRadius = "50%";
  datesKnob.style.transition = ".4s";
  datesKnob.style.transform = hideDates ? "translateX(14.4px)" : "translateX(0)";
  datesSlider.appendChild(datesKnob);
  datesSwitchWrapper.appendChild(datesSwitchInput);
  datesSwitchWrapper.appendChild(datesSlider);
  datesSwitchInput.addEventListener("change", () => {
    hideDates = datesSwitchInput.checked;
    datesSlider.style.backgroundColor = hideDates ? colorGreen : prefersDark() ? "#555" : "#ccc";
    datesKnob.style.transform = hideDates ? "translateX(14.4px)" : "translateX(0)";
    renderStreakTiles(streakContainer);
  });
  datesToggle.appendChild(datesLabel);
  datesToggle.appendChild(datesSwitchWrapper);

  // Add all toggles to container
  toggleContainer.appendChild(datesToggle);
  toggleContainer.appendChild(monthlyToggle);
  toggleContainer.appendChild(heatmapToggle);

  return toggleContainer;
}

function updateHeatmapVisibility() {
  if (showHeatmap) {
    heatmapContainer.style.display = "block";
  } else {
    heatmapContainer.style.display = "none";
  }
}

function renderLegend(container) {
  const legend = document.createElement("div");
  legend.style.display = "flex";
  legend.style.justifyContent = "center";
  legend.style.gap = "14.4px";
  legend.style.marginTop = "18px";
  legend.style.fontSize = "0.75em";
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
    { color: "transparent", label: "Today", icon: "", border: `2px solid ${colorPurple}` }
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
    if (item.border) dot.style.border = item.border;
    dot.innerHTML = item.icon;
    itemDiv.appendChild(dot);
    itemDiv.appendChild(document.createTextNode(item.label));
    legend.appendChild(itemDiv);
  }

  container.appendChild(legend);
}

function renderDateNavigation() {
  const navContainer = document.createElement("div");
  navContainer.style.position = "absolute";
  navContainer.style.right = "0";
  navContainer.style.top = "0";
  navContainer.style.display = "flex";
  navContainer.style.alignItems = "center";
  navContainer.style.gap = "7.2px";

  let centerText = "";
  if (!showMonthly) {
    const weekStart = window.moment().startOf("isoWeek").add(weekOffset, "weeks");
    const weekEnd = weekStart.clone().endOf("isoWeek");
    centerText = `${weekStart.format("MMM D")} – ${weekEnd.format("MMM D")}`;
  } else {
    const monthStart = window.moment().startOf("month").add(monthOffset, "months");
    centerText = monthStart.format("MMMM YYYY");
  }

  const dateLabel = document.createElement("span");
  dateLabel.textContent = centerText;
  dateLabel.style.fontSize = "12.6px";
  dateLabel.style.fontWeight = "500";

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
  if (!showMonthly) {
    leftBtn.onclick = () => { weekOffset--; renderStreakTiles(streakContainer); };
  } else {
    leftBtn.onclick = () => { monthOffset--; renderStreakTiles(streakContainer); };
  }

  const rightBtn = document.createElement("button");
  rightBtn.textContent = "→";
  btnStyle(rightBtn);
  if (!showMonthly) {
    rightBtn.onclick = () => { weekOffset++; renderStreakTiles(streakContainer); };
  } else {
    rightBtn.onclick = () => { monthOffset++; renderStreakTiles(streakContainer); };
  }

  navContainer.appendChild(leftBtn);
  navContainer.appendChild(dateLabel);
  navContainer.appendChild(rightBtn);

  return navContainer;
}

function renderStreakNavigation(container) {
  const navWrapper = document.createElement("div");
  navWrapper.style.position = "relative";
  navWrapper.style.marginBottom = "12.6px";
  navWrapper.style.height = "30px";

  // Add toggle group to left
  navWrapper.appendChild(renderToggleGroup(container));

  // Add date navigation to right
  navWrapper.appendChild(renderDateNavigation());

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
    const isBeforeTracking = date.isBefore(trackingStart, 'day');
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
    } else if (!isFuture && !isToday) {
      background = colorRed;
      content = `<svg width="9" height="9" viewBox="0 0 12 12" fill="none" stroke="#fff" stroke-width="1.8" stroke-linecap="round"><line x1="2" y1="2" x2="10" y2="10"/><line x1="10" y1="2" x2="2" y2="10"/></svg>`;
      textColor = colorRed;
    } else {
      border = `2px solid var(--divider-color, #ccc)`;
      content = hideDates ? `<span style="font-size: 0.65em; color: ${textColor};">${date.date()}</span>` : '';
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

function renderMonthGrid(tag, weekdaysOnly) {
  const today = window.moment().format("YYYY-MM-DD");
  const monthStart = window.moment().startOf("month").add(monthOffset, "months");
  const monthEnd = monthStart.clone().endOf("month");
  const firstDayOfWeek = (parseInt(monthStart.format("d")) + 6) % 7;
  const daysInMonth = monthEnd.date();
  const totalCells = Math.ceil((firstDayOfWeek + daysInMonth) / 7) * 7;
  const weekDayLabels = ['M', 'T', 'W', 'T', 'F', 'S', 'S'];

  let html = `
    <div style="display: grid; grid-template-columns: repeat(7, 1fr); gap: 4.5px; margin: 9px 0;">
      ${weekDayLabels.map(label => `
        <div style="text-align: center; font-size: 10.8px; font-weight: 500; color: var(--text-muted);">${label}</div>
      `).join("")}
  `;

  for (let i = 0; i < totalCells; i++) {
    const dayOffset = i - firstDayOfWeek;
    const date = monthStart.clone().add(dayOffset, "days");
    const dateStr = date.format("YYYY-MM-DD");
    const isInMonth = dayOffset >= 0 && dayOffset < daysInMonth;
    const isToday = dateStr === today;
    const isWeekend = ["0", "6"].includes(date.format("d"));
    const isBeforeTracking = date.isBefore(trackingStart, 'day');
    const isFuture = date.isAfter(today, 'day');
    const skip = weekdaysOnly && isWeekend;

    let background = "var(--background-primary)";
    let content = '';
    let border = '';
    let textColor = "var(--text-muted)";

    if (!isInMonth) {
      html += `<div></div>`;
      continue;
    }

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
    } else if (!isFuture && !isToday) {
      background = colorRed;
      content = `<svg width="9" height="9" viewBox="0 0 12 12" fill="none" stroke="#fff" stroke-width="1.8" stroke-linecap="round"><line x1="2" y1="2" x2="10" y2="10"/><line x1="10" y1="2" x2="2" y2="10"/></svg>`;
      textColor = colorRed;
    } else {
      border = `2px solid var(--divider-color, #ccc)`;
      content = hideDates ? `<span style="font-size: 0.65em; color: ${textColor};">${date.date()}</span>` : '';
    }

    if (isToday) {
      border = `2px solid ${colorPurple}`;
    }

    html += `
      <div style="display: flex; justify-content: center;">
        <div style="width: 24px; height: 24px; border-radius: 50%; background: ${background}; border: ${border}; display: flex; justify-content: center; align-items: center;">
          ${content}
        </div>
      </div>`;
  }

  html += '</div>';
  return html;
}

function getStreakData(tag, weekdaysOnly) {
  const start = trackingStart.clone();
  const end = window.moment();
  const range = end.diff(start, 'days') + 1;

  let longest = 0;
  let currentStreak = 0;
  let completed = 0;
  let tracked = 0;

  for (let i = 0; i < range; i++) {
    const date = start.clone().add(i, 'days');
    const dateStr = date.format("YYYY-MM-DD");
    const isToday = date.isSame(end, 'day');
    const isWeekend = ["0", "6"].includes(date.format("d"));
    const isTrackedDay = !weekdaysOnly || !isWeekend;

    const page = dv.page(dateStr);
    const done = page?.file?.tasks?.some(t => t.text.includes(tag) && t.completed);

    if (isTrackedDay || done) {
      if (!isToday || done) tracked++;
    }

    if (done) completed++;

    if (isTrackedDay || done) {
      if (done) {
        currentStreak++;
        longest = Math.max(longest, currentStreak);
      } else {
        if (!isToday) currentStreak = 0;
      }
    }
  }

  const todayDone = dv.page(end.format("YYYY-MM-DD"))?.file?.tasks?.some(t => t.text.includes(tag) && t.completed);
  const current = todayDone ? currentStreak : currentStreak;
  const consistency = tracked ? Math.round((completed / tracked) * 100) : 0;

  return { current, longest, completed, tracked, consistency };
}

function renderStreakTiles(container) {
  container.innerHTML = '';
  container.appendChild(renderStreakNavigation(container));

  const grid = document.createElement("div");
  grid.style.display = "grid";
  grid.style.gridTemplateColumns = "repeat(auto-fit, minmax(198px, 1fr))";
  grid.style.gap = "10.8px";

  for (let habit of habitTags) {
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
      ${
        showMonthly
          ? renderMonthGrid(habit.tag, habit.weekdaysOnly)
          : renderWeekCircles(habit.tag, habit.weekdaysOnly)
      }
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

  container.appendChild(grid);
  renderLegend(container);

  if (trackingStart) {
    const note = document.createElement("div");
    note.style.marginTop = "9px";
    note.style.fontSize = "11px";
    note.style.color = "var(--text-muted)";
    note.style.textAlign = "center";
    const formattedDate = trackingStart.format("MMMM D, YYYY");
    note.textContent = `Note: All streak and consistency stats are measured from ${formattedDate}.`;
    container.appendChild(note);
  }
}

// ===== HEATMAP CODE =====
const dayNames = ["MON", "TUE", "WED", "THU", "FRI", "SAT", "SUN"];
const cellSize = 24;

// Process habit data for heatmap
const habitData = {};
let trackingStartDate = trackingStart.format("YYYY-MM-DD");

for (let p of journalFiles) {
  const date = p.file.name;
  const momentDate = window.moment(date, "YYYY-MM-DD");

  // Only process dates on or after tracking start
  if (momentDate.isBefore(trackingStart, 'day')) continue;

  const isWeekday = !["Saturday", "Sunday"].includes(momentDate.format("dddd"));
  const allTasks = p.file.tasks;
  let expectedHabits = 0;
  let completedHabits = 0;

  for (const tag of habitTags.map(h => h.tag)) {
    const task = allTasks.find(t => t.text.includes(tag));
    if (!task) continue;

    const isWeekdayOnly = habitTags.find(h => h.tag === tag)?.weekdaysOnly;
    const shouldCount = !isWeekdayOnly || isWeekday;

    if (shouldCount) {
      expectedHabits += 1;
      if (task.completed) completedHabits += 1;
    }
  }

  habitData[date] = { completed: completedHabits, expected: expectedHabits };
}

// Backfill missing days after tracking start
const start = trackingStart.clone();
const end = today.clone().subtract(1, "day");

for (let d = start.clone(); d.isSameOrBefore(end); d.add(1, "day")) {
  const dateStr = d.format("YYYY-MM-DD");
  if (habitData[dateStr]) continue;

  const isWeekday = !["Saturday", "Sunday"].includes(d.format("dddd"));
  let expectedHabits = 0;

  for (const habit of habitTags) {
    const shouldCount = !habit.weekdaysOnly || isWeekday;
    if (shouldCount) expectedHabits += 1;
  }

  habitData[dateStr] = { completed: 0, expected: expectedHabits };
}

const getCellColor = (dateStr, data) => {
  if (dateStr < trackingStartDate) return "transparent";
  if (!data || data.expected === 0) return "transparent";

  const ratio = data.completed / data.expected;

  if (dateStr === todayStr) return null;
  if (dateStr > todayStr) return "transparent"; // future

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
  const encodedLink = `obsidian://open?vault=${encodeURIComponent("Obsidian Vault")}&file=${encodeURIComponent(filePath)}`;

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

const renderHeatmapNavigation = () => {
  const navWrapper = document.createElement("div");
  navWrapper.style.position = "relative";
  navWrapper.style.height = "28px";
  navWrapper.style.marginBottom = "14px";

  const controls = document.createElement("div");
  controls.style.display = "flex";
  controls.style.alignItems = "center";
  controls.style.gap = "8px";
  controls.style.position = "absolute";
  controls.style.right = "0";
  controls.style.top = "0";

  const prevBtn = document.createElement("button");
  prevBtn.textContent = "←";

  const styleNavButton = (btn) => {
    btn.style.fontSize = "12.6px";
    btn.style.padding = "1.8px 5.4px";
    btn.style.border = "none";
    btn.style.background = "transparent";
    btn.style.cursor = "pointer";
    btn.style.color = "var(--text-muted)";
  };

  styleNavButton(prevBtn);
  prevBtn.onclick = () => {
    if (selectedQuarter === 1) {
      selectedQuarter = 4;
      selectedYear -= 1;
    } else {
      selectedQuarter -= 1;
    }
    renderHeatmapCalendars();
  };

  const title = document.createElement("span");
  title.textContent = `Q${selectedQuarter} ${selectedYear}`;
  title.style.fontSize = "12.6px";
  title.style.fontWeight = "500";

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
    renderHeatmapCalendars();
  };

  controls.appendChild(prevBtn);
  controls.appendChild(title);
  controls.appendChild(nextBtn);
  navWrapper.appendChild(controls);

  return navWrapper;
};

const renderHeatmapCalendars = () => {
  heatmapWrapper.innerHTML = "";
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
    heatmapWrapper.appendChild(div);
  });
};

// Heatmap grid container
const heatmapWrapper = document.createElement("div");
heatmapWrapper.style.display = "grid";
heatmapWrapper.style.gridTemplateColumns = "repeat(3, 1fr)";
heatmapWrapper.style.gap = "10px";
heatmapWrapper.style.maxWidth = "100%";
heatmapWrapper.style.overflowX = "hidden";

// Heatmap legend
const heatmapLegend = document.createElement("div");
heatmapLegend.innerHTML = `
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

// ===== COMBINED LAYOUT =====
dv.container.style.display = "flex";
dv.container.style.flexDirection = "column";
dv.container.style.gap = "18px";
dv.container.style.padding = "12px";

// Streak View Container
const streakContainer = document.createElement("div");
streakContainer.style.border = "1px solid var(--divider-color)";
streakContainer.style.borderRadius = "8px";
streakContainer.style.padding = "12px";
streakContainer.style.background = "var(--background-primary)";
dv.container.appendChild(streakContainer);

// Heatmap Container
const heatmapContainer = document.createElement("div");
heatmapContainer.style.border = "1px solid var(--divider-color)";
heatmapContainer.style.borderRadius = "8px";
heatmapContainer.style.padding = "12px";
heatmapContainer.style.background = "var(--background-primary)";
heatmapContainer.appendChild(renderHeatmapNavigation());
heatmapContainer.appendChild(heatmapWrapper);
heatmapContainer.appendChild(heatmapLegend);
dv.container.appendChild(heatmapContainer);

// Initialize both views
renderStreakTiles(streakContainer);
renderHeatmapCalendars();

// Set initial visibility
updateHeatmapVisibility();
```
