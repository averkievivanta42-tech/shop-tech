<script setup>
defineProps({
  page: {
    type: String,
    required: true
  }
})

const emit = defineEmits(['back'])
</script>

<template>
  <section class="info-page">
    <nav class="breadcrumbs">
      <button type="button" class="crumb-link" @click="emit('back')">Каталог</button>
      <span class="crumb-sep">/</span>
      <span class="crumb-current">
        {{
          page === 'delivery'
            ? 'Доставка і оплата'
            : page === 'warranty'
            ? 'Гарантія та повернення'
            : page === 'privacy'
            ? 'Політика конфіденційності'
            : page === 'terms'
            ? 'Публічна оферта'
            : 'Про магазин та контакти'
        }}
      </span>
    </nav>

    <div class="back-row">
      <button type="button" class="back-btn" @click="emit('back')">
        ← Повернутися до каталогу
      </button>
    </div>

    <article v-if="page === 'delivery'" class="info-card">
      <h1 class="info-title">🚚 Доставка і оплата</h1>
      <p class="info-lead">
        Інтернет-магазин <strong>МегаТренд</strong> здійснює щоденне відправлення замовлень по всій території України. Усі товари перед відправкою проходять обов'язкову перевірку комплектності.
      </p>

      <div class="info-grid">
        <div class="info-box">
          <h3>📦 Доставка у відділення «Нова Пошта»</h3>
          <ul>
            <li>Термін доставки: 1-2 робочі дні у більшість міст України.</li>
            <li>При оформленні замовлення до 16:00 відправка здійснюється в той самий день.</li>
            <li>При замовленні на суму від 3 000 ₴ доставка у відділення безкоштовна.</li>
            <li>Після відправки ви отримаєте SMS або повідомлення з номером експрес-накладної (ТТН).</li>
          </ul>
        </div>

        <div class="info-box">
          <h3>📮 Доставка у поштомат «Нова Пошта»</h3>
          <ul>
            <li>Цілодобове отримання без черг у зручний для вас час.</li>
            <li>Доступно для малогабаритних товарів (смартфони, навушники, смарт-годинники, планшети, периферія, аксесуари).</li>
            <li>Термін зберігання посилки у поштоматі - 3 календарні дні.</li>
          </ul>
        </div>

        <div class="info-box">
          <h3>🚗 Адресна доставка кур'єром</h3>
          <ul>
            <li>Доставка кур'єром служби «Нова Пошта» безпосередньо до дверей вашого дому або офісу.</li>
            <li>Кур'єр попередньо телефонує за 30-60 хвилин до прибуття для узгодження часу.</li>
          </ul>
        </div>

        <div class="info-box">
          <h3>🏪 Самовивіз із пункту видачі</h3>
          <ul>
            <li>Самовивіз доступний за попереднім узгодженням з менеджером за номером <strong>0670001234</strong>.</li>
            <li>Графік роботи пункту видачі: Пн-Нд з 09:00 до 20:00.</li>
          </ul>
        </div>
      </div>

      <h2 class="info-subtitle">💳 Способи оплати</h2>
      <div class="info-grid">
        <div class="info-box">
          <h3>💵 Оплата під час отримання (Накладений платіж)</h3>
          <p>
            Оплата готівкою або банківською карткою у відділенні «Нова Пошта» після повного огляду та перевірки товару. Ви платите лише після того, як переконаєтесь у цілісності упаковки та працездатності пристрою.
          </p>
        </div>

        <div class="info-box">
          <h3>💳 Оплата карткою онлайн (Visa / Mastercard)</h3>
          <p>
            Миттєва оплата банківською карткою під час оформлення замовлення, а також підтримка Apple Pay та Google Pay без додаткових комісій.
          </p>
        </div>

        <div class="info-box">
          <h3>🏦 Безготівковий розрахунок за реквізитами ФОП</h3>
          <p>
            Оплата на офіційний розрахунковий рахунок IBAN (<strong>ФОП Аверкієв І.О.</strong>) через будь-який мобільний банкінг (Приват24, Monobank тощо) або у касі банку. Рахунок-фактура та видаткова накладна надаються разом із товаром.
          </p>
        </div>
      </div>
    </article>

    <article v-else-if="page === 'warranty'" class="info-card">
      <h1 class="info-title">🛡️ Гарантія, обмін та повернення</h1>
      <p class="info-lead">
        Уся техніка в магазині <strong>МегаТренд</strong> супроводжується офіційною гарантією від 12 до 36 місяців залежно від категорії товару та політики виробника.
      </p>

      <div class="text-section">
        <h2>1. Умови гарантійного обслуговування</h2>
        <p>
          Гарантійний термін зазначається на сторінці кожного товару та в гарантійному талоні, який додається до замовлення. Гарантія покриває будь-які заводські дефекти, виявлені протягом усього гарантійного періоду за умови дотримання правил експлуатації.
        </p>
        <p>
          Для звернення за гарантійним обслуговуванням достатньо зв'язатися з нашою службою підтримки за телефоном <strong>0670001234</strong> або написати на електронну пошту <strong>info@megatrend.com.ua</strong>.
        </p>
      </div>

      <div class="text-section">
        <h2>2. Обмін та повернення протягом 14 днів</h2>
        <p>
          Відповідно до Закону України «Про захист прав споживачів», покупець має право повернути або обміняти товар належної якості протягом <strong>14 календарних днів</strong> з моменту отримання, якщо дотримані наступні умови:
        </p>
        <ul>
          <li>Товар не використовувався, не має слідів експлуатації, подряпин, сколів чи потертостей;</li>
          <li>Повністю збережено товарний вигляд, заводські захисні плівки, пломби та ярлики;</li>
          <li>Збережено повну комплектацію, оригінальну цілісну упаковку та розрахунковий документ (чек або видаткову накладну);</li>
          <li>У пристроях Apple / Samsung не була активована облікова запис користувача (iCloud / Knox).</li>
        </ul>
        <p>
          Повернення коштів здійснюється протягом 1-3 робочих днів з моменту отримання та перевірки товару сервісним відділом на банківську картку або за реквізитами покупця.
        </p>
      </div>
    </article>

    <article v-else-if="page === 'privacy'" class="info-card">
      <h1 class="info-title">🔒 Політика конфіденційності та захисту персональних даних</h1>
      <p class="info-lead">
        Ця Політика конфіденційності регулює порядок збору, обробки та зберігання персональних даних користувачів інтернет-магазину <strong>МегаТренд</strong> (володілець бази даних - <strong>ФОП Аверкієв І.О.</strong>) відповідно до Закону України «Про захист персональних даних».
      </p>

      <div class="text-section">
        <h2>1. Які дані ми збираємо</h2>
        <p>
          Під час оформлення замовлення на сайті ми запитуємо виключно ту інформацію, яка необхідна для виконання доставки та зворотного зв'язку з покупцем:
        </p>
        <ul>
          <li>Прізвище та ім'я отримувача замовлення;</li>
          <li>Контактний номер телефону;</li>
          <li>Адреса електронної пошти (для надсилання підтвердження замовлення);</li>
          <li>Місто та номер відділення/поштомату або адреса доставки;</li>
          <li>Дані кошика покупок, які зберігаються локально у вашому браузері (<code>localStorage</code>).</li>
        </ul>
      </div>

      <div class="text-section">
        <h2>2. Мета обробки персональних даних</h2>
        <p>
          Персональні дані використовуються виключно для:
        </p>
        <ul>
          <li>Оформлення, комплектації та відправлення замовлень службами доставки;</li>
          <li>Інформування клієнта про статус виконання замовлення та надання номера ТТН;</li>
          <li>Виконання гарантійних зобов'язань, оформлення бухгалтерських і розрахункових документів;</li>
          <li>Надання консультацій за зверненням користувача.</li>
        </ul>
      </div>

      <div class="text-section">
        <h2>3. Захист інформації та передача третім особам</h2>
        <p>
          Ми не продаємо і не передаємо персональні дані стороннім організаціям, за винятком логістичних операторів (ТОВ «Нова Пошта») та платіжних систем виключно в обсязі, необхідному для доставки та оплати вашого замовлення.
        </p>
        <p>
          Користувач має право в будь-який момент запросити зміну або повне видалення своїх персональних даних, звернувшись за номером <strong>0670001234</strong> або на email <strong>info@megatrend.com.ua</strong>.
        </p>
      </div>
    </article>

    <article v-else-if="page === 'terms'" class="info-card">
      <h1 class="info-title">📄 Договір публічної оферти</h1>
      <p class="info-lead">
        Цей документ є офіційною публічною пропозицією (офертою) <strong>ФОП Аверкієв І.О.</strong> (надалі - Продавець) укласти Договір купівлі-продажу товарів дистанційним способом через інтернет-магазин <strong>МегаТренд</strong> відповідно до ст. 633, 641 Цивільного кодексу України.
      </p>

      <div class="text-section">
        <h2>1. Загальні положення</h2>
        <p>
          Оформлення замовлення на сайті інтернет-магазину означає повне і беззастережне прийняття (акцепт) Покупцем умов цього Договору публічної оферти. Ціни на товари в каталозі вказані у національній валюті України - гривні (₴).
        </p>
      </div>

      <div class="text-section">
        <h2>2. Порядок оформлення замовлення</h2>
        <p>
          Покупець самостійно обирає товар у каталозі, додає його до кошика та заповнює форму оформлення замовлення із зазначенням достовірних контактних даних і реквізитів доставки. Після реєстрації замовлення Продавець підтверджує наявність товару та узгоджує строк відправлення.
        </p>
      </div>

      <div class="text-section">
        <h2>3. Права та обов'язки сторін</h2>
        <ul>
          <li>Продавець зобов'язується передати Покупцеві товар належної якості у повній комплектації та з відповідним гарантійним строком.</li>
          <li>Покупець зобов'язується перевірити цілісність та комплектність товару під час його отримання у відділенні служби доставки та оплатити замовлення відповідно до обраного способу оплати.</li>
        </ul>
      </div>

      <div class="text-section">
        <h2>4. Реквізити Продавця</h2>
        <p>
          <strong>Фізична особа-підприємець Аверкієв І.О.</strong><br />
          Контактний телефон: <strong>0670001234</strong><br />
          Електронна пошта: <strong>info@megatrend.com.ua</strong>
        </p>
      </div>
    </article>

    <article v-else class="info-card">
      <h1 class="info-title">🏢 Про магазин МегаТренд та контакти</h1>
      <p class="info-lead">
        <strong>МегаТренд</strong> - український інтернет-магазин сучасної електроніки, комп'ютерної техніки, смартфонів, планшетів, аудіосистем та периферії.
      </p>

      <div class="info-grid">
        <div class="info-box">
          <h3>📞 Контактна інформація</h3>
          <ul>
            <li><strong>Оператор магазину:</strong> ФОП Аверкієв І.О.</li>
            <li><strong>Телефон:</strong> <a href="tel:0670001234" class="phone-link">0670001234</a></li>
            <li><strong>Email:</strong> info@megatrend.com.ua</li>
            <li><strong>Графік роботи кол-центру:</strong> Пн-Нд з 09:00 до 20:00 (без вихідних)</li>
          </ul>
        </div>

        <div class="info-box">
          <h3>⚡ Наші переваги</h3>
          <ul>
            <li>Тільки оригінальна продукція провідних світових брендів (Apple, Samsung, ASUS, Lenovo, Sony, Garmin, Marshall, LG, Dell, Logitech).</li>
            <li>Актуальні ціни та реальна наявність на складі.</li>
            <li>Швидка відправка в день замовлення та офіційна гарантія.</li>
          </ul>
        </div>
      </div>
    </article>
  </section>
