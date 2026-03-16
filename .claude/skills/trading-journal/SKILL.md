---
name: trading-journal
description: Use when user wants to record trades, write journal entries, view daily/monthly statistics, analyze errors, or track trading performance. Invoke proactively for any mention of "запиши сделку", "журнал", "итоги дня", "статистика", "анализ ошибок", "trade journal", "record trade", "daily summary", "monthly stats", "performance analysis", "что я наторговал". Essential for discipline and continuous improvement.
---

# Trading Journal — Recording Trades and Performance Analysis

## Overview

The trading journal captures every completed trade and session outcome. Without systematic recording, patterns go undetected, errors repeat, and improvement is impossible. This skill covers four workflows: recording individual trades, daily summaries, monthly statistics, and error analysis.

## File Structure

```
journal/
├── YYYY-MM/
│   ├── YYYY-MM-DD.md     ← daily journal (trades + session notes)
│   └── ...
└── ...
```

Statistics are derived from journal files — there is no separate stats database.

---

## Workflow 1: Record a Trade ("Запиши сделку")

### Required Information

Before writing, confirm with user (or extract from context):
- **Symbol**: ticker (AAPL, BTCUSDT, EURUSD)
- **Direction**: Long / Short
- **Entry price** and **exit price**
- **Position size** (shares / contracts / coins)
- **Stop-loss** used
- **Take-profit** target
- **Result**: profit/loss in $ and % of deposit
- **Exit reason**: TP reached / SL hit / Manual exit
- **Notes**: what worked, what didn't, emotional state

### Trade Entry Format

Append to `journal/YYYY-MM/YYYY-MM-DD.md` (create file if needed):

```markdown
## Сделка: {SYMBOL} {Long/Short} — {TIME}

| Параметр | Значение |
|----------|----------|
| Символ | {SYMBOL} |
| Направление | Long / Short |
| Вход | {entry_price} |
| Стоп-лосс | {sl_price} (риск: {risk_pct}%) |
| Тейк-профит | {tp_price} (R:R {rr_ratio}:1) |
| Выход | {exit_price} |
| Размер позиции | {size} шт/контр/монет |
| Результат | {+/-}${pnl} ({+/-}{pnl_pct}% депозита) |
| Причина выхода | TP / SL / Ручной выход |

**Заметки:**
- {what worked or didn't}
- {emotional state if relevant}
```

---

## Workflow 2: Daily Summary ("Итоги дня")

### When to Use
- End of trading session
- User asks "что было сегодня", "итоги дня", "подведи итог"

### Process

1. Read today's journal file `journal/YYYY-MM/YYYY-MM-DD.md`
2. If file doesn't exist — session had no trades, note that
3. Calculate metrics from the day's trades
4. Append summary section to the file

### Daily Summary Format

```markdown
---

## Итоги дня — {DATE}

### Сводка
- **Торговых сессий**: {session_count}
- **Сделок**: {total_trades} (лонг: {long_count}, шорт: {short_count})
- **Прибыльных**: {wins} ({win_rate}%)
- **Убыточных**: {losses}
- **Результат дня**: {+/-}${daily_pnl} ({+/-}{daily_pnl_pct}% депозита)
- **Лучшая сделка**: {best_trade} (+${best_pnl})
- **Худшая сделка**: {worst_trade} (-${worst_pnl})

### Соблюдение правил
- [ ] Домашнее задание выполнено перед сессией
- [ ] Все сделки по сценарию
- [ ] Стоп-лоссы использованы в каждой сделке
- [ ] Не превышен дневной лимит (3-5 сделок, макс 3-5% риска)
- [ ] Остановился после 3 убыточных подряд (если было)

### Выводы
{1-3 конкретных вывода: что сработало, что нет, что улучшить}
```

---

## Workflow 3: Monthly Statistics ("Статистика за месяц")

### When to Use
- User asks "статистика за месяц", "итоги месяца", "как торговал в {month}"

### Process

1. List all daily journal files in `journal/YYYY-MM/`
2. Parse each file to extract trade records
3. Calculate aggregate metrics
4. Present formatted report

### Metrics to Calculate

**Performance:**
- Total trades, win rate, average R:R
- Total PnL, best/worst day
- Maximum drawdown (consecutive losses)
- Expectancy per trade = (Win% × Avg Win) − (Loss% × Avg Loss)

**By instrument:** Rank by win rate and PnL

**By strategy type** (if noted in trades): breakout vs bounce vs retest

**By day of week:** Which days perform best

**Risk management compliance:**
- Average risk per trade (should be ≤1%)
- Days where daily limit was exceeded
- Instances of trading without SL

### Monthly Report Format

```markdown
# Статистика — {YYYY-MM}

## Общая статистика
| Метрика | Значение |
|---------|----------|
| Всего сделок | {N} |
| Прибыльных | {wins} ({win_rate}%) |
| Убыточных | {losses} |
| Средний R:R | 1:{avg_rr} |
| Expectancy | ${expectancy} на сделку |
| Итого PnL | {+/-}${total_pnl} |
| Макс. просадка | -${max_drawdown} |

## По инструментам
| Тикер | Сделок | Win Rate | Итого PnL |
|-------|--------|----------|-----------|
| ... | ... | ...% | ... |

## По дням недели
| День | Сделок | Win Rate | PnL |
|------|--------|----------|-----|
| ... | ... | ...% | ... |

## Соблюдение риск-менеджмента
- Средний риск на сделку: {avg_risk}% (норма ≤1%)
- Нарушений дневного лимита: {violations}
- Сделок без стоп-лосса: {no_sl_count}

## Выводы и рекомендации
1. {strength}
2. {weakness}
3. {action_item}
```

---

## Workflow 4: Error Analysis ("Анализ ошибок")

### When to Use
- User asks "анализ ошибок", "где я ошибаюсь", "почему убытки", "разбор проигрышных сделок"

### Process

1. Collect all losing trades from the specified period
2. Look for recurring patterns in the **Notes** field
3. Categorize errors by type
4. Prioritize by frequency and impact

### Common Error Categories

| Тип ошибки | Признаки |
|------------|----------|
| **Вход не по плану** | Цена входа не соответствует сценарию |
| **Слабый уровень** | Уровень не проходил проверку по силе |
| **Нарушение R:R** | R:R меньше 2:1 на момент входа |
| **Эмоциональный вход** | Вход после серии убытков или без домашки |
| **Ранний выход** | Вышел до TP без технической причины |
| **Передвинул стоп** | Сдвинул SL против позиции |
| **Торговля в боковике** | Вход в середине диапазона |

### Error Report Format

```markdown
## Анализ ошибок — {period}

**Всего убыточных сделок:** {N}

### Топ ошибок
1. **{error_type}** — {count} раз ({pct}% убытков)
   - Примеры: {trade1}, {trade2}
   - Рекомендация: {specific_action}

2. **{error_type}** — {count} раз
   ...

### Приоритетное улучшение
{1 конкретное действие, которое даст наибольший эффект}
```

---

## Important Rules

1. **Record every trade** — even losses, especially losses
2. **Notes are mandatory** — a trade without notes provides no learning value
3. **Be honest** — don't rationalize bad trades; note what actually happened
4. **Look for patterns** — one error is a mistake; three is a habit to fix

## References

- @trading-risk-management — Risk rules to check compliance against
- @trading-homework — Homework workflow (source of trade scenarios)
- @trading-entry-exit — Entry/exit rules (for evaluating whether trade followed plan)
