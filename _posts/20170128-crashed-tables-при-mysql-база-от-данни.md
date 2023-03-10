---
title: 'Crashed tables при MySQL база от данни'
excerpt: 'Много често се случва да има проблеми със сървърите, за които никой и нищо не знае. Редовният отговор е „Аз не съм го пипал„, „То само се счупи„, „‘То така си беше„. Нещата никога не се случват сами. Не случайно съществува Нютоновия закон „Всяко действие има равно по големина и противоположно по посока противодействие.'
coverImage: '/assets/blog/hello-world/pcvt-cover.jpg'
date: '2017-01-28T05:35:07.322Z'
author:
  name: Любомир Филипов
  picture: '/assets/blog/authors/avatar.png'
ogImage:
  url: '/assets/blog/hello-world/pcvt-cover.jpg'
category: DevOps
---

Много често се случва да има проблеми със сървърите, за които никой и нищо не знае. Редовният отговор е „Аз не съм го пипал„, „То само се счупи„, „‘То така си беше„. Нещата никога не се случват сами. Не случайно съществува Нютоновия закон „Всяко действие има равно по големина и противоположно по посока противодействие.“

Един от проблемите, които  може да срещнете е: „/myDatabaseName/my_awesome_table’ is marked as crashed and last (automatic?) repair failed„. Причината да виждате тази грешка е, че таблицата се е „счупила“, а MySQL автоматично не може да я поправи.

Ако изпаднете в ситуация, когато се налага сами да се справяте с проблема, а няма системен администратор или DevOps guru наблизо може да следвате тези няколко стъпки.

Проверете дали сървърът работи като проверката при Debian базирана система става така:
service mysql status

Ще получите съобщение, което ще ви посочи дали сървърът работи. Следващото нещо, което е добра практиката, е да проверите логовете за грешки. За Unix базирана система те се намират най-често тук:

tail /var/log/mysql/error.log –lines 100

Запазете резултата от командата някъде(Все пак е добре да се погледне и от специалист, ако не сте сигурни какво се случва).

Ако сървърът работи, трябва да го спрете:

service mysql stop

Когато се уверите, че сървърът е вече спрян трябва да намерим къде са файловете от базата данни. Данните, които са организирани при вас в таблици се пазят във файлове на твърдия диск като за всяка таблица има файл. По принцип те се намират във /var/lib/mysql.

Отворете папката, в която се намира файла, свързан с таблицата:

cd /var/lib/mysql/DB_NAME

Ако имате проблем с достъпване на директорията се уверете, че сте влезли като root потребител.

Покажете всички файлове в директорията:

ls -al

В списъка от файлове трябва да виждате: my_awesome_table.MYI

Ако потърсите информация в интернет относно грешката, свързана със „счупена“ таблица – най-често ще намерите препоръка да се изпълни REPAIR команда. Според официалната документация се препоръчва да се изпълни също  тази команда, но от личен опит препоръчвам да изпълните, ако е възможно myisamchk, защото REPAIR отнема изключитлено много време и не винаги завършва успешно.

myisamchk -r -o my_awesome_table.MYI

Ще поправи автоматично данните в таблицата като процесът ще отнеме много по-малко време, отколкото ако се изпълни REPAIR в MySQL конзола.

Ако горната команда ви връща грешка, проверете дали има: my_awesome_table.TMD файл, който трябва да бъде изтрит. Може да гледате на .TMD файла като работен файл, който е необходим, когато се поправя таблица. След като изтриете файла, стартирайте отново myisamchk -r -o my_awesome_table.MYI

След като успешно приключи изпълнението на командата, трябва отново да се стартира сървърът:

service mysql start

Понякога можем и сами да се справим с проблемите, които са възникнали, но винаги препоръчвам да се допитваме до специалистите.
