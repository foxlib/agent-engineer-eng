# Урок 2: Как думают агенты — языковые модели как механизм рассуждений

## Введение

В уроке 1 мы говорили, что агент состоит из трех частей: модели (мозг), инструментов (руки) и уровня оркестровки (контур управления). В этом уроке мы подробно рассмотрим мозг.

Языковая модель — наиболее важный компонент агента. Именно она считывает цель пользователя, рассуждает о том, что нужно делать, решает, какие инструменты использовать, интерпретирует результаты и генерирует окончательные ответы. Все остальное в системе агента существует для поддержки или расширения возможностей модели.

Понимание того, как работают языковые модели — даже на высоком уровне — значительно улучшит ваши навыки создания агентов. Вы будете знать, почему одни подсказки работают, а другие нет. Вы будете понимать, почему агенты иногда сбиваются с пути. И вы сможете принимать обоснованные решения о выборе модели, разработке подсказок и архитектуре системы.

---

## Большие языковые модели как мозг агента

Большая языковая модель — это нейронная сеть, обученная на огромных массивах текстовых данных. По своей сути, она делает одно: **предсказывает следующий токен** (примерно, следующее слово или часть слова), учитывая все, что было до него.

Звучит просто, но возникающие в результате этого обучения возможности поразительны:

- **Понимание**: Понимание сложных вопросов и инструкций
- **Рассуждение**: Решение многошаговых логических задач
- **Планирование**: Разбиение цели на подзадачи
- **Генерация кода**: Написание и отладка программного обеспечения
- **Выбор инструмента**: Определение того, какую функцию вызвать и с какими параметрами
- **Резюмирование**: Сведение длинных документов к ключевым моментам
- **Перевод**: Преобразование между языками, форматами и представлениями

### Что большие языковые модели умеют делать хорошо

| Возможности | Пример в контексте агента |
|---|---|
| Понимание естественного языка | Анализ запроса пользователя: "Отменить мой последний заказ" |
| Рассуждение и планирование | Принятие решения: "Сначала мне нужно найти заказ, затем проверить, можно ли его отменить, а затем отменить" |
| Выбор инструмента | Выбор вызова `lookup_order(user_id, sort="recent")` |
| Форматирование вывода | Возврат чистого JSON-ответа или дружественного сообщения |
| Интерпретация ошибок | Чтение ошибки API и принятие решения о повторной попытке с другими параметрами |
| Синтез контекста | Объединение результатов нескольких вызовов инструментов в связный ответ |

### Что не могут делать LLM (без посторонней помощи)

| Ограничение | Почему это важно |
|---|---|
| Нет доступа к данным в реальном времени | Знания модели имеют дату окончания обучения |
| Нет гарантий вычислений | Модели LLM могут ошибаться в математике — они предсказывают токены, а не вычисляют |
| Отсутствие постоянной памяти | Каждый разговор начинается заново, если вы не создадите память |
| Отсутствие возможности действовать | Без инструментов модель может только генерировать текст |
| Риск галлюцинаций | Модели могут генерировать правдоподобную, но неверную информацию |
| Ограничения контекстного окна | Существует максимальное количество текста, которое модель может обрабатывать одновременно |

Именно поэтому существуют агенты. Инструменты компенсируют неспособность модели действовать и получать доступ к данным в реальном времени. Оркестрация компенсирует отсутствие у модели постоянной памяти и склонность к отклонениям от намеченного пути.

---

## Как языковые модели обрабатывают информацию

Вам не нужно досконально разбираться в архитектуре трансформеров, чтобы создавать агентов, но понимание трех ключевых концепций поможет вам писать более качественные подсказки и проектировать более эффективные системы.

### Токены: единица языка

Языковые модели не читают символы или слова. Они читают **токены** — фрагменты текста, которые модель научилась распознавать во время обучения. Токен может представлять собой целое слово, часть слова или знак препинания.

**Примеры:**

| Текст | Примерное количество токенов |
|---|---|
| "Привет" | 1 токен |
| "Привет, мир!" | 3 токена |
| "ChatGPT — потрясающий" | 4 токена |
| Типичная функция кода (20 строк) | 100-300 токенов |
| Целая страница английского текста | ~500-700 токенов |

**Почему это важно для агентов:**

