# _AGENT_BOOTSTRAP.md — как агенту быстро получить файлы проекта

Канал обмена: этот репозиторий. Загрузка файлов в чат у игрока сломана (upload failed).
Tокен (fine-grained PAT) игрок даёт в чате. Проверено 2026-11-11.

## Быстрый старт
1._zipball репозитория: https://codeload.github.com/Rustam358/HandoffPack358/zip/refs/heads/main
   (~5 МБ/с, публичный, токен не нужен). Внутри: Paths of Empyrean RU 1.5 (2).zip (zip истины v1.5.25).
2. Полный пак v6.00 — release qwe, asset id 734721760 (27760738 байт, md5 97e20e86a3eb818096f732987bc4e9f6).
3. Читать 00_ПРОЧТИ_ПЕРВЫМ.md -> handoff/03_агентская_память/state.md -> user.md.

## Как скачать release-ассет (CDN медленный, ~50-70 КБ/с на соединение)
- GET api.github.com/repos/Rustam358/HandoffPack358/releases/assets/734721760
  с Accept: application/octet-stream, allow_redirects=False -> заголовок Location (signed URL).
- Качать параллельно Range-чанками 256 КБ, 192-320 потоков -> ~2-3 МБ/с суммарно.
  Отстающие чанки добирать поддиапазонами 16-32 КБ. Всего 2-3 вызова песочницы.
- 7z распаковывается БЕЗ 7zz/py7z (их нет, pypi ~18 КБ/с), чистым stdlib:
    import struct, lzma
    off, _ = struct.unpack("<QQ", data[12:28])
    d = lzma.LZMADecompressor(format=lzma.FORMAT_RAW,
          filters=[{"id": lzma.FILTER_LZMA1, "dict_size": 1<<23, "lc": 3, "lp": 0, "pb": 2}])
    inner_zip = d.decompress(data[32:32+off])  # 31211595 байт, md5 fa1363541ad70a89644a9f7456116b5f

## Особенности песочницы (важно!)
- Жёсткий лимит ~20 с на вызов. Песочница часто сбрасывается между вызовами (файлы исчезают):
  всё важное делать за один вызов; источник истины — этот репозиторий, не диск песочницы.
- Запись в репо: работает Contents API (PUT /contents/{path}, base64).
  POST /git/tree с fine-grained PAT даёт 404 — путь Git Data tree/commit не использовать.
- Egress-фильтр: заблокированы catbox, pixeldrain, tmpfiles, Google Drive, Dropbox,
  Discord CDN, rentry, Яндекс.Диск, transfer.sh, gofile. Доступны: github (api/codeload/
  release-assets), paste.rs, pastebin.com, bitbucket, gitlab, temp.sh (медленно).

## Отдача результатов игроку
Текст — в чат или на paste.rs (ссылка игроку). Бинарники — temp.sh POST или коммит в этот репо.