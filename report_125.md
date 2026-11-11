# report_125.md — Этап 125 (аналитический): гэп-анализ EN 1.6 → RU

**Статус:** аналитический, черновик v1 (сеть песочницы штормит; точная нумерация 6 изменённых
полей и детальный разбор KID/MCM — pending, см. §9). Правок ESP НЕТ. Выпуск не менялся: v1.5.25.
**Вход:** директива игрока «Нам нужно сделать перевод 1.6, т.к. автор выпустил обновление PoE сегодня».
**База:** RU v1.5.25 (zip истины, md5 508b3b087e059fdeb0ed7edbad43cd4e). EN 1.6 — текстовый слой
из репо (75 файлов, 40 ESP). EN 1.4 — снимок пака v6.00.

## 1. Метод
Дамп текстовых полей (канон: FULL/DESC/ITXT/NNAM + MGEF DNAM + GMST DATA; скрипт dump150.py)
трёх источников → выравнивание по ключу (file, rtype, formid, field, occ) + EDID-выравнивание
для Spears. Дампы: `_mirror/dumps_ru1525_en16.zip` (RU+EN16), EN14 — из пака.

## 2. Счётчики полей
| Источник | Текстовых полей | Записей (все ESP) |
|---|---|---|
| EN 1.4 | 12 317 | 22 576 |
| EN 1.6 | 13 223 | 21 519 |
| RU v1.5.25 | 12 338 | 22 597 |

EN14→EN16: **новых 906, удалённых 0, изменённых 6**.

## 3. Главный вывод
**Нового переводимого контента в 1.6 практически нет.** Новых полей EN16 вне Spears = **0**.
Весь прирост EN16 vs EN14 (906) — это 898 WEAP аддона Spears + 8 служебных (2 MGEF, 2 SPEL,
2 PERK, 1 CONT, 1 CELL). Из 906 новых 12 уже покрыты RU.

## 4. Spears — смена префикса formid 07→06 (НЕ новый контент)
RU sp использует formid `07xxxxxx`, EN16 sp — `06xxxxxx` (сдвиг индекса мастера, вероятно
из-за смены требований 1.5: SkyPatcher→SPID). EDID-ключи: RU 900, EN16 900, **общих 864**
(остальные — пустые FULL/DESC пары). Переводы копий 1:1 по EDID. Пример:
`PoE_SpearsAddonQAChest` RU 07000001 «Копья (QA)» → EN16 06000001 «Spears Addon QA».
**Действие:** механический ремап formid в RU Spears по EDID, тексты сохранить.

## 5. Шесть изменённых автором полей (EN14→EN16)
Переработка механик Quality/Masterwork (чейнджлог 1.6). Идентифицированы 2 ключевых DESC:

**Tailor** (`IWP_PERK_SMITH_Tailor`):
- EN16: «You can create rare or expensive types of clothing at the tanning rack. All clothing can be improved to Quality level at the armor workbench; Quality items can be sold for a higher price and enchantments placed upon them are 20% more powerful.»
- RU (устар.): «Позволяет создавать редкие и дорогие виды одежды на дубильном станке. Вся созданная вами одежда продается дороже и усиливает зачарования на 20%.»
- Дельта: добавлена механика «улучшение до Quality на верстаке брони» — в RU отсутствует.

**Dragonblood Masterwork** (`PoE_PERK_SMITH_DragonbloodMasterwork`):
- EN16: «Dragon souls can be sacrificed at the forge. For each dragon soul sacrificed, you can temper 1 weapon or armor to Masterwork level. Masterwork tempered items are 50% stronger and enchantments placed upon them are 25% more powerful.»
- RU (устар.): «Вы можете приносить драконьи души в жертву у кузнечного горна, чтобы создавать шедевры кузнечного мастерства. Такое снаряжение легче и крепче обычного, поддается закалке на 50% эффективнее, а любые наложенные на него чары обретают на 25% больше силы.»
- Дельта: новая формулировка «закалка до Masterwork, +50% силы, +25% чары»; RU несёт старую
  («легче и крепче, на 50% эффективнее»). Канон Этапа 102: stronger→«сильнее».

Полная нумерация всех 6 — при дампе EN14 (pending). Варианты Tailor1OLD/Tailor2OLD — legacy.

## 6. Структурная переработка 1.6 (текста не затронула)
- Убраны статические записи masterwork/quality предметов → динамическая генерация.
  main 17 017→16 360 записей; CC-патчи уменьшены (ECP 1760→1024, S&S 442→222,
  FearsomeFists 98→50, Fishing 93→49 и др.). Удалённых ТЕКСТОВЫХ полей = 0.
- Скрипты консолидированы: **12 PEX → 2 PEX** (PathsOfEmpyrean_DLLFunctions,
  PathsOfEmpyrean_ModObjectsScript). Наши санкц. MCM pex (isp__modconfig, iap__modmanager) — проверить.
- Новые runtime-слои EN: DISTR.ini (SPID), KID.ini (расширен Material Mastery),
  DMMF.json, ContainerDistributionFramework JSON, CraftingCategories JSON, ScrambledBugs.json.
- Аддон Spears: вариант OAR выделен отдельным ESP (фикс OAR-анимаций, чейнджлог 1.6).

## 7. RU-only записи (сохранить)
15 полей гендерных маскировок (MGEF/MESG 065D69D7–E0, Этап 115) — наши санкц. добавления,
в EN отсутствуют. Перенести в v1.6.0 без изменений.

## 8. Runtime-слой (языкозависимый) — адаптация
- **PoE_RU_KID.ini** (наш компенсатор, 14 EDID-правил, Этап 99) — ребейз под KID 1.6
  (расширено авто-тегирование Material Mastery).
- **DESC.ini** (MGEF по formid) — formid main стабильны; проверить покрытие.
- **MCM** ENGLISH.txt → RUSSIAN.txt (89 строк EN).
- **KID name-правила**: динамическая генерация имён Masterwork/Quality из «материал + база» —
  проверить протекание русских базовых имён (семья Masterwork ×812 из Этапа 118).
- Строки EN-слоёв: KID=1618, DESC=288, DISTR=118.

## 9. Открытые вопросы / pending
- **125-а:** динамическая генерация имён Masterwork/Quality в 1.6 — механизм (KID/скрипт/DLL),
  нужен ли отдельный RU-слой генерации? Исследовать KID.ini + DLLFunctions.
- **125-б:** MCM — сменилась ли система (2 PEX вместо 12)? Применим ли наш RU MCM?
- **125-в:** имя файла на Nexus «RU 1.5» → «RU 1.6»? (решение игрока; внутренняя версия v1.6.0)
- Pending: точные 6 изменённых полей (дамп EN14); детальный разбор KID/MCM; ребейз PoE_RU_KID.ini.

## 10. План исполнительского этапа (v1.6.0)
1. Ребейз RU-пакета на структуру EN16: Spears ремап formid по EDID; сохранить 15 санкц. записей;
   учесть удалённые статические записи и новые runtime-слои.
2. Обновить 6 изменённых полей (Tailor/Dragonblood DESC + остальные) по канонам.
3. Адаптировать runtime-слой (KID/DESC/MCM/DISTR).
4. Патчер (master91/103) → сборка v1.6.0 → батарея верификации → отчёт.

## 11. Артефакты в репо
`_mirror/dumps_ru1525_en16.zip` (дампы RU+EN16), `_mirror/stage_kit.zip` (скрипты+perks_v150),
`_mirror/core_docs.zip`, `_AGENT_BOOTSTRAP.md`. EN16 текстовый слой: `PoE_EN_16_text.zip`.
