# Документация по настройке Bubble — Правовой маршрутизатор
## Раздел: страница `route` → передача данных в `result`

---

## 1. Контекст

Страница `/route` реализует пошаговый мастер выбора юридического маршрута. На странице реализованы две ветки:

- **А-1** — Новый договор (шаги: Group_A1_1 → Group_A1_2 → Group_A1_3)
- **А-2** — Дополнительное соглашение (шаги: Group_A2_1 → Group_A2_2 → Group_A2_3)

Навигация между шагами управляется через **Custom States** страницы `route`:

| State | Тип | Назначение |
|---|---|---|
| `current_step` | number | Номер текущего шага (0 = выбор типа) |
| `route_type` | text | Выбранная ветка: `"A1"` или `"A2"` |

В конце каждой ветки находится кнопка **`Button to result`**, которая:
1. Создаёт запись в таблице `Request`
2. Переходит на страницу `result`, передавая созданную запись

---

## 2. Структура элементов страницы `route`

### Ветка А-1 — Новый договор

```
Group_A1_1  (current_step = 1, route_type = "A1")
  └── Group Kto_kontragent
        ├── Input "ООО Ромашка или ИП"     ← counterparty_name
        └── Checkbox A                      ← counterparty_unknown

Group_A1_2  (current_step = 2, route_type = "A1")
  ├── Dropdown choice type of dogovor       ← contract_subject
  └── Input Drugoe (typy of dogovor)        ← other_basis (если выбрано "Иное")

Group_A1_3  (current_step = 3, route_type = "A1")
  └── Group Choice_of_form_(tipic-untipic)
        └── RadioButtons TipicalDogovor     ← contract_form_type
  └── Group Назад-Далее
        ├── Button Back A1_2
        └── Button to result               ← НАСТРАИВАЕТСЯ В ЭТОМ РАЗДЕЛЕ
```

### Ветка А-2 — Дополнительное соглашение

```
Group_A2_1  (current_step = 1, route_type = "A2")
  └── Group Kto_kontragent_DS
        ├── Input "ООО Ромашка или ИП"     ← counterparty_name
        └── Checkbox A                      ← counterparty_unknown

Group_A2_2  (current_step = 2, route_type = "A2")
  └── Group Rekviziti_dogovora_DS
        └── [Input реквизиты]               ← contract_details

Group_A2_3  (current_step = 3, route_type = "A2")
  ├── Checkbox DS_srok_postavki             ↘
  ├── Checkbox DS_obem_postavki             ↘
  ├── Checkbox DS_srok_dogovora             ↘  ds_subjects (list)
  ├── Checkbox DS_summ_dogovora             ↗
  ├── Checkbox DS_predmet_dogovora          ↗
  ├── Checkbox DS_rekvizitov                ↗
  └── Group DS_izmenenie_inoe
        ├── Checkbox DS_inoe
        └── Input Drugoe (typy of dogovor)  ← other_basis
  └── Group Назад-Далее
        ├── Button Back A2_2
        └── Button to result               ← НАСТРАИВАЕТСЯ В ЭТОМ РАЗДЕЛЕ
```

---

## 3. Предварительные требования

Перед настройкой workflow убедиться, что:

- [ ] В таблице `Request` добавлено поле `pdf_doc2` (file)
- [ ] В таблице `Request` добавлено поле `counterparty_unknown` (yes/no)
- [ ] В таблице `RouteType` в App Data созданы записи с кодами `A1` и `A2`
- [ ] Страница `result` существует в проекте

---

## 4. Настройка Workflow — Button to result (ветка А-1)

**Расположение кнопки:** `Group_A1_3` → `Group Назад-Далее` → `Button to result`

### Как открыть workflow

1. Кликни на кнопку `Button to result` в Group_A1_3
2. Перейди на вкладку **Workflow** в правой панели
3. Нажми **Click here to add an action**

---

### Action 1 — Create a new Request

**Путь:** `Data (Things)` → `Create a new thing...`

| Параметр | Значение |
|---|---|
| Type | `Request` |

**Поля для заполнения:**

| Поле | Значение в Bubble |
|---|---|
| `author` | `Current User` |
| `route_type` | `Search for RouteType` → Add constraint: `code = "A1"` → `:first item` |
| `status` | `draft` |
| `counterparty_name` | `Input "ООО Ромашка или ИП"'s value` (из Group_A1_1) |
| `counterparty_unknown` | `Checkbox A's value` (из Group_A1_1) |
| `contract_subject` | `Dropdown choice type of dogovor's value` (из Group_A1_2) |
| `other_basis` | `Input Drugoe (typy of dogovor)'s value` (из Group_A1_2) |
| `contract_form_type` | `RadioButtons TipicalDogovor's value` (из Group_A1_3) |

> **Важно:** поле `route_type` требует поиска по БД, а не текстового значения. Используй `Do a search for RouteType` → Constraint: `code = "A1"` → `:first item`

---

### Action 2 — Navigate to result

**Путь:** `Navigation` → `Navigate to page...`

