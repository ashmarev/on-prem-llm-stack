# on-prem-llm-stack
 
**Локальный LLM-стек для корпоративной среды · On-premises LLM stack for enterprise**
 
Практические материалы по развёртыванию LLM без облака — для сред с требованиями
к периметру, регуляторике и критической информационной инфраструктуре.
 
Practical guides for deploying LLMs without cloud dependency — for environments
with perimeter requirements, regulatory constraints, and critical infrastructure.
 
---
 
## Зачем · Why on-prem
 
Облачный LLM — не вариант если:
 
- данные не покидают периметр (187-ФЗ, КИИ, внутренние политики)
- нужен контроль над моделью, версией и поведением
- требуется audit trail для регулятора или внутренней безопасности
  
Cloud LLM is not an option when:
 
- data must not leave the perimeter (regulatory, internal policy)
- you need control over model, version, and behavior
- audit trail is required for compliance or security
---
 
## Содержимое · Contents
 
| Этап | Описание | Статус |
|---|---|---|
| [01-inference](./01-inference/) | Локальный инференс: выбор модели, железо, vLLM · Local inference: model selection, hardware, vLLM | ✅ Ready |
| [02-proxy](./02-proxy/) | Прокси: роутинг, rate limiting, логирование · Proxy: routing, rate limiting, logging | 🔜 Coming |
| [03-agents](./03-agents/) | Агентные архитектуры, tool call, audit trail · Agent architectures, tool call, audit trail | 🔜 Coming |
| [04-auth](./04-auth/) | Авторизация агентов, token exchange · Inter-agent authorization, token exchange | 🔜 Coming |
  
---
 
## Связанные материалы · Related
 
- Хабр / Habr — статья в процессе · 🔜 Coming
  
---
 
## Автор · Author
 
**Артём Шмарёв · Artem Shmarev**  
Data Governance & AI Architect ·  Unicon BSL  
Lab Head · Herzen University  
[linkedin.com/in/ashmarev](https://linkedin.com/in/ashmarev)  
