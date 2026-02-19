# Архітектура адмінки на Filament

## Загальний опис
Адмінка буде побудована на Filament 3 з підтримкою багатомовності та різних ролей користувачів.

## Ролі користувачів

### 1. Admin (Адміністратор)
- Повний доступ до всіх функцій адмінки
- Управління користувачами та ролями
- Налаштування системи
- Доступ до всіх даних

### 2. User (Звичайний користувач)
- Доступ тільки до фронтенду
- Можливість робити замовлення
- Особистий кабінет

### 3. Seller (Продавець)
- Доступ до панелі продавця
- Управління своїми товарами
- Перегляд замовлень
- Статистика продажів

## Структура адмінки

### 1. Dashboard (Головна панель)
**Файл**: `app/Filament/Pages/Dashboard.php`

**Віджети**:
- Статистика замовлень (сьогодні, тиждень, місяць)
- Графік продажів
- Топ товари
- Останні замовлення
- Статистика продавців
- Нагадування (неопрацьовані замовлення, товари на модерації)

### 2. Управління категоріями
**Файл**: `app/Filament/Resources/CategoryResource.php`

**Поля для редагування**:
- **Основна інформація**:
  - `name` (string, required) - Назва категорії
  - `slug` (string, required) - URL slug
  - `description` (textarea) - Опис категорії
  - `meta_title` (string) - Meta title для SEO
  - `meta_description` (textarea) - Meta description
  - `meta_keywords` (string) - Meta keywords

- **Структура**:
  - `parent_id` (select) - Батьківська категорія
  - `sort_order` (number) - Порядок сортування
  - `status` (boolean) - Активна/неактивна

- **Зображення**:
  - `image` (file upload) - Основне зображення
  - `icon` (file upload) - Іконка категорії

- **Багатомовність**:
  - `translations` (relation) - Переклади для інших мов
  - Підтримка української та російської мов

**Функції**:
- Дерево категорій з можливістю переміщення
- Масове редагування
- Експорт/імпорт категорій
- SEO налаштування

### 3. Управління товарами
**Файл**: `app/Filament/Resources/ProductResource.php`

**Поля для редагування**:
- **Основна інформація**:
  - `name` (string, required) - Назва товару
  - `slug` (string, required) - URL slug
  - `sku` (string) - Артикул
  - `model` (string) - Модель
  - `description` (rich text editor) - Опис товару
  - `short_description` (textarea) - Короткий опис

- **Ціна та наявність**:
  - `price` (decimal, required) - Ціна
  - `special_price` (decimal) - Спеціальна ціна
  - `special_price_from` (datetime) - Дата початку акції
  - `special_price_to` (datetime) - Дата закінчення акції
  - `quantity` (integer) - Кількість на складі
  - `stock_status_id` (select) - Статус наявності
  - `min_quantity` (integer) - Мінімальна кількість для замовлення

- **Категорії**:
  - `main_category_id` (select, required) - Основна категорія (для URL)
  - `categories` (multi-select) - Додаткові категорії для показу

- **Продавець**:
  - `seller_id` (select) - Продавець товару

- **Разом з ним купують**:
  - `items_id` (multi select) - визначення додаткових товарів що будуть виводитись в карточці товару.

- **Разом з ним купують 2**:
  - `items_id` (multi select) - додатковий модуль з аналогічним функціоналом.

- **SEO та статус**:
  - `meta_title` (string) - Meta title
  - `meta_description` (textarea) - Meta description
  - `meta_keywords` (string) - Meta keywords
  - `status` (boolean) - Активний/неактивний

- **Зображення**:
  - `images` (multiple file upload) - Галерея зображень
  - `main_image` (file upload) - Основне зображення

- **Атрибути**:
  - `attributes` (relation) - Додаткові атрибути товару

**Функції**:
- Масове редагування товарів
- Імпорт/експорт товарів
- Копіювання товарів
- Модерація товарів від продавців
- SEO оптимізація

### 4. Управління замовленнями
**Файл**: `app/Filament/Resources/OrderResource.php`

**Поля для перегляду**:
- **Основна інформація**:
  - `order_number` (string) - Номер замовлення
  - `customer` (relation) - Клієнт
  - `status` (select) - Статус замовлення
  - `total` (decimal) - Загальна сума
  - `created_at` (datetime) - Дата створення

- **Деталі замовлення**:
  - `items` (relation) - Товари в замовленні
  - `shipping_address` (json) - Адреса доставки
  - `billing_address` (json) - Адреса оплати
  - `payment_method` (string) - Спосіб оплати
  - `shipping_method` (string) - Спосіб доставки

- **Продавці**:
  - `sellers` (relation) - Продавці в замовленні
  - `seller_orders` (relation) - Розподіл по продавцях

**Функції**:
- Зміна статусу замовлення
- Друк накладних
- Відстеження доставки
- Повернення та обмін

### 5. Управління продавцями
**Файл**: `app/Filament/Resources/SellerResource.php`

**Поля для редагування**:
- **Особиста інформація**:
  - `name` (string, required) - Ім'я
  - `email` (email, required) - Email
  - `phone` (string) - Телефон
  - `company_name` (string) - Назва компанії

- **Профіль**:
  - `description` (textarea) - Опис продавця
  - `logo` (file upload) - Логотип
  - `address` (textarea) - Адреса

