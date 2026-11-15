# Подготовить спецификацию мини-трекера

> Сначала подготовь спецификацию мини-трекера в формате артефактов OpenSpec, код пока не пиши.\
> \
> Выдай четыре отдельных Markdown-блока:\
> \
> 1\. proposal.md — цель и границы.\
> 2\. specs/tracker/spec.md — требования с MUST и сценариями WHEN/THEN.\
> 3\. design.md — данные и расчёты.\
> 4\. tasks.md — чек-лист реализации и проверок.\
> \
> Используй все требования ниже и не расширяй задачу. Это работа в веб-чате: не утверждай, что запускал OpenSpec CLI или валидатор. После четырёх документов остановись.\
> \
> Результат: автономный мини-трекер без CDN, внешних библиотек и API-ключей. Один файл tracker-v2.html с CSS и JavaScript внутри.\
> \
> Цель: мини-трекер учебных задач на неделю.\
> \
> Данные задачи: id, title, hours, done.\
> \
> Начальные данные:\
> — «Чтение» — 2 часа, done=true;\
> — «Практика» — 4 часа, done=false;\
> — «Повторение» — 2 часа, done=false.\
> \
> Требования:\
> \
> 1\. Добавление, удаление и переключение done должны реально менять данные.\
> \
> 2\. Название после trim не должно быть пустым. Часы — число от 0.5 до 24 включительно с шагом 0.5. Значение 0 запрещено. Ошибка должна быть видна рядом с полем, а введённые данные должны сохраняться.\
> \
> 3\. Прогресс рассчитывается так:\
> \
> progress = round(100 × сумма часов выполненных задач / сумма часов всех задач).\
> \
> Если задач нет, прогресс равен 0%, без NaN и Infinity.\
> \
> 4\. Фильтр влияет только на видимый список. Итоговые часы и прогресс рассчитываются по всем задачам.\
> \
> 5\. Для начальных данных: всего 8 часов, выполнено 2 часа, прогресс 25%.\
> \
> 6\. После отметки «Практики» готовой: выполнено 6 из 8 часов, прогресс 75%.\
> \
> 7\. При фильтре «активные» видны «Практика» и «Повторение», а общий прогресс остаётся 25%.\
> \
> 8\. Пустое название, пробелы, пустые, нулевые и отрицательные часы, а также 5.24 должны отклоняться. Значения 0.5 и 2 должны приниматься.\
> \
> 9\. После удаления всех задач должны отображаться 0 часов и 0%.\
> \
> 10\. Интерфейс должен быть читаемым на ширине 390 px без горизонтальной прокрутки. Поля и кнопки должны иметь понятные подписи.\
> \
> Документы и реализация выполняются разными запросами. Сейчас код не пиши и не отмечай задачи выполненными.

```markdown
# proposal.md

## Цель

Создать автономный мини-трекер учебных задач на неделю. Пользователь может добавлять и удалять задачи, переключать статус выполнения, просматривать активные задачи и отслеживать прогресс по учебным часам.

## Границы

Результат реализации — один файл `tracker-v2.html` со встроенными CSS и JavaScript. Трекер работает без CDN, внешних библиотек и API-ключей.

В объём входят начальный список задач, управление задачами, проверка полей формы, фильтр списка, сводка по часам и читаемый интерфейс на ширине 390 px.
```

```markdown
# specs/tracker/spec.md

## ADDED Requirements

### Requirement: Начальный список задач

Трекер MUST открываться с тремя задачами. Каждая задача MUST содержать `id`, `title`, `hours` и `done`.

#### Scenario: Первое открытие трекера

- WHEN пользователь впервые открывает трекер.
- THEN в списке видны «Чтение» (2 часа, выполнено), «Практика» (4 часа, активно) и «Повторение» (2 часа, активно).

### Requirement: Изменение данных задач

Трекер MUST изменять данные при добавлении, удалении и переключении статуса задачи. После каждого изменения трекер MUST обновлять видимый список и сводку.

#### Scenario: Добавление задачи

- WHEN пользователь отправляет форму с корректным названием и количеством часов.
- THEN трекер добавляет задачу с уникальным `id`, названием после `trim`, числовым значением `hours` и `done=false`.

#### Scenario: Переключение статуса в обе стороны

- WHEN пользователь отмечает активную задачу выполненной.
- THEN её `done` становится `true`.
- WHEN пользователь возвращает ту же задачу в активные.
- THEN её `done` становится `false`.

#### Scenario: Удаление задачи

- WHEN пользователь удаляет задачу.
- THEN трекер удаляет её из данных и пересчитывает сводку по оставшимся задачам.

### Requirement: Проверка полей

Трекер MUST принимать название, которое остаётся непустым после `trim`, и число часов от 0.5 до 24 включительно с шагом 0.5. При ошибке трекер MUST сохранять оба введённых значения, показывать сообщение рядом с каждым ошибочным полем и оставлять список задач без изменений.

#### Scenario: Недопустимое название

- WHEN пользователь отправляет форму с пустым названием или названием из одних пробелов.
- THEN трекер отклоняет добавление, показывает ошибку рядом с названием и сохраняет оба введённых значения.

#### Scenario: Недопустимое количество часов

- WHEN пользователь отправляет форму с пустым полем часов, нечисловым значением, нулём, отрицательным значением, значением выше 24 или значением вне шага 0.5, например 5.24.
- THEN трекер отклоняет добавление, показывает ошибку рядом с полем часов и сохраняет оба введённых значения.

#### Scenario: Допустимое количество часов

- WHEN пользователь отдельно отправляет форму с корректным названием и значением 0.5, 2 или 24.
- THEN трекер добавляет задачу с указанным числом часов.

### Requirement: Сводка по часам и прогресс

Трекер MUST отображать общие часы, выполненные часы и прогресс. Прогресс MUST равняться `round(100 × выполненные часы / общие часы)`. При пустом списке трекер MUST отображать 0 общих часов, 0 выполненных часов и 0% без `NaN` и `Infinity`.

#### Scenario: Начальная сводка

- WHEN в трекере находятся три начальные задачи с исходными статусами.
- THEN общие часы равны 8, выполненные часы равны 2, а прогресс равен 25%.

#### Scenario: Выполнение «Практики»

- WHEN пользователь отмечает «Практику» выполненной при исходном состоянии остальных задач.
- THEN выполнено 6 из 8 часов, а прогресс равен 75%.

#### Scenario: Удаление всех задач

- WHEN пользователь удаляет последнюю задачу.
- THEN общие и выполненные часы равны 0, а прогресс равен 0%.

### Requirement: Фильтр списка

Трекер MUST предоставлять режимы «Все» и «Активные». Фильтр MUST менять только видимый список. Сводка MUST рассчитываться по всем задачам.

#### Scenario: Фильтрация начального списка

- WHEN пользователь включает фильтр «Активные» при исходных статусах задач.
- THEN видны только «Практика» и «Повторение», а сводка по-прежнему показывает 2 выполненных часа из 8 и прогресс 25%.

### Requirement: Автономность и читаемость

Трекер MUST работать из одного файла `tracker-v2.html` со встроенными CSS и JavaScript без CDN, внешних библиотек и API-ключей. Интерфейс MUST быть читаемым на ширине 390 px без горизонтальной прокрутки. Поля и кнопки MUST иметь понятные подписи.

#### Scenario: Работа на узком экране без сети

- WHEN пользователь открывает файл без доступа к сети при ширине области просмотра 390 px.
- THEN он может прочитать сводку и список, понять назначение полей и кнопок и пользоваться трекером без горизонтальной прокрутки.
```

