<p align="center">
  <img src="https://github.com/amberframework/site-assets/raw/master/images/amber.png" width="200">
  <p align="center"><strong>Introduction to the Amber Web Framework</strong><br>
  And its Out-of-the-Box Features<p>
  <p align="center">
    <sup>
      <i>
        Amber is Rails-like to an extent, but simpler, reasonable, and easy to understand and use.
      </i>
    </sup>
  </p>
  <p align="center">
  </p>
</p>

# Introduction

**Amber** is a web application framework written in [Crystal](http://www.crystal-lang.org). Homepage is at [amberframework.org](https://amberframework.org/), docs are on [Amber Docs](https://docs.amberframework.org), Github repository is at [amberframework/amber](https://github.com/amberframework/amber), and the chat is on FreeNode IRC (channel #amber) or on [Gitter](https://gitter.im/amberframework/amber).

Amber is simple to get used to, and much more intuitive than frameworks like Rails. (But it does inherit the concepts from Rails that are good.)

This document is here to describe everything that Amber offers out of the box, sorted in a logical order and easy to consult repeatedly over time. The Crystal level is not described; it is expected that the readers coming here have a formed understanding of Crystal and its features.

# Installation

```shell
git clone https://github.com/amberframework/amber
cd amber
make
# The result of 'make' is just one file -- bin/amber

# To install it, or to symlink the system-wide executable to current directory, run one of:
make install # default PREFIX is /usr/local
make install PREFIX=/usr/local/stow/
make force_link # can also specify PREFIX=...
```

# Creating New Amber App

```shell
amber new <app_name> [-d DATABASE] [-t TEMPLATE_LANG] [-m ORM_MODEL]
```

Supported databases: [PostgreSQL](https://www.postgresql.org/) (pg, default), [MySQL](https://www.mysql.com/) (mysql), and [SQLite](https://sqlite.org/) (sqlite).

Supported template languages: [slang](https://github.com/jeromegn/slang) (default) and [ecr](https://crystal-lang.org/api/0.21.1/ECR.html). (ecr is very similar to Ruby's erb.)

Slang is extremely elegant, but very different from the traditional perception of HTML.
ECR is HTML-like and beyond mediocre when compared to slang, but may be the best choice for your application if you intend to use some HTML site template (from e.g. [themeforest](https://themeforest.net/)) whose pages are in HTML + CSS or SCSS.

In any case, you can even combine templates in various languages in a project, and regardless of the language, have in mind that the templates are compiled along with the application. This makes them extremely fast, as well as read-only which is a very welcome side-benefit!

Supported ORM models: [granite](https://github.com/amberframework/granite-orm) (default) and [crecto](https://github.com/Crecto/crecto).

Granite is a very nice and simple, effective ORM model, where you mostly write your own SQL (i.e. all search queries typically look like YourModel.all("WHERE field1 = ? AND field2 = ?", [value1, value2])). But it also has belongs/has relations, and some other little things. (If you have by chance known and loved [Class::DBI](http://search.cpan.org/~tmtm/Class-DBI-v3.0.17/lib/Class/DBI.pm) for Perl, it might remind you of it in some ways.)

Supported migrations engines: [micrate](https://github.com/juanedi/micrate). Micrate is very, very simple and you basically write raw SQL in your migrations. There are just two keywords in the migration file which give instructions whether the SQLs that follow pertain to migrating up or down. These keywords are "-- +micrate Up" and "-- +micrate Down".

# Running the App

The app can be started as soon as you have created it.

To run it, you can use a couple different approaches. Some are of course suitable for development, some for production, etc.:

```shell
# For development, clean and simple - compiles and runs your app:
crystal src/<app_name>.cr

# For development, clean and simple - compiles and runs your app, but
# also watches for changes in files and rebuilds/re-runs automatically.
amber watch

# For production, compiles app with optimizations and places it in bin/app.
# Crystal by default compiles using 8 threads (tune with --threads NUM)
crystal build --no-debug --release --verbose -t -s -p -o bin/app src/app.cr
```

The watch command currently has some issues in edge cases. For example, it may try to run things even if some steps fail ([#499](https://github.com/amberframework/amber/issues/499)) or start re-building the application twice concurrently ([#507](https://github.com/amberframework/amber/issues/507)), and it is generally non-configurable ([#476](https://github.com/amberframework/amber/issues/476)).

Amber itself also currently has problems in edge cases. For example, if you create a new model but do not specify any fields for it, then until you add at least one field, Amber won't start due to a compile error in Granite ([#112](https://github.com/amberframework/granite-orm/issues/112)).

Please ignore these temporary problems until they are solved.

Amber by default uses a feature called "port reuse" available in newer Linux kernels. If you get an error "setsockopt: Protocol not available", it means your kernel does not have it. Please edit `config/environments/development.yml` and set "port_reuse" to false.

# Building the App and Troubleshooting

The application is always built, regardless of whether one is using the Crystal command 'run' (the default) or 'build'. It is just that in run mode, the resulting binary won't be saved to a file, but will be executed and later discarded.

For faster build speed, development versions are compiled without the --release flag. With the --release flag, the compilation takes noticeably longer, but the resulting binary has incredible performance.

Crystal caches partial results of the compilation (*.o files etc.) under `~/.cache/crystal/` for faster subsequent builds.

Sometimes building the App will fail on the C level because of missing header files or libraries. If Crystal doesn't print the actual C error, it will at least print the compiler line that caused it.

The best way to see the actual error from there is to copy-paste the command reported and run it manually in the terminal. The error will be shown and from there the cause will be determined easily.

There are some issues with the `libgc` library here and there. Crystal comes with built-in `libgc`, but it may conflict with the system one. In my case the solution was to install and then remove package `libgc-dev`.

# REPL

Often times, it is very useful to enter an interactive console (think of IRB shell) with all applications classes initialized etc. In Ruby this would be done with IRB or with a command like `rails console`.

Due to its nature, Crystal does not have a free-form [REPL](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop), but you can save and execute scripts in the context of the application. One way to do it is via command `amber x [filename]`. This command will allow you to type or edit the contents, and then execute the script.

Another, more professional way to do it is via REPL-like script tools [cry](https://github.com/elorest/cry) and [icr](https://github.com/crystal-community/icr). `cry` began as an experiment and a predecessor to `amber x`, but now offers additional functionality such as repeatedly editing and running the script if `cry -r` is invoked.

In any case, running a script "in application context" simply means requiring `config/application.cr` (or more generally, `config/**`), Therefore, be sure to list all your requires in `config/application.cr` so that everything works as expected.

# File Structure

So, at this point you might be wanting to know what's placed where in an Amber application. The default structure looks like this:

```
./spec                     - Tests (named *_spec.cr)
./config                   - All configuration
./config/application.cr    - Main configuration file
./config/routes.cr         - All routes
./config/environments      - Environment-specific YAML configurations
./config/webpack           - Webpack (asset bundler) configuration
./config/initializers      - Initializers
./db/migrations            - All DB migration files (created with 'amber g migration ...')
./src                      - Main source directory, with <app_name>.cr being the main/entry file
./src/controllers          - All controllers
./src/models               - All models
./src/views                - All views
./src/views/layouts        - All layouts
./src/views/home           - Views for HomeController (path "/")
./src/assets               - Static assets which will be bundled and placed into ./public/dist/
./src/assets/stylesheets
./src/assets/fonts
./src/assets/images
./src/assets/javascripts
./public                   - The "public" directory for static files
./public/dist              - Directory inside "public" for generated files and bundles
./public/dist/images
```

I prefer to have some of these directories accessible directly in the root directory of the application and to have the config directory named `etc`, so I run:

```
ln -sf config etc
ln -sf src/assets
ln -sf src/controllers
ln -sf src/models
ln -sf src/views
ln -sf src/views/layouts
```

# Database Commands

Amber provides a group of commands under the 'db' group to allow working with the database. The simple commands you will most probably want to run just to see basic things working are:

```shell
amber db create
amber db status
amber db version
```

But, beware:

Before these commands will work, you will need to take care of a couple things.

First, create a user to access the database. For PostgreSQL, this is done by invoking something like:

```shell
$ sudo su - postgres
$ createuser -dElPRS myuser
Enter password for new role: 
Enter it again: 
```

Then, edit `config/environments/development.yml` and configure "database_url:" to match your settings. If nothing else, the part that says "postgres:@" should be replaced with "yourusername:yourpassword@".

Then, please note that none of the database commands will work until you generate something that involves a migration ([#519](https://github.com/amberframework/amber/issues/519)). If you want to fix this manually, assuming that you are using Granite ORM, just run:

```shell
echo "Granite::ORM.settings.database_url = Amber.settings.database_url" >> \
  config/initializers/granite.cr
```

Then, to avoid seeing another minor unnecessary error, please run `mkdir -p db/migrations` ([#522](https://github.com/amberframework/amber/issues/522)).

And then try the three database commands from the beginning of this section, but have in mind that `amber db create` *must* be the first command.

Please note that for the database connection to succeed, all parameters must be correct (hostname, port, username, password, database name), database server must be accessible, and the database must actually exist (unless you are invoking 'amber db create' to create it). In case of *any error in any of the stages* of connecting to the database, the error message will be very terse and just say "Connection unsuccessful: <database_url>". The solution is simple, though - simply use the printed database_url to manually attempt a connection to the database, and the problem will most likely quickly reveal itself.

# Routes

Routes are very easy to understand. Routes connect HTTP methods (and the paths with which they were invoked) to controllers and methods on the Amber side.

Amber includes a wonderful command `amber routes` to display current routes. By default, the routes table looks like the following:

```shell
$ amber routes

╔══════╦═══════════════════════════╦════════╦══════════╦═══════╦═════════════╗
║ Verb | Controller                | Action | Pipeline | Scope | URI Pattern ║
╠──────┼───────────────────────────┼────────┼──────────┼───────┼─────────────╣
║ get  | Amber::Controller::Static | index  | static   |       | /*          ║
╠──────┼───────────────────────────┼────────┼──────────┼───────┼─────────────╣
║ get  | HomeController            | index  | web      |       | /           ║
╚══════╩═══════════════════════════╩════════╩══════════╩═══════╩═════════════╝
```

Here's an example of an actual route definition that routes HTTP POST requests to "/registration" to the method create() in class RegistrationController:

```
post "/registration", RegistrationController, :create
```

Standard HTTP verbs (GET, HEAD, POST, PUT, PATCH, DELETE) by convention go to standard methods on the controllers (show, new, create, edit, update, destroy). However, there is nothing preventing you from routing URLs to any methods you want in the controllers.

# Views

Information about views can be summarized in bullet points:

- Views in Amber are located in `src/views/`
- They are rendered using `render()`
- The first argument given to `render()` is the template name (e.g. `render("index.slang")`)
- If we are in the context of a controller, `render("index.slang")` will look for view using the path `src/views/<controller_name>/index.slang`
- If we are not rendering a partial, by default the template will be wrapped in a layout
- If the layout name isn't given, the default layout will be `views/layouts/application.slang`
- There is no unnecessary magic applied to template names &mdash; name given is the name that is looked up on disk
- Partials begin with "_" by convention, but that is not required
- To render a partial, use `render( partial: "_name.ext")`

# Static Pages

It can be pretty much expected that a website will need a set of simple, "static" pages. Those pages are served by the application, but do not come from a database nor typically use any complex code. Such pages might include About and Contact pages, Terms of Conditions, etc. Making this work is trivial.

Let's say that, for simplicity and grouping, we want all "static" pages to be served by PageController. We will group all these pages under a common web-accessible prefix of /page/, and finally we will route page requests to controller methods. (Because these pages will not be powered by a database or have any methods really, we won't need a model nor more than a single view file per page.)

Let's start by creating a controller:

```shell
amber g controller page
```

Afterwards, we edit `config/routes.cr` to link URL "/about" to method about() in PageController. We do this inside the "routes :web" block:

```
routes :web do 
  ...
  get "/about", PageController, :about
  ...
end
```

Then, we edit the controller and actually add method about(). This method can just directly return some string in response, or it can render a view, and then the expanded view contents will be returned as the response.

```shell
$ vi src/controllers/page_controller.cr

# Inside the file, we add:

def about
  # "return" can be omitted here. It is included only for clarity.
  return render "about.ecr"
end
```

Since this is happening in the "page" controller, the view directory for finding the templates defaults to `src/views/page/`. We will create the directory and the file "about.ecr" in it:

```shell
$ mkdir -p src/views/page/
$ vi src/views/page/about.ecr

# Inside the file, we add:

Hello, World!
```

Because we have called render() without additional arguments, the template will default to being rendered within the default application layout, `views/layouts/application.cr`.

And that's it! Visiting `/about` will go to the router, router will invoke `PageController::about()`, that method will render template `src/views/page/about.ecr` in the context of layout `views/layouts/application.cr`, the result of rendering will be a full page with content `Hello, World!` in the body, that result will be returned to the controller, and from there it will be returned to the client.

# Variables in Views

In Amber, templates are compiled in the same scope as controller methods. This means you do not need instance variables for passing the information from controllers to views. Any variable you define in the controller method is automagically visible in the template.

For example, let's add the current date and time display to our /about page:

```shell
$ vi src/controllers/page_controller.cr

def about
	time = Time.now
	render "about.ecr"
end

$ vi src/views/page/about.ecr

Hello, World! The time is now <%= time %>.
```

# Pipelines

