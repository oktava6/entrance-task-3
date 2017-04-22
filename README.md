## Entrance task 3 💪
[Задание 3](https://academy.yandex.ru/events/frontend/shri_msk-2017/)

## Ход мыслей
В тексте задания нашел упоминание о ServiceWorkers, с которыми раньше не сталкивался. Почитав про них на MDN и Хабре, а также опробовав их в домашних проектах, приступил к изучению приложения из задания.  

Форкнул репозиторий, склонировал на компьютер. В readme прочитал, что входной точной является gifs.html. Запустил в директории проекта http-сервер и обратился к этому файлу в браузере. В результате увидел работоспособное приложение, которое умеет все, что написано в тексте задания, кроме работы без подключения к сети: с галочкой offline в devtools браузер показывал динозавра.  

Приступил к изучению исходного кода. В файле blocks.js обнаружил использование kv-keeper, jQuery и bem-components, с которым разбирался довольно долго из-за непривычного подхода к определению блоков. В блоке service-worker нашел регистрацию ServiceWorker, который находится в файле ./assets/service-worker.js. Начал его смотреть и сразу увидел следующую строчку:
```
importScripts('../vendor/kv-keeper.js-1.0.4/kv-keeper.js');
```
В [статье](https://developer.mozilla.org/ru/docs/Web/API/Service_Worker_API/Using_Service_Workers) на MDN было написано: "Максимальная видимость scope сервис-воркера равна его location". Это значит, что ServiceWorker не сможет получить доступ к ресурсам, находящимся выше его области видимости, а именно к "../vendor". Возможно, до рефакторинга под названием "Разложить файлы красиво" ServiceWorker находился в корневой директории.  

Переместил ServiceWorker в корень проекта, изменил все относительные пути и перезагрузил страницу с очисткой кэша. В результате получил сообщение в консоли об успешной установке и активации ServiceWorker-а. Приложение научилось в offline брать ресурсы из кэша:  

![Screenshot](https://github.com/oktava6/imhonet_tasks/blob/master/task2/inet.png)

Правда, осталось ограничение: файлы кэшировались только после второй загрузки страницы. Это происходит из-за того, что ServiceWorker добавляет обработчик события [fetch](https://developer.mozilla.org/ru/docs/Web/API/FetchEvent) уже после того, как все ресурсы приложения загружены. После повторной загрузки страницы обработчик отработает на каждом обращении к ресурсам и добавит необходимые файлы в кэш.  

Самое простое и логичное решение этой проблемы - закэшировать ресурсы в момент установки ServiceWorker-а с помощью метода [cache.addAll()](https://developer.mozilla.org/ru/docs/Web/API/Cache/addAll). Он принимает на вход массив URLs, получает данные и сохраняет их в кэш.  

## Запуск
$ npm i http-server  
$ http-server  
-> localhost:8080/gifs.html
