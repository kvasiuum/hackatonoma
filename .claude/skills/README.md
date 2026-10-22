# Скиллы проекта

Подключаются автоматически, когда Claude Code открывает этот репозиторий.

| Скиллы | Источник | Лицензия |
|---|---|---|
| `motion-animations` | архив motion-animations.zip (Motion / motion.dev) | — |
| `animate`, `animate-expo`, `animation-vocabulary`, `apple-design`, `ask-sonner`, `emil-design-eng`, `find-animation-opportunities`, `improve-animations`, `pick-ui-library`, `prototype`, `review-animations`, `write-swift` | архив skills-main.zip | `LICENSE-skills-main` |
| `ui-ux-pro-max`, `design`, `design-system`, `ui-styling`, `brand`, `banner-design`, `slides` | архив ui-ux-pro-max-skill-main.zip | `LICENSE-ui-ux-pro-max` |
| `improve` | github.com/motiondivision/motion, `.agents/skills/improve` (автор shadcn) | MIT |

При установке удалены папки с тестами. В `ui-ux-pro-max/SKILL.md` путь к скрипту заменён на `.claude/skills/ui-ux-pro-max/scripts/search.py` (от корня репозитория).
Генерация логотипов и иконок в `design` обращается к внешним API (Gemini, Atlas Cloud, MuAPI) только при заданных ключах `*_API_KEY`.
