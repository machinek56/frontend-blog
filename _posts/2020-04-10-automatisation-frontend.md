---
layout: post
title:  "Автоматизация линтинга и форматирования. Как не отправить \"плохой\" код в репозиторий."
date:   2020-04-10 00:00:00 +0300
categories: webpack husky lint-staged
---

Как часто вы пишете команды для запуска линтинга, тестов, форматирования кода? <br>
А как часто пушите лишние коммиты, содержащие исправления по code style проекта? <br>
В команде разработчиков сложно зачастую договориться о правилах форматирования кода,
и на помощь нам приходит известный инструмент *Prettier*, который делает за нас всю грязную работу,
а именно форматирует код согласно единому стилю по одной команде.

В мире фронтенда появились инструменты, позволяющие автоматизировать такие вещи без ручного запуска команд при срабатывании [гит-хуков](https://git-scm.com/docs/githooks).

### Husky

[Инструмент](https://github.com/typicode/husky) для запуска команд при гит-хуках (commit, push).
Для начала установим его:
```bash
npm install -D husky
```
Затем в **package.json** добавим секцию:
```json
"husky": {
   "hooks": {
     "pre-commit": "YOUR_COMMAND_HERE",
     "pre-push": "YOUR_COMMAND_HERE"
   }
}
```

### Lint-staged

Второй [инструмент](https://github.com/okonet/lint-staged), позволяющий выполнять команды только над индексированными файлами.
Установим его:
```bash
npm install -D lint-staged
```
Допишем в **package.json** секцию:
```json
"lint-staged": {
    "*.{ts,tsx,js,css,less,json,md}": [
      "prettier --write"
    ]
}
```

Данная запись означает, что для файлов, чье расширение подходит по маске будет применяться команда `prettier --write`.
С *lint-staged v10.x* больше не нужно писать вторую команду `gid add`, файлы автоматически будут добавлены в индекс.

Мы можем добавлять другие маски файлов и запускать над ними команды.

> Как я понимаю, *lint-staged* проходит только по тем файлам, которые были изменены. В этом его и фишка, что нам не нужно проверять все файлы проекта подходящие под маску.

Итоговая конфигурация этих двух инструментов может выглядеть так:
```json
"husky": {
    "hooks": {
      "pre-commit": "lint-staged",
      "pre-push": "yarn run tslint --fix && yarn run test"
    }
}
```

В этом примере перед пушем производится линтинг кода и запуск тестов.
Если какая-то команда завершится неудачно (EXIT_CODE != 0), то операция *commit* или *push* не будет произведена.

Еще пример с другого проекта:
```json
"husky": {
    "hooks": {
      "pre-commit": "npm run ts-compile-check && lint-staged" // компиляция ts и запуск lint-staged
    }
}
"lint-staged": {
   "*.(js|jsx|ts|tsx)": [
     "npm run lint" // запуск eslint --fix
   ]
}
```

Часто *eslint* и *prettier* используются вместе для автоматического исправления кода.
Их настройку и работу вместе разберем в другом посте.

Таким образом поддерживается консистентность кодовой базы проекта, мы не даем и коммитить и пушить "плохой" код.
И что больше всего радует - это происходит автоматически.