```markdown
# design.md

## Данные

Источником данных служит общий список задач в памяти страницы. При открытии файла он инициализируется такими значениями.

| id | title | hours | done |
| --- | --- | ---: | --- |
| 1 | Чтение | 2 | true |
| 2 | Практика | 4 | false |
| 3 | Повторение | 2 | false |

`id` однозначно определяет задачу. Новая задача получает следующий числовой идентификатор, начиная с 4. `title` хранится как строка после `trim`, `hours` — как число, `done` — как логическое значение.

## Изменения и проверка формы

При отправке формы проверить оба поля до изменения списка. Название допустимо, если после `trim` остался хотя бы один символ. Для часов сначала проверить, что поле непустое, затем преобразовать значение целиком в число. Число должно быть конечным, лежать в диапазоне от 0.5 до 24 включительно и при умножении на 2 давать целое число.

При ошибке вывести сообщение рядом с соответствующим полем и сохранить исходные значения обоих полей. При успешной проверке добавить задачу с `done=false`. Удаление и переключение статуса выполняются по `id`.

## Расчёты

- Общие часы — сумма `hours` всех задач.
- Выполненные часы — сумма `hours` задач с `done=true`.
- Если общие часы равны нулю, прогресс равен 0%. Иначе прогресс равен `round(100 × выполненные часы / общие часы)`.

После добавления, удаления или переключения статуса обновить список и сводку из текущих данных. Фильтр «Активные» показывает задачи с `done=false`, а «Все» показывает полный список. Смена фильтра не изменяет данные, используемые для расчётов.
```

```markdown
# tasks.md

## 1. Реализация

- [ ] Создать `tracker-v2.html` со встроенными CSS и JavaScript и без CDN, внешних библиотек и API-ключей.
- [ ] Задать модель `id`, `title`, `hours`, `done` и три начальные задачи.
- [ ] Добавить форму с понятными подписями, проверкой названия после `trim` и проверкой часов на диапазон и шаг.
- [ ] Показывать ошибки рядом с ошибочными полями и сохранять оба введённых значения при отклонении формы.
- [ ] Реализовать добавление, удаление и переключение `done` в обе стороны с изменением данных и обновлением интерфейса.
- [ ] Добавить режимы списка «Все» и «Активные» и считать сводку по полному списку.
- [ ] Рассчитывать общие часы, выполненные часы и округлённый прогресс. Обработать пустой список до деления.
- [ ] Обеспечить читаемость на ширине 390 px, понятные подписи полей и кнопок и отсутствие горизонтальной прокрутки.

## 2. Проверки

- [ ] Открыть файл без сети и проверить работу трекера, подписи и читаемость при ширине 390 px без горизонтальной прокрутки.
- [ ] Проверить начальные задачи и сводку — 2 выполненных часа из 8, прогресс 25%.
- [ ] Отметить «Практику» выполненной и проверить 6 из 8 часов и 75%. Вернуть её в активные и проверить 2 из 8 часов и 25%.
- [ ] С исходными статусами включить «Активные» и проверить, что видны только «Практика» и «Повторение», а сводка показывает 2 из 8 часов и 25%. Вернуться в режим «Все» и проверить полный список.
- [ ] По отдельности проверить отклонение пустого названия, названия из пробелов, пустых, нечисловых, нулевых и отрицательных часов, значения 5.24 и значения выше 24. Для каждого случая проверить сообщение рядом с полем, сохранение обоих значений формы и отсутствие новой задачи.
- [ ] Проверить добавление задач с 0.5, 2 и 24 часами, удаление пробелов по краям названия, начальный статус `done=false` и пересчёт сводки.
- [ ] Удалить одну задачу и проверить её исчезновение и пересчёт сводки. Удалить остальные и проверить 0 общих часов, 0 выполненных часов и 0% без `NaN` и `Infinity`.
```