- **Оплата**: Большинство API взимают плату за токен (вход + выход). Рабочие процессы агентов используют гораздо больше токенов, чем одиночные запросы, потому что каждая итерация цикла отправляет полный контекст.
- **Скорость**: Больше токенов = больше времени на генерацию. Делайте описания инструментов краткими.
- **Ограничения контекста**: Существует максимальное количество токенов, которые модель может обработать за один вызов. Если накопленный контекст вашего агента превышает это значение, вы теряете информацию.

### Окна контекста: рабочая память модели

**Окно контекста** — это общее количество токенов, которые модель может обрабатывать одновременно. Представьте это как стол модели — все, на что ей нужно ссылаться, должно помещаться на этом столе.

| Модель | Окно контекста |
|---|---|
| Gemini 2.5 Pro | 1 000 000 токенов |
| Gemini 2.0 Flash | 1 000 000 токенов |
| GPT-4o | 128 000 токенов |
| Claude 3.5 Sonnet | 200 000 токенов |

**Что отображается в контекстном окне во время звонка агента:**

```
+------------------------------------------+
| Системные инструкции ("Вы - это...")     |  ~200-500 токенов
+------------------------------------------+
| Определения инструментов (имена,         |  ~500-2000 токенов
| описания, схемы параметров)              |
+------------------------------------------+
| История разговоров                       |  Переменная
+------------------------------------------+
| Предыдущие вызовы инструментов и         |  Переменная (может быстро увеличиваться)  
| результаты                               |
+------------------------------------------+
| Сообщение текущего пользователя          |  Переменная
+------------------------------------------+
| = Всего должно поместиться в контекстное |
| окно                                     |
+------------------------------------------+
```

**Почему это важно для агентов:**

По мере выполнения агентом множества шагов контекст расширяется с каждым вызовом инструмента и результатом. В пятишаговом рабочем процессе агента могут накапливаться тысячи токенов результатов работы инструментов. Если не быть осторожным, можно исчерпать контекстное окно в середине задачи.

Стратегии управления этим:
- **Обобщать** промежуточные результаты вместо сохранения исходных данных
- **Сокращать** длинные выходные данные инструментов до релевантных частей
- **Использовать модели с большими контекстными окнами** для сложных многошаговых задач
- **Реализовать скользящее окно**, которое отбрасывает более старый, менее релевантный контекст

### Внимание: как модель фокусируется

**Механизм внимания** позволяет модели определять, какие части контекста релевантны текущему решению. При принятии решения о том, какой токен сгенерировать следующим, модель присваивает разные веса разным частям входных данных.

Представьте это как чтение длинного документа и выделение важных частей. Модель «выделяет» токены, наиболее релевантные тому, что она пытается сделать в данный момент.

**Почему это важно для агентов:**

- **Размещайте важную информацию там, где модель может её найти.** Модели, как правило, уделяют больше внимания началу и концу контекста. Важные инструкции следует размещать в системном запросе (в начале) или рядом с запросом пользователя (в конце).
- **Будьте конкретны и ясны.** Расплывчатые инструкции заставляют модель гадать, что важно. Конкретные инструкции облегчают механизму внимания захват нужной информации.
- **Структура помогает.** Четкие заголовки, нумерованные списки и согласованное форматирование помогают модели анализировать и обрабатывать правильный контент.

---

## Стратегии рассуждения

То, как агент «думает» о проблеме, во многом зависит от того, как вы задаете модели вопросы. Различные стратегии рассуждения дают совершенно разные результаты, особенно для сложных задач.

### Метод «цепочки рассуждений» Chain-of-Thought (CoT)

**Что это:** Побуждение модели к пошаговому анализу проблемы перед предоставлением окончательного ответа.

**Как это работает:** Вместо того чтобы сразу переходить к ответу, модель генерирует промежуточные этапы рассуждений. Это значительно повышает точность в задачах, требующих логики, математики или многоэтапного анализа.

**Пример без CoT:**
```
Prompt: «Если сервер обрабатывает 100 запросов в секунду, и у нас есть 3 сервера,
        при этом 40% трафика приходится на сервер 1, сколько запросов в секунду
        обрабатывает сервер 1?»

Ответ модели: "120 запросов в секунду" (неверно)
```

