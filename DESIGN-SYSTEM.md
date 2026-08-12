# Дизайн-система

Инструкция, как перенести это оформление в другой проект. Всё сделано на голом
HTML + Tailwind из CDN, без сборки, без React и без библиотек компонентов —
поэтому переносится копированием, а не установкой.

Файл описывает три вещи:

1. **Фундамент** — токены, конфиг Tailwind, каркас страницы. Копируется целиком.
2. **Компоненты** — что из чего собрано и какими классами.
3. **Правила** — почему сделано именно так. Это важнее кода: код можно
   переписать, правила — то, что делает интерфейс узнаваемым.

Рядом лежит готовый образец — [`starter/index.html`](starter/index.html): в нём
собрано всё описанное ниже, и его стоит открыть в браузере до чтения. Токены и
классы вынесены в [`starter/tokens.css`](starter/tokens.css).

Оформление выросло из приложения «Распаковка конкурентов»
([christophermaasi/unpack-competitors](https://github.com/christophermaasi/unpack-competitors),
папка `ТЕСТ-веб-приложение/3-скрипт-веб-приложения-новый/`): `Index.html`
(каркас, токены, общие помощники), `Dashboard.html` (сетка таблиц), `Card.html`,
`Queries.html`, `Heatmap.html`. Там же — живые примеры всего, о чём идёт речь.

---

## 0. Чем это является

Это **shadcn/ui (тема zinc), переложенный на ванильный HTML**. Мы не тянем сам
shadcn — берём только его систему токенов и визуальный язык:

- нейтральная серо-синяя гамма (zinc), цвет только там, где он что-то значит;
- всё на карточках: рамка `1px`, скругление `0.5rem`, фон `card`;
- две темы из коробки, переключение классом `dark` на `<html>`;
- плотная типографика: базовый текст `13–14px`, подписи `11–12px`;
- никаких теней-«подушек», кроме всплывающих панелей и подсказок.

---

## 1. Фундамент — копировать как есть

### 1.1. Подключение и конфиг Tailwind

```html
<script src="https://cdn.tailwindcss.com"></script>
<script>
  tailwind.config = {
    darkMode: 'class',
    theme: {
      extend: {
        colors: {
          border: 'hsl(var(--border))',
          background: 'hsl(var(--background))',
          foreground: 'hsl(var(--foreground))',
          card:    { DEFAULT: 'hsl(var(--card))',    foreground: 'hsl(var(--card-foreground))' },
          muted:   { DEFAULT: 'hsl(var(--muted))',   foreground: 'hsl(var(--muted-foreground))' },
          accent:  { DEFAULT: 'hsl(var(--accent))',  foreground: 'hsl(var(--accent-foreground))' },
          primary: { DEFAULT: 'hsl(var(--primary))', foreground: 'hsl(var(--primary-foreground))' },
          sidebar: { DEFAULT: 'hsl(var(--sidebar))', foreground: 'hsl(var(--sidebar-foreground))' }
        },
        borderRadius: { lg: 'var(--radius)', md: 'calc(var(--radius) - 2px)', sm: 'calc(var(--radius) - 4px)' }
      }
    }
  };
</script>
```

Ключевой приём: цвета объявлены как `hsl(var(--…))`, а сами переменные хранят
**тройку HSL без обёртки** (`240 10% 3.9%`). Это позволяет писать
`hsl(var(--primary) / .25)` — цвет с прозрачностью без второй переменной. Так во
всём проекте задаются полупрозрачные подложки.

### 1.2. Токены

```css
:root {
  --background: 0 0% 100%;      --foreground: 240 10% 3.9%;
  --card: 0 0% 100%;            --card-foreground: 240 10% 3.9%;
  --muted: 240 4.8% 95.9%;      --muted-foreground: 240 3.8% 46.1%;
  --accent: 240 4.8% 95.9%;     --accent-foreground: 240 5.9% 10%;
  --primary: 240 5.9% 10%;      --primary-foreground: 0 0% 98%;
  --border: 240 5.9% 90%;
  --sidebar: 240 4.8% 97.5%;    --sidebar-foreground: 240 5.3% 26.1%;
  --head: 240 6% 90%;           /* полоса шапки таблицы */
  --radius: 0.5rem;
}
.dark {
  --background: 240 10% 3.9%;   --foreground: 0 0% 98%;
  --card: 240 10% 3.9%;         --card-foreground: 0 0% 98%;
  --muted: 240 3.7% 15.9%;      --muted-foreground: 240 5% 64.9%;
  --accent: 240 3.7% 15.9%;     --accent-foreground: 0 0% 98%;
  --primary: 0 0% 98%;          --primary-foreground: 240 5.9% 10%;
  --border: 240 3.7% 15.9%;
  --sidebar: 240 5.9% 7%;       --sidebar-foreground: 240 5% 84.9%;
  --head: 240 5% 14.5%;
}
body { font-family: ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, sans-serif;
       -webkit-font-smoothing: antialiased; }
```

Два токена сверх стандартного shadcn — они наши:

- `--sidebar` — фон боковой панели. Чуть отличается от `background`, чтобы
  панель читалась без разделительной тени.
- `--head` — фон шапки таблицы. **Заметно темнее карточки в обеих темах**: у
  таблиц залипающая шапка, и она обязана быть непрозрачной и отличимой.

Для таблиц объявляются ещё две линии (в `Dashboard.html`), потому что обычный
`--border` для плотной сетки слишком контрастный:

```css
:root { --db-line: 240 6% 89%; --db-line-strong: 240 6% 78%; }
.dark  { --db-line: 240 4% 23%; --db-line-strong: 240 4% 33%; }
```

### 1.3. Переключение темы

Светлая по умолчанию, выбор запоминается:

```js
try { if (localStorage.getItem('theme') === 'dark') document.documentElement.classList.add('dark'); } catch (e) {}

themeButton.onclick = function () {
  var dark = document.documentElement.classList.toggle('dark');
  try { localStorage.setItem('theme', dark ? 'dark' : 'light'); } catch (e) {}
};
```

`try/catch` не для красоты: в iframe с закрытыми куками `localStorage` бросает
исключение и без него ломается вся страница.

### 1.4. Базовые классы поверх Tailwind

```css
.scroll-area::-webkit-scrollbar { height: 10px; width: 10px; }
.scroll-area::-webkit-scrollbar-thumb { background: hsl(var(--border)); border-radius: 6px; }
.scroll-area::-webkit-scrollbar-track { background: transparent; }

.nav-item.active { background: hsl(var(--accent)); color: hsl(var(--foreground)); font-weight: 500; }

.field { border-radius: calc(var(--radius) - 2px); border: 1px solid hsl(var(--border));
         background: hsl(var(--background)); padding: .35rem .5rem; font-size: .8125rem; color: inherit; }
.field:focus { outline: none; box-shadow: 0 0 0 2px hsl(var(--primary) / .25); }
```

`.field` — единственный класс поля ввода на весь проект. Им оформлены `input`,
`select`, `input[type=date]`, `input[type=search]` и кнопка, открывающая
выпадающий список. Одинаковый вид у всего, что «принимает ввод», — половина
аккуратности интерфейса.

---

## 2. Каркас страницы

Схема: сайдбар слева фиксирован, справа колонка «шапка + прокручиваемый
контент». Скролл только у `main`, страница целиком не скроллится никогда.

```html
<body class="bg-background text-foreground">
<div class="flex h-screen overflow-hidden">

  <aside id="sidebar"
    class="fixed inset-y-0 left-0 z-50 w-[260px] shrink-0 border-r border-border bg-sidebar
           flex flex-col -translate-x-full lg:translate-x-0 lg:static transition-transform duration-200">

    <!-- логотип: квадрат 32px + два ряда текста -->
    <div class="h-16 flex items-center gap-2.5 px-5 border-b border-border">
      <div class="h-8 w-8 rounded-md bg-primary text-primary-foreground grid place-items-center
                  text-sm font-bold shrink-0">Р</div>
      <div class="min-w-0">
        <div class="text-sm font-semibold truncate">Распаковка</div>
        <div class="text-xs text-muted-foreground truncate">конкурентов</div>
      </div>
    </div>

    <nav id="nav" class="flex-1 overflow-y-auto p-3 space-y-1"></nav>

    <!-- нижний блок: внешние ссылки, настройки, тема -->
    <div class="p-3 border-t border-border space-y-1"> … </div>
  </aside>

  <div id="overlay" class="fixed inset-0 z-40 bg-black/50 hidden lg:hidden"></div>

  <div class="flex-1 flex flex-col min-w-0">
    <header class="h-14 shrink-0 border-b border-border flex items-center gap-3 px-5">
      <button id="burger" class="lg:hidden -ml-1 p-2 rounded-md hover:bg-accent">…</button>
      <div class="min-w-0 flex-1">
        <h1 id="pageTitle" class="text-base font-semibold tracking-tight truncate">Заголовок</h1>
        <p id="pageSubtitle" class="text-xs text-muted-foreground truncate">Подзаголовок</p>
      </div>
      <button class="…вторичная кнопка…">Обновить</button>
      <button class="…главная кнопка…">Действие</button>
    </header>

    <main class="flex-1 overflow-auto scroll-area">
      <section data-screen="dashboard" class="hidden p-4"> … </section>
      <section data-screen="settings"  class="hidden p-4"> … </section>
    </main>
  </div>
</div>
```

Размеры каркаса, если переносить один в один: сайдбар **260px**, шапка сайдбара
**64px** (`h-16`), шапка контента **56px** (`h-14`), поля секций **16px**
(`p-4`), брейкпоинт мобильной раскладки **lg (1024px)**.

### Роутер

Роутера как такового нет — есть переключение секций и один источник правды о
разделах:

```js
var SCREENS = [
  { id: 'dashboard', label: 'Дашборд', subtitle: 'Сравнение товаров по всем показателям',
    icon: '<rect width="7" height="9" x="3" y="3" rx="1"/>…' },
  …
];

function go(id) {
  document.querySelectorAll('[data-screen]').forEach(function (s) {
    s.classList.toggle('hidden', s.getAttribute('data-screen') !== id);
  });
  document.querySelectorAll('[data-go]').forEach(function (b) {
    b.classList.toggle('active', b.getAttribute('data-go') === id);
  });
  pageTitle.textContent = screen.label;
  pageSubtitle.textContent = screen.subtitle;
  closeSidebar();
  …загрузка данных раздела…
}
```

Каждый раздел описан один раз: пункт меню, заголовок, подзаголовок и иконка
берутся из одного объекта. Иконки — **строки с содержимым `<svg>`**, а обёртка
рисуется в `navButton()`; так у всех иконок гарантированно одинаковые размер,
толщина линии и скругления.

Иконки — Lucide, вставленные как inline-путь. Обёртка всегда одна:

```html
<svg class="h-4 w-4 shrink-0" viewBox="0 0 24 24" fill="none" stroke="currentColor"
     stroke-width="2" stroke-linecap="round" stroke-linejoin="round">…</svg>
```

### Пункт меню

```html
<button class="nav-item w-full flex items-center gap-3 rounded-md px-3 py-2 text-sm
               text-sidebar-foreground hover:bg-accent hover:text-foreground transition-colors"
        data-go="dashboard">
  <svg class="h-4 w-4 shrink-0" …>…</svg>
  <span>Дашборд</span>
</button>
```

Внешние ссылки в сайдбаре выглядят так же, но это `<a target="_blank">`, и под
названием идёт вторая строка `text-xs text-muted-foreground` — адрес или
пояснение.

---

## 3. Компоненты

### 3.1. Карточка — основа всего

```html
<div class="rounded-lg border border-border bg-card">
  <div class="px-5 py-4 border-b border-border">
    <h2 class="font-semibold">Заголовок</h2>
    <p class="text-sm text-muted-foreground mt-0.5">Пояснение одной строкой</p>
  </div>
  <div class="p-5 space-y-4"> … </div>
</div>
```

Правила: рамка всегда `border-border`, скругление `rounded-lg`, шапка отделена
`border-b`, поля `px-5 py-4` у шапки и `p-5` у тела. Карточки в колонке
разделяются `space-y-5`. Ширина контента ограничивается по смыслу:
`max-w-3xl` — настройки, `max-w-4xl` — формы, таблицы — во всю ширину.

**Акцентная карточка** — когда на экране несколько одинаковых серых блоков и
один из них главный:

```html
<div class="rounded-lg border border-sky-500/50 dark:border-sky-400/40 bg-card shadow-sm overflow-hidden">
  <div class="px-5 py-4 border-b border-sky-500/40 dark:border-sky-400/30 bg-sky-50 dark:bg-sky-950/40">
    …
```

Приём применяется **один раз на экран**. Два акцента = ни одного.

### 3.2. Кнопки

Три вида, больше не заводить:

```html
<!-- главная -->
<button class="rounded-md bg-primary text-primary-foreground px-4 py-2 text-sm font-medium
               hover:opacity-90 transition-opacity">Сохранить</button>

<!-- вторичная -->
<button class="inline-flex items-center gap-2 rounded-md border border-border bg-background
               px-3 py-1.5 text-sm font-medium hover:bg-accent transition-colors">Обновить</button>

<!-- ссылка-действие -->
<button class="text-sky-600 dark:text-sky-400 hover:underline">Показать все</button>
```

У главной наведение — прозрачность, у вторичной — фон. Не наоборот: у главной
кнопки фон уже `primary`, менять его некуда.

Кнопки в шапке контента — компактные (`px-3 py-1.5`), кнопки внутри форм —
крупнее (`px-4 py-2`). Рядом с кнопкой действия почти всегда стоит
`<span class="text-sm text-muted-foreground">` под статус («Сохранено»,
«Проверяю…»).

### 3.3. Поле ввода

```html
<div>
  <label class="text-sm font-medium mb-1.5 block">Название</label>
  <input type="text" class="field w-full" placeholder="ИП Иванов">
  <p class="text-xs text-muted-foreground mt-1.5">Подсказка под полем</p>
</div>
```

Обязательность — звёздочкой в подписи: `<span class="text-rose-500">*</span>`.
Технические значения (ключи, артикулы, id) — `class="field w-full font-mono"`.
Сетка форм: `grid gap-4 sm:grid-cols-2`, широкое поле — `sm:col-span-2`.

### 3.4. Пилюля и чип — два переключателя

`.pill` — выбор метрики в легенде, с цветной точкой ряда:

```css
.pill { display: inline-flex; align-items: center; gap: .4rem; cursor: pointer; user-select: none;
        font-size: .75rem; white-space: nowrap; padding: .15rem .1rem; transition: opacity .15s; }
.pill.off { opacity: .38; }
.pill.off .pill-dot { background: hsl(var(--muted-foreground)) !important; }
.pill-dot { width: 9px; height: 9px; border-radius: 999px; flex: none; }
```

`.chip` — выбор объекта (товара) с картинкой:

```css
.chip { display: inline-flex; align-items: center; gap: .4rem; cursor: pointer; user-select: none;
        font-size: .75rem; white-space: nowrap; border: 1px solid hsl(var(--border));
        border-radius: 999px; padding: .2rem .6rem .2rem .25rem; transition: all .15s; }
.chip.off { opacity: .45; }
.chip.on  { background: hsl(var(--accent)); border-color: hsl(var(--primary) / .35); }
.chip img { width: 20px; height: 20px; border-radius: 999px; object-fit: cover; flex: none; }
```

Разница осмысленная: пилюля выключается **прозрачностью** (её цвет — это цвет
линии на графике, рамка бы с ним спорила), чип — **рамкой и фоном**.

Выключенное состояние никогда не прячется — оно приглушается. Человек должен
видеть, что ещё можно включить.

### 3.5. Переключатель режимов (сегменты)

```css
.seg { display: inline-flex; gap: 2px; padding: 2px; border-radius: 999px;
       background: hsl(var(--muted-foreground) / .12); }
.seg button { font-size: .6875rem; line-height: 1.35; padding: .15rem .6rem; border-radius: 999px;
              white-space: nowrap; opacity: .7; transition: all .15s; }
.seg button.on { background: hsl(var(--card)); opacity: 1; font-weight: 500;
                 box-shadow: 0 1px 2px rgba(0,0,0,.14); }
.seg button:not(.on):hover { opacity: 1; }
```

Активная кнопка — светлая «таблетка» с еле заметной тенью на приглушённом фоне.
Мягче, чем рамка с заливкой, и не тянет на себя внимание в шапке таблицы.

### 3.6. Плашка-перечисление и счётчик

```css
.db-tag { display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical;
          max-width: 100%; font-size: .6875rem; line-height: 1.3; overflow: hidden;
          padding: .1rem .375rem; border-radius: 5px; background: hsl(var(--muted-foreground) / .1); }
.db-tags { display: flex; flex-wrap: wrap; gap: .2rem; }

.db-count { font-size: .6875rem; line-height: 1; padding: .2rem .4rem; border-radius: 999px;
            background: hsl(var(--muted-foreground) / .12); font-weight: 500; }
```

Список значений в узкой ячейке — не строка через `;`, а плашки: по ним видно
хотя бы количество и начало каждого. Обрезка ровно в две строки, полный текст —
в `title`.

Булево — точкой, а не словом: столбец тогда читается сканированием.

```css
.db-flag { display: inline-flex; align-items: center; gap: .35rem; font-size: .75rem; }
.db-flag i { width: 7px; height: 7px; border-radius: 999px; flex: none; background: currentColor; }
```

### 3.7. Прогресс операции

```html
<div class="rounded-lg border border-border bg-card p-5">
  <div class="flex items-start justify-between gap-4">
    <div class="min-w-0">
      <div class="font-medium truncate">Шаг</div>
      <div class="text-sm text-muted-foreground truncate mt-0.5">Последнее сообщение</div>
    </div>
    <button class="shrink-0 …вторичная кнопка…">Отменить</button>
  </div>
  <div class="mt-3 h-2 w-full rounded-full bg-muted overflow-hidden">
    <div class="h-full bg-primary transition-all duration-500" style="width:0%"></div>
  </div>
  <details class="mt-3">
    <summary class="cursor-pointer select-none text-sm text-muted-foreground hover:text-foreground">
      Показать лог</summary>
    <pre class="mt-2 max-h-64 overflow-auto rounded-md bg-muted p-3 text-xs leading-relaxed
                whitespace-pre-wrap"></pre>
  </details>
</div>
```

Подробности всегда под `<details>` — по умолчанию свёрнуты, но доступны.

### 3.8. Пустое состояние

Не «нет данных» точкой, а объяснение и кнопка выхода из тупика:

```js
function emptyState(reason) {
  return '<div class="font-medium text-foreground">Данных пока нет</div>' +
    '<p class="mt-1">Соберите их из кабинета — это займёт несколько минут.</p>' +
    (reason ? '<p class="mt-2 text-xs opacity-70">' + esc(reason) + '</p>' : '') +
    '<button onclick="go(&quot;collect&quot;)" class="mt-4 rounded-md bg-primary ' +
    'text-primary-foreground px-4 py-2 text-sm font-medium hover:opacity-90 ' +
    'transition-opacity">Перейти к выгрузке</button>';
}
```

Контейнер под него: `<div class="text-sm text-muted-foreground py-24 text-center">`.
Он же показывает «Загружаю данные…» и ошибку:

```html
<div class="text-rose-500 font-medium">Ошибка</div><div class="mt-1">текст</div>
```

Три состояния раздела — загрузка, пусто, ошибка — живут в одном узле, тело
раздела рядом под `hidden`. Переключение — двумя `classList.toggle`.

### 3.9. Выпадающая панель и подсказка

```css
#menu { position: fixed; z-index: 70; background: hsl(var(--card));
        border: 1px solid hsl(var(--border)); border-radius: calc(var(--radius) - 2px);
        box-shadow: 0 12px 32px rgba(0,0,0,.28); overflow: hidden; }
#menu label:hover { background: hsl(var(--accent)); }

#tip { position: absolute; z-index: 60; pointer-events: none; display: none;
       background: hsl(var(--card)); border: 1px solid hsl(var(--border));
       border-radius: calc(var(--radius) - 2px); padding: .5rem .6rem; font-size: .75rem;
       box-shadow: 0 8px 24px rgba(0,0,0,.28); min-width: 150px; }
```

Панель **живёт в `<body>` и позиционируется `fixed`**, а не внутри кнопки: кнопка
стоит в шапке таблицы с `overflow: auto`, и любая всплывашка внутри была бы
обрезана. Позиция считается от `getBoundingClientRect()` кнопки; у нижнего края
экрана панель раскрывается вверх. Закрывают её `mousedown` мимо и `Escape` —
обработчики вешаются на документ **по одному разу**, а не при каждом открытии.

---

## 4. Таблицы — главное в этом дизайне

Всё интересное здесь: широкие таблицы с замороженными колонками и залипающей
шапкой. Правила выстраданы, менять их не стоит.

### 4.1. `separate`, а не `collapse`

```css
#table { border-collapse: separate; border-spacing: 0; }
```

**Принципиально.** При `border-collapse: collapse` граница принадлежит таблице, а
не ячейке: залипающая колонка едет вбок, граница остаётся на месте, и в шов
видно уезжающее содержимое. При `separate` каждая ячейка везёт свои границы с
собой. Границы при этом рисуют сами ячейки (`border-b`, `border-r`), поэтому
линии не удваиваются.

### 4.2. Залипающие колонки

Ширины колонок фиксируются числами в JS и оттуда же считаются смещения:

```js
var W_METRIC = 210, W_BEST = 88, W_AVG = 88;
var W_LEFT = W_METRIC + W_BEST + W_AVG;
```

```css
.db-c1 { position: sticky; left: 0;     z-index: 20; }
.db-c2 { position: sticky; left: 210px; z-index: 20; }
.db-c3 { position: sticky; left: 298px; z-index: 20; }
.sticky-head { position: sticky; top: 0; z-index: 30; }
.sticky-head .db-c1, .sticky-head .db-c2, .sticky-head .db-c3 { z-index: 40; }
```

Этажи `z-index`: **20** — залипающие колонки, **30** — залипающая шапка, **40** —
их пересечение, **60** — подсказка, **70** — выпадающая панель, **80** —
полноэкранный просмотр. Держать эту лестницу.

Таблица — `style="table-layout:fixed"` с явным `<colgroup>`: иначе браузер
пересчитывает ширины по содержимому и залипающие смещения разъезжаются.

**У всех залипающих ячеек фон обязан быть непрозрачным** (`bg-card`,
`.head-cell`). Полупрозрачный — и сквозь колонку видно уезжающие ячейки.

Отсюда же правило про зебру: полосатость строк задаётся **градиентом поверх
фона**, а не `background-color`, чтобы не затирать непрозрачную подложку:

```css
.db-odd td   { background-image: linear-gradient(hsl(var(--muted-foreground) / .035),
                                                 hsl(var(--muted-foreground) / .035)); }
.db-total td { background-image: linear-gradient(hsl(var(--muted-foreground) / .1),
                                                 hsl(var(--muted-foreground) / .1)); }
```

### 4.3. Строка-разделитель, растянутая на всю ширину

Такая ячейка прилипнуть не может — она и так начинается у левого края и уезжает
вместе с ним. Прилипает её **содержимое**:

```css
.row-pin { position: sticky; left: 0; padding-right: 12px; }
```

```html
<tr><td colspan="…" class="bg-muted border-y-2 border-border px-3 py-2 text-xs font-semibold
        uppercase tracking-wide text-muted-foreground">
  <span class="row-pin inline-block">Заголовок блока</span></td></tr>
```

Между блоками — не отступ, а строка-распорка цветом фона страницы:

```js
'<tr><td colspan="' + cols + '" class="p-0" style="height:10px;background:hsl(var(--background))"></td></tr>'
```

### 4.4. Сколько линий рисовать

Вертикальной сетки нет вовсе — колонок много, и линия у каждой расчертила бы
экран в клетку. Вертикаль везут только два места:

- правый край замороженной зоны (`--db-line-strong`);
- граница между объектами (`.db-sep`).

Горизонталь — у каждой строки, но слабой линией `--db-line`. Шапка отрывается от
тела мягкой тенью, а не жирной линией:

```css
.sticky-head th { box-shadow: 0 4px 10px -8px rgba(0,0,0,.45); }
```

### 4.5. Выделение опорной колонки

Точка отсчёта (у нас — карточка-лидер) подсвечивается подложкой и краями. Края —
**тенью внутрь, а не рамкой**: рамка добавила бы ширину и разошлась бы с сеткой
соседей.

```css
.db-lead   { background-color: hsl(var(--card));
             background-image: linear-gradient(rgba(99,102,241,.07), rgba(99,102,241,.07)); }
.dark .db-lead { background-image: linear-gradient(rgba(129,140,248,.12), rgba(129,140,248,.12)); }
.sticky-head .db-lead { background-color: hsl(var(--head)); }
.db-lead-l { box-shadow: inset  2px 0 0 rgba(99,102,241,.55); }
.db-lead-r { box-shadow: inset -2px 0 0 rgba(99,102,241,.55); }
.db-lead-l.db-lead-r { box-shadow: inset 2px 0 0 rgba(99,102,241,.55),
                                   inset -2px 0 0 rgba(99,102,241,.55); }
.db-lead-tag { background: rgba(99,102,241,.15); color: #4f46e5; font-size: .625rem;
               border-radius: 999px; padding: .05rem .4rem; letter-spacing: .04em; }
.dark .db-lead-tag { color: #a5b4fc; }
```

### 4.6. Тепловая заливка ячейки

```js
function heatStyle(value, min, max, invert) {
  if (typeof value !== 'number' || !isFinite(value) || max === min) return '';
  var t = (value - min) / (max - min);
  if (invert) t = 1 - t;
  var alpha = (Math.abs(t - 0.5) * 2 * 0.14).toFixed(3);
  return 'background-color:rgba(' + (t >= 0.5 ? '16,185,129' : '244,63,94') + ',' + alpha + ')';
}
```

Максимальная непрозрачность — **0.14**. Заливка должна намекать, а не кричать;
текст поверх неё обязан остаться читаемым в обеих темах. Середина шкалы
прозрачна: подсвечиваются только края.

### 4.7. Ширина колонки по данным

Числа не сокращаем — точность важнее компактности, «1,5 млн» вместо
«1 500 239 ₽» не годится. Значит, ширину колонки надо подбирать под самое
длинное значение, которое в неё попадёт, и мерить **настоящую ширину текста**, а
не количество символов:

```js
var measureNode = null;
function textWidth_(text) {
  if (!measureNode) {
    measureNode = document.createElement('span');
    measureNode.style.cssText = 'position:absolute;left:-9999px;top:0;visibility:hidden;' +
      'white-space:pre;font-size:.75rem;font-variant-numeric:tabular-nums';
    document.body.appendChild(measureNode);
  }
  measureNode.textContent = text;
  return measureNode.getBoundingClientRect().width;
}
function dayColumnWidthExact(texts) {
  var widest = 0;
  texts.forEach(function (t) { if (t) widest = Math.max(widest, textWidth_(t)); });
  return Math.max(62, Math.min(180, Math.ceil(widest) + 11));
}
```

Оценка «символ × 7.3px» ошибается на пробелах, неразрывных пробелах и знаке
валюты, а ошибка повторяется столько раз, сколько колонок, — таблица уезжает на
пол-экрана.

Ширина считается **один раз, при первой отрисовке**, и дальше держится:
переключение метрики не должно двигать сетку под курсором.

---

## 5. Данные на экране

### 5.1. Числа

Все числа — `tabular-nums` (Tailwind-класс `tabular-nums`), выравнивание по
правому краю в колонках итогов, по центру — в дневных.

```js
var fmtInt  = new Intl.NumberFormat('ru-RU', { maximumFractionDigits: 0 });
var fmtNum  = new Intl.NumberFormat('ru-RU', { maximumFractionDigits: 1 });
var fmtNum2 = new Intl.NumberFormat('ru-RU', { minimumFractionDigits: 2, maximumFractionDigits: 2 });

function formatValue(value, format) {
  if (format === 'bool') return value == null || value === '' ? '—' : (value ? 'да' : 'нет');
  if (value === null || value === undefined || value === '') return '—';
  if (typeof value !== 'number' || !isFinite(value)) return '—';
  switch (format) {
    case 'money':    return fmtInt.format(Math.round(value)) + ' ₽';
    case 'percent':  return fmtNum.format(value) + ' %';
    case 'percent2': return fmtNum2.format(value) + ' %';
    case 'rating':   return value.toFixed(2).replace('.', ',');
    case 'int':      return fmtInt.format(Math.round(value));
    case 'hours':    return fmtInt.format(Math.round(value)) + ' ч';
    default:         return fmtNum.format(value);
  }
}
```

Правила отсюда:

- пусто — это `—`, а не `0` и не пустая ячейка;
- знаки после запятой не срезаются даже у круглых чисел: столбик «5,00 %» рядом
  с «0,37 %» читается лучше, чем «5 %»;
- формат — свойство метрики, а не место вызова. Формат объявлен рядом с данными,
  отрисовка про него ничего не знает.

### 5.2. Рост и падение

```js
var GOOD_CLASS = 'text-emerald-700/85 dark:text-emerald-400/80';
var BAD_CLASS  = 'text-rose-700/85 dark:text-rose-400/80';
```

Цвет — приглушённый: в таблице таких значений сотни, полная насыщенность
превращает экран в гирлянду. Направление показывается **стрелкой `↑`/`↓` плюс
цвет**, никогда цветом одним: у метрик бывает `better: 'lower'`, и «вниз =
хорошо» без стрелки не прочитать. Ноль и «нечего сравнить» — серым
(`text-muted-foreground/70`), а не зелёным.

### 5.3. Цвета рядов на графиках

```js
var SERIES_COLORS = ['#a855f7','#f59e0b','#10b981','#3b82f6','#ec4899','#14b8a6','#ef4444',
                     '#8b5cf6','#84cc16','#f97316','#06b6d4','#d946ef','#eab308','#0ea5e9','#94a3b8'];
```

Цвет выдаётся по **индексу сущности в списке**, а не по порядку отрисовки: тогда
у метрики один и тот же цвет в легенде, на графике, в точке строки и в
подсказке. Это единственное место, где в интерфейсе появляется насыщенный цвет.

### 5.4. Акценты блоков

Семь пар «фон + текст», непрозрачных, плюс цветная полоса слева:

```css
.acc-sky     { background: #f3f8ff; color: #0369a1; }
.acc-amber   { background: #fffaf2; color: #b45309; }
.acc-emerald { background: #f2fdf8; color: #047857; }
.acc-violet  { background: #f8f6ff; color: #6d28d9; }
.acc-indigo  { background: #f4f5ff; color: #4338ca; }
.acc-rose    { background: #fff5f7; color: #be123c; }
.acc-teal    { background: #f2fdfb; color: #0f766e; }
.dark .acc-sky     { background: #0b1724; color: #7dd3fc; }
.dark .acc-amber   { background: #21160a; color: #fcd34d; }
.dark .acc-emerald { background: #081b15; color: #6ee7b7; }
.dark .acc-violet  { background: #16102a; color: #c4b5fd; }
.dark .acc-indigo  { background: #10132b; color: #a5b4fc; }
.dark .acc-rose    { background: #250d14; color: #fda4af; }
.dark .acc-teal    { background: #081b1a; color: #5eead4; }

.bar-sky { box-shadow: inset 3px 0 0 #0ea5e9; }   /* и так для каждого цвета */
```

Точка и счётчик в заголовке блока берут цвет через `currentColor` — блок красится
одним классом, а не тремя.

### 5.5. График

Рисуется вручную в SVG, без библиотек. Что важно перенести:

- сетка — 5 горизонталей `stroke="hsl(var(--border))"`, вертикали по границам
  дней с `opacity="0.35"`;
- линия — `stroke-width="2"`, `stroke-linejoin="round"`, точки радиусом 3 с
  заливкой `hsl(var(--card))` и обводкой цветом ряда: точка «прорезает» линию и
  видна на пересечениях;
- **у каждой метрики своя шкала** — показы в тысячах и CTR в процентах на одной
  оси нечитаемы. Подписи оси показываются, только когда метрика одна;
- наведение — не на линию, а на **прозрачные прямоугольники** во всю высоту
  графика, по одному на день. Попасть в столбец мышью легко, в линию — нет;
- подсказка у правого края переворачивается влево (`left: auto; right: …`).

Спарклайн в ячейке: SVG 56×20, `preserveAspectRatio="none"`,
`vector-effect="non-scaling-stroke"` — иначе линия растянется вместе с
координатами и станет разной толщины.

---

## 6. Правила, которые держат всё вместе

1. **Цвет — носитель смысла, а не украшение.** Интерфейс серый. Цветное:
   ряд графика, рост/падение, акцент блока, опорная колонка, одна акцентная
   карточка на экран. Всё остальное — `foreground` / `muted-foreground`.
2. **Приглушать, а не прятать.** Выключенный фильтр, недоступная метрика,
   прошлый период — прозрачностью или `muted-foreground`. Пропавший элемент
   человек ищет, приглушённый — понимает.
3. **Плотность.** Это рабочий инструмент, а не лендинг: `text-sm` в интерфейсе,
   `text-[13px]` в таблицах, `text-xs`/`text-[11px]` в подписях, поля ячеек
   `.45rem .625rem`. Воздух добавляется телу таблицы, но не шапке с зашитыми
   высотами залипания.
4. **Заголовок всегда с подзаголовком.** У раздела, у карточки, у колонки. Верхняя
   строка — что это, нижняя `text-muted-foreground` — что это значит.
5. **Никогда не оставлять пустой экран.** У каждого раздела три состояния —
   «Загружаю данные…», пустое с кнопкой выхода, ошибка красным.
6. **Подсказка стоит там, где возникает вопрос.** Инструкция «где взять ключи» —
   **перед** формой, а не после: человек приходит с пустыми полями и вопросом.
   Пояснение к таблице — строкой над ней, `px-4 py-2 text-xs text-muted-foreground`.
7. **Переходы короткие и только по делу**: `.15s` у переключателей, `transition-colors`
   у наведения, `duration-200` у выезда сайдбара, `duration-500` у прогресса.
   Ничего не появляется с анимацией.
8. **Мобильная раскладка — одна:** сайдбар уезжает за экран и выезжает поверх
   затемнения по бургеру. Таблицы на телефоне скроллятся вбок как есть и не
   перестраиваются.

---

## 7. Как переносить

1. Скопировать в `<head>` новый проект: подключение Tailwind + `tailwind.config`
   + блок `:root`/`.dark` + базовые классы (§1). Это самодостаточно.
2. Взять каркас из §2 и оставить в `SCREENS` свои разделы.
3. Собирать экраны из карточек §3.1 и кнопок §3.2. Пока хватает их — новых
   компонентов не заводить.
4. Таблицы — только если действительно нужна широкая сетка. Тогда переносить §4
   целиком, включая `separate`, лестницу `z-index` и непрозрачные фоны: половина
   правил там неочевидна и всплывает багом через неделю.
5. Формат чисел и цвета динамики (§5.1–5.2) взять сразу — они дешёвые и сильно
   влияют на впечатление.

Что **не** переносится и должно быть переделано под новый проект: тексты, состав
метрик, ширины колонок в JS-константах (`W_METRIC`, `COLS_LEFT` и прочие) и
`max-height: calc(100vh - …)` у прокручиваемых областей — эти числа посчитаны под
конкретную высоту шапок этого приложения.
