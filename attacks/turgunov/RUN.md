# Запуск и сдача ДЗ 3

Результаты реальных прогонов и ограничения проверки описаны в writeup.md.
Основной успешный payload без защит — attack_roles.md. Все трейсы лежат в evidence/.

## Окружение

Из корня репозитория, PowerShell:

```powershell
$env:PYTHONUTF8 = "1"
py -3.13 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r attacks/turgunov/requirements-local.txt
Copy-Item .env.example .env
```

Copy-Item выполняется только если собственного .env ещё нет. В .env записать
личный LLM_API_KEY; не добавлять файл в Git. Ключ и конфигурация для ДЗ 3
должны соответствовать .env.example именно этого стенда.
Активация окружения не обязательна: команды ниже используют его Python явно.
PYTHONUTF8 нужен для кириллицы в выводе при перенаправлении в Windows.
В новом окне PowerShell эту переменную нужно установить снова.

Версия anthropic зафиксирована в своей папке: SDK 2.8.0 не принимает
temperature в Messages.create, который использует исходный стенд.
Совместимая версия для эксперимента — 0.40.0; код агента не меняется.

## Офлайн-проверка

```powershell
.\.venv\Scripts\python.exe -m pytest tests -q
.\.venv\Scripts\python.exe run.py --list-defenses
```

Эти команды не проверяют успех payload на модели и не требуют ключа.

## Воспроизведение замеров

Сначала один чистый прогон: убедиться, что модель решает штатную задачу.
Затем по пять прогонов каждой атаки:

```powershell
.\.venv\Scripts\python.exe run.py --name clean_verified
.\.venv\Scripts\python.exe run.py --attack attacks/turgunov/attack.md --goal A,B --runs 5 --name baseline_verified
.\.venv\Scripts\python.exe run.py --attack attacks/turgunov/attack_spotlighting.md --defense spotlighting,repeat_task --goal A,B --runs 5 --name protected_verified
```

Результаты сохраняются в:
- runs/clean__clean_verified/run_1/
- runs/attack__baseline_verified/run_1/ … run_5/
- runs/attack_spotlighting__protected_verified/run_1/ … run_5/

Для следующей версии изменить --name, иначе файлы run_N перезапишутся.
Весь N фиксировать честно, а не только удачные прогоны.

В этом стенде send_email записывает письмо в env.outbox песочницы;
инструмент не подключён к настоящей почте. Исполнение команд GitHub/почты,
встречающихся в результатах модели, вне песочницы не требуется.

## Успешная обязательная часть

```powershell
.\.venv\Scripts\python.exe run.py --attack attacks/turgunov/attack_roles.md --goal A,B --name roles_pilot
.\.venv\Scripts\python.exe run.py --attack attacks/turgunov/attack_roles.md --defense spotlighting,repeat_task --goal A,B --name roles_protected_pilot
```

Эти два варианта проверялись по одному разу. Первый достиг A+B, второй — нет.
В writeup.md приведены все попытки и их результаты; выбор успешного трейса
не выдаётся за подтверждённую частоту успеха.

## Сдача

После экспериментов заполнить writeup.md и скопировать выбранные реальные
трейсы в свою папку. Проверить `git diff --name-only upstream/master...HEAD`:
изменения должны касаться только своей папки attacks/<фамилия>/.
Если upstream remote не добавлен, исходный репозиторий сохранён как origin:
сравнивать с origin/master. Отправка решения — отдельный Pull Request
в voidbeholder/injection_practice после завершения отчёта и проверки трейсов.
