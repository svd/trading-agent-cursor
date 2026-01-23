# Trading Agent - AI помощник для трейдинга

AI-агент для помощи в трейдинге по методологии Александра Герчика.

## Описание

Этот агент помогает трейдеру:
- Выполнять "домашнее задание" (скрининг, анализ уровней, формирование сценариев)
- Отслеживать рыночную ситуацию и условия входа
- Вести статистику и анализировать результаты

## Структура

```
trading-agent-cursor/
├── .cursor/rules/      # Правила для Cursor
│   ├── methodology/    # Методология Герчика
│   ├── homework/       # Домашнее задание
│   ├── levels/         # Работа с уровнями
│   ├── entry-exit/     # Точки входа/выхода
│   ├── risk-management/# Управление рисками
│   └── statistics/     # Ведение статистики
├── .claude/            # Для Claude Code
│   └── CLAUDE.md       # Главный системный промпт
├── prompts/            # Переиспользуемые промпты
│   ├── screening.md
│   ├── level-analysis.md
│   ├── trade-setup.md
│   └── journal-entry.md
└── schemas/            # JSON схемы
    ├── instrument.schema.json
    ├── trade-setup.schema.json
    └── journal-entry.schema.json
```

## Использование

### Подключение к проекту

#### Вариант 1: Симлинки (рекомендуется)

```bash
cd trading-workspace
mkdir -p .cursor
ln -sf ../../trading-agent-cursor/.cursor/rules .cursor/rules
ln -sf ../trading-agent-cursor/.claude .claude
```

**Важно**: Пути симлинков должны быть относительно папки workspace, где они расположены.

#### Вариант 2: Git submodule

```bash
cd trading-workspace
git submodule add ../trading-agent-cursor agent
ln -s agent/.cursor/rules .cursor/rules
ln -s agent/.claude .claude
```

### Требования

- Claude Code / OpenCode / Cursor с поддержкой MCP
- MCP сервер markethub-mcp для рыночных данных
- MCP сервер для RAG (опционально, для доступа к knowledge base)

## Основные команды

### Домашнее задание
- "Проведи скрининг для интрадея"
- "Проанализируй {SYMBOL}"
- "Добавь {SYMBOL} на лонг-лист"

### Торговая сессия
- "Проверь условия для {SYMBOL}"
- "Рассчитай позицию для {SYMBOL}"
- "Что говорит Герчик о...?"

### Статистика
- "Запиши сделку..."
- "Итоги дня"
- "Статистика за месяц"

## Методология

Агент следует методологии Герчика:
- Обязательное домашнее задание перед торговлей
- Работа с уровнями поддержки/сопротивления
- Строгий риск-менеджмент (1% на сделку, минимум 2:1 R:R)
- Ведение статистики и анализ результатов

## Лицензия

Для личного использования.
