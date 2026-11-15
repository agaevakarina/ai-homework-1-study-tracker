# Создай мини-трекер учебных задач

> Создай мини-трекер учебных задач в одном автономном HTML-файле.
>
> Нужны название задачи, оценка в часах, отметка «готово», добавление и удаление задач, фильтр «все / активные / готовые», суммарные часы и процент выполнения.
>
> Добавь примеры: «Чтение» — 2 часа, выполнено; «Практика» — 4 часа, активно; «Повторение» — 2 часа, активно.
>
> Сделай удобный интерфейс. Всё должно работать в обычном браузере без установки библиотек и внешних ресурсов. Верни полный HTML-код.

Сохрани код как `study-tracker.html` и открой файл в браузере. Суммарные часы учитывают все задачи, а процент выполнения считается по числу готовых задач. Изменения сохраняются между открытиями файла, если браузер разрешает локальное хранение.

```html
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Учебный трекер</title>
  <style>
    :root {
      color-scheme: light;
      font-family: system-ui, sans-serif;
      color: #172c25;
      background: #f2f6f3;
      --surface: #fff;
      --line: #d8e4dc;
      --muted: #4d6257;
      --accent: #176443;
      --accent-soft: #e3f2e8;
      --radius: 18px;
    }

    * { box-sizing: border-box; }
    body { margin: 0; }
    h1, h2, p { margin: 0; }
    button, input { font: inherit; }

    button {
      min-height: 44px;
      padding: 0.55rem 0.9rem;
      border: 1px solid transparent;
      border-radius: 10px;
      cursor: pointer;
    }

    :focus-visible {
      outline: 3px solid #236eaf;
      outline-offset: 3px;
    }

    main {
      width: min(100% - 2rem, 44rem);
      margin: 0 auto;
      padding: clamp(2rem, 7vw, 5rem) 0 3rem;
    }

    h1 {
      margin-bottom: 1.5rem;
      font-size: clamp(2rem, 7vw, 3.5rem);
      line-height: 1.05;
      letter-spacing: -0.055em;
    }

    h2 { font-size: 1.15rem; }

    .panel {
      padding: clamp(1rem, 4vw, 1.5rem);
      margin-bottom: 1rem;
      border: 1px solid var(--line);
      border-radius: var(--radius);
      background: var(--surface);
    }

    .summary {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 0.75rem 1rem;
    }

    .metric-label, .task-hours, .empty, .storage-note {
      color: var(--muted);
      font-size: 0.9rem;
    }

    .metric {
      display: block;
      margin-top: 0.2rem;
      font-size: clamp(1.35rem, 5vw, 2rem);
      font-weight: 750;
      overflow-wrap: anywhere;
    }

    progress {
      grid-column: 1 / -1;
      width: 100%;
      height: 0.7rem;
      accent-color: var(--accent);
    }

    form {
      display: grid;
      grid-template-columns: minmax(0, 1fr) 7rem auto;
      align-items: end;
      gap: 0.75rem;
      margin-top: 0.9rem;
    }

    .field {
      display: grid;
      gap: 0.35rem;
      font-size: 0.9rem;
      font-weight: 600;
    }

    input[type="text"], input[type="number"] {
      width: 100%;
      min-height: 44px;
      padding: 0.55rem 0.65rem;
      border: 1px solid #93a99b;
      border-radius: 10px;
      background: var(--surface);
      color: inherit;
      font-weight: 400;
    }

    .add {
      background: var(--accent);
      color: #fff;
      font-weight: 650;
    }

    .add:hover { background: #104c32; }

    .list-header, .filters {
      display: flex;
      align-items: center;
      flex-wrap: wrap;
      gap: 0.35rem;
    }

    .list-header {
      justify-content: space-between;
      gap: 0.75rem;
      margin-bottom: 0.5rem;
    }

    .filters button {
      padding-inline: 0.65rem;
      background: #f2f6f3;
      color: var(--muted);
    }

    .filters button:hover { background: #edf5ef; }

    .filters button[aria-pressed="true"] {
      border-color: var(--accent);
      background: var(--accent-soft);
      color: var(--accent);
      font-weight: 650;
    }

    ul { margin: 0; padding: 0; list-style: none; }

    .task {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      padding: 0.55rem 0;
      border-bottom: 1px solid var(--line);
    }

    .task:last-child { border-bottom: 0; }

    .task-label {
      display: flex;
      flex: 1;
      align-items: center;
      gap: 0.75rem;
      min-width: 0;
      min-height: 44px;
      cursor: pointer;
    }

    .task-label input {
      flex: none;
      width: 1.2rem;
      height: 1.2rem;
      margin: 0;
      accent-color: var(--accent);
      cursor: pointer;
    }

    .task-text {
      display: grid;
      gap: 0.2rem;
      min-width: 0;
      overflow-wrap: anywhere;
    }

    .task-name { font-weight: 650; }

    .task-label input:checked + .task-text .task-name {
      color: var(--muted);
      text-decoration: line-through;
    }

    .delete { background: transparent; color: #942e34; }
    .delete:hover { background: #fff0f0; }
    .empty { padding: 0.85rem 0 0.25rem; }

    .sr-only {
      position: absolute;
      width: 1px;
      height: 1px;
      padding: 0;
      overflow: hidden;
      clip-path: inset(50%);
      white-space: nowrap;
    }

    @media (max-width: 520px) {
      form { grid-template-columns: minmax(0, 1fr) auto; }
      .field:first-child { grid-column: 1 / -1; }
    }
  </style>
</head>
<body>
  <main>
    <h1>Учебный трекер</h1>

    <section class="panel summary" aria-label="Сводка">
      <div>
        <span class="metric-label">Всего часов</span>
        <strong class="metric" id="total-hours"></strong>
      </div>
      <div>
        <span class="metric-label" id="completion-label">Задач выполнено</span>
        <strong class="metric" id="completion"></strong>
      </div>
      <progress id="progress" max="100" value="0"
                aria-labelledby="completion-label"></progress>
    </section>

    <section class="panel" aria-labelledby="add-heading">
      <h2 id="add-heading">Новая задача</h2>
      <form id="task-form">
        <label class="field">
          Название
          <input id="task-name" type="text" maxlength="160" required>
        </label>
        <label class="field">
          Часы
          <input id="task-hours" type="number" inputmode="decimal"
                 min="0.01" step="any" required>
        </label>
        <button class="add" type="submit">Добавить</button>
      </form>
    </section>

    <section class="panel" aria-labelledby="list-heading">
      <div class="list-header">
        <h2 id="list-heading">Задачи</h2>
        <div class="filters" id="filters" role="group" aria-label="Фильтр задач">
          <button type="button" data-filter="all" aria-pressed="true">Все</button>
          <button type="button" data-filter="active" aria-pressed="false">Активные</button>
          <button type="button" data-filter="done" aria-pressed="false">Готовые</button>
        </div>
      </div>
      <ul id="task-list"></ul>
      <p class="empty" id="empty-message" hidden></p>
    </section>

    <p class="storage-note" id="storage-note" hidden>
      Браузер не разрешил сохранить задачи. Изменения доступны до закрытия
      или перезагрузки страницы.
    </p>
    <p class="sr-only" id="status" role="status"></p>
  </main>

  <script>
    const storageKey = "study-task-tracker-v1";
    const examples = [
      { name: "Чтение", hours: 2, done: true },
      { name: "Практика", hours: 4, done: false },
      { name: "Повторение", hours: 2, done: false }
    ];

    const form = document.querySelector("#task-form");
    const nameInput = document.querySelector("#task-name");
    const hoursInput = document.querySelector("#task-hours");
    const filterGroup = document.querySelector("#filters");
    const filterButtons = [...filterGroup.querySelectorAll("button")];
    const list = document.querySelector("#task-list");
    const emptyMessage = document.querySelector("#empty-message");
    const totalHours = document.querySelector("#total-hours");
    const completion = document.querySelector("#completion");
    const progress = document.querySelector("#progress");
    const storageNote = document.querySelector("#storage-note");
    const status = document.querySelector("#status");
    const numberFormat = new Intl.NumberFormat("ru-RU", {
      maximumFractionDigits: 10
    });

    let tasks = loadTasks();
    let filter = "all";

    function loadTasks() {
      try {
        const saved = JSON.parse(localStorage.getItem(storageKey));
        if (Array.isArray(saved) && saved.every(isTask)) return saved;
      } catch {
        // Если прочитать данные не удалось, открываем список с примерами.
      }
      return examples.map(task => ({ ...task }));
    }

    function isTask(task) {
      return task !== null && typeof task === "object"
        && typeof task.name === "string" && task.name.trim() !== ""
        && Number.isFinite(task.hours) && task.hours >= 0.01
        && typeof task.done === "boolean";
    }

    function saveTasks() {
      try {
        localStorage.setItem(storageKey, JSON.stringify(tasks));
        storageNote.hidden = true;
      } catch {
        storageNote.hidden = false;
      }
    }

    function setFilter(nextFilter) {
      filter = nextFilter;
      for (const button of filterButtons) {
        button.setAttribute("aria-pressed", String(button.dataset.filter === filter));
      }
    }

    function updateSummary() {
      const hours = tasks.reduce((sum, task) => sum + task.hours, 0);
      const doneCount = tasks.filter(task => task.done).length;
      const percentage = tasks.length ? Math.round(100 * doneCount / tasks.length) : 0;

      totalHours.textContent = numberFormat.format(hours);
      completion.textContent = `${percentage}\u00a0%`;
      progress.value = percentage;
    }

    function render() {
      updateSummary();
      const visibleTasks = tasks.filter(task =>
        filter === "all" || task.done === (filter === "done")
      );

      list.replaceChildren(...visibleTasks.map(createTaskRow));

      const emptyText = {
        all: "Задач пока нет. Добавь первую.",
        active: "Нет активных задач.",
        done: "Нет готовых задач."
      };

      emptyMessage.textContent = emptyText[filter];
      emptyMessage.hidden = visibleTasks.length > 0;
    }

    function createTaskRow(task) {
      const row = document.createElement("li");
      row.className = "task";

      const label = document.createElement("label");
      label.className = "task-label";

      const checkbox = document.createElement("input");
      checkbox.type = "checkbox";
      checkbox.checked = task.done;

      const text = document.createElement("span");
      text.className = "task-text";

      const name = document.createElement("span");
      name.className = "task-name";
      name.textContent = task.name;

      const hours = document.createElement("span");
      hours.className = "task-hours";
      hours.textContent = `${numberFormat.format(task.hours)}\u00a0ч`;

      const deleteButton = document.createElement("button");
      deleteButton.className = "delete";
      deleteButton.type = "button";
      deleteButton.textContent = "Удалить";
      deleteButton.setAttribute("aria-label", `Удалить задачу «${task.name}»`);

      text.append(name, hours);
      label.append(checkbox, text);
      row.append(label, deleteButton);

      checkbox.addEventListener("change", () => {
        task.done = checkbox.checked;
        saveTasks();

        if (filter === "all") {
          updateSummary();
        } else {
          refreshAndMoveFocus(row, 'input[type="checkbox"]');
        }

        status.textContent = task.done
          ? `Задача «${task.name}» выполнена.`
          : `Задача «${task.name}» снова активна.`;
      });

      deleteButton.addEventListener("click", () => {
        tasks = tasks.filter(item => item !== task);
        saveTasks();
        refreshAndMoveFocus(row, ".delete");
        status.textContent = `Задача «${task.name}» удалена.`;
      });

      return row;
    }

    function refreshAndMoveFocus(row, controlSelector) {
      const index = [...list.children].indexOf(row);
      render();

      const nextRow = list.children[index] || list.lastElementChild;
      const nextControl = nextRow?.querySelector(controlSelector)
        || filterGroup.querySelector('[aria-pressed="true"]');
      nextControl.focus();
    }

    nameInput.addEventListener("input", () => nameInput.setCustomValidity(""));

    form.addEventListener("submit", event => {
      event.preventDefault();

      const name = nameInput.value.trim();
      if (!name) {
        nameInput.setCustomValidity("Введи название задачи.");
        nameInput.reportValidity();
        return;
      }

      tasks.push({ name, hours: hoursInput.valueAsNumber, done: false });
      saveTasks();

      // При добавлении из списка готовых показываем новую активную задачу.
      if (filter === "done") setFilter("active");
      render();

      form.reset();
      nameInput.focus();
      status.textContent = `Задача «${name}» добавлена.`;
    });

    for (const button of filterButtons) {
      button.addEventListener("click", () => {
        setFilter(button.dataset.filter);
        render();
      });
    }

    render();
  </script>
</body>
</html>
```