</template>

<style scoped>
.info-page {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.breadcrumbs {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  color: var(--text-muted);
}

.crumb-link {
  color: var(--text-secondary);
  font-weight: 500;
}

.crumb-link:hover {
  color: var(--blue);
  text-decoration: underline;
}

.crumb-current {
  color: var(--navy);
  font-weight: 600;
}

.back-row {
  display: flex;
}

.back-btn {
  height: 36px;
  padding: 0 14px;
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  font-size: 13px;
  font-weight: 600;
  color: var(--navy);
}

.back-btn:hover {
  border-color: var(--blue);
  color: var(--blue);
}

.info-card {
  background-color: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-lg);
  padding: 32px;
  box-shadow: var(--shadow-sm);
}

.info-title {
  font-size: 26px;
  font-weight: 800;
  color: var(--navy);
  margin-bottom: 12px;
}

.info-subtitle {
  font-size: 20px;
  font-weight: 700;
  color: var(--navy);
  margin: 28px 0 14px;
}

.info-lead {
  font-size: 15px;
  color: var(--text-secondary);
  line-height: 1.6;
  margin-bottom: 24px;
  max-width: 820px;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 16px;
}

.info-box {
  background-color: var(--bg);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 20px;
}

.info-box h3 {
  font-size: 16px;
  font-weight: 700;
  color: var(--navy);
  margin-bottom: 10px;
}

.info-box p {
  font-size: 14px;
  color: var(--text-secondary);
  line-height: 1.55;
}

.info-box ul {
  padding-left: 18px;
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 14px;
  color: var(--text-secondary);
  line-height: 1.5;
}

.text-section {
  margin-top: 22px;
  padding-top: 18px;
  border-top: 1px solid var(--border);
}

.text-section h2 {
  font-size: 18px;
  font-weight: 700;
  color: var(--navy);
  margin-bottom: 10px;
}

.text-section p {
  font-size: 14px;
  color: var(--text-secondary);
  line-height: 1.65;
  margin-bottom: 10px;
}

.text-section ul {
  padding-left: 20px;
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 14px;
  color: var(--text-secondary);
  margin-bottom: 10px;
}

.phone-link {
  color: var(--blue);
  font-weight: 700;
}

@media (max-width: 640px) {
  .info-card {
    padding: 20px 16px;
  }

  .info-title {
    font-size: 21px;
  }
}
</style>