- **Налаштування**:
  - `status` (select) - Статус (активний, на модерації, заблокований)
  - `payment_methods` (multi select) - можливість обрати із доступних способи оплати продавця. 
  - `shipping_methods` (multi select) - можливість обрати із доступних способи доставки продавця. 

**Функції**:
- Модерація продавців
- Статистика продажів

### 6. Налаштування магазину
**Файл**: `app/Filament/Pages/Settings.php`

**Розділи налаштувань**:

#### Загальні налаштування:
- `store_name` (string) - Назва магазину
- `store_email` (email) - Email магазину
- `store_phone` (string) - Телефон
- `store_address` (textarea) - Адреса
- `store_logo` (file upload) - Логотип
- `favicon` (file upload) - Фавікон
- `timezone` (select) - Часовий пояс
- `currency` (select) - Валюта
- `language` (select) - Мова за замовчуванням

#### SEO налаштування:
- `meta_title` (string) - Meta title
- `meta_description` (textarea) - Meta description
- `meta_keywords` (string) - Meta keywords
- `google_analytics` (string) - Google Analytics ID
- `google_tag_manager` (string) - Google Tag Manager ID

#### Налаштування доставки:
- `shipping_methods` (json) - Способи доставки

#### Налаштування оплати:
- `payment_methods` (json) - Способи оплати

### 7. Управління мовами
**Файл**: `app/Filament/Resources/LanguageResource.php`

**Поля**:
- `name` (string) - Назва мови
- `code` (string) - Код мови (ua, ru)
- `locale` (string) - Локаль
- `status` (boolean) - Активна/неактивна
- `sort_order` (number) - Порядок сортування

**Функції**:
- Переклади контенту

### 8. Додаткові модулі

#### Блог/Новини
**Файл**: `app/Filament/Resources/ArticleResource.php`
- Управління статтями
- Категорії статей
- Коментарі
- SEO налаштування

#### Акції та знижки
**Файл**: `app/Filament/Resources/PromotionResource.php`
- Створення акцій
- Купони
- Знижки
- Таймери акцій

#### Відгуки та рейтинги
**Файл**: `app/Filament/Resources/ReviewResource.php`
- Модерація відгуків
- Рейтинги товарів
- Відгуки про продавців

### 9. Модулі

#### Модуль генерації фідів XML для Google Merchant
**Файл**: `app/Filament/Pages/GoogleMerchantFeed.php`
- Генерація XML фідів для Google Shopping
- Налаштування полів для експорту
- Автоматичне оновлення фідів
- Валідація даних перед експортом
- Планувальник генерації фідів

#### Модуль генерації Sitemap
**Файл**: `app/Filament/Pages/SitemapGenerator.php`
- Автоматична генерація sitemap.xml
- Налаштування пріоритетів сторінок
- Частота оновлення сторінок
- Виключення сторінок з sitemap
- Підтримка багатомовності в sitemap

#### Модуль Нової Пошти
**Файл**: `app/Filament/Resources/NovaPoshtaResource.php`
- Синхронізація регіонів України
- Синхронізація міст та населених пунктів
- Синхронізація відділень Нової Пошти
- Налаштування способів доставки
- Розрахунок вартості доставки
- Відстеження відправлень

#### Модуль Sticker PRO
**Файл**: `app/Filament/Pages/StickerPro.php`
- Управління стикерами для товарів
- Налаштування умов відображення стикерів
- Кастомні стикери для акцій
- Автоматичне додавання стикерів
- Візуальний редактор стикерів

#### Модуль "Варіанти вибору товарів"
**Файл**: `app/Filament/Resources/ProductVariantsResource.php`
- Створення варіантів товарів (розмір, колір, матеріал)
- Управління цінами для варіантів
- Наявність по варіантах
- Зображення для кожного варіанту
- SEO оптимізація варіантів

## Технічні особливості

### Багатомовність
- Використання пакету `spatie/laravel-translatable`
- Автоматичне збереження перекладів
- Fallback на мову за замовчуванням

### Права доступу
- Використання `spatie/laravel-permission`
- Гнучка система ролей та дозволів
- Middleware для захисту маршрутів

### Файли та зображення
- Використання `spatie/laravel-medialibrary`
- Автоматичне створення thumbnail'ів
- Оптимізація зображень

### Кешування
- Кешування категорій та товарів
- Redis для сесій та кешу
- Автоматичне оновлення кешу

## Структура файлів

```
app/
├── Filament/
│   ├── Pages/
│   │   ├── Dashboard.php
│   │   └── Settings.php
│   ├── Resources/
│   │   ├── CategoryResource.php
│   │   ├── ProductResource.php
│   │   ├── OrderResource.php
│   │   ├── SellerResource.php
│   │   ├── LanguageResource.php
│   │   ├── ArticleResource.php
│   │   ├── PromotionResource.php
│   │   └── ReviewResource.php
│   └── Widgets/
│       ├── OrderStatsWidget.php
│       ├── SalesChartWidget.php
│       └── TopProductsWidget.php
├── Models/
│   ├── Category.php
│   ├── Product.php
│   ├── Order.php
│   ├── Seller.php
│   └── Language.php
└── Services/
    ├── CategoryService.php
    ├── ProductService.php
    └── OrderService.php
```
