# Перевод документации Laravel

Этот репозиторий содержит русскоязычный перевод документации фреймворка Laravel. 
Он не создает веб-сайт laravel.su/docs только содержит только контент для него.

Каждая ветка соответствует определенной версии фреймворка, таким образом, для внесения изменений в документацию,
относящуюся к Laravel 11.x, следует перейти в ветку `11.x`.

> Если вы только знакомитесь, то удобнее будет читать материалы на сайте [laravel.su](https://laravel.su/docs).

**Проверка актуальности документации:**

Для удобства использования и внесения своевременных обновлений мы отслеживаем и отображаем текущее состояние каждого
раздела документации на странице [Статус перевода](https://laravel.su/status), что позволяет постоянно быть в курсе
последних обновлений.

## Процесс внесения вклада

Мы рады каждому, кто желает помочь улучшить и актуализировать перевод документации.
Независимо от того, хотите ли вы исправить опечатку или дополнить перевод новым разделом, ваш вклад всегда
приветствуется!

### Исправление мелких недочетов

Для исправления опечаток, неточностей или некорректных терминов не нужно быть экспертом в `git`, достаточно следующего:

1. Найдите файл, который требует корректировки.
2. Нажмите на иконку карандаша в правом верхнем углу блока файла, чтобы начать редактирование.
3. Внесите нужные изменения и предложите **Pull Request** с вашими правками, следуя инструкциям GitHub.

Если отсутствует раздел, который есть в английской версии, потребуется более формальный подход к переводу и добавлению
материалов который описан на
странице [Как правильно переводить документацию Laravel](https://laravel.su/articles/rus-documentation-contribution-guide)

**Важно:** Все изменения должны следовать принятым стандартам качества перевода и будут проверяться перед публикацией.

## Стандарты перевода

Перевод документации — ответственная работа, и мы стремимся обеспечить высокое качество материалов. Пожалуйста,
ознакомьтесь со следующими принципами:

### Согласованность

Следите за тем, чтобы термины и фразы использовались единообразно по всему переводу.
<details><summary>Например</summary>

❌ **Плохой перевод:**
> Ввод новой записи в базу данных может быть совершен с помощью Eloquent моделей. Для обновления записи следует
> использовать метод `save`.

✅ **Хороший перевод:**
> Добавление новой записи в базу данных осуществляется через Eloquent модели. Для обновления существующей записи также
> используется метод `save`.


*Плохой перевод демонстрирует использование разных терминов ("ввод" и "обновление") для обозначения схожих операций, что
может сбивать с толку. Хороший перевод использует согласованный язык, делая текст более понятным.*
</details>


### Понятность и доступность

Ваш перевод должен быть понятен широкой аудитории. Старайтесь использовать простые и четкие формулировки. 
<details><summary>Например</summary>

❌ **Плохой перевод:**
> Фасады обеспечивают статичный интерфейс к классам, которые доступны в сервис-контейнере приложения.

✅ **Хороший перевод:**
> Фасады предоставляют удобный способ обращения к классам из контейнера служб Laravel, используя простой синтаксис, как
> при работе со статическими методами.

*Плохой перевод не дает читателю понимания зачем используются фасады и какой у них интерфейс. Хороший перевод доступным
языком объясняет преимущества использования фасадов.*
</details>

### Избегайте дословного перевода

Внимательно относитесь к контексту, чтобы избежать ошибок из-за неучтенных нюансов языка.
<details><summary>Например</summary>

❌ **Плохой перевод:**
> Если вы приведены когортой методов маршрутизации замысловатым образом, вы можете использовать контроллеры.

✅ **Хороший перевод:**
> Если ваши маршруты становятся сложными из-за большого количества логики обработки, стоит использовать контроллеры.

*Плохой перевод кажется неестественным и трудно понимается, в то время как хороший перевод корректно и понятно передает
суть предложения, сохраняя идиоматичность русского языка.*
</details>

### Точность терминологии

Техническая точность крайне важна, не допускайте искажения смысла из-за неправильного перевода терминов и конструкций.
<details><summary>Например</summary>

❌ **Плохой перевод:**
> Middleware это середина между запросом и ответом.

✅ **Хороший перевод:**
> Посредник (Middleware) — программный компонент, который выступает в роли фильтра между полученным запросом и
> отправляемым ответом.


*В плохом переводе термин "Middleware" некорректно интерпретирован, что затрудняет понимание его функции. Хороший
перевод точно объясняет роль Middleware в контексте запросов и ответов веб-приложения.*
</details>

## Обратная связь и взаимодействие с сообществом

Если у вас есть предложения или вы столкнулись с трудностями, не стесняйтесь открывать **issue** в репозитории или
комментировать существующие.
Мы стараемся оперативно реагировать на обратную связь и совершенствовать наши материалы.

---

**Станьте ценной частью сообщества Laravel, начинайте изучение фреймворка с помощью качественно переведенной
документации!**
