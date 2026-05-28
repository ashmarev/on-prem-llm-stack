# on-prem-llm-stack
 
**Локальный LLM-стек для корпоративной среды · On-premises LLM stack for enterprise**
 
Практические материалы по развёртыванию LLM без облака — для сред с требованиями
к периметру, регуляторике и критической информационной инфраструктуре.
 
Practical guides for deploying LLMs without cloud dependency — for environments
with perimeter requirements, regulatory constraints, and critical infrastructure.
 
---
 
## Содержимое · Contents
 
| Раздел | Описание | Статус |
|---|---|---|
| [hardware/rtx-2080ti](./hardware/rtx-2080ti/) | vLLM на RTX 2080 Ti · vLLM on RTX 2080 Ti | ✅ Ready |
| hardware/rtx-3060 | vLLM на RTX 3060 · vLLM on RTX 3060 | 🔜 Coming |
| [hardware/rtx-4070](./hardware/rtx-4070/) | vLLM на RTX 4070 · vLLM on RTX 4070 | ✅ Ready |
| hardware/rtx-5090 | vLLM на RTX 5090 · vLLM on RTX 5090 | 🔜 Coming |
| proxy/ | LLM-прокси: роутинг, rate limiting, логирование · LLM proxy: routing, rate limiting, logging | 🔜 Coming |
| agents/ | Агентные архитектуры, tool call, audit trail · Agent architectures, tool call, audit trail | 🔜 Coming |
| auth/ | Авторизация между агентами, token exchange · Inter-agent authorization, token exchange | 🔜 Coming |
 
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
 
## Железо · Hardware
 
Репозиторий строится на потребительских GPU — доступных большинству команд
без enterprise-бюджета на A100/H100.
 
Built on consumer GPUs — accessible to most teams without enterprise-grade A100/H100 budget.
 
| GPU | VRAM | Модели / Models |
|---|---|---|
| RTX 2080 Ti | 11 GiB | Qwen3-4B-AWQ и аналоги до 7B |
| RTX 3060 | 12 GiB | Qwen3-4B-AWQ и аналоги до 7B |
| RTX 4070 | 12 GiB | Qwen3-4B-AWQ и аналоги до 7B |
| RTX 5090 | 32 GiB | модели до 32B |

---
 
## Стек · Stack
 
- **Inference:** [vLLM](https://github.com/vllm-project/vllm)
- **Quantization:** AWQ / awq_marlin
- **OS:** Ubuntu 24.04 LTS
- **Proxy:** coming — llm_proxy поверх vLLM / llm_proxy on top of vLLM
 
---
 
## Связанные материалы · Related
 
- Хабр / Habr — статья в процессе · article in progress
  
---
 
## Автор · Author
 
**Артём Шмарёв · Artem Shmarev**  
Data Governance & AI Architect · Lab Head · Unicon BSL / Herzen Universit
[linkedin.com/in/ashmarev](https://linkedin.com/in/ashmarev) 
[https://github.com/ashmarev](https://github.com/ashmarev)