> Исправь документы до реализации:\
> \
> 1\. Во всех применимых документах добавь третий фильтр «Готовые». Он показывает только задачи с done=true и, как остальные фильтры, не влияет на общие часы, выполненные часы и прогресс.\
> \
> 2\. В proposal.md явно укажи, что сервер, аккаунты, синхронизация и внешние API не входят в проект.\
> \
> 3\. В tasks.md расположи этапы в порядке: данные → расчёты → интерфейс → тесты → ручная проверка. Не отмечай пункты выполненными.\
> \
> Верни полные обновлённые версии всех четырёх документов. Код пока не пиши.

```markdown
# proposal.md

## Цель

Создать автономный мини-трекер учебных задач на неделю. Пользователь может добавлять и удалять задачи, переключать статус выполнения, фильтровать список и отслеживать прогресс по учебным часам.

## Границы

Результат реализации — один файл `tracker-v2.html` со встроенными CSS и JavaScript. Трекер работает без CDN, внешних библиотек и API-ключей.

В объём входят начальный список задач, управление задачами, проверка полей формы, фильтры «Все», «Активные» и «Готовые», сводка по часам и читаемый интерфейс на ширине 390 px. Все три фильтра управляют только видимым списком. Общие часы, выполненные часы и прогресс рассчитываются по всем задачам.

Сервер, аккаунты, синхронизация и внешние API не входят в проект.
```

```markdown
# specs/tracker/spec.md

## ADDED Requirements

### Requirement: Начальный список задач

Трекер MUST открываться с тремя задачами. Каждая задача MUST содержать `id`, `title`, `hours` и `done`.

#### Scenario: Первое открытие трекера

- WHEN пользователь открывает трекер.
- THEN в списке видны «Чтение» (2 часа, `done=true`), «Практика» (4 часа, `done=false`) и «Повторение» (2 часа, `done=false`).

### Requirement: Изменение данных задач

Трекер MUST изменять данные при добавлении, удалении и переключении статуса задачи. После каждого изменения трекер MUST обновлять видимый список и сводку.

#### Scenario: Добавление задачи

- WHEN пользователь отправляет форму с корректным названием и количеством часов.
- THEN трекер добавляет задачу с уникальным `id`, названием после `trim`, числовым значением `hours` и `done=false`.

#### Scenario: Отметка задачи выполненной

- WHEN пользователь отмечает активную задачу выполненной.
- THEN её `done` становится `true`.

#### Scenario: Возврат задачи в активные

- WHEN пользователь снимает отметку с выполненной задачи.
- THEN её `done` становится `false`.

#### Scenario: Удаление задачи

- WHEN пользователь удаляет задачу.
- THEN трекер удаляет её из данных и пересчитывает сводку по оставшимся задачам.

### Requirement: Проверка полей

Трекер MUST принимать название, которое остаётся непустым после `trim`, и число часов от 0.5 до 24 включительно с шагом 0.5. При ошибке трекер MUST сохранять оба введённых значения, показывать сообщение рядом с каждым ошибочным полем и оставлять список задач без изменений.

#### Scenario: Недопустимое название

- WHEN пользователь отправляет форму с пустым названием или названием из одних пробелов.
- THEN трекер отклоняет добавление, показывает ошибку рядом с названием и сохраняет оба введённых значения.

#### Scenario: Недопустимое количество часов

- WHEN пользователь отправляет форму с пустым полем часов, нечисловым значением, нулём, отрицательным значением, значением выше 24 или значением вне шага 0.5, например 5.24.
- THEN трекер отклоняет добавление, показывает ошибку рядом с полем часов и сохраняет оба введённых значения.

#### Scenario: Допустимое количество часов

- WHEN пользователь отдельно отправляет форму с корректным названием и значением 0.5, 2 или 24.
- THEN трекер добавляет задачу с указанным числом часов.

### Requirement: Сводка по часам и прогресс

Трекер MUST отображать общие часы, выполненные часы и прогресс. Прогресс MUST равняться `round(100 × выполненные часы / общие часы)`. При отсутствии задач трекер MUST отображать 0 общих часов, 0 выполненных часов и 0% без `NaN` и `Infinity`.

#### Scenario: Начальная сводка

- WHEN в трекере находятся три начальные задачи с исходными статусами.
- THEN общие часы равны 8, выполненные часы равны 2, а прогресс равен 25%.

#### Scenario: Выполнение «Практики»

- WHEN пользователь отмечает «Практику» выполненной при исходном состоянии остальных задач.
- THEN выполнено 6 из 8 часов, а прогресс равен 75%.

#### Scenario: Удаление всех задач

- WHEN пользователь удаляет последнюю задачу.
- THEN общие и выполненные часы равны 0, а прогресс равен 0%.

### Requirement: Фильтры списка

Трекер MUST предоставлять три фильтра. «Все» показывает все задачи, «Активные» показывает только задачи с `done=false`, а «Готовые» показывает только задачи с `done=true`. Фильтры MUST менять только видимый список. Общие часы, выполненные часы и прогресс MUST рассчитываться по всем задачам при любом фильтре.

#### Scenario: Активные задачи

- WHEN пользователь включает фильтр «Активные» при исходных статусах задач.
- THEN видны только «Практика» и «Повторение», а сводка показывает 8 общих часов, 2 выполненных часа и 25%.

#### Scenario: Готовые задачи

- WHEN пользователь включает фильтр «Готовые» при исходных статусах задач.
- THEN видно только «Чтение», а сводка показывает 8 общих часов, 2 выполненных часа и 25%.

#### Scenario: Возврат к полному списку

- WHEN пользователь переключается с фильтра «Активные» или «Готовые» на «Все» при исходных статусах задач.
- THEN видны все три начальные задачи, а сводка показывает 8 общих часов, 2 выполненных часа и 25%.

### Requirement: Автономность и читаемость

Трекер MUST работать из одного файла `tracker-v2.html` со встроенными CSS и JavaScript без CDN, внешних библиотек и API-ключей. Интерфейс MUST быть читаемым на ширине 390 px без горизонтальной прокрутки. Поля и кнопки MUST иметь понятные подписи.

#### Scenario: Работа на узком экране без сети

- WHEN пользователь открывает файл без доступа к сети при ширине области просмотра 390 px.
- THEN он может прочитать сводку и список, понять назначение полей и кнопок и пользоваться трекером без горизонтальной прокрутки.
```

