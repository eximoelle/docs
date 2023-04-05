## Первоначальная настройка

```
git config --global user.name "John Doe" //Ваше имя
git config --global user.email "johndoe@example.com" //Ваш e-mail
git config --global core.editor nano //Текcтовый редактор для git
git config --global merge.tool vimdiff //Утилита сравнения
git config --list //Проверка всех настроек
```

### Настройка SSH-авторизации

[Подключение к GitHub по SSH](https://docs.github.com/ru/authentication/connecting-to-github-with-ssh)

```
❯ ssh-keygen -t ed25519 -C "git:<email>" -f ~/.ssh/id_github
```

Добавить ключ через Github WebUI: Профиль > Settings >  SSH and GPG keys.

В `~/.ssh/config` добавить:
```
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_github
```

Для проверки авторизации можно использовать
```
❯ ssh -T git@github.com
Hi eximoelle! You've successfully authenticated, but GitHub does not provide shell access.
```

### Как изменить тип авторизации на SSH при пуше в репозиторий

If it is asking you for a **username** and password, your origin remote is pointing at the HTTPS URL rather than the SSH URL.

Change it to ssh.

For example, a GitHub project like Git will have an HTTPS URL:

```
https://github.com/<Username>/<Project>.git
```

And the SSH one:

```
git@github.com:<Username>/<Project>.git
```

You can do:

```
git remote set-url origin git@github.com:<Username>/<Project>.git
```

to change the URL.

## Создание репозитория

## Форк

### Создание форка

Создать форк проекта (легче через WebUI).

Клонировать свой форк:
```
❯ git clone git@github.com:eximoelle/my-zdotdir $ZDOTDIR
```

Создается локальная копия форка. Автоматически ему присваивается имя `origin`.

### Настройка веток, окружения

Основа: [Настраиваем Git для правильной работы с опенсорс-проектами](https://proglib.io/p/nastraivaem-git-dlya-pravilnoy-raboty-s-opensors-proektami-2022-09-28)

#### Добавление удаленного upstream-репозитория

Сейчас форк удаленного репозитория `origin` не обновляется автоматически вместе с основным проектом. Если изменения будут слиты в основной репозиторий проекта, клонированный форк не будет знать о них. Это важно для перебазирования PR, чтобы привести его в соответствие с репозиторием основного проекта.

Нужно добавить апстрим, который будет указывать на основной проект:
```
❯ git remote add upstream git@github.com:getantidote/zdotdir.git
❯ git remote -v
origin	git@github.com:eximoelle/my-zdotdir (fetch)
origin	git@github.com:eximoelle/my-zdotdir (push)
upstream	git@github.com:getantidote/zdotdir.git (fetch)
upstream	git@github.com:getantidote/zdotdir.git (push)
```

#### Создание базовой ветки форка

Нужно создать ветку, которая будет использоваться для отслеживания изменений в репозитории. Она не будет использоваться для внесения изменений в код. На основе нее будут перебазироваться ветки разработки и на ее основе будут создаваться новые рабочие ветки.

Базовая ветка форка будет названа `upstream`. Все ветки разработки будут основаны на ней.

Получаем апстрим основного проекта :
```
❯ git fetch upstream
remote: Enumerating objects: 8, done.
remote: Counting objects: 100% (8/8), done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 8 (delta 3), reused 8 (delta 3), pack-reused 0
Unpacking objects: 100% (8/8), 1.19 KiB | 122.00 KiB/s, done.
From github.com:getantidote/zdotdir
 * [new branch]      advanced                -> upstream/advanced
 * [new branch]      demo-antidote-issue-100 -> upstream/demo-antidote-issue-100
 * [new branch]      main                    -> upstream/main
```

Создаем свою ветку `upstream`, которая будет отражать состояние оригинального репозитория:
```
❯ git checkout -b upstream upstream/main
branch 'upstream' set up to track 'upstream/main'.
Switched to a new branch 'upstream'
```

Получаем изменения (которых пока нет):
```
❯ git push origin upstream:refs/heads/main
Everything up-to-date
```

### Workflow

Есть базовая ветка и можно приступать к изменению кода. Этот workflow направлен на внесение изменений, отправку PR, а при необходимости на обновление этого PR.

#### Создание рабочей ветки

От ветки `upstream` будут идти рабочие ветки.

Убедимся, что мы в ветке `upstream`:
```
❯ git checkout upstream
Already on 'upstream'
Your branch is up to date with 'upstream/main'.

❯ git pull --rebase
Already up to date.
```

Создаем и переключаемся на рабочую ветку `refining`:
```
❯ git checkout -b refining
Switched to a new branch 'refining'
```

Мы находимся в ветке `refining` и все дальнейшие изменения форка будут поступать в эту ветку.

#### Коммит

Если были созданы новые файлы, их нужно добавить в отслеживаемые командой `git add <file/directory>`.

Проверить, какие файлы подготовлены для коммита — `git status`.

Удалить файл из коммита — `git rm file`.

Сам коммит делается командой `git commit` с опциями `-s` (?) и `-m <коммент>` для коротких подписей о том, что изменено в этом коммите.

#### Отправка изменений в удаленный репозиторий форка

```
❯ git push origin refining
```

Если нужно сделать ре-пуш в удаленный репозиторий после внесения изменений в уже отправленный коммит, используется опция `-f` (force push). Это опасная опция и при неправильном использовании она может переписать историю коммитов в удаленном репозитории, но в данном случае она необходима чтобы внести изменения в старый коммит и намеренно переписать историю, чтобы включить последние правки.

[Как правильно писать коммиты, чтобы всем было хорошо](https://proglib.io/p/kak-pravilno-pisat-soobshcheniya-kommitov-v-git-chtoby-vsem-bylo-horosho-2022-08-11)