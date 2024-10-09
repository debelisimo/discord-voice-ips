# Что это и зачем?

Этот репозиторий содержит список IP-адресов, используемых голосовыми серверами Discord.
Эти списки могут быть полезны для пользователей, которые хотят настроить 'корректную' маршрутизацию для обеспечения 'стабильной' работы голосовых каналов Discord.
Бонусом добавлен список всех основных доменов и голосовых каналов.

## Структура Репозитория

- `discord-domains-list` - список доменных имен, принадлежащих Discord.
- `discord-voice-domains-list` - список из доменов голосовых каналов Discord для регионов: Россия, Нидерланды, Швеция, Италия, Германия, Испания, Польша, Румыния
- `discord-voice-ip-list` - список из IP голосовых каналов Discord для вышеназванных регионов
- `discord-voice-ipset-list` - список в формате IPset с командой `add unblock 'IP-адрес'`, где `'IP-адрес'` — это адреса из списка `discord-voice-ip-list`. Для использования этого файла с командой `restore`, необходимо предварительно создать соответствующий ipset список `unblock`.
- `ips-by-region` - фолдер со списками IP голосовых каналов разбитым по регионам
- `custom-solutions` - фолдер с решениями от **заинтересованных** и **неравнодушных**
- `parser-voice-ip.sh` - про него подробнее будет ниже

## Использование IPset

Несколько шагов:

1. **Создайте новый список `unblock`**:
```bash
~# ipset create unblock hash:net
```
2. **Склонируйте этот репозиторий**:
```bash
~# git clone https://github.com/GhostRooter0953/discord-voice-ips.git
```
3. **Перейдите в директорию с клонированным репозиторием**:
```bash
~# cd discord-voice-ips
```
4. **Добавьте адреса из файла `discord-voice-ipset-list` в ваш ipset**:
```bash
~# ipset restore < discord-voice-ipset-list
```
5. **Добавьте соответствующее правило в ваш фаерволл, чтобы настроить маршрутизацию**
```bash
ДАННЫЕ_УДАЛЕНЫ
```
_p.s. ещё можно использовать менее элегантное решение тупо запихнув списки в `белый список` вашего клиента наделённого `корректной` маршрутизацией_

# Скрипт parser-voice-ip.sh

Предназначен для парсинга IP-адресов голосовых серверов Discord из файла со списком доменов и их дальнейшего добавления в IPset.

Скрипт может:
- Автоматически почистить предыдущие списки (_как IPset так и файлы_)
- Создать список `unblock` если таковой отсутствует
- Предоставить пользователю возможность подтверждать действия (_касающиеся IPset_) вручную
- Просто порезолвить IP адреса из `discord-voice-domains-list` и записать их в `discord-voice-ip-list`

## Подготовка и использование

1. Убедитесь, что у вас установлены необходимые утилиты:
   - dig (_часть пакета dnsutils_)
   - ipset

2. Склонируйте репозиторий:
```bash
   git clone https://github.com/GhostRooter0953/discord-voice-ips.git
```

3. Перейдите в директорию репо:
```bash
/opt/tmp # cd discord-voice-ips
```

4. Запустите скрипт:
```bash
/opt/tmp/discord-voice-ips # ./parser-voice-ip.sh
```

### Процесс работы скрипта

1. Очистка старых списков: Скрипт очищает файлы `discord-voice-ip-list` и `discord-voice-ipset-list`.

2. Сброс кэша DNS: Производится сброс кэша DNS через отправку сигнала к `dnsmasq`.

3. Парсинг доменов: Скрипт читает домены из файла `discord-voice-domains-list`, резолвит их в IP-адреса и записывает в `discord-voice-ip-list` и `discord-voice-ipset-list`.

4. Создание списка unblock:
    - Если список unblock не существует, будет запрошено подтверждение на его создании.
    - В автоматическом режиме список создается без подтверждения.

5. Очистка списка unblock:
    - Если список существует, будет запрошено подтверждение на его очистку.
    - В автоматическом режиме список очищается без подтверждения.

6. Загрузка IP адресов в IPset:
    - После завершения парсинга, пользователю предлагается загрузить адреса из файла `discord-voice-ipset-list` в IPset.

7. Вывод результатов: После выполнения всех действий выводится количество загруженных IP адресов.

### Пример запуска в ручном режиме (_без опций_)

```bash
/opt/tmp # ./parser-voice-ip.sh
Очистка IP листов
Начинаем парсить IP голосовых серверов Discord
Парсим... Прогресс: 100%
Парсинг завершён
Чистим список 'unblock'? (Y/N): Y
Список 'unblock' очищен
Загружаем адреса в IPset? (Y/N): Y
Загружено 1430 IP адреса(ов) в список 'unblock'
```

### Опции

1. `auto`: Автоматический режим. В этом режиме все действия выполняются без запроса подтверждения у пользователя, а именно:
  - Создаётся IPset список unblock, если его нет
  - Очищается IPset список unblock
  - Загружаются IP в список unblock из `discord-voice-ipset-list`

Пример запуска в автоматическом режиме:
```bash
/opt/tmp # ./parser-voice-ip.sh auto
Очистка IP листов
Начинаем парсить IP голосовых серверов Discord
Парсим... Прогресс: 100%
Парсинг завершён
Список 'unblock' очищен
Загружено 1429 IP адреса(ов) в список 'unblock'
```

2. `noipset`: Автоматический режим без IPset. Нужен для простого резолва IP с последующей записью вывода в файлы `discord-voice-ip-list` и `discord-voice-ipset-list`.

Пример запуска в режиме `noipset`:
```bash
/opt/tmp # ./parser-voice-ip.sh noipset
Очистка IP листов
Начинаем парсить IP голосовых серверов Discord
Парсим... Прогресс: 100%
Парсинг завершён
Пропускаем танцы с IPset... Список с IP можно найти в 'discord-voice-ip-list'
```

## Примечания

- Убедитесь, что вы запускаете скрипт с правами, достаточными для выполнения команд ipset и управления сетевыми настройками.
- Убедитесь, что у вас установлены необходимые утилиты:
   - dig (_часть пакета dnsutils_)
   - ipset
- Для получения информации об использовании утилиты ipset можно использовать следующую команду:
```bash
man ipset
```
# TO DO
- Генератор списка сабдоменов
- Резолв по сгенерированному списку с таймаутом в 2 секунды
---