```markdown
# design.md

## Данные

Источником данных служит общий список задач в памяти страницы. При открытии файла он инициализируется следующими значениями.

| id | title | hours | done |
| --- | --- | ---: | --- |
| 1 | Чтение | 2 | true |
| 2 | Практика | 4 | false |
| 3 | Повторение | 2 | false |

`id` однозначно определяет задачу. Новая задача получает следующий числовой идентификатор, начиная с 4. `title` хранится как строка после `trim`, `hours` — как число, `done` — как логическое значение.

## Изменения и проверка формы

При отправке формы проверить оба поля до изменения списка. Название допустимо, если после `trim` остался хотя бы один символ. Для часов сначала проверить, что поле непустое, затем преобразовать значение целиком в число. Число должно быть конечным, лежать в диапазоне от 0.5 до 24 включительно и при умножении на 2 давать целое число.

При ошибке вывести сообщение рядом с каждым ошибочным полем и сохранить исходные значения обоих полей. При успешной проверке добавить задачу с `done=false`. Удаление и переключение статуса выполняются по `id`.

## Расчёты

- Общие часы — сумма `hours` всех задач.
- Выполненные часы — сумма `hours` задач с `done=true`.
- Если общие часы равны нулю, прогресс равен 0%. Иначе прогресс равен `round(100 × выполненные часы / общие часы)`.

После добавления, удаления или переключения статуса обновить видимый список и сводку из текущих данных.

## Фильтры

- «Все» показывает полный список.
- «Активные» показывает только задачи с `done=false`.
- «Готовые» показывает только задачи с `done=true`.

При смене фильтра обновляется только видимый список. Общие часы, выполненные часы и прогресс при любом фильтре рассчитываются по полному списку задач.
```

```markdown
# tasks.md

## 1. Данные

- [ ] Задать модель `id`, `title`, `hours`, `done` и три начальные задачи.
- [ ] Реализовать проверку названия после `trim` и проверку часов на непустое значение, допустимый диапазон и шаг.
- [ ] Реализовать добавление задачи с уникальным `id` и `done=false`, удаление по `id` и переключение `done` в обе стороны.

## 2. Расчёты

- [ ] Рассчитывать общие и выполненные часы по полному списку задач.
- [ ] Рассчитывать округлённый прогресс по часам. При пустом списке возвращать 0% до деления.

## 3. Интерфейс

- [ ] Создать `tracker-v2.html` со встроенными CSS и JavaScript. Обеспечить работу без CDN, внешних библиотек и API-ключей.
- [ ] Добавить форму и кнопки с понятными подписями. Показывать ошибки рядом с ошибочными полями и сохранять оба введённых значения при отклонении формы.
- [ ] Вывести список и сводку. Обновлять их после добавления, удаления и переключения статуса задачи.
- [ ] Добавить фильтры «Все», «Активные» (`done=false`) и «Готовые» (`done=true`). Применять выбранный фильтр только к видимому списку.
- [ ] Обеспечить читаемость на ширине 390 px без горизонтальной прокрутки.

## 4. Тесты

- [ ] Проверить начальные задачи и сводку — 2 выполненных часа из 8, прогресс 25%.
- [ ] Отметить «Практику» выполненной и проверить 6 из 8 часов и 75%. Вернуть её в активные и проверить 2 из 8 часов и 25%.
- [ ] При исходных статусах проверить, что «Активные» показывает «Практику» и «Повторение», «Готовые» показывает только «Чтение», а «Все» показывает все три задачи. При каждом фильтре проверить 8 общих часов, 2 выполненных часа и 25%.
- [ ] По отдельности проверить отклонение пустого названия, названия из пробелов, пустых, нечисловых, нулевых и отрицательных часов, значения 5.24 и значения выше 24. В каждом случае проверить сообщение рядом с полем, сохранение обоих значений формы и отсутствие новой задачи.
- [ ] Проверить добавление задач с 0.5, 2 и 24 часами, удаление пробелов по краям названия, начальный статус `done=false` и пересчёт сводки.
- [ ] Удалить одну задачу и проверить её исчезновение и пересчёт сводки. Удалить остальные и проверить 0 общих часов, 0 выполненных часов и 0% без `NaN` и `Infinity`.

## 5. Ручная проверка

- [ ] Открыть `tracker-v2.html` без доступа к сети и проверить добавление, удаление, переключение статуса и работу трёх фильтров.
- [ ] При ширине области просмотра 390 px проверить читаемость списка и сводки, понятность подписей полей и кнопок, видимость ошибок рядом с полями и отсутствие горизонтальной прокрутки.
```