| Параметр | Значение |
|---|---|
| Destination | `result` |
| Data to send | `Result of Step 1` |

> **Важно:** `Result of Step 1` — это автоматически созданная ссылка на запись из Action 1. Выбирается из выпадающего списка в поле `Data to send`.

---

## 5. Настройка Workflow — Button to result (ветка А-2)

**Расположение кнопки:** `Group_A2_3` → `Group Назад-Далее` → `Button to result`

### Action 1 — Create a new Request

**Путь:** `Data (Things)` → `Create a new thing...`

| Параметр | Значение |
|---|---|
| Type | `Request` |

**Поля для заполнения:**

| Поле | Значение в Bubble |
|---|---|
| `author` | `Current User` |
| `route_type` | `Search for RouteType` → Add constraint: `code = "A2"` → `:first item` |
| `status` | `draft` |
| `counterparty_name` | `Input "ООО Ромашка или ИП"'s value` (из Group_A2_1) |
| `counterparty_unknown` | `Checkbox A's value` (из Group_A2_1) |
| `contract_details` | `Input реквизиты's value` (из Group_A2_2) |
| `ds_subjects` | см. блок ниже |
| `other_basis` | `Input Drugoe (typy of dogovor)'s value` (из Group_A2_3) |

#### Заполнение поля `ds_subjects` (List of DsSubjects)

Поле принимает список объектов `DsSubject`. Для каждого отмеченного чекбокса нужно добавить соответствующую запись из таблицы `DsSubject`.

Механика в Bubble:

```
ds_subjects = Search for DsSubject:
  constraint: code is in [список кодов отмеченных чекбоксов]
```

Альтернативный способ (проще для MVP): вместо поиска по списку — использовать условия `when Checkbox DS_srok_postavki is checked → add item`. Реализуется через **List of DsSubjects** с ручным добавлением через `:plus item`:

```
ds_subjects:
  + Search for DsSubject where code = "DS_srok_postavki" :first item
    (only when Checkbox DS_srok_postavki is checked)
  + Search for DsSubject where code = "DS_obem_postavki" :first item
    (only when Checkbox DS_obem_postavki is checked)
  ... и так далее для каждого чекбокса
```

> **Примечание:** В Bubble это реализуется через составной список с условными `:filtered` или через отдельный Custom State типа `list of DsSubject`, который наполняется на шаге А-2.3 по мере отметки чекбоксов.

---

### Action 2 — Navigate to result

**Путь:** `Navigation` → `Navigate to page...`

| Параметр | Значение |
|---|---|
| Destination | `result` |
| Data to send | `Result of Step 1` |

---

## 6. Настройка страницы `result`

> Выполняется после завершения настройки workflow на странице `route`.

### Шаг 1 — Установить Content type

1. Открой страницу `result` в редакторе
2. Кликни на пустую область страницы (не на элемент)
3. В правой панели найди **Type of content**
4. Выбери `Request`

После этого страница будет получать объект `Request` при переходе с `route`, и можно будет обращаться к его полям через `Current Page Request's [поле]`.

### Шаг 2 — Подключить поля к элементам

Каждый текстовый элемент на странице `result` подключается через **Insert dynamic data**:

| Элемент на result | Dynamic data |
|---|---|
| Имя автора | `Current Page Request's author's first_name` + `last_name` |
| Подразделение | `Current Page Request's author's department's name` |
| Контактные данные | `Current Page Request's author's email` |
| Тип запроса | `Current Page Request's route_type's title` |
| Контрагент | `Current Page Request's counterparty_name` |
| Предмет договора | `Current Page Request's contract_subject` |
| Реквизиты договора | `Current Page Request's contract_details` |
| Тип формы | `Current Page Request's contract_form_type` |
| Предмет ДС | `Current Page Request's ds_subjects's label` (список) |
| Срок согласования | `Search for AdminSettings where route_type = Current Page Request's route_type :first item's approval_days` |
| Адресат «Кому» | `Search for AdminSettings where route_type = Current Page Request's route_type :first item's addressee` |

---

## 7. Проверка после настройки

После настройки обоих workflow провести тестовый прогон:

- [ ] Пройти маршрут А-1 до конца → нажать `Button to result`
- [ ] Убедиться, что в `App Data → Request` появилась новая запись с правильными полями
- [ ] Убедиться, что страница `result` открылась и отображает данные из созданной записи
- [ ] Повторить для маршрута А-2
- [ ] Проверить, что поле `route_type` заполнено объектом (не пустое)
- [ ] Проверить, что `ds_subjects` содержит список отмеченных предметов ДС

---

## 8. Известные ограничения MVP

| Ограничение | Описание |
|---|---|
| `ds_subjects` — ручной маппинг | Для каждого чекбокса нужен отдельный поиск по `DsSubject`. При добавлении новых предметов ДС — обновлять workflow |
| `pdf_doc1`, `pdf_doc2` | Генерация PDF не реализована на данном этапе. Поля резервируются в БД, функция добавляется позже |
| Ветки Б-1, Б-2 | Не реализуются в текущей итерации MVP |

