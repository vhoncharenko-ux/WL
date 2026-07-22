# White labeling — як підключити зміну кольору

## 1. Звідки береться колір превʼю

Обидві ілюстрації Application shell — це **інлайн `<svg>`** прямо в HTML
(`index.html`, класи `.sp` всередині `.tb-shell`), а не `<img src="...">`.
Це важливо: тільки інлайн SVG дозволяє перефарбовувати заливки через CSS
змінні — зовнішній файл через `<img>` так не можна.

Усі елементи ілюстрації, які раніше мали колір бренду чи акценту, тепер
використовують дві CSS-змінні замість хардкоду:

```css
:root {
  --preview-primary: #00695c; /* дефолтний Primary */
  --preview-accent:  #ff5722; /* дефолтний Accent */
}
```

Ці змінні підставлені **і в `fill`, і в `stroke`** всередині SVG (обидва
типи атрибутів зустрічались у вихідних ілюстраціях — якщо замінити лише
`fill`, обведення чекбокса/радіо лишиться старого кольору).

## 2. Як відбувається перефарбування

У кожного дропдауна палітри (`<details class="tb-dd">`) є атрибут
`data-var`, який каже, яку CSS-змінну він контролює:

```html
<details class="tb-dd" data-default-color="#00695c" data-var="--preview-primary">  <!-- Primary -->
<details class="tb-dd" data-default-color="#ff5722" data-var="--preview-accent">   <!-- Accent -->
```

При кліку на колір у списку (`<script>` унизу файлу):

```js
if (cssVar) root.style.setProperty(cssVar, col); // root = document.documentElement
```

Тобто зміна одного `--preview-*` на `:root` миттєво перефарбовує всі
елементи ілюстрації, бо вони посилаються на цю змінну через `var(...)`.

## 3. Як підключити реальне бек-енд значення

Зараз дефолти захардкоджені в `:root` і в `data-default-color`. Щоб
підʼєднати справжні збережені кольори White labeling:

1. При завантаженні сторінки отримайте `primaryPaletteColor` /
   `accentPaletteColor` з налаштувань (API / стейт додатку).
2. Встановіть їх одразу після завантаження DOM:
   ```js
   document.documentElement.style.setProperty('--preview-primary', primaryPaletteColor);
   document.documentElement.style.setProperty('--preview-accent', accentPaletteColor);
   ```
3. Синхронізуйте UI дропдаунів (закритий свотч + підсвічений пункт у
   списку) з тим самим значенням — зараз це робить внутрішня функція
   всередині `forEach(".tb-dd", ...)`; при переносі в компонент
   (React/Angular/Vue) відповідний стан варто підняти в один спільний
   store, а не тримати в DOM-класах, як зараз.
4. На «Reset to default» — поверніть обидві змінні до дефолтних значень
   палітри Material (`#00695c` / `#ff5722`) і скиньте `disabled` кнопки
   Save назад у `true`.

## 4. Контраст тексту "Aa" у списку кольорів

За специфікацією Figma лише 4 з 20 кольорів мають темний текст поверх
світлого фону: **Lime, Yellow, Amber, Orange** (`#212121`), решта — білий.
Це керується прапорцем `dark: true` в масиві `COLORS` у `<script>`:

```js
{ name: "Lime", color: "#c9de00", dark: true },
```

і класом-модифікатором `.tb-swatch--dark` (колір тексту `#212121`),
який навішується і на пункт у списку, і на закритий тригер при виборі.
Якщо буде додаватись новий колір — перевіряйте контраст і виставляйте
`dark: true`, якщо фон світліше приблизно WCAG AA поріг для чорного тексту.

## 5. Файли ілюстрацій (як передані дизайном, без CSS-змінних)

Оригінальні SVG з фіксованими кольорами `#00695C` / `#FF5722` — для
довідки чи якщо треба інша інтеграція (наприклад, генерація на сервері):

- [`handoff-assets/shell-preview-light.svg`](handoff-assets/shell-preview-light.svg) — картка "Light" (нейтральний сайдбар)
- [`handoff-assets/shell-preview-primary.svg`](handoff-assets/shell-preview-primary.svg) — картка "Light" (пофарбований сайдбар/topbar)

Версія, що реально працює на сторінці (з `var(--preview-primary)` /
`var(--preview-accent)` замість хардкоду), лежить інлайн у самому
`index.html` — шукайте `class="sp"`.