> Теперь реализуй tracker-v2.html строго по согласованным документам выше.\
> \
> Создай один автономный HTML-файл с CSS и JavaScript внутри. Не используй CDN, внешние библиотеки, сервер или API.\
> \
> Реализуй все требования, включая:\
> — три начальные задачи;\
> — расчёт прогресса по часам;\
> — добавление, удаление и переключение статуса;\
> — фильтры «Все», «Активные» и «Готовые»;\
> — проверку диапазона 0.5–24 с шагом 0.5;\
> — сообщения об ошибках рядом с полями с сохранением введённых значений;\
> — пустое состояние 0 часов и 0%;\
> — интерфейс без горизонтальной прокрутки при ширине 390 px.\
> \
> Верни полный tracker-v2.html одним блоком, без разделения на части. После кода перечисли, что мне нужно проверить вручную. Не утверждай, что браузерные проверки выполнены, если ты их фактически не запускал.

<details><summary>2 previous messages</summary>

> Создам автономный HTML-файл и проверю расчёты, фильтры и обработку формы.
>
> <details><summary>Ran 3 commands, called a tool</summary>
>
> - Ran `ls -la; command -v node; ls /Applications`
>
> - Ran `cat > outputs/tracker-v2.html <<'EOF'
> <!doctype html>
> <html lang="ru">
> <head>
>   <meta charset="utf-8">
>   <meta name="viewport" content="width=device-width, initial-scale=1">
>   <title>Учебные задачи на неделю</title>
>   <style>
>     * { box-sizing: border-box; }
>     body {
>       margin: 0;
>       background: #edf2ef;
>       color: #172720;
>       font: 1rem/1.5 system-ui, sans-serif;
>     }
>     main { max-width: 42rem; margin: auto; padding: 1rem; }
>     h1 { font-size: 1.75rem; line-height: 1.2; }
>     h2 { margin: 0 0 .75rem; font-size: 1.125rem; }
>     section {
>       margin-block: 1rem;
>       padding: 1rem;
>       background: white;
>       border: 1px solid #ccd8d0;
>       border-radius: .75rem;
>     }
>     label, input, button { font: inherit; }
>     input[type="text"] {
>       display: block;
>       width: 100%;
>       margin-top: .25rem;
>       padding: .5rem .625rem;
>       border: 1px solid #63776b;
>       border-radius: .375rem;
>     }
>     button {
>       padding: .45rem .7rem;
>       border: 1px solid #63776b;
>       border-radius: .375rem;
>       background: white;
>       color: inherit;
>       cursor: pointer;
>       min-height: 2.75rem;
>     }
>     button[type="submit"], button[aria-pressed="true"] {
>       border-color: #245d46;
>       background: #245d46;
>       color: white;
>     }
>     :focus-visible { outline: 2px solid #215ab0; outline-offset: 2px; }
>     .field { margin-bottom: .75rem; }
>     .error { margin: .25rem 0 0; color: #a52222; }
>     input[aria-invalid="true"] { border-color: #a52222; }
>     .filters { display: flex; flex-wrap: wrap; gap: .375rem; }
>     #summary { margin: 0; }
>     #tasks { list-style: none; margin: .75rem 0 0; padding: 0; }
>     #tasks li {
>       display: flex;
>       flex-wrap: wrap;
>       align-items: center;
>       justify-content: space-between;
>       gap: .5rem;
>       padding: .625rem 0;
>       border-top: 1px solid #ccd8d0;
>     }
>     .task-label {
>       display: flex;
>       flex: 1 1 10rem;
>       align-items: flex-start;
>       gap: .5rem;
>       min-width: 0;
>       overflow-wrap: anywhere;
>     }
>     .task-label input { flex: none; margin: .3em 0 0; }
>     #empty { margin-bottom: 0; }
>   </style>
> </head>
> <body>
>   <main>
>     <h1>Учебные задачи на неделю</h1>
>
>     <section aria-labelledby="summary-heading">
>       <h2 id="summary-heading">Сводка</h2>
>       <p id="summary" role="status">
>         Всего: <strong id="total"></strong> ч ·
>         Выполнено: <strong id="completed"></strong> ч ·
>         Прогресс: <strong id="progress"></strong>
>       </p>
>     </section>
>
>     <section aria-labelledby="form-heading">
>       <h2 id="form-heading">Новая задача</h2>
>       <form id="task-form" novalidate>
>         <div class="field">
>           <label for="title">Название</label>
>           <input id="title" type="text" required aria-describedby="title-error">
>           <p id="title-error" class="error" aria-live="polite"></p>
>         </div>
>         <div class="field">
>           <label for="hours">Часы (от 0,5 до 24, шаг 0,5)</label>
>           <input id="hours" type="text" inputmode="decimal" required aria-describedby="hours-error">
>           <p id="hours-error" class="error" aria-live="polite"></p>
>         </div>
>         <button type="submit">Добавить задачу</button>
>       </form>
>     </section>
>
>     <section aria-labelledby="tasks-heading">
>       <h2 id="tasks-heading">Задачи</h2>
>       <div class="filters" role="group" aria-label="Фильтр задач">
>         <button type="button" data-filter="all" aria-pressed="true">Все</button>
>         <button type="button" data-filter="active" aria-pressed="false">Активные</button>
>         <button type="button" data-filter="done" aria-pressed="false">Готовые</button>
>       </div>
>       <ul id="tasks"></ul>
>       <p id="empty" hidden>Нет задач для отображения.</p>
>     </section>
>   </main>
>
>   <script>
>     const tasks = [
>       { id: 1, title: 'Чтение', hours: 2, done: true },
>       { id: 2, title: 'Практика', hours: 4, done: false },
>       { id: 3, title: 'Повторение', hours: 2, done: false }
>     ];
>     let nextId = 4;
>     let filter = 'all';
>
>     const form = document.getElementById('task-form');
>     const titleInput = document.getElementById('title');
>     const hoursInput = document.getElementById('hours');
>     const taskList = document.getElementById('tasks');
>     const empty = document.getElementById('empty');
>     const filterButtons = document.querySelectorAll('[data-filter]');
>     const numberFormat = new Intl.NumberFormat('ru-RU');
>
>     function renderSummary() {
>       let total = 0;
>       let completed = 0;
>       for (const task of tasks) {
>         total += task.hours;
>         if (task.done) completed += task.hours;
>       }
>       const progress = total === 0 ? 0 : Math.round(100 * completed / total);
>
>       document.getElementById('total').textContent = numberFormat.format(total);
>       document.getElementById('completed').textContent = numberFormat.format(completed);
>       document.getElementById('progress').textContent = progress + '%';
>     }
>
>     function renderList() {
>       const visibleTasks = filter === 'all'
>         ? tasks
>         : tasks.filter(task => task.done === (filter === 'done'));
>       const rows = visibleTasks.map(task => {
>         const row = document.createElement('li');
>         const label = document.createElement('label');
>         label.className = 'task-label';
>         const checkbox = document.createElement('input');
>         checkbox.type = 'checkbox';
>         checkbox.checked = task.done;
>         checkbox.setAttribute('aria-label', 'Выполнено: ' + task.title);
>         const taskText = document.createElement('span');
>         taskText.textContent = task.title + ' · ' + numberFormat.format(task.hours) + ' ч';
>         label.append(checkbox, taskText);
>
>         const remove = document.createElement('button');
>         remove.type = 'button';
>         remove.textContent = 'Удалить';
>         remove.setAttribute('aria-label', 'Удалить задачу «' + task.title + '»');
>
>         checkbox.addEventListener('change', () => {
>           task.done = checkbox.checked;
>           renderSummary();
>           if (filter !== 'all') {
>             renderList();
>             focusFilter();
>           }
>         });
>         remove.addEventListener('click', () => {
>           const index = tasks.findIndex(item => item.id === task.id);
>           tasks.splice(index, 1);
>           renderList();
>           renderSummary();
>           focusFilter();
>         });
>
>         row.append(label, remove);
>         return row;
>       });
>
>       taskList.replaceChildren(...rows);
>       empty.hidden = rows.length > 0;
>     }
>
>     function focusFilter() {
>       document.querySelector('[data-filter][aria-pressed="true"]').focus();
>     }
>
>     function showError(input, message) {
>       document.getElementById(input.id + '-error').textContent = message;
>       input.setAttribute('aria-invalid', String(Boolean(message)));
>     }
>
>     form.addEventListener('submit', event => {
>       event.preventDefault();
>       const title = titleInput.value.trim();
>       const hoursText = hoursInput.value.trim();
>       const hours = /^\d+(?:[.,]\d+)?$|^[.,]\d+$/.test(hoursText)
>         ? Number(hoursText.replace(',', '.'))
>         : NaN;
>       const validHours = hours >= 0.5 && hours <= 24 && Number.isInteger(hours * 2);
>
>       showError(titleInput, title ? '' : 'Введите название задачи.');
>       showError(hoursInput, validHours ? '' : 'Введите число от 0,5 до 24 с шагом 0,5.');
>       if (!title || !validHours) {
>         (title ? hoursInput : titleInput).focus();
>         return;
>       }
>
>       tasks.push({ id: nextId++, title, hours, done: false });
>       form.reset();
>       renderList();
>       renderSummary();
>       titleInput.focus();
>     });
>
>     for (const button of filterButtons) {
>       button.addEventListener('click', () => {
>         filter = button.dataset.filter;
>         for (const option of filterButtons) {
>           option.setAttribute('aria-pressed', String(option === button));
>         }
>         renderList();
>       });
>     }
>
>     renderList();
>     renderSummary();
>   </script>
> </body>
> </html>
> EOF
> ls -l outputs`
>
> - Ran `which osascript; ls /System/Library/Frameworks/JavaScriptCore.framework/Versions/A/Helpers/`
>
> MCP tool call
>
> node_repl.js
>
> ```json
> {
>   "title": "Проверяю логику трекера",
>   "code": "const fs = await import('node:fs'); const vm = await import('node:vm'); const assert = (await import('node:assert/strict')).default;\nconst html = fs.readFileSync('outputs/tracker-v2.html','utf8');\nclass El { constructor(id) { this.id=id; this.value=''; this.events={}; this.attrs={}; } setAttribute(k,v){this.attrs[k]=v} append(...xs){this.children=xs} replaceChildren(...xs){this.children=xs} addEventListener(k,f){this.events[k]=f} fire(k){this.events[k]({preventDefault(){}})} focus(){} reset(){fields.title.value=fields.hours.value=''} }\nconst fields = Object.fromEntries([...html.matchAll(/ id=\"([^\"]+)\"/g)].map(m=>[m[1],new El(m[1])]));\nconst buttons = ['all','active','done'].map(filter=>Object.assign(new El(),{dataset:{filter}}));\nconst document = {getElementById:id=>fields[id],createElement:()=>new El(),querySelectorAll:()=>buttons,querySelector:()=>buttons.find(b=>b.attrs['aria-pressed']==='true')};\nvm.runInNewContext(html.match(/<script>([\\s\\S]*?)<\\/script>/)[1],{document});\nconst summary = (...values)=>assert.deepEqual(['total','completed','progress'].map(id=>fields[id].textContent),values);\nconst names = (...values)=>assert.deepEqual(fields.tasks.children.map(r=>r.children[0].children[1].textContent.split(' · ')[0]),values);\nsummary('8','2','25%'); names('Чтение','Практика','Повторение');\nbuttons[1].fire('click'); names('Практика','Повторение'); summary('8','2','25%');\nbuttons[2].fire('click'); names('Чтение'); summary('8','2','25%');\nbuttons[0].fire('click'); const check=fields.tasks.children[1].children[0].children[0]; check.checked=true; check.fire('change'); summary('8','6','75%'); check.checked=false; check.fire('change'); summary('8','2','25%');\nfor(const [title,hours,error] of [['','2','title'],['   ','2','title'],...['','0','-1','5.24','25','abc'].map(h=>['Новая',h,'hours'])]){\nfields.title.value=title;fields.hours.value=hours;fields['task-form'].fire('submit');assert.ok(fields[error+'-error'].textContent);assert.equal(fields.title.value,title);assert.equal(fields.hours.value,hours);summary('8','2','25%');\n}\nfor(const hours of ['0.5','2','24','0,5']){fields.title.value=' Новая ';fields.hours.value=hours;fields['task-form'].fire('submit');assert.equal(fields.tasks.children.at(-1).children[0].children[1].textContent.split(' · ')[0],'Новая');}\nsummary('35','2','6%'); while(fields.tasks.children.length) fields.tasks.children[0].children[1].fire('click'); summary('0','0','0%');\nnodeRepl.write('Passed');"
> }
> ```
>
> ```text
> Passed
> ```
>
> </details>
</details>

