%{
  version: "2.0.4",
  title: "Distillery (Basics)",
  excerpt: """
  Distillery - это менеджер релизов, написанный на чистом Elixir.
  Он позволяет создавать релизы, которые можно развернуть в другом месте, практически без настройки.
  """
}
---

## Что такое релиз?

Релиз - это пакет, содержащий ваш скомпилированный Erlang/Elixir код (например, [BEAM](https://en.wikipedia.org/wiki/BEAM_(Erlang_virtual_machine)) [байт-код] (https://ru.wikipedia.org/wiki/%D0%91%D0%B0%D0%B9%D1%82-%D0%BA%D0%BE%D0%B4)).
Он также предоставляет любые скрипты, необходимые для запуска вашего приложения.

После создания одного или нескольких приложений у Вас может возникнуть желание создать полноценный набор программного обеспечения, включающий эти приложения и набор других приложений Erlang/OTP. Это и называется релизом. - [Erlang documentation](http://erlang.org/doc/design_principles/release_structure.html)

Резилы дают возможность упрощенной доставки ПО: они не требуют никаких сторонних зависимостей и предоставляют всё необходимое для своего запуска. Ими легко управлять при помощи поставляемых вместе с ними шелл-скриптов: запуск удалённого терминала, пуск/останов/перезапуск релиза, запуск в фоновом режиме, отправление дистанционных команд и многое другое. Релизы это архивируемые артефакты сборки, что означает, что Вы, имея его исходный архив (tarball), можете восстановить старый релиз в любое время, если возникнут проблемы совместимости с ОС или системными библиотеками. Использование релизов открывает Вам доступ к одной из самых мощных особенностей Erlang VM: установка/откат обновления (апгрейд/даунгрейд) на горячую. - [Distillery Documentation](https://hexdocs.pm/distillery/introduction/understanding_releases.html) 

Релиз будет содержать следующее:
* папка /bin
   * Содержит сценарий, который является отправной точкой для запуска всего вашего приложения.
* папка /lib
   * Содержит скомпилированный байт-код приложения вместе с любыми зависимостями.
* папка /Release
   * Содержит метаданные о выпуске, а также хуки и пользовательские команды.
* a /erts-VERSION
   * Содержит среду выполнения Erlang, которая позволит машине запускать ваше приложение без установленного Erlang или Elixir.


### Начало работы/установка

Для того, чтобы добавить менеджер релизов Distillery в ваш проект, укажите его в качестве зависимости в файле `mix.exs`.
*Примечание* - если вы работаете с зонтичным проектор, то это должен быть файл mix.exs в корневом каталоге вашего проекта.

```elixir
defp deps do
  [{:distillery, "~> 2.0"}]
end
```

Затем в терминале выполните:

```shell
mix deps.get
mix compile
```


### Создание релиза

В терминале выполните

```shell
mix release.init
```

Эта команда создаст директорию `rel` с файлами конфигурации в ней.

Чтобы сгенерировать релиз в вашем терминале, запустите `mix release`.

После того, как релиз будет собран, вы должны увидеть некоторые инструкции в своем терминале.

```
==> Assembling release..
==> Building release book_app:0.1.0 using environment dev
==> You have set dev_mode to true, skipping archival phase
Release successfully built!
To start the release you have built, you can use one of the following tasks:

    # start a shell, like 'iex -S mix'
    > _build/dev/rel/book_app/bin/book_app console

    # start in the foreground, like 'mix run --no-halt'
    > _build/dev/rel/book_app/bin/book_app foreground

    # start in the background, must be stopped with the 'stop' command
    > _build/dev/rel/book_app/bin/book_app start

Если это дистанционный релиз, и вы хотите к нему подключиться:

    # connects a local shell to the running node
    > _build/dev/rel/book_app/bin/book_app remote_console

    # connects directly to the running node's console
    > _build/dev/rel/book_app/bin/book_app attach

Для получения полного перечня команд:

    > _build/dev/rel/book_app/bin/book_app help
```

Для запуска приложения введите это в терминале: ` _build/dev/rel/MYAPP/bin/MYAPP foreground`
Замените MYAPP на имя вашего проекта.
Теперь мы работаем с релизной сборкой нашего приложения.

## Использование Distillery вместе с Phoenix

Если вы собираетесь использовать менеджер релизов Distillery вместе с фреймворком Phoenix, то для этого нужно выполнить несколько следующих шагов.

Сперва нам нужно отредактировать файл `config/prod.exs`.

Измените следующие строки с этого:

```elixir
config :book_app, BookAppWeb.Endpoint,
  load_from_system_env: true,
  url: [host: "example.com", port: 80],
  cache_static_manifest: "priv/static/cache_manifest.json"
```
на это:

```elixir
config :book_app, BookAppWeb.Endpoint,
  http: [port: {:system, "PORT"}],
  url: [host: "localhost", port: {:system, "PORT"}],
  cache_static_manifest: "priv/static/cache_manifest.json",
  server: true,
  root: ".",
  version: Application.spec(:book_app, :vsn)
```

Здесь мы выполнили несколько вещей:
- `server` - при старте приложения мы запускаем приложение Cowboy, которое обрабатывает http endpoint;
- `root` - задаёт корневой каталог приложения, в котором хранятся все статические файлы;
- `version` - принудительно обновляет кеш приложения при горячем обновлении версии приложения;
- `port` - установка порта через переменную окружения позволяет нам передавать номер порта при запуске приложения;
Например, мы можем задать порт, запуская `PORT=4001 _build/prod/rel/book_app/bin/book_app foreground`

Если вы выполнили приведенную выше команду, то вы могли заметить, что приложение прекратило работу, потому что оно неспособно соединиться с базой данных, так как никакой базы данных ещё не существует. Это может быть исправлено запуском команды `mix`.
В терминале введите следующее:

```shell
MIX_ENV=prod mix ecto.create
```

Эта команда создаст для вас базу данных.
После перезапуска приложения должно успешно запуститься.
Однако, вы можете заметить, что миграции для базы данных не были запущены.
Обычно в разработке мы запускаем все миграции вручную вызовом `mix ecto.migrate`.
В случае релиза нам придется настроить его так, чтобы он запускал все миграции самостоятельно.

## Запуск миграций в Production

Distillery provides us with the ability to execute code at different points in a release's lifecycle.
These points are known as [boot-hooks](https://hexdocs.pm/distillery/1.5.2/boot-hooks.html).
The hooks provided by Distillery include

* pre_start
* post_start
* pre/post_configure
* pre/post_stop
* pre/post_upgrade


For our purposes, we're going to be using the `post_start` hook to run our apps migrations in production.
Let's first go and create a new release task called `migrate`.
A release task is a module function that we can call on from the terminal that contains code that is separate from the inner workings of our application.
It is useful for tasks that the application itself will typically not need to run.

```elixir
defmodule BookAppWeb.ReleaseTasks do
  def migrate do
    {:ok, _} = Application.ensure_all_started(:book_app)

    path = Application.app_dir(:book_app, "priv/repo/migrations")

    Ecto.Migrator.run(BookApp.Repo, path, :up, all: true)
  end
end
```

*Note* It is good practice to ensure that your applications have all started up properly before running these migrations.
The [Ecto.Migrator](https://hexdocs.pm/ecto/2.2.8/Ecto.Migrator.html) allows us to run our migrations with the connected database.

Далее, создадим новый файл - `rel/hooks/post_start/migrate.sh` и добавим следующий код:


```bash
echo "Running migrations"

bin/book_app rpc "Elixir.BookApp.ReleaseTasks.migrate"

```

In order for this code to run properly, we are using Erlang's `rpc` module which allows us Remote Procedure Call service.
Basically, this allows us to call a function on a remote node and get the answer.
When running in production it is likely that our application will be running in several different nodes

Finally, in our `rel/config.exs` file we're going to add the hook to our prod configuration.

Заменим

```elixir
environment :prod do
  set include_erts: true
  set include_src: false
  set cookie: :"TkJuF,3nc4)OWPBpPxPDb6mz$>)>a>/v/,l2}W*sUFaz<)bG,v*3pPESE,`XOk{,"
  set vm_args: "rel/vm.args"
end
```

на

```elixir
environment :prod do
  set include_erts: true
  set include_src: false
  set cookie: :"TkJuF,3nc4)OWPBpPxPDb6mz$>)>a>/v/,l2}W*sUFaz<)bG,v*3pPESE,`XOk{,"
  set vm_args: "rel/vm.args"
  set post_start_hooks: "rel/hooks/post_start"
end
```

*Примечание* - This hook only exists in the production release of this application.
If we used the default development release it would not run.

## Custom Commands

When working with a release, you may not have access to `mix` commands as `mix` may not be installed to the machine the release is deployed to.
We can solve this by creating custom commands.

> Custom commands are extensions to the boot script, and are used in the same way you use foreground or remote_console, in other words, they have the appearance of being part of the boot script. Like hooks, they have access to the boot scripts helper functions and environment - [Distillery Docs](https://hexdocs.pm/distillery/1.5.2/custom-commands.html)

Commands are similar to release tasks in that they are both method functions but are different from them in that they are executed through the terminal as opposed to being run by the release script.

Now that we can run our migrations, we may want to be able to seed our database with information through running a command.
First, add a new method to our release tasks.
In `BookAppWeb.ReleaseTasks`, add the following:

```elixir
def seed do
  seed_path = Application.app_dir(:book_app_web, "priv/repo/seeds.exs")
  Code.eval_file(seed_path)
end
```

Затем создайте новый файл `rel/commands/seed.sh` и добавьте следующий код:

```bash
#!/bin/sh

release_ctl eval "BookAppWeb.ReleaseTasks.seed/0"
```


*Примечание* - `release_ctl()` is a shell script provided by Distillery that allows us to execute commands locally or in a clean node.
If you need to run this against a running node you can run `release_remote_ctl()`

See more about shell_scripts from Distillery [here](https://hexdocs.pm/distillery/extensibility/shell_scripts.html)

Finally, add the following to your `rel/config.exs` file
```elixir
release :book_app do
  ...
  set commands: [
    seed: "rel/commands/seed.sh"
  ]
end

```

Be sure, to recreate the release by running `MIX_ENV=prod mix release`.
Once this is complete, you can now run in your terminal `PORT=4001 _build/prod/rel/book_app/bin/book_app seed`.
