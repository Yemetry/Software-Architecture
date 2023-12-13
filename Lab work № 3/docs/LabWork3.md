# Диаграмма компонентов:
![image](https://github.com/Yemetry/Software-Architecture/assets/107578601/8a1b74e7-da22-4afd-8f0c-18ee6dd3864b)

![image](https://github.com/Yemetry/Software-Architecture/assets/107578601/b866adc6-4954-4fd5-9fae-f7ef1b607439)

# Модель базы данных

![image](https://github.com/Yemetry/Software-Architecture/assets/107578601/027dd875-1345-498e-bceb-1b3c2a6c1962)

 Краткое пояснение к модели базы данных:

- users: Содержит информацию о пользователях.
- presentations: Представляет собой презентации и ссылается на пользователя, создавшего их.
- slides: Содержит информацию о слайдах презентаций, включая их содержимое и тип (информационный или вопрос).
- questions: Хранит открытые вопросы, связанные с конкретными слайдами.
- quiz_responses: Содержит ответы на вопросы викторины с указанием правильности.
- text_responses: Хранит текстовые ответы на вопросы.
- image_responses: Хранит ссылки на изображения с ответами на вопросы.


# Код с учетом принципов KISS, YAGNI, DRY и SOLID.
```
from fastapi import FastAPI, HTTPException
from fastapi.middleware.cors import CORSMiddleware
from pydantic import BaseModel
from typing import List

app = FastAPI()

# Для простоты примера, используем in-memory базу данных
reports_db = []


class Report(BaseModel):
    user_id: int
    presentation_id: int
    slide_id: int
    response: str
    is_correct: bool


@app.post("/reports", response_model=Report)
async def create_report(report: Report):
    # Логика сохранения отчета в базе данных
    reports_db.append(report)
    return report


@app.get("/reports/{user_id}", response_model=List[Report])
async def get_user_reports(user_id: int):
    # Логика получения отчетов для конкретного пользователя
    user_reports = [report for report in reports_db if report.user_id == user_id]
    if not user_reports:
        raise HTTPException(status_code=404, detail="Reports not found for this user")
    return user_reports


# Добавим middleware для обработки CORS
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"], 
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

# Запуск сервера
if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="127.0.0.1", port=8000) 
```
```
<template>
  <div id="app">
    <h1>User Reports</h1>
    <ul>
      <li v-for="report in reports" :key="report.id">
        User ID: {{ report.user_id }}, Slide ID: {{ report.slide_id }}, Response: {{ report.response }}, Correct: {{ report.is_correct }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      reports: [],
    };
  },
  mounted() {
    // Получаем отчеты пользователя при загрузке компонента
    this.fetchUserReports(1);  // Предполагаем, что пользователь с ID=1
  },
  methods: {
    fetchUserReports(userId) {
      this.$http.get(`/reports/${userId}`)
        .then(response => {
          this.reports = response.data;
        })
        .catch(error => {
          console.error('Error fetching user reports:', error);
        });
    },
  },
};
</script>

<style>
#app {
  text-align: center;
  margin-top: 20px;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  margin: 10px 0;
}
</style>
```

Пояснение:

- KISS (Keep It Simple, Stupid): Код написан просто и понятно. В FastAPI, мы используем минимальный набор инструментов для обработки HTTP-запросов и создания отчетов. В Vue.js, компонент App.vue прост и легко читаем.

- YAGNI (You Ain't Gonna Need It): Мы добавляем только те функции, которые действительно необходимы для текущего функционала. Например, не добавляются сложные механизмы аутентификации или авторизации, если они не требуются в текущем контексте.

- DRY (Don't Repeat Yourself): Код организован так, чтобы избежать повторения. Например, базовый URL для HTTP-запросов в Vue.js определен один раз, и его можно легко изменить в одном месте.

- SOLID: Принципы SOLID следованы в структуре кода FastAPI, где каждая функция выполняет одну задачу, и в Vue.js, где компонент App.vue ответственен только за отображение данных и управление жизненным циклом компонента.

# Иные принципы разработки:

## BDUF (Big Design Up Front)
**Масштабное проектирование прежде всего**
- Применимость: BDUF может быть полезным при разработке больших и сложных систем, где требуется детальное планирование до начала работы. Это может помочь увидеть общую картину и уменьшить возможные проблемы в будущем.
- Отказ: BDUF может быть неприменимым в быстро меняющихся условиях рынка или проекта, так как детальное планирование заранее может оказаться устаревшим или неприменимым при изменении требований. Это также может привести к излишней потере времени на планирование, вместо начала разработки.

## SoC (Separation of Concerns):
**Принцип разделения ответственности**
- Применимость: SoC улучшает поддерживаемость и расширяемость кода, разделяя его на отдельные компоненты или модули, каждый из которых отвечает только за свою конкретную функциональность. Это облегчает понимание кода и его модификацию.
- Отказ: В некоторых случаях строгое соблюдение SoC может привести к излишнему разделению функциональности, что затруднит понимание общей логики программы или увеличит накладные расходы на взаимодействие между компонентами.

## MVP (Minimum Viable Product):
**Минимально жизнеспособный продукт**
- Применимость: MVP полезен при разработке продукта, так как он позволяет быстро вывести его на рынок с минимальным набором функций, необходимым для удовлетворения основных потребностей пользователей. Это позволяет получить обратную связь и определить дальнейшее развитие продукта на основе реальных потребностей пользователей.
- Отказ: Иногда MVP может быть недостаточным для привлечения пользователей или конкурентоспособности продукта на рынке, особенно если минимальный набор функций слишком ограничен для решения проблемы.

## PoC (Proof of Concept):
**Доказательство концепции**
- Применимость: PoC полезен при исследовании новых идей или технологий. Он позволяет демонстрировать осуществимость концепции или технической идеи перед инвестированием времени и ресурсов в полноценную разработку.
- Отказ: PoC может быть неприменимым, если его цель нечетко определена или если фокус слишком сильно смещен на доказательство технической осуществимости, игнорируя реальные потребности рынка или конечных пользователей.
