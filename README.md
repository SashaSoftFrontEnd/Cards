
# **Интерактивная карточка**

---

## **Описание**

Интерактивная карточка — это UI-элемент, который:

* открывается по клику
* показывает скрытый контент
* может выезжать из любой стороны
* автоматически закрывает другие карточки
* поддерживает клавиатуру (`Enter`)
* работает без hover

Подходит для:

* галерей
* продуктов
* превью контента (Netflix-style)
* сервисных блоков
* FAQ-виджетов

---

## **Особенности реализации**

* чистый HTML + CSS + JS
* анимации через `transform` (GPU-ускорение)
* без перерисовок layout
* по методологии
* состояние управляется классом `.card--active`
* направления реализуются модификаторами блока

---

## **Структура **

```
.card                         ← блок
  .card__image                ← элемент (фон/картинка)
  .card__front                ← видимая сторона
  .card__content              ← скрытый контент

card--from-top                ← модификатор направления
card--from-bottom
card--from-left
card--from-right
card--active                  ← модификатор состояния
```

---

## **HTML рендер**

```html
<div class="card card--from-bottom">
  <img class="card__image" src="img/1.jpg" alt="" />

  <div class="card__front">Снизу ↑</div>

  <div class="card__content">
    <p class="card__heading">From Bottom</p>
    <p class="card__text">Описание карточки...</p>
  </div>
</div>
```

---

## **CSS логика анимации**

```css
.card--from-bottom .card__content {
  transform: translateY(100%); /* скрыто */
}

.card--from-bottom.card--active .card__content {
  transform: translateY(0); /* выезд */
}

.card--from-bottom.card--active .card__front {
  opacity: 0;
  transform: translateY(-20%);
}
```

Все направления работают так же, только меняются оси:

```
сверху   → translateY(-100%)
снизу    → translateY(100%)
слева    → translateX(-100%)
справа   → translateX(100%)
```

---

## **JS логика (автозакрытие)**

```js
const cards = document.querySelectorAll('.card');

cards.forEach(card => {
  card.addEventListener('click', () => {
    const isActive = card.classList.contains('card--active');

    if (isActive) {
      card.classList.remove('card--active');
      return;
    }

    cards.forEach(other => other.classList.remove('card--active'));
    card.classList.add('card--active');
  });
});
```

---

# **Версия для React**

---

## **Файловая структура**

```
src/
  components/
    Card/
      Card.jsx
      Card.module.css
  App.jsx
```

---

## **Card.module.css**

```css
.card {
  position: relative;
  width: 100%;
  aspect-ratio: 1;
  border-radius: 14px;
  overflow: hidden;
  cursor: pointer;
  background: #222;
  transition: box-shadow 0.3s ease;
}

.cardActive {
  box-shadow: 0 20px 36px rgba(0, 0, 0, 0.55);
}

.image {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: filter 0.4s ease;
}

.cardActive .image {
  filter: brightness(0.45);
}

.front {
  position: absolute;
  inset: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  color: #fff;
  font-size: 28px;
  font-weight: 700;
  transition: opacity 0.45s ease, transform 0.45s ease;
}

.content {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  background: rgba(0, 0, 0, 0.65);
  color: #fff;
  gap: 12px;
  transition: transform 0.6s cubic-bezier(0.23, 1, 0.32, 1);
}

/* направление */
.fromBottom .content {
  transform: translateY(100%);
}

.cardActive.fromBottom .content {
  transform: translateY(0);
}

.cardActive.fromBottom .front {
  opacity: 0;
  transform: translateY(-20%);
}
```

---

## **Card.jsx**

```jsx
import { useState } from "react";
import styles from "./Card.module.css";

export default function Card({ image, label, heading, text, direction, isOpen, onToggle }) {
  return (
    <div
      className={`${styles.card} ${isOpen ? styles.cardActive : ""} ${styles[direction]}`}
      onClick={onToggle}
    >
      <img className={styles.image} src={image} alt="" />

      <div className={styles.front}>{label}</div>

      <div className={styles.content}>
        <p>{heading}</p>
        <p>{text}</p>
      </div>
    </div>
  );
}
```

---

## **App.jsx — автозакрытие одной активной**

```jsx
import { useState } from "react";
import Card from "./components/Card/Card";

export default function App() {
  const [activeIndex, setActiveIndex] = useState(null);

  const cards = [
    { label: "Снизу ↑", direction: "fromBottom", image: "/img/1.jpg" },
    { label: "Сверху ↓", direction: "fromTop", image: "/img/2.jpg" },
    { label: "Слева →", direction: "fromLeft", image: "/img/3.jpg" },
    { label: "Справа ←", direction: "fromRight", image: "/img/4.jpg" }
  ];

  return (
    <div style={{ display: "grid", gap: "30px", gridTemplateColumns: "repeat(auto-fit, minmax(240px, 1fr))" }}>
      {cards.map((card, index) => (
        <Card
          key={index}
          {...card}
          heading="Заголовок"
          text="Описание карточки"
          isOpen={activeIndex === index}
          onToggle={() =>
            setActiveIndex(activeIndex === index ? null : index)
          }
        />
      ))}
    </div>
  );
}
```