**Пример с CoT:**
```
Prompt: "Подумайте шаг за шагом. Если сервер обрабатывает 100 запросов в секунду, и
        у нас есть 3 сервера, при этом 40% трафика приходится на сервер 1, сколько
        запросов в секунду обрабатывает сервер 1?"

Ответ модели:
""Шаг 1: Общая пропускная способность составляет 3 сервера x 100 запросов/с = 300 запросов/с
  Шаг 2: Сервер 1 получает 40% от общего трафика
  Шаг 3: 40% от 300 = 120 запросов/с
  Шаг 4: Сервер 1 может обрабатывать 100 запросов/с, но получает 120 запросов/с
Ответ: Сервер 1 получает 120 запросов/с, но может обрабатывать только 100 запросов/с,
следовательно, он перегружен на 20 запросов/с"
```

Пошаговый подход выявил условие перегрузки, которое не было обнаружено в прямом ответе.

**Когда использовать дерево мыслей (CoT) для агентов:**
- Принятие сложных решений по выбору инструмента («Учитывая эти 5 инструментов, какой из них поможет в данном случае?»)
- Многоэтапное планирование («Какая последовательность действий позволит достичь этой цели?»)
- Диагностика ошибок («Инструмент вернул ошибку — что пошло не так и что мне следует попробовать дальше?»)

### Дерево мыслей Tree-of-Thoughts (ToT)

**Что это:** Расширение дерева мыслей, в котором модель исследует несколько путей рассуждений, оценивает их и выбирает лучший.

**Как это работает:** Вместо одной цепочки рассуждений модель генерирует несколько возможных подходов, оценивает или критикует каждый из них и переходит к наиболее перспективному пути.

```
Цель: «Оптимизировать этот медленный запрос к базе данных»

Путь A: «Добавить индекс по столбцу в предложении WHERE»
  -> Оценка: «Вероятно, эффективно, низкий риск, легко реализовать»

Путь B: «Переписать как материализованное представление»
  -> Оценка: «Может помочь, но добавляет сложности и требует обслуживания»

Путь C: «Денормализовать структуру таблицы»
  -> Оценка: «Может сработать, но высокий риск, влияет на другие запросы»

Решение: Сначала перейти к пути A, если A недостаточно эффективен, попробовать путь B
```

**When to use ToT for agents:**
- When there are multiple valid approaches and you want the model to consider trade-offs
- Complex debugging where the root cause is uncertain
- Architecture decisions that require evaluating alternatives

**Trade-off:** ToT uses more tokens and takes more time. Reserve it for decisions where the cost of choosing wrong is high.

### Step-by-Step Decomposition

**What it is:** Breaking a complex goal into a sequence of simpler subtasks before executing any of them.

**How it works:** The agent first creates a plan, then executes each step of the plan, checking progress along the way.

```
User goal: "Set up monitoring for our new API endpoint"

Plan:
1. Check what monitoring tools are currently configured
2. Determine what metrics matter for this endpoint (latency, error rate, throughput)
3. Create the monitoring dashboard
4. Set up alerting thresholds
5. Test that alerts fire correctly
6. Document the monitoring setup

Execution: [proceeds step by step, with each step potentially using tools]
```

**When to use decomposition for agents:**
- Multi-step tasks where the order matters
- Tasks where you want the agent to be transparent about its approach
- Complex workflows that benefit from checkpoints

---

## Model selection: picking the right model for the job

Not all tasks need the most powerful model. Choosing the right model is an engineering decision that balances capability, cost, speed, and reliability.

### The model spectrum

```
Lighter / Faster / Cheaper                  Heavier / Smarter / More Expensive
|----------------------------------------------------------|
Gemini Flash          Gemini Pro          Gemini 2.5 Pro
(simple tasks)        (balanced)          (complex reasoning)
```

### When to use what

| Task Type | Recommended Tier | Why |
|---|---|---|
| Classification ("Is this spam?") | Light (Flash) | Simple decision, no complex reasoning needed |
| Data extraction ("Pull the date from this email") | Light (Flash) | Pattern matching, well-defined output |
| Summarization | Light to Medium | Depends on length and complexity of source |
| Multi-step reasoning | Medium to Heavy (Pro) | Requires sustained logical chains |
| Complex code generation | Heavy (2.5 Pro) | Needs deep understanding of patterns and edge cases |
| Agentic tool use | Medium to Heavy | Tool selection and result interpretation need strong reasoning |
| Creative writing | Medium | Good results without the heaviest models |

### Model routing: using different models for different steps

