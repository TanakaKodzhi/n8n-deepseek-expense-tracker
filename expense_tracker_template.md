{
  "name": "Трекер Расходов",
  "nodes": [
    {
      "parameters": {
        "content": "**Проект: Персональный трекер расходов в Notion**\n\n**Суть:** Бот принимает сообщения вида «Обед 500» или «Такси 300», n8n парсит текст через DeepSeek, определяет категорию и записывает строку в таблицу Notion.",
        "height": 240,
        "color": 6
      },
      "type": "n8n-nodes-base.stickyNote",
      "typeVersion": 1,
      "position": [
        -96,
        -368
      ],
      "id": "aa7fba37-261c-47a5-85e0-63c644d402dd",
      "name": "Sticky Note"
    },
    {
      "parameters": {
        "resource": "databasePage",
        "databaseId": {
          "__rl": true,
          "mode": "list",
          "value": ""
        },
        "title": "={{ $json.output.item }}",
        "propertiesUi": {
          "propertyValues": [
            {
              "key": "Категория📃|rich_text",
              "textContent": "={{ $json.output.category }}"
            },
            {
              "key": "Сумма💵(RUB)|number",
              "numberValue": "={{ $json.output.amount }}"
            },
            {
              "key": "Дата📅|date",
              "date": "={{ $now.setZone('Asia/Vladivostok').toISO() }}"
            }
          ]
        },
        "options": {}
      },
      "type": "n8n-nodes-base.notion",
      "typeVersion": 3,
      "position": [
        640,
        -224
      ],
      "id": "2765827a-8fdd-439d-950a-fbc1bee9648c",
      "name": "Create a database page"
    },
    {
      "parameters": {
        "updates": [
          "message"
        ],
        "additionalFields": {}
      },
      "type": "n8n-nodes-base.telegramTrigger",
      "typeVersion": 1.5,
      "position": [
        -112,
        -80
      ],
      "id": "d9bb5a72-e54f-498b-aaac-d8b0c044d8cf",
      "name": "Telegram Trigger"
    },
    {
      "parameters": {
        "conditions": {
          "options": {
            "caseSensitive": true,
            "leftValue": "",
            "typeValidation": "strict",
            "version": 4
          },
          "conditions": [
            {
              "id": "d0ad9982-5ccb-4078-8d58-2525afa876b4",
              "leftValue": "={{ $json.message.text }}",
              "rightValue": "^[а-яА-ЯёЁa-zA-Z0-9\\s\"«»№\\-]+\\s+\\d+$",
              "operator": {
                "type": "string",
                "operation": "regex"
              }
            }
          ],
          "combinator": "and"
        },
        "options": {}
      },
      "type": "n8n-nodes-base.if",
      "typeVersion": 2.4,
      "position": [
        64,
        -80
      ],
      "id": "b716344a-be96-4d4e-9a27-ef7000a80b26",
      "name": "If"
    },
    {
      "parameters": {
        "chatId": "={{ $('Telegram Trigger').item.json.message.chat.id }}",
        "text": "=<b>⚠️Не удалось распознать расход</b>\n<i>Я пока не понял этот формат. Пожалуйста, напишите</i> <b>название расхода,</b> <i>а в конце через пробел</i> <b>сумму цифрами</b> <i>без дополнительных знаков в конце</i> \nПримеры: \n• <i>Такси 300</i>\n• <i>Обед в столовой 570</i>\n• <i>Еда магазин \"Пятёрочка\" 1100</i>\n• <i>Бензин АИ-95 1500</i>",
        "additionalFields": {
          "appendAttribution": false,
          "parse_mode": "HTML"
        }
      },
      "type": "n8n-nodes-base.telegram",
      "typeVersion": 1.2,
      "position": [
        64,
        128
      ],
      "id": "23f2e2db-7320-4244-8036-ccfc3506f087",
      "name": "Send a text message"
    },
    {
      "parameters": {
        "promptType": "define",
        "text": "=Ты — специализированный API-парсер финансовых расходов. Твоя единственная задача — извлечь данные о трате из текста пользователя и классифицировать категорию.\n\nКАТЕГОРИИ:\n1. Еда и продукты (супермаркеты, доставка продуктов, бакалея, мясо, вода)\n2. Кафе и перекусы (рестораны, фастфуд, кофе, столовые, готовая еда вне дома)\n3. Транспорт (такси, общественный транспорт, бензин, каршеринг, автоуслуги)\n4. Жильё и коммуналка (аренда, ЖКХ, связь для дома, бытовая химия, хозтовары)\n5. Подписки и связь (мобильная связь, хостинги, домены, облачные сервисы, IT-подписки)\n6. Одежда и покупки (одежда, обувь, маркетплейсы, электроника, техника)\n7. Спорт и здоровье (аптеки, медицина, добавки, спортзал, спорттовары)\n8. Обучение и развитие (книги, курсы, софт для учебы, тренинги)\n9. Развлечения и отдых (игры, донаты, кино, бары, досуг с друзьями)\n10. Другое (подарки, переводы людям, комиссии, штрафы и всё неучтенное)\n\nПРАВИЛА ИЗВЛЕЧЕНИЯ:\n1. \"category\": выбери СТРОГО одно точное название из списка выше. Никаких вариаций. Если сомневаешься или категория не очевидна — ставь \"Другое\".\n2. \"amount\": число (тип number, float или integer). Конвертируй сокращения в цифры (например, \"1.5к\" -> 1500, \"2кк\" -> 2000000). Не включай валюту, пробелы и символы. Если суммы нет в тексте — верни null.\n3. \"item\": краткое очищенное название покупки (string) в именительном падеже (например, \"шаурма\", а не \"купил шаурму\"). Если назначение траты не указано — верни \"Расход\".\n4. \"is_valid\": boolean (true, если найдена хотя бы сумма; false, если текст не содержит информации о финансовой операции).\n\nТРЕБОВАНИЕ К ФОРМАТУ ВЫВОДА:\nВерни ИСКЛЮЧИТЕЛЬНО валидный JSON-объект без пояснений, без вводных слов и БЕЗ Markdown-форматирования (НЕ используй тройные кавычки ```json).\nСхема ответа:\n{\n  \"item\": string,\n  \"amount\": number | null,\n  \"category\": string,\n  \"is_valid\": boolean\n}\nТЕКСТ ПОЛЬЗОВАТЕЛЯ:\n{{ $json.message.text }}",
        "hasOutputParser": true,
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.agent",
      "typeVersion": 3.1,
      "position": [
        288,
        -224
      ],
      "id": "d78f355b-6e75-432e-8a59-80e8bffe6c87",
      "name": "AI Agent"
    },
    {
      "parameters": {
        "model": "deepseek-flash",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatDeepSeek",
      "typeVersion": 1,
      "position": [
        256,
        -32
      ],
      "id": "bc55e274-ea10-4dbb-bfb5-7e9b3d4f44b6",
      "name": "DeepSeek Chat Model"
    },
    {
      "parameters": {
        "jsonSchemaExample": "{\n  \"item\": \"Кока-Кола\",\n  \"amount\": 100,\n  \"category\": \"Еда и продукты\",\n  \"is_valid\": true\n}\n",
        "autoFix": true
      },
      "type": "@n8n/n8n-nodes-langchain.outputParserStructured",
      "typeVersion": 1.3,
      "position": [
        464,
        -48
      ],
      "id": "00599f88-d0e9-40c7-82af-f3617257e4ae",
      "name": "Structured Output Parser"
    },
    {
      "parameters": {
        "model": "deepseek-flash",
        "options": {}
      },
      "type": "@n8n/n8n-nodes-langchain.lmChatDeepSeek",
      "typeVersion": 1,
      "position": [
        480,
        112
      ],
      "id": "0e153157-8e59-47b3-8428-e281e7865863",
      "name": "DeepSeek Chat Model1"
    }
  ],
  "pinData": {},
  "connections": {
    "Telegram Trigger": {
      "main": [
        [
          {
            "node": "If",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "If": {
      "main": [
        [
          {
            "node": "AI Agent",
            "type": "main",
            "index": 0
          }
        ],
        [
          {
            "node": "Send a text message",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "DeepSeek Chat Model": {
      "ai_languageModel": [
        [
          {
            "node": "AI Agent",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "Structured Output Parser": {
      "ai_outputParser": [
        [
          {
            "node": "AI Agent",
            "type": "ai_outputParser",
            "index": 0
          }
        ]
      ]
    },
    "DeepSeek Chat Model1": {
      "ai_languageModel": [
        [
          {
            "node": "Structured Output Parser",
            "type": "ai_languageModel",
            "index": 0
          }
        ]
      ]
    },
    "AI Agent": {
      "main": [
        [
          {
            "node": "Create a database page",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  },
  "active": false,
  "settings": {
    "executionOrder": "v1"
  }
}