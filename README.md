# Predictions Service

**Цель:** создать в админке отдельный раздел (сервис) для добавления прогнозов на спортивные матчи и ранжирования участников по набранным баллам.

Платформа должна быть мульти-клиентной. То есть, мы создаём конкурсы с привязкой к проекту (домену).

---

## Сущность “Projects”

Список добавленных доменов (редактировать, удалить)

**Columns:**
- Name
- Domain
- Path
- Buttons Save, Edit, Delete\*

\* Вы можете редактировать или удалить созданные проекты только в том случае, если у них нет активных конкурсов.

### Кнопка “Add New Project”

На экране добавления/редактирования нового проекта доступны следующие поля:

| Field name | Field type | Value |
|---|---|---|
| Name | Text | Min 3 chars |
| Domain\* | Text | Min 3 chars |
| Path\* | URL | https://domain.com/predictions/\* or https://tips.domain.com/\* |
| Sports\*\* | Dropdown multiselect | Football, Basketball |
| Points & Ranges\*\*\* | Ranges from X to Y, number |  |

### Points & Ranges

| Probability from % | Probability to % | Points | Action |
|---:|---:|---:|---|
| 0 | 30 | 12 | Delete |
| 31 | 50 | 10 | Delete |
| 51 | 60 | 8 | Delete |
| 61 | 70 | 6 | Delete |
| 71 | 80 | 4 | Delete |
| 81 | 90 | 2 | Delete |
| 91 | 100 | 1 | Delete |
|  |  |  | Add |

**Buttons:** “Save Project” and “Delete Project”

*Мы можем редактировать Domain и Path проекта, если в существующем проекте создано 0 лиг.
**Мы не можем удалить Sports, если проект содержит лиги с выбранными Sports.
***Мы не можем редактировать Points & Ranges, если в проекте есть хотя бы одна игра со статусом started или finished.

---

## Сущность “Contests”

Показываем список созданных contests (редактировать, удалить)

- Поле Search by name
- Filters by date range, visibility, sports, league

Таблица созданных contests с следующими колонками:

- Contest Name
- Start date
- Finish date
- Visibility (public or private)
- Sports (sports added)
- Leagues (count of leagues added)
- Tipsters (participants count +link)
- Edit
- Delete

### Кнопка “Add New Contest”

На экране добавления/редактирования нового Contest доступны следующие поля:

| Field name | Field type | Value |
|---|---|---|
| Name | text | Min 5 max 100 chars |
| Short Description | HTML | Min 20 chars |
| Rules & Prizes\* | HTML | No chars limits |
| Logo | IMG | webp, png, jpeg |
| start | date | Jan 22, 2026 |
| finish | date | Jan 31, 2026 or {league last event} |
| creator\*\*\*\* | dropdown | Admin |
| visibility\*\*\*\*\* | dropdown | Public |
| Add sports | Dropdown select | Football |
| Add leagues | Dropdown select | Premier League |
| Select games | Dropdown select | All games or pick games by date Jan 23, 2026 |
| Add markets | Dropdown select | Match Result |
| Odds range\*\* | range | From 0 to 10 |
| Minimum number of tips to be eligible for prizes | text | =>0 |
| Rank tipsters by | dropdown | Points |
| CRUD | buttons | Save / Delete\*\*\* |

*Поле “Rules & Prizes” обязательно только если visibility = public.
**Если Odds range не отмечен, считаем, что ограничений нет.
***Вы можете удалить созданный contest только если первая игра ещё не началась.
****MVP позволяет создавать лиги только для администраторов
*****MVP позволяет только public тип contests

---

## Сущность “Tipsters”

Показываем список зарегистрированных пользователей, которые сделали хотя бы один прогноз.

Таблица со списком пользователей и колонки:

**Columns:**
- Tipster Name
- Points
- Profit
- Yield
- Average Odds
- WinRate
- Total Tips (count)
- Tips Won (count)
- Tips Lost (count)
- Tips Void (count)
- Tips Open (count)
- See Tipster Stats (открывается Tipster Stats)

### Фильтры
- Search by nickname
- Filters by contest, year, year & month, visibility, sports, leagues, Last X Tips
- Sort by points, profit, yield, winrate

Показываем Total number of results внизу страницы. По умолчанию — 20 строк на страницу (можно изменить на показ 20, 50, 100 строк). Добавляем пагинацию для перехода между страницами.

---

## Tipster Stats

По клику на “See Tipster Stats” открывается таблица с данными по конкретному типстеру.

[Смотрите демонстрационную таблицу с примером расчётов параметров для одного типстера](https://docs.google.com/spreadsheets/d/1-L8ejXHC8OJ8xQJJhYD706sg9Hf8S9v4kTyrO8R49SQ/edit?usp=sharing)

---

##Параметры и расчёты

###Points

Points — это сумма points по всем одиночным tips. Может быть 0 или больше.

Пользователи получают points за один tip согласно правилам, заданным в настройках проекта. За tips со статусом Lost или Void пользователь получает 0 points.

Расчёт points для одного tip (пример)

- Выигрышный одиночный tip (probability между 81% - 90%) — 2 points
- Выигрышный одиночный tip (probability между 71% - 80%) — 4 points
- Выигрышный одиночный tip (probability между 61% - 70%) — 6 points
- Выигрышный одиночный tip (probability между 51% - 60%) — 8 points
…

###Profit

Profit = SUM profit по всем одиночным tips. Может быть положительным, нулевым или отрицательным.

Profit одного tip рассчитывается следующим образом:

Каждый tip стоит 100 coins.

- Если tip выигрывает, возврат рассчитывается так (100 coins x Odds — например, 100 X 1.45 = 145).

- Если tip проигрывает, сумма вычитается из общего результата (-100 coins x Odds — например, -100 X 1.45 = -145).

- Если tip имеет статус Void или Open, он стоит 0 coins (0 coins x Odds — например, 0 X 1.45 = 0).

###YIELD

YIELD = (P / S) х 100%, где

P – прибыль или убыток (Profit).
S – сумма ставок (coins).

YIELD показывает, насколько эффективна та или иная стратегия ставок, и на короткой дистанции уже можно видеть прогресс прогнозиста.

Например, игрок сделал 20 ставок по 100, чистая сумма ставок составит 2 000. Эти ставки принесли вам прибыль 50. Принцип расчёта будет следующим:

YIELD = (50 / 2000) х 100% = 2.5%

###WinRate

WinRate = W / W + L

W - number of tips with status Won
L - number of tips with status Lost

Мы исключаем tips со статусом Void или Open из расчётов.

##Tip Statuses

- Tip Won - если API возвращает Is Won=true для tip

- Tip Lost - если API возвращает Is Won=false для tip

- Tip Void - если API возвращает Is Won=void для tip

- Tip Open - если game Status=planned