A sophisticated agent system does not use the same model for every step. This is called **model routing** - directing different parts of the workflow to different models based on complexity.

**Example architecture:**

```
User query arrives
    |
    v
[Light model: Classify intent]  --> "order_status"
    |
    v
[Light model: Extract parameters]  --> order_id: 12345
    |
    v
[Tool call: Look up order]  --> status data
    |
    v
[Light model: Format response]  --> "Your order #12345 shipped on March 15"
```

In this flow, every step uses a fast, cheap model because none of the individual steps require heavy reasoning. The total cost and latency are much lower than using a frontier model for the entire interaction.

**Compare with a harder task:**

```
User: "Review this pull request and suggest improvements"
    |
    v
[Heavy model: Analyze code changes, reason about patterns,
 identify bugs, suggest improvements]
    |
    v
[Return detailed review]
```

This task needs deep reasoning, so it warrants a more capable model.

### Google Cloud Model options

Google Cloud provides access to Gemini models through Vertex AI:

- **Gemini 2.0 Flash** - Fast and efficient for most agent tasks. Large context window (1M tokens). Good balance of capability and speed.
- **Gemini 2.5 Pro** - Top-tier reasoning for complex tasks. Use when the task requires deep analysis, complex multi-step logic, or nuanced understanding.
- **Gemini 2.0 Flash Lite** - Fastest and cheapest option for simple tasks like classification and extraction.

> **Learn more:** [Vertex AI Model Documentation](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/models)

