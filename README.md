# Game Platform Telegram Bot

Telegram-бот для игровой платформы с квестами, рейтингами, наградами, рефералами и inline-навигацией.

## Что уже реализовано

- Полная регистрация игрока: никнейм, возраст, страна, платформы, игровые интересы
- Личный кабинет: уровень, XP, монеты, квесты, рейтинг, серия входов, достижения
- Система квестов: категории, карточка квеста, взятие в работу, отправка отчёта скриншотом, видео, документом или ссылкой
- Модерация: очередь заявок, одобрение, отклонение, запрос уточнений
- Автоматические награды: начисление XP и монет после одобрения
- Рейтинги: общий и недельный
- Реферальная система: персональная `start`-ссылка и бонусы за приглашения
- Магазин наград: каталог и заявка администратору на выдачу
- Админ-панель: создание квестов, базовое редактирование, ручные бонусы, рассылки, статистика
- Seed-данные: стартовые квесты, награды и новости

## Технологии

- Java 21
- Spring Boot 3
- TelegramBots long polling
- Spring Data JPA
- H2 file database
- Docker multi-stage build

## Локальный запуск

Рекомендуемый JDK для локальной сборки: 21 или 23.

```bash
export JAVA_HOME=$(/usr/libexec/java_home -v 23)
export PATH="$JAVA_HOME/bin:$PATH"

mvn -DskipTests package
TELEGRAM_BOT_TOKEN=123456:token \
TELEGRAM_BOT_USERNAME=your_game_bot \
INITIAL_ADMIN_ID=123456789 \
APP_MODERATOR_IDS=123456789 \
APP_SUPPORT_USERNAME=support_manager \
DB_PATH=./data/game-platform-bot \
java -jar target/game-platform-bot-1.0.0.jar
```

## Docker

Сборка образа:

```bash
docker build -t game-platform-bot .
```

Запуск контейнера в привычном стиле без проброса порта:

```bash
docker rm -f game-platform-bot 2>/dev/null || true

docker run -d \
  --name game-platform-bot \
  --restart unless-stopped \
  --network host \
  --add-host api.telegram.org:149.154.167.220 \
  -e TELEGRAM_BOT_TOKEN='123456:token' \
  -e TELEGRAM_BOT_USERNAME='your_game_bot' \
  -e INITIAL_ADMIN_ID='123456789' \
  -e APP_MODERATOR_IDS='123456789,987654321' \
  -e APP_SUPPORT_USERNAME='support_manager' \
  -e APP_CLUB_NAME="Game Quest Club" \
  -e DB_PATH='/data/game-platform-bot' \
  -e JAVA_TOOL_OPTIONS='-Djava.net.preferIPv4Stack=true -Djava.net.preferIPv6Addresses=false' \
  -v "$(pwd)/data:/data" \
  game-platform-bot
```

Если захотите, можно по-прежнему использовать `APP_ADMIN_IDS` вместо `INITIAL_ADMIN_ID`. Для одного первого администратора удобнее `INITIAL_ADMIN_ID`, для нескольких сразу удобнее `APP_ADMIN_IDS`.

## Переменные окружения

- `TELEGRAM_BOT_TOKEN` - основной токен Telegram-бота
- `TELEGRAM_BOT_USERNAME` - username бота без `@`
- `INITIAL_ADMIN_ID` - первый администратор, удобен для быстрого старта
- `APP_ADMIN_IDS` - Telegram ID администраторов через запятую
- `APP_MODERATOR_IDS` - Telegram ID модераторов через запятую
- `APP_SUPPORT_USERNAME` - username аккаунта поддержки
- `APP_CLUB_NAME` - отображаемое название платформы
- `DB_PATH` - базовый путь для H2-файла
- `SPRING_DATASOURCE_URL` - URL базы данных

## Важные заметки

- Все основные экраны используют inline-кнопки.
- Для кратких кнопок применяется компактная раскладка по две в строке; длинные идут по одной.
- База данных хранится в H2-файле внутри volume `/data` и переживает перезапуск контейнера.
- Недельный рейтинг обнуляется по расписанию каждый понедельник.
- Боту не нужен HTTP-порт, поэтому `-p 8080:8080` убран. Это заодно убирает вашу текущую ошибку `address already in use`.