[Скачать tracker-v2.html](./outputs/tracker-v2.html)

```html
<!doctype html>
<html lang="ru">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Учебные задачи на неделю</title>
  <style>
    * { box-sizing: border-box; }
    body {
      margin: 0;
      background: #edf2ef;
      color: #172720;
      font: 1rem/1.5 system-ui, sans-serif;
    }
    main { max-width: 42rem; margin: auto; padding: 1rem; }
    h1 { font-size: 1.75rem; line-height: 1.2; }
    h2 { margin: 0 0 .75rem; font-size: 1.125rem; }
    section {
      margin-block: 1rem;
      padding: 1rem;
      background: white;
      border: 1px solid #ccd8d0;
      border-radius: .75rem;
    }
    label, input, button { font: inherit; }
    input[type="text"] {
      display: block;
      width: 100%;
      margin-top: .25rem;
      padding: .5rem .625rem;
      border: 1px solid #63776b;
      border-radius: .375rem;
    }
    button {
      padding: .45rem .7rem;
      border: 1px solid #63776b;
      border-radius: .375rem;
      background: white;
      color: inherit;
      cursor: pointer;
      min-height: 2.75rem;
    }
    button[type="submit"], button[aria-pressed="true"] {
      border-color: #245d46;
      background: #245d46;
      color: white;
    }
    :focus-visible { outline: 2px solid #215ab0; outline-offset: 2px; }
    .field { margin-bottom: .75rem; }
    .error { margin: .25rem 0 0; color: #a52222; }
    input[aria-invalid="true"] { border-color: #a52222; }
    .filters { display: flex; flex-wrap: wrap; gap: .375rem; }
    #summary { margin: 0; }
    #tasks { list-style: none; margin: .75rem 0 0; padding: 0; }
    #tasks li {
      display: flex;
      flex-wrap: wrap;
      align-items: center;
      justify-content: space-between;
      gap: .5rem;
      padding: .625rem 0;
      border-top: 1px solid #ccd8d0;
    }
    .task-label {
      display: flex;
      flex: 1 1 10rem;
      align-items: flex-start;
      gap: .5rem;
      min-width: 0;
      overflow-wrap: anywhere;
    }
    .task-label input { flex: none; margin: .3em 0 0; }
    #empty { margin-bottom: 0; }
  </style>
</head>
<body>
  <main>
    <h1>Учебные задачи на неделю</h1>

    <section aria-labelledby="summary-heading">
      <h2 id="summary-heading">Сводка</h2>
      <p id="summary" role="status">
        Всего: <strong id="total"></strong> ч ·
        Выполнено: <strong id="completed"></strong> ч ·
        Прогресс: <strong id="progress"></strong>
      </p>
    </section>

    <section aria-labelledby="form-heading">
      <h2 id="form-heading">Новая задача</h2>
      <form id="task-form" novalidate>
        <div class="field">
          <label for="title">Название</label>
          <input id="title" type="text" required aria-describedby="title-error">
          <p id="title-error" class="error" aria-live="polite"></p>
        </div>
        <div class="field">
          <label for="hours">Часы (от 0,5 до 24, шаг 0,5)</label>
          <input id="hours" type="text" inputmode="decimal" required aria-describedby="hours-error">
          <p id="hours-error" class="error" aria-live="polite"></p>
        </div>
        <button type="submit">Добавить задачу</button>
      </form>
    </section>

    <section aria-labelledby="tasks-heading">
      <h2 id="tasks-heading">Задачи</h2>
      <div class="filters" role="group" aria-label="Фильтр задач">
        <button type="button" data-filter="all" aria-pressed="true">Все</button>
        <button type="button" data-filter="active" aria-pressed="false">Активные</button>
        <button type="button" data-filter="done" aria-pressed="false">Готовые</button>
      </div>
      <ul id="tasks"></ul>
      <p id="empty" hidden>Нет задач для отображения.</p>
    </section>
  </main>

  <script>
    const tasks = [
      { id: 1, title: 'Чтение', hours: 2, done: true },
      { id: 2, title: 'Практика', hours: 4, done: false },
      { id: 3, title: 'Повторение', hours: 2, done: false }
    ];
    let nextId = 4;
    let filter = 'all';

    const form = document.getElementById('task-form');
    const titleInput = document.getElementById('title');
    const hoursInput = document.getElementById('hours');
    const taskList = document.getElementById('tasks');
    const empty = document.getElementById('empty');
    const filterButtons = document.querySelectorAll('[data-filter]');
    const numberFormat = new Intl.NumberFormat('ru-RU');

    function renderSummary() {
      let total = 0;
      let completed = 0;
      for (const task of tasks) {
        total += task.hours;
        if (task.done) completed += task.hours;
      }
      const progress = total === 0 ? 0 : Math.round(100 * completed / total);

      document.getElementById('total').textContent = numberFormat.format(total);
      document.getElementById('completed').textContent = numberFormat.format(completed);
      document.getElementById('progress').textContent = progress + '%';
    }

    function renderList() {
      const visibleTasks = filter === 'all'
        ? tasks
        : tasks.filter(task => task.done === (filter === 'done'));
      const rows = visibleTasks.map(task => {
        const row = document.createElement('li');
        const label = document.createElement('label');
        label.className = 'task-label';
        const checkbox = document.createElement('input');
        checkbox.type = 'checkbox';
        checkbox.checked = task.done;
        checkbox.setAttribute('aria-label', 'Выполнено: ' + task.title);
        const taskText = document.createElement('span');
        taskText.textContent = task.title + ' · ' + numberFormat.format(task.hours) + ' ч';
        label.append(checkbox, taskText);

        const remove = document.createElement('button');
        remove.type = 'button';
        remove.textContent = 'Удалить';
        remove.setAttribute('aria-label', 'Удалить задачу «' + task.title + '»');

        checkbox.addEventListener('change', () => {
          task.done = checkbox.checked;
          renderSummary();
          if (filter !== 'all') {
            renderList();
            focusFilter();
          }
        });
        remove.addEventListener('click', () => {
          const index = tasks.findIndex(item => item.id === task.id);
          tasks.splice(index, 1);
          renderList();
          renderSummary();
          focusFilter();
        });

        row.append(label, remove);
        return row;
      });

      taskList.replaceChildren(...rows);
      empty.hidden = rows.length > 0;
    }

    function focusFilter() {
      document.querySelector('[data-filter][aria-pressed="true"]').focus();
    }

    function showError(input, message) {
      document.getElementById(input.id + '-error').textContent = message;
      input.setAttribute('aria-invalid', String(Boolean(message)));
    }

    form.addEventListener('submit', event => {
      event.preventDefault();
      const title = titleInput.value.trim();
      const hoursText = hoursInput.value.trim();
      const hours = /^\d+(?:[.,]\d+)?$|^[.,]\d+$/.test(hoursText)
        ? Number(hoursText.replace(',', '.'))
        : NaN;
      const validHours = hours >= 0.5 && hours <= 24 && Number.isInteger(hours * 2);

      showError(titleInput, title ? '' : 'Введите название задачи.');
      showError(hoursInput, validHours ? '' : 'Введите число от 0,5 до 24 с шагом 0,5.');
      if (!title || !validHours) {
        (title ? hoursInput : titleInput).focus();
        return;
      }

      tasks.push({ id: nextId++, title, hours, done: false });
      form.reset();
      renderList();
      renderSummary();
      titleInput.focus();
    });

    for (const button of filterButtons) {
      button.addEventListener('click', () => {
        filter = button.dataset.filter;
        for (const option of filterButtons) {
          option.setAttribute('aria-pressed', String(option === button));
        }
        renderList();
      });
    }

    renderList();
    renderSummary();
  </script>
</body>
</html>
```

### Что проверить вручную

Логику проверил вне браузера. Форма принимает десятичные значения с точкой и запятой.

- Откройте файл без сети и установите ширину области просмотра 390 px. Проверьте читаемость списка и сводки, подписи полей и кнопок и отсутствие горизонтальной прокрутки, в том числе с длинным названием задачи.
- Проверьте начальную сводку — 2 из 8 часов и 25%. При исходных статусах «Активные» показывает «Практику» и «Повторение», «Готовые» — только «Чтение», «Все» — все три задачи. Сводка одинакова во всех трёх режимах.
- В режиме «Все» отметьте «Практику» выполненной. Сводка должна показать 6 из 8 часов и 75%. Снимите отметку и проверьте возврат к 2 из 8 часов и 25%.
- Отправьте пустое название, название из пробелов, пустые часы и значения `0`, `-1`, `5.24`, `25` и `abc`. Проверьте, что ошибка появляется рядом с соответствующим полем, оба значения сохраняются и задача не добавляется.
- Добавьте задачи с `0.5`, `2` и `24` часами. Проверьте, что они появляются как активные, пробелы по краям названия убираются, а сводка пересчитывается.
- Удалите все задачи. В сводке должны отобразиться 0 общих часов, 0 выполненных часов и 0%.