> **Learn more:** [Gemini API Documentation](https://ai.google.dev/gemini-api/docs)

---

## The role of system instructions

System instructions are the agent's "job description." They tell the model who it is, what it can do, how it should behave, and what it should not do.

### What goes in system instructions

A well-written system instruction for an agent typically includes:

1. **Role definition**: What the agent is and who it serves
2. **Capabilities**: What tools are available and when to use them
3. **Constraints**: What the agent should not do
4. **Output format**: How to structure responses
5. **Error handling**: What to do when things go wrong
6. **Personality/tone**: How to communicate (if relevant)

### Example: customer support agent

```
You are a customer support agent for Acme Corp, an online electronics retailer.

Your role:
- Help customers with order inquiries, returns, and product questions
- Be friendly, professional, and concise

Available tools:
- lookup_order(order_id): Returns order status, items, and shipping info
- initiate_return(order_id, reason): Starts a return process
- search_products(query): Searches the product catalog
- escalate_to_human(reason): Transfers to a human agent

Guidelines:
- Always verify the customer's identity before accessing order information
- If you cannot resolve an issue in 3 attempts, escalate to a human agent
- Never make up order information - always use the lookup tool
- Do not offer discounts or refunds beyond standard policy
- Keep responses under 3 paragraphs

When you encounter an error from a tool:
- If it is a temporary error (timeout, 500), retry once
- If it is a permanent error (not found, unauthorized), explain the issue to the customer
- If you are unsure, escalate to a human agent
```

### Tips for writing effective system instructions

| Do | Do Not |
|---|---|
| Be specific about tool usage | Leave tool selection ambiguous |
| Define clear boundaries | Assume the model knows your business rules |
| Include error handling guidance | Hope the model figures out errors on its own |
| Specify output format | Let the model choose its own format each time |
| Use concrete examples | Write abstract, vague instructions |
| Keep instructions concise | Write a 10-page essay (wastes context) |

### Ordering matters

Models pay more attention to instructions at the beginning and end of the system prompt. Structure your system instructions like this:

```
1. Most critical rules (identity, safety constraints)     <-- Start
2. Tool usage guidelines
3. Output format
4. Examples
5. Edge case handling
6. Reminder of most critical rules                        <-- End
```

This takes advantage of the "primacy and recency" effects in attention.

---

## Temperature, sampling, and agent behavior

When a language model generates text, it does not just pick the single most likely next token. It samples from a probability distribution over all possible tokens. The parameters that control this sampling have a big impact on agent behavior.

### Temperature

**Temperature** controls how random the model's outputs are.

- **Temperature 0 (or very low)**: The model almost always picks the most likely token. Outputs are deterministic and focused.
- **Temperature 1**: The model samples proportionally to token probabilities. Outputs are more varied and creative.
- **Temperature > 1**: The model becomes increasingly random. Outputs become unpredictable.

**Visual analogy:**

```
Temperature 0:    "The capital of France is Paris."
                  (Always the same answer)

Temperature 0.7:  "The capital of France is Paris, a city known for..."
                  (Slight variation in elaboration)

Temperature 1.5:  "The capital of France is historically rooted in..."
                  (More creative, potentially off-track)
```

### What temperature to use for agents

| Agent Task | Recommended Temperature | Why |
|---|---|---|
| Tool selection | 0 - 0.2 | You want deterministic, correct tool calls |
| Data extraction | 0 | Exact answers, no creativity needed |
| Code generation | 0 - 0.3 | Correctness matters more than variety |
| Planning | 0.2 - 0.5 | Some flexibility helps explore options |
| Creative writing | 0.7 - 1.0 | Variety and originality are valued |
| Brainstorming | 0.8 - 1.0 | Want diverse ideas |

**For most agent use cases, keep temperature low (0 to 0.3).** Agents need to make reliable decisions about tool use, parameter extraction, and reasoning. High temperature introduces randomness where you want consistency.

### Top-K and Top-P Sampling

These are additional controls on how the model selects tokens.

**Top-K:** Only consider the K most likely tokens. If K=50, the model ignores every token outside the top 50 candidates.

**Top-P (nucleus sampling):** Only consider tokens whose cumulative probability reaches P. If P=0.9, the model considers the smallest set of tokens that together have a 90% probability.

```
Token probabilities: [0.4, 0.25, 0.15, 0.08, 0.05, 0.03, 0.02, ...]

Top-K=3:  Consider only [0.4, 0.25, 0.15]
Top-P=0.8: Consider only [0.4, 0.25, 0.15] (cumulative = 0.8)
Top-P=0.9: Consider only [0.4, 0.25, 0.15, 0.08] (cumulative = 0.88... round up)
```

**For agents:** Use conservative settings. Top-P around 0.9 and moderate Top-K values are reasonable defaults. The model's default settings are usually fine for agent work - temperature is the parameter you are most likely to want to adjust.

### How sampling affects the agent loop

Consider an agent that needs to decide which tool to call. With low temperature, it will consistently pick the same (usually correct) tool for a given situation. With high temperature, it might pick different tools on different runs, leading to inconsistent behavior.

```
User: "What is the weather in Tokyo?"

Low temperature (0):
  -> Agent thinks: "I need the weather tool"
  -> Calls: get_weather(city="Tokyo")
  -> Consistent, predictable

High temperature (1.2):
  -> Run 1: Calls get_weather(city="Tokyo")
  -> Run 2: Calls web_search("Tokyo weather forecast")
  -> Run 3: Tries to answer from training data (no tool call)
  -> Inconsistent, hard to debug
```

For production agents, deterministic behavior is almost always what you want.

---

## ELI5: how the LLM brain works

### Think of the LLM like a chef

Imagine you are running a restaurant kitchen, and the LLM is your head chef.

**The chef's training (model training):**
The chef has spent years studying thousands of cookbooks, watching cooking shows, and practicing recipes. They have not memorized every recipe word for word, but they have developed deep intuitions about what flavors go together, what techniques work for what ingredients, and how to improvise when something is missing.

**Tokens are like ingredients:**
The chef does not think in terms of complete dishes all at once. They think in terms of individual ingredients and steps. "First the onion, then the garlic, then the tomatoes..." Each ingredient choice informs the next one. That is how token prediction works - each token is chosen based on all the tokens before it.

**The context window is like the counter space:**
The chef can only work with what fits on the kitchen counter. If the counter is huge (1 million tokens), they can have lots of ingredients, recipes, and prep work visible at once. If the counter is small, they have to put things away to make room, and might forget what they were doing.

**Temperature is like the chef's mood:**
- Low temperature: The chef is focused and methodical. They follow the recipe exactly. Every time you order the same dish, it tastes the same.
- High temperature: The chef is feeling creative. They improvise, substitute ingredients, try new things. Sometimes the result is amazing, sometimes it is weird.

**System instructions are like the restaurant concept:**
"You are a French bistro. You use traditional techniques. You do not serve sushi." This shapes every decision the chef makes without having to be repeated for each dish.

**Tools are like kitchen equipment:**
The chef's knowledge alone does not cook food. They need an oven, a stove, knives, and measuring tools. Similarly, the LLM's reasoning alone does not look up data or call APIs. It needs tools.

**The agent loop is like a cooking show challenge:**
The chef gets a challenge ("Make a three-course meal for someone who is gluten-free"). They plan their approach, start cooking, taste as they go, adjust seasoning, plate the food, and evaluate the result. If the sauce breaks, they troubleshoot and adapt. That plan-act-observe-adjust loop is exactly what an agent does.

---

## Putting it together: how model choice affects agent quality

Here is a practical example of how these concepts combine in a real agent scenario.

### Scenario: a bug triage agent

Your team wants an agent that reads new bug reports, categorizes them by severity, assigns them to the right team, and drafts an initial investigation plan.

**Model selection decision:**

| Step | Model Choice | Reasoning |
|---|---|---|
| Classify severity (P0-P3) | Flash (light) | Simple classification with clear criteria |
| Assign to team | Flash (light) | Lookup-style decision based on component |
| Draft investigation plan | Pro (heavy) | Requires understanding the bug, related systems, and suggesting diagnostic steps |

**Temperature decisions:**

| Step | Temperature | Reasoning |
|---|---|---|
| Classify severity | 0 | Must be deterministic - same bug should always get the same severity |
| Assign to team | 0 | Must be consistent - same component should always route to the same team |
| Draft investigation plan | 0.3 | Slight flexibility helps generate more useful and varied investigation ideas |

**System instruction excerpt:**

```
You are a bug triage agent for the Platform Engineering team.

Severity classification:
- P0: Service is down or data loss is occurring
- P1: Major feature is broken, no workaround
- P2: Feature is impaired but a workaround exists
- P3: Minor issue, cosmetic, or improvement request

Team routing:
- Auth/login issues -> Identity team
- API errors -> Platform team
- UI issues -> Frontend team
- Database/performance -> Infrastructure team

When drafting an investigation plan:
- Start with the most likely root cause
- List 3-5 diagnostic steps in order of priority
- Include relevant log queries or dashboard links
- Note any recent deployments that might be related
```

This example shows how understanding the model's capabilities, setting appropriate parameters, and writing clear instructions all contribute to a reliable agent.

---

## Common mistakes when working with LLMs in agents

### 1. trusting the model for math

LLMs predict tokens, not compute equations. For any calculation that matters, use a code execution tool.

```
Bad:  "Calculate the total cost of 47 items at $23.99 each"
      -> Model might say $1,127.53 (the correct answer is $1,127.53,
         but it got lucky - it often gets these wrong)

Good: Have the agent call a calculator tool or code execution tool
      -> calculate("47 * 23.99") -> $1,127.53 (guaranteed correct)
```

### 2. assuming perfect memory

The model only "remembers" what is in its current context window. If information from step 1 gets truncated by step 10, the model will not remember it.

### 3. over-relying on a single model

Using a frontier model for every step wastes money and adds latency. Use model routing to match model capability to task complexity.

### 4. ignoring the system prompt

A well-crafted system prompt can be the difference between an agent that works 50% of the time and one that works 95% of the time. Invest time in writing and iterating on your system instructions.

### 5. not accounting for hallucination

LLMs will confidently generate plausible but incorrect information. For any fact that matters, ground the agent's response in tool results rather than the model's training data.

---

## Key takeaways

1. **The LLM is the reasoning engine** of your agent. Understanding its capabilities and limitations is foundational to building good agents.

2. **Tokens, context windows, and attention** are the three key concepts. Tokens determine cost and speed. Context windows determine how much information the model can work with. Attention determines what the model focuses on.

3. **Reasoning strategies matter.** Chain-of-Thought, Tree-of-Thoughts, and step-by-step decomposition can dramatically improve agent performance on complex tasks.

4. **Pick the right model for each step.** Use lighter models for simple tasks and heavier models for complex reasoning. Model routing reduces cost and latency.

5. **System instructions are your main lever for controlling agent behavior.** Write them carefully, be specific, and iterate based on testing.

6. **Keep temperature low for agents.** Deterministic behavior is almost always better for production agent systems.

---

## What is next?

The brain is important, but an agent that can only think is just a chatbot. In the next lesson, we give our agent hands - tools that let it interact with the world, call APIs, search the web, and execute code.

[Next: Lesson 3 - Tools: Giving Agents Hands -->](../03-tools-giving-agents-hands/README.md)
