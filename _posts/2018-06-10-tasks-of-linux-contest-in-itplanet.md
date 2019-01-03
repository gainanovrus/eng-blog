---
published: true
layout: single
title:  "Tasks of Linux contest in IT-Planet"
excerpt: >-
  IT-Planet is an international olympiad for students and young specialists in IT. 
  I participated in the contest of Linux Administration and I became one of the winners. 
  The tasks that presented below have been in the final of the contest. 
categories: linux
tags: itplanet contest centos linux
toc: true
header:
  teaser: /assets/images/tasks-of-linux-contest-in-itplanet-teaser.jpg
  og_image: /assets/images/tasks-of-linux-contest-in-itplanet-teaser.jpg
---

**IT-Planet** is an international olympiad for students and young specialists in IT. 
I participated in the contest of Linux Administration and I became one of the winners. 
The tasks that presented below have been in the final of the contest. 
{: .notice--info}

[«IT-Планета»][itplanet] — одно из самых масштабных состязаний в области информационных технологий, учрежденных в России. 
Учредителями конкурсов являются ведущие российские и международные ИТ-компании: 1С, Huawei, Cisco, Oracle, ГНУ/Линуксцентр и СКБ Контур. 
Олимпиада проводится по нескольким направлениям. 

Я участвовал в конкурсе по номинации "Администрирование Linux", прошел несколько отборочных этапов и стали одним из победителей в финальных соревнованиях.
И здесь решил выложить сами задания финала, которые показались мне достаточно интересными. Если найду время - выложу и решения.

---

{% capture notice-1 %}
В нашем распоряжении была машина с OC Centos 7 минимальной конфигурации, подключенная физически к интернету. Необходимо было:
  * авторизоваться в системе, взломав пароль пользователя root
  * починить сломанные настройки сети
  * найти веб-сервер в локальной сети, на котором лежат задания
  * скачать текстовый файл с заданиями и начать их решать
{% endcapture %}

<div class="notice--info">{{ notice-1 | markdownify }}</div>

---

### Задания финала конкурса "Администрирование Linux"

{% capture notice-2 %}
#### Представиться в /root/motd

* Не забывайте, что ваша рабочая станция после перезапуска должна автоматически
 подключиться к сети.
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

1. Нужно создать раздел на диске размером 1000MB, примонтировать в /data/
   * Диск должен монтироваться при старте системы
   * Должны быть приняты меры на случай смены имени диска

2. Завести пользователей orta, atmos, hexen, plyaski, ai
   * orta, atmos должны иметь доступ read в /data
   * ai и plyaski rw доступ
   * Чтобы избежать переполнения раздела, нужно выставить квоту на запись для
     юзеров ai и plyaski, soft 7000kb, hard 8000kb

3. Настроить nfs-server
   * Каталог /var/nfs должен быть доступен для монтирования из локальной сети
   * root доступ должен быть отключен
   * пользовательская авторизация включена
   * firewall должен работать

4. Настроить кеширующий dns-сервер unbound на локальную сеть.
   * В качестве forward-zone использовать 8.8.8.8 / 8.8.4.4
   * запросы должны приниматься только из подсети 192.168.171.0/24

5. Юзеру hexen ограничить использование CPU до 10% а MEM до 128MB
   * проверка будет производиться приложением stress/stress-ng

6. Запустить в контейнере (docker/lxc) socks5-прокси сервер с авторизацией по
   логин:пароль (it:planet), порт 1080
   * Проверить работоспособность прокси при помощи curl/wget/aria2/etc

7. Установить и настроить prometheus для сбора метрик с рабочей станции
   сервис должен работать на стандартном порту (9090), отдавать базовую
   информацию о машине, быть доступным из локальной сети.

8. На сервере с задачами находится файл [answers.txt][answers-txt]
   Содержит он вопросы восстановления паролей у пользователей почтового
   сервиса. Для удобства обработки списка нужно написать скрипт, который
   * Забирает список с сервера
   * На лету преобразовывает его в json 
     (реализация принимается на bash, python, готовый скрипт положить в /root/answer_to_json.py,{sh})
   * json сохранить в /root/result.json
   * Бонусный балл можно заработать отфильтровав наиболее редкие вопросы восстановления
     (результат положить в /root/answers_filtered.txt)

9. На сервере с заданиями можно обнаружить странный архив [botnet.tar.gz][botnet-tar-gz] с бинарниками,
   происхождение и назначение которых вам неизвестны. После краткого
   расследования выяснилось, что злоумышленник проник в систему, потому что у
   одного из пользователей логин совпадал с паролем. Есть подозрение, что это
   часть какого-то ботнета. Задача собрать как можно больше информации о
   бинарном файле, о том что делает программа, при возможности деанонимизировать автора.
   Информацию о файле и свои мысли записать в /root/botnet.answer

   Все манипуляции с файлами из архива с ботнетом производить в контейнере. Выполнение на хост-системе ведет
   к дисквалификации.


[itplanet]:http://world-it-planet.org/
[answers-txt]:{{ site.baseurl }}/assets/files/itplanet-linux/answers.txt
[botnet-tar-gz]:{{ site.baseurl }}/assets/files/itplanet-linux/botnet.tar.gz