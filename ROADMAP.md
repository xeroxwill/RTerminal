# RTerminal — дорожная карта форка

## Основа и текущий статус

RTerminal начинается с исходного Android-приложения [Termux](https://github.com/termux/termux-app), зафиксированного на upstream-коммите `8629e632fcb95da272221be327db653fb24befe9`. Исходный проект отдельно указывает, что устанавливаемые через `pkg` пакеты живут в [`termux-packages`](https://github.com/termux/termux-packages). Лицензия приложения — GPLv3-only; исходные лицензионные файлы и уведомления сохранены.

Начальный технический ребрендинг выполнен в исходниках:

- Android `applicationId`: `com.reloot.rterminal`; Java/Gradle namespace намеренно остаётся `com.termux`.
- Обновлены метка приложения, Android authorities/permissions, внутренний prefix path, shortcut targets и отдельные package-derived константы.
- Сменён launcher icon; добавлены legacy-density и adaptive варианты.
- Для release Gradle по умолчанию формирует ABI-specific APKs и universal APK. Устройство должно получать сборку своей архитектуры; фактический размер APK ещё не измерен.
- **Сборка пока намеренно блокируется**, если не предоставлены локальные bootstrap ZIP и SHA-256 для всех ABI. Официальные архивы Termux зашивают путь `/data/data/com.termux/files/usr`, поэтому их нельзя безопасно использовать для RTerminal. Свой bootstrap и репозиторий `pkg` ещё не собраны.

Этот исходник пока нельзя считать готовым или проверенным APK. Совместимость с существующими Termux plugins также не заявляется: их package IDs и подписи остаются upstream.

## Цели продукта

- **Самостоятельное Android-приложение.** Проверить новый application ID, разрешения, authorities и запуск shell после подготовки своего bootstrap.
- **Рабочий `pkg`.** Подготовить fork пакетных build-рецептов, bootstrap под `/data/data/com.reloot.rterminal/files/usr` и совместимый подписанный репозиторий бинарных пакетов. Одного изменения ID в Android-проекте недостаточно; bootstrap и бинарные пакеты должны быть собраны под новый prefix. См. [forking notes Termux](https://github.com/termux/termux-app#forking) и [руководство сопровождающего пакеты](https://github.com/termux/termux-packages/wiki/For-maintainers).
- **Стабильный Python.** Закреплять поддерживаемую версию по релизам, проверить стандартные модули, обновление и установку пакетов через `pkg`. Решить, включать ли Python в первичную установку или предлагать установку позже: это влияет на начальный размер.
- **GUI-библиотеки.** Проверять Tkinter, Dear PyGui (в запросе — PyDearGui), PyQt5, Qt, SDL2 и другие выбранные библиотеки вместе с графической сессией, зависимостями и устройством. Установка библиотеки сама по себе не гарантирует отображение GUI на Android.
- **Стабильность и память.** Сравнивать воспроизводимые замеры памяти и процессов на одинаковых устройствах и версиях Android; снижать накладные расходы форка, не обещая управлять системными ограничениями Android.
- **Место на диске.** Измерять отдельно APK, bootstrap, устанавливаемые пакеты, apt-кэш и временные файлы. Заявленная цель 20 ГБ → 10 ГБ пока не подтверждена: набор пакетов и метод измерения не заданы.

## План измерения и оптимизации размера

На одинаковом устройстве, Android-версии, ABI и наборе пакетов сравнить исходный Termux и RTerminal: размер выбранного APK; объём после первого запуска; размер расширенного `$PREFIX`; список и размеры пакетов; кэш до и после очистки. Сначала сравнить universal и ABI-specific APK. Затем собрать базовый bootstrap и оценить осторожное исключение только необязательных convenience-пакетов, сохранив `bash`, `apt`/`dpkg`, `termux-tools`, `termux-core`, `termux-exec` и их зависимости. Проверить чистую установку, `pkg update`, `pkg install` и `pkg upgrade` на устройстве. Не удалять вручную файлы из уже установленных пакетов и не обещать экономию до измерений.

## Этапы

1. **Брендинг и application ID:** исходники обновлены; остаются проверка Android manifest merge, установки рядом с Termux и запуска на устройстве после создания bootstrap.
2. **Пакетный слой:** подготовить fork `termux-packages`, задать `TERMUX_APP__PACKAGE_NAME=com.reloot.rterminal`, оценить build system и собрать bootstrap для поддерживаемых ABI.
3. **`pkg` и обновления:** подготовить свой подписанный APT-репозиторий и проверить обновление и установку пакетов.
4. **Python и GUI:** закрепить Python, составить матрицу поддерживаемых GUI-пакетов и проверить их на устройстве.
5. **Стабильность и ресурсы:** получить baseline, устранять найденные узкие места и публиковать сравнение по воспроизводимой методике.
6. **Релиз:** не публиковать APK до сборки/проверки bootstrap, установки, `pkg`-smoke-тестов и проверки поддерживаемых архитектур.

## Источники

- [Termux application source and forking notes](https://github.com/termux/termux-app)
- [Termux package build system](https://github.com/termux/termux-packages)
- [Termux package maintainer guide: bootstrap and custom package names](https://github.com/termux/termux-packages/wiki/For-maintainers)
