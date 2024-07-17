# <img src="https://img.hhcdn.ru/employer-logo/6044806.png" alt="Leads" style="background-color: #fff;" />

Задача - провести code review Vue приложения, код которого можно найти по ссылке https://github.com/ComsiComsa/vue2-app-review

Проанализировать код проекта и подготовить отчет, в котором указать:
1. Найденные ошибки, потенциальные проблемы и места для улучшения.
2. Насколько код следует best practices Vue разработки и как его можно оптимизировать.
3. Соответствует ли код стандартам форматирования и соглашениям Vue сообщества.
4. Любые другие комментарии и рекомендации по коду.

---

Reviewer:  Dmitriy Solod\
Contact me: +7(951)612-18-31 / @sodmian

---

## ❗ Критические замечания

1. Использование LocalStorage для хранения данных
2. Не обеспечивающее уникальность значение свойства key в списке элементов грида, как следствие отсутствия идентичности сущности на уровне бизнес-логики
3. Отсутствие разделения среды разработки на develop и production
4. Отсутствие организации автоматической сборки, pre-commit хуков и правил оформления коммита
5. Отсутствие линтеров
6. Отсутствие конфигураций сборщика для обеспечения поддержки предыдущих стандартов ES
7. Не проработана структура приложения, у компонентов отсутствует однозначная зона ответственности
8. Неверный подход к работе с глобальным хранилищем данных
9. Полное отсутствие типизации и модели данных, наличие ошибок связанных с данным подходом
10. Нарушена принятая индентация для языка программирования JS/TS в 2 пробела

## 🔍 Ревью кода

