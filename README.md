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

\* You can edit or delete the created projects only if it does not have active contests.

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

\*We can Edit the project’s Domain and Path if the existing project has zero leagues created.  
\*\*We cannot Delete Sports if the project contains leagues with the selected Sports.  
\*\*\*We cannot Edit Points & Ranges if the project has at least one game with status started or finished.

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

\*Field “Rules & Prizes” is only required if visibility = public.  
\*\*If Odds range is not checked, we assume there are no limits.  
\*\*\*You can only delete the created contest if the first game has not started yet.  
\*\*\*\*MVP allows creating leagues for admins only  
\*\*\*\*\*MVP allows only public type of contests

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

See the demo table with example of parameters’ calculations for a single tipster

---

## Parameters & Calculations

### Points

Points are the sum of points for all single tips. Can be zero or higher.

Users can earn points for a single tip according to the rules specified in the project's settings. The user gets 0 points for lost or void tips.

Points calculation for a single tip (example)

Winning single tip (probability between 81% - 90%) - 2 points  
Winning single tip (probability between 71% - 80%) - 4 points  
Winning single tip (probability between 61% - 70%) - 6 points  
Winning single tip (probability between 51% - 60%) - 8 points  
…

### Profit

Profit = SUM of profit of all single tips. Can be positive, zero or negative.

Profit of a single tip is calculated the following way:

Each tip costs 100 coins.

If the tip wins, returns are calculated (100 coins x Odds - e.g., 100 X 1.45 = 145).

If the tip loses, the amount is deducted from the total (-100 coins x Odds - e.g., -100 X 1.45 = -145).

If the tip is void or open, it costs zero coins (0 coins x Odds - e.g., 0 X 1.45 = 0).

### YIELD

YIELD = (P / S) х 100%, где

P – прибыль или убыток (Profit).  
S – сумма ставок (coins).

YIELD показывает, насколько эффективна та или иная стратегия ставок, и на короткой дистанции уже можно видеть прогресс прогнозиста.

Например, игрок сделал 20 ставок по 100, чистая сумма ставок составит 2 000. Эти ставки принесли вам прибыль 50. Принцип расчёта будет следующим:

YIELD = (50 / 2000) х 100% = 2.5%

### WinRate

WinRate = W / W + L

W - number of tips with status Won  
L - number of tips with status Lost

We exclude tips with status Void or Open from the calculations.

---

## Tip Statuses

Tip Won - if API returns Is Won=true for the tip

Tip Lost - if API returns Is Won=false for the tip

Tip Void - if API returns Is Won=void for the tip

Tip Open - if game Status=planned
