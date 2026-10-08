# Deploy / sync notes

## Где живёт SKILL.md

- **Source of truth:** этот репозиторий, `SKILL.md` в корне. Все правки делаются здесь.
- **Зеркало для агент-discovery:** `/.well-known/agent-skills/easypay/SKILL.md` на `https://easypay.thenextgen.store` (репозиторий `EasyPay-Labs/easypay-landing`). Опубликовано в индексе `https://easypay.thenextgen.store/.well-known/agent-skills/index.json` по схеме Agent Skills Discovery RFC v0.2.0.

## При обновлении SKILL.md

1. Закоммитить и запушить изменения в `EasyPay-Labs/easypay-skill@main` (этот репо).
2. Обновить зеркало на лендинге:
   - Клонировать / `git pull` репозиторий `EasyPay-Labs/easypay-landing`.
   - Запустить `python scripts/sync_skill.py` — скрипт скачает свежий SKILL.md из main этого репо, перезапишет зеркало и пересчитает sha256 digest в `index.json`.
   - Закоммитить изменения в `easypay-landing`: файлы `.well-known/agent-skills/easypay/SKILL.md` и `.well-known/agent-skills/index.json`.
   - Запушить — GitHub Pages задеплоит за ~30 сек.
3. Проверить:
   - `curl -fsS https://easypay.thenextgen.store/.well-known/agent-skills/index.json` — обновлённый `digest`.
   - `curl -fsS https://easypay.thenextgen.store/.well-known/agent-skills/easypay/SKILL.md` — обновлённый текст.
   - Опционально: `curl -X POST https://isitagentready.com/api/scan -H 'Content-Type: application/json' -d '{"url":"https://easypay.thenextgen.store"}'` → `checks.discovery.agentSkills.status == "pass"`.

## Версионирование

Версия в frontmatter `SKILL.md` (`version: x.y.z`) — semver:

- **major** — breaking changes в контракте тулов или семантике JTBD-флоу.
- **minor** — новые тулы, новые JTBD-флоу, расширения без удаления существующего.
- **patch** — правки текста, опечатки, уточнения формулировок.

При bump'е версии — sync на лендинг обязателен, иначе агенты будут видеть устаревшую карточку.