![](https://skrinshoter.ru/s/120724/Uq7vcPS7.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2008:15:29.jpg)

Для начала обновим зависимости проекта

![](https://skrinshoter.ru/s/120724/2Wdj8qtZ.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2008:48:43.jpg)

Вынесем настройку в переменные окружения

![](https://skrinshoter.ru/s/120724/5VvtJspQ.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2008:51:41.jpg)

Добавим действия для управления состоянием через мутации и выполнения асинхронных операций

![](https://skrinshoter.ru/s/120724/JJyMZRRB.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2008:53:58.jpg)

Далее, обратим внимание на следующее:

❌ `<TaskProgress :progress="progress"/>`\
✔️ `<ProgressBar :progress="getProgress"/>`

❌ `<TaskForm @create-new-task="createNewTask($event)"/>`\
✔️ `<Form @submit="handleFormSubmit"/>`

❌ `<TaskGrid @task-delete="taskDelete($event)" @task-toggle="taskToggle($event)" />`\
✔️ `<GridList :data-source="tasks" />`

Директивам `@task-delete` и `@task-toggle` здесь не место, компонент должен быть ответственен только за представление элементов грида.

Добавим ESLint и Prettier, чтобы поддерживать код в чистоте. Значение атрибута `id` тега `div` лишено смысла и дублирует аналогичное значение атрибута в шаблоне `index.html`. Поступим следующим образом, [изучим разделение слоев приложения](https://feature-sliced.design/ru/docs) и перенесем компоненты в отдельный слой `widgets`:

```
<template>
  <TaskWidget />
</template>
```

![](https://skrinshoter.ru/s/120724/dxoG9JBE.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2008:56:11.jpg)

Внедрим методологию БЭМ

![](https://skrinshoter.ru/s/120724/k6QV7nKW.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2016:21:33.jpg)

Заменим данный блок кода на Vuex action `initTasks`, перенесем в него инициализацию данных

![](https://skrinshoter.ru/s/120724/X7st8SIr.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2016:40:12.jpg)

Вычисляемое свойство использует мутацию в сеттере 😨, в объекте присутствуют вложенные объекты 😱

![](https://skrinshoter.ru/s/120724/BLW4mTTV.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2016:59:56.jpg)

Пересмотрим этот подход

![](https://skrinshoter.ru/s/120724/Sbckybvd.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2017:01:20.jpg)

Видим лишние отступы, однако здесь проблемы по серьезнее:

- Управление состоянием переносим в Vuex
- Задача (Task) - сущность, а сущности не помешает уникальный идентификатор и проверка на не пустой `title`. C этим нам поможет `Symbol` и дефолтное значение `title`
- Метод `taskToggle` переименовать в `setDone`. Атрибутом метода передать ссылку на объект, нежели индекс объекта в массиве
- Метод `createNewTask` лучше переименовать в `add`, атрибутом передать объект `task`, вместо строкового значения атрибута `taskTitle`

![](https://skrinshoter.ru/s/120724/KeXJ7G30.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2017:13:00.jpg)

Секции `style` укажем тип используемого препроцессора, глобальные стили перенесем на другой слой приложения, сброс дефолтных значений организуем через подключение файла `reset.css`, цветам, шрифтам и т.д. зададим переменные и вынесем в отдельный файл, подключим библиотеку атомарных классов, например: Tailwind

![](https://skrinshoter.ru/s/120724/Aoy4arle.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2017:20:57.jpg)

Переходим дальше. Не будем повторяться про БЭМ и форматирование кода, сосредоточимся на следующем:

❌ `<div class="task" :class="taskClass"  @click="$emit('task-toggle', task)">`\
✔️
```
<!-- task (task_done) -->
<div :class="['task', getClassModifier]" @click="handleToggle">
```

❌ `<div class="task-delete-button" :class="taskClass" @click.stop="$emit('task-delete', task)">x</div>`\
✔️ `<Button class="task__btn task__btn_destroy" @click="handleDestroy"><Svg name="close" /></Button>`

❌ `<span class="task-text" :class="taskClass">{{ task.title }}</span>`\
✔️ `<span class="task__title">{{ task.title }}</span>`

![](https://skrinshoter.ru/s/120724/UGza0t4W.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2018:28:10.jpg)

Нужно учесть, что вывод значения `task.title` может привести к проблеме, т.к. task может оказаться строкой

![](https://skrinshoter.ru/s/120724/taLDCHHv.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2017:41:57.jpg)

Снова БЭМ, не используются вложенные условия препроцессора

![](https://skrinshoter.ru/s/120724/r012I2vS.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2017:45:48.jpg)

Теперь разберем следующий блок кода:

❌ `e.preventDefault();`\
✔️ Директива `@submit` уже использует модификатор `prevent`

❌
```
if ( this.title != ''){
  this.$emit('create-new-task', this.title);
  this.title = "";
}
```
✔️
```
if (!this.title.length){
  // какой-то код
  return;
}
      
this.$emit('submit', this.title);
resetForm();
```

![](https://skrinshoter.ru/s/120724/eua5TbSU.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2017:55:14.jpg)

Подключим к проекту Stylelint для исключения подобных ситуаций

![](https://skrinshoter.ru/s/120724/9xAnFQEy.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2018:00:03.jpg)

Переходим к последнему компоненту:

- При использовании компонента в качестве виджета с атрибутом `id="task-grid"` будут проблемы, т.к. `id` может оказаться не уникальным, исправляем
- `animation=250` и `fallbackTolerance=3` заменим корректной записью
- `@start="drag=true"` и `@end="drag=false"` перепишем на `@start="() => setDragAndDrop(true)"` и `@end="() => setDragAndDrop(false)"`
- `:key="task.title"` не гарантирует уникальность значения. На предыдущих шагах мы использовали в качестве уникального идентификатора сущности примитивный тип данных `Symbol`, воспользуемся этим решением
- Убираем привязку директив
  ```
  @task-delete="$emit('task-delete', i)"
  @task-toggle="$emit('task-toggle', i)"
  ```
- `tasks.length===0` не нуждается в проверке на тождественное равенство, исправляем
- ![](https://skrinshoter.ru/s/120724/ZctmcpbM.jpg?download=1&name=%D0%A1%D0%BA%D1%80%D0%B8%D0%BD%D1%88%D0%BE%D1%82-12-07-2024%2018:00:27.jpg)

  Подключенный на предыдущих шагах Stylelint обозначит данную проблему