> Я проверила приложение и обнаружила две ошибки.
>
> 1. Процент выполнения считается по количеству задач. Он должен считаться по часам:
>
> progress = round(100 × сумма часов выполненных задач / сумма часов всех задач).
>
> Для начальных данных должно быть 25%, а после выполнения «Практики» — 75%.
>
> 2. Приложение принимает 5.24 часа. Допустимы только положительные значения от 0.5 до 24 включительно с шагом 0.5. Значения 0.5 и 2 допустимы, а 5.24 должно отклоняться.
>
> Исправь эти ошибки, не удаляя остальные функции. Верни полный обновлённый HTML-код одним блоком.

```html
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Учебный трекер</title>
  <style>
    :root {
      color-scheme: light;
      font-family: system-ui, sans-serif;
      color: #172c25;
      background: #f2f6f3;
      --surface: #fff;
      --line: #d8e4dc;
      --muted: #4d6257;
      --accent: #176443;
      --accent-soft: #e3f2e8;
      --radius: 18px;
    }

    * { box-sizing: border-box; }
    body { margin: 0; }
    h1, h2, p { margin: 0; }
    button, input { font: inherit; }

    button {
      min-height: 44px;
      padding: 0.55rem 0.9rem;
      border: 1px solid transparent;
      border-radius: 10px;
      cursor: pointer;
    }

    :focus-visible {
      outline: 3px solid #236eaf;
      outline-offset: 3px;
    }

    main {
      width: min(100% - 2rem, 44rem);
      margin: 0 auto;
      padding: clamp(2rem, 7vw, 5rem) 0 3rem;
    }

    h1 {
      margin-bottom: 1.5rem;
      font-size: clamp(2rem, 7vw, 3.5rem);
      line-height: 1.05;
      letter-spacing: -0.055em;
    }

    h2 { font-size: 1.15rem; }

    .panel {
      padding: clamp(1rem, 4vw, 1.5rem);
      margin-bottom: 1rem;
      border: 1px solid var(--line);
      border-radius: var(--radius);
      background: var(--surface);
    }

    .summary {
      display: grid;
      grid-template-columns: repeat(2, minmax(0, 1fr));
      gap: 0.75rem 1rem;
    }

    .metric-label, .task-hours, .empty, .storage-note, .hours-hint {
      color: var(--muted);
      font-size: 0.9rem;
    }

    .metric {
      display: block;
      margin-top: 0.2rem;
      font-size: clamp(1.35rem, 5vw, 2rem);
      font-weight: 750;
      overflow-wrap: anywhere;
    }

    progress {
      grid-column: 1 / -1;
      width: 100%;
      height: 0.7rem;
      accent-color: var(--accent);
    }

    form {
      display: grid;
      grid-template-columns: minmax(0, 1fr) 7rem auto;
      align-items: end;
      gap: 0.75rem;
      margin-top: 0.9rem;
    }

    .field {
      display: grid;
      gap: 0.35rem;
      font-size: 0.9rem;
      font-weight: 600;
    }

    .hours-hint { grid-column: 1 / -1; }

    input[type="text"], input[type="number"] {
      width: 100%;
      min-height: 44px;
      padding: 0.55rem 0.65rem;
      border: 1px solid #93a99b;
      border-radius: 10px;
      background: var(--surface);
      color: inherit;
      font-weight: 400;
    }

    .add {
      background: var(--accent);
      color: #fff;
      font-weight: 650;
    }

    .add:hover { background: #104c32; }

    .list-header, .filters {
      display: flex;
      align-items: center;
      flex-wrap: wrap;
      gap: 0.35rem;
    }

    .list-header {
      justify-content: space-between;
      gap: 0.75rem;
      margin-bottom: 0.5rem;
    }

    .filters button {
      padding-inline: 0.65rem;
      background: #f2f6f3;
      color: var(--muted);
    }

    .filters button:hover { background: #edf5ef; }

    .filters button[aria-pressed="true"] {
      border-color: var(--accent);
      background: var(--accent-soft);
      color: var(--accent);
      font-weight: 650;
    }

    ul { margin: 0; padding: 0; list-style: none; }

    .task {
      display: flex;
      align-items: center;
      gap: 0.5rem;
      padding: 0.55rem 0;
      border-bottom: 1px solid var(--line);
    }

    .task:last-child { border-bottom: 0; }

    .task-label {
      display: flex;
      flex: 1;
      align-items: center;
      gap: 0.75rem;
      min-width: 0;
      min-height: 44px;
      cursor: pointer;
    }

    .task-label input {
      flex: none;
      width: 1.2rem;
      height: 1.2rem;
      margin: 0;
      accent-color: var(--accent);
      cursor: pointer;
    }

    .task-text {
      display: grid;
      gap: 0.2rem;
      min-width: 0;
      overflow-wrap: anywhere;
    }

    .task-name { font-weight: 650; }

    .task-label input:checked + .task-text .task-name {
      color: var(--muted);
      text-decoration: line-through;
    }

    .delete { background: transparent; color: #942e34; }
    .delete:hover { background: #fff0f0; }
    .empty { padding: 0.85rem 0 0.25rem; }

    .sr-only {
      position: absolute;
      width: 1px;
      height: 1px;
      padding: 0;
      overflow: hidden;
      clip-path: inset(50%);
      white-space: nowrap;
    }

    @media (max-width: 520px) {
      form { grid-template-columns: minmax(0, 1fr) auto; }
      .field:first-child { grid-column: 1 / -1; }
    }
  </style>
</head>
<body>
  <main>
    <h1>Учебный трекер</h1>

    <section class="panel summary" aria-label="Сводка">
      <div>
        <span class="metric-label">Всего часов</span>
        <strong class="metric" id="total-hours"></strong>
      </div>
      <div>
        <span class="metric-label" id="completion-label">Выполнено по часам</span>
        <strong class="metric" id="completion"></strong>
      </div>
      <progress id="progress" max="100" value="0"
                aria-labelledby="completion-label"></progress>
    </section>

    <section class="panel" aria-labelledby="add-heading">
      <h2 id="add-heading">Новая задача</h2>
      <form id="task-form">
        <label class="field">
          Название
          <input id="task-name" type="text" maxlength="160" required>
        </label>
        <label class="field">
          Часы
          <input id="task-hours" type="number" inputmode="decimal"
                 min="0.5" max="24" step="0.5" required
                 aria-describedby="hours-hint">
        </label>
        <button class="add" type="submit">Добавить</button>
        <p class="hours-hint" id="hours-hint">От 0,5 до 24 часов с шагом 0,5.</p>
      </form>
    </section>

    <section class="panel" aria-labelledby="list-heading">
      <div class="list-header">
        <h2 id="list-heading">Задачи</h2>
        <div class="filters" id="filters" role="group" aria-label="Фильтр задач">
          <button type="button" data-filter="all" aria-pressed="true">Все</button>
          <button type="button" data-filter="active" aria-pressed="false">Активные</button>
          <button type="button" data-filter="done" aria-pressed="false">Готовые</button>
        </div>
      </div>
      <ul id="task-list"></ul>
      <p class="empty" id="empty-message" hidden></p>
    </section>

    <p class="storage-note" id="storage-note" hidden>
      Браузер не разрешил сохранить задачи. Изменения доступны до закрытия
      или перезагрузки страницы.
    </p>
    <p class="sr-only" id="status" role="status"></p>
  </main>

  <script>
    const storageKey = "study-task-tracker-v1";
    const examples = [
      { name: "Чтение", hours: 2, done: true },
      { name: "Практика", hours: 4, done: false },
      { name: "Повторение", hours: 2, done: false }
    ];

    const form = document.querySelector("#task-form");
    const nameInput = document.querySelector("#task-name");
    const hoursInput = document.querySelector("#task-hours");
    const filterGroup = document.querySelector("#filters");
    const filterButtons = [...filterGroup.querySelectorAll("button")];
    const list = document.querySelector("#task-list");
    const emptyMessage = document.querySelector("#empty-message");
    const totalHours = document.querySelector("#total-hours");
    const completion = document.querySelector("#completion");
    const progress = document.querySelector("#progress");
    const storageNote = document.querySelector("#storage-note");
    const status = document.querySelector("#status");
    const numberFormat = new Intl.NumberFormat("ru-RU", {
      maximumFractionDigits: 10
    });

    let tasks = loadTasks();
    let filter = "all";

    function loadTasks() {
      try {
        const saved = JSON.parse(localStorage.getItem(storageKey));
        if (Array.isArray(saved) && saved.every(isSavedTask)) return saved;
      } catch {
        // Если прочитать данные не удалось, открываем список с примерами.
      }
      return examples.map(task => ({ ...task }));
    }

    function isSavedTask(task) {
      // Сохраняем оценки, введённые в старой версии. Новые задачи
      // проходят проверку диапазона и шага в форме.
      return task !== null && typeof task === "object"
        && typeof task.name === "string" && task.name.trim() !== ""
        && Number.isFinite(task.hours) && task.hours >= 0.01
        && typeof task.done === "boolean";
    }

    function saveTasks() {
      try {
        localStorage.setItem(storageKey, JSON.stringify(tasks));
        storageNote.hidden = true;
      } catch {
        storageNote.hidden = false;
      }
    }

    function setFilter(nextFilter) {
      filter = nextFilter;
      for (const button of filterButtons) {
        button.setAttribute("aria-pressed", String(button.dataset.filter === filter));
      }
    }

    function updateSummary() {
      let hours = 0;
      let completedHours = 0;

      for (const task of tasks) {
        hours += task.hours;
        if (task.done) completedHours += task.hours;
      }

      const percentage = hours > 0
        ? Math.round(100 * (completedHours / hours))
        : 0;

      totalHours.textContent = numberFormat.format(hours);
      completion.textContent = `${percentage}\u00a0%`;
      progress.value = percentage;
    }

    function render() {
      updateSummary();
      const visibleTasks = tasks.filter(task =>
        filter === "all" || task.done === (filter === "done")
      );

      list.replaceChildren(...visibleTasks.map(createTaskRow));

      const emptyText = {
        all: "Задач пока нет. Добавь первую.",
        active: "Нет активных задач.",
        done: "Нет готовых задач."
      };

      emptyMessage.textContent = emptyText[filter];
      emptyMessage.hidden = visibleTasks.length > 0;
    }

    function createTaskRow(task) {
      const row = document.createElement("li");
      row.className = "task";

      const label = document.createElement("label");
      label.className = "task-label";

      const checkbox = document.createElement("input");
      checkbox.type = "checkbox";
      checkbox.checked = task.done;

      const text = document.createElement("span");
      text.className = "task-text";

      const name = document.createElement("span");
      name.className = "task-name";
      name.textContent = task.name;

      const hours = document.createElement("span");
      hours.className = "task-hours";
      hours.textContent = `${numberFormat.format(task.hours)}\u00a0ч`;

      const deleteButton = document.createElement("button");
      deleteButton.className = "delete";
      deleteButton.type = "button";
      deleteButton.textContent = "Удалить";
      deleteButton.setAttribute("aria-label", `Удалить задачу «${task.name}»`);

      text.append(name, hours);
      label.append(checkbox, text);
      row.append(label, deleteButton);

      checkbox.addEventListener("change", () => {
        task.done = checkbox.checked;
        saveTasks();

        if (filter === "all") {
          updateSummary();
        } else {
          refreshAndMoveFocus(row, 'input[type="checkbox"]');
        }

        status.textContent = task.done
          ? `Задача «${task.name}» выполнена.`
          : `Задача «${task.name}» снова активна.`;
      });

      deleteButton.addEventListener("click", () => {
        tasks = tasks.filter(item => item !== task);
        saveTasks();
        refreshAndMoveFocus(row, ".delete");
        status.textContent = `Задача «${task.name}» удалена.`;
      });

      return row;
    }

    function refreshAndMoveFocus(row, controlSelector) {
      const index = [...list.children].indexOf(row);
      render();

      const nextRow = list.children[index] || list.lastElementChild;
      const nextControl = nextRow?.querySelector(controlSelector)
        || filterGroup.querySelector('[aria-pressed="true"]');
      nextControl.focus();
    }

    nameInput.addEventListener("input", () => nameInput.setCustomValidity(""));

    form.addEventListener("submit", event => {
      event.preventDefault();

      const name = nameInput.value.trim();
      if (!name) {
        nameInput.setCustomValidity("Введи название задачи.");
        nameInput.reportValidity();
        return;
      }

      const hours = hoursInput.valueAsNumber;
      const halfHours = hours * 2;
      if (!Number.isInteger(halfHours) || halfHours < 1 || halfHours > 48) {
        status.textContent = "Введи от 0,5 до 24 часов с шагом 0,5.";
        hoursInput.focus();
        return;
      }

      tasks.push({ name, hours, done: false });
      saveTasks();

      // При добавлении из списка готовых показываем новую активную задачу.
      if (filter === "done") setFilter("active");
      render();

      form.reset();
      nameInput.focus();
      status.textContent = `Задача «${name}» добавлена.`;
    });

    for (const button of filterButtons) {
      button.addEventListener("click", () => {
        setFilter(button.dataset.filter);
        render();
      });
    }

    render();
  </script>
</body>
</html>
```
