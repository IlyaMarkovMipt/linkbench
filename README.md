# Linkbench fork with Tarantool plugin
- - -

##Content
- - -
* [Instructions(Eng)](#instructionseng)
    * [Getting and Building LinkBench](#getting-and-building-linkbench)
    * [Running a Benchmark with Tarantool](#running-with-tarantool)
        * [Tarantool Launch](#tarantool-launch-eng)
        * [Configuration files](#configuration-files)
        * [Loading data](#loading-data)
        * [Request Phase](#request-phase)
    * [Using Docker image](using-docker-image)
* [Instructions(Rus)](#instructionsrus)
* [Original Readme](#original-readme)
* [Simply way to launch](#simply-way)

## Instructions(Eng)
- - -

### Prerequisites
- - -

These instructions assume you are using a UNIX-like system such as a Linux distribution
or Mac OS X.

**Java**: You will need a Java 7+ runtime environment.  LinkBench by default
      uses the version of Java on your path.  You can override this by setting the
      JAVA\_HOME environment variable to the directory of the desired
      Java runtime version.  You will also need a Java JDK to compile from source.

**Maven**: To build LinkBench, you will need the Apache Maven build tool. If
    you do not have it already, it is available from http://maven.apache.org .


**Tarantool**: To run Linkbench with Tarantool database you will need to install it. 
    github: https://github.com/tarantool/tarantool, official site: http://tarantool.org

[Back to content](#content)


### Getting and Building LinkBench
----------------------------
First get the source code

    git clone git@github.com:facebook/linkbench.git

Then enter the directory and build LinkBench

    cd linkbench
    mvn clean package

In order to skip slower tests (some run quite long), type

    mvn clean package -P fast-test

To skip all tests

    mvn clean package -DskipTests

If the build is successful, you should get a message like this at the end of the output:

    BUILD SUCCESSFUL
    Total time: 3 seconds

If the build fails while downloading required files, you may need to configure Maven,
for example to use a proxy.  Example Maven proxy configuration is shown here:
http://maven.apache.org/guides/mini/guide-proxies.html

Now you can run the LinkBench command line tool:

    ./bin/linkbench

Running it without arguments will show a brief help message:

    Did not select benchmark mode
    usage: linkbench [-c <file>] [-csvstats <file>] [-csvstream <file>] [-D
           <property=value>] [-L <file>] [-l] [-r]
     -c <file>                       Linkbench config file
     -csvstats,--csvstats <file>     CSV stats output
     -csvstream,--csvstream <file>   CSV streaming stats output
     -D <property=value>             Override a config setting
     -L <file>                       Log to this file
     -l                              Execute loading stage of benchmark
     -r                              Execute request stage of benchmark

[Back to content](#content)

### Running a Benchmark with Tarantool
-------------
In this section we will document the process of launching
a new Tarantool database and running a benchmark with LinkBench.

#### Tarantool Launch

We need to launch Tarantool server. All setup is written in directory src/tarantool_scripts,  
so move to the directory and launch server:
    
    # assuming, you are in linkbench directory
    cd src/tarantool_scripts
    tarantool app.lua &

This will launch tarantool server as daemon. You can change setup in app.lua file.
As default it launches Tarantool with engine 'vynil' and logs to tarantol.log.

[Back to content](#content)

#### Configuration Files

LinkBench requires several configuration files that specify the
benchmark setup, the parameters of the graph to be generated, etc.
Before benchmarking you will want to make a copy of the example config file:

    cp config/LinkConfigTarantool.properties config/MyConfig.properties

Open MyConfig.properties.  At a minimum you will need to fill in the
settings under *Tarantool Connection Information* to match the server, user
and database you set up earlier. E.g.

    # Tarantool connection information
    host = localhost
    user = linkbench
    password = your_password
    port = 3306
    dbid = linkdb

You can read through the settings in this file.  There are a lot of settings
that control the benchmark itself, and the output of the LinkBench
command link tool.  Notice that MyConfig.properties
references another file in this line:

    workload_file = config/FBWorkload.properties

This workload file defines how the social
graph should be generated and what mix of operations should make
up the benchmark.  The included workload file has been tuned to
match our production workload in query mix.  If you want to change
the scale of the benchmark (the default graph is quite small for
benchmarking purposes), you should look at the maxid1 setting.  This
controls the number of nodes in the initial graph created in the load
phase: increase it to get a larger database.

      # start node id (inclusive)
      startid1 = 1

      # end node id for initial load (exclusive)
      # With default config and MySQL/InnoDB, 1M ids ~= 1GB
      maxid1 = 10000001

[Back to content](#content)

####Loading Data

First we need to do an initial load of data using our new config file:

    ./bin/linkbench -c config/MyConfig.properties -l

This will take a while to load, and you should get frequent progress updates.
Once loading is finished you should see a notification like:

    LOAD PHASE COMPLETED.  Loaded 10000000 nodes (Expected 10000000).
      Loaded 47423071 links (4.74 links per node).  Took 620.4 seconds.
      Links/second = 76435

At the end LinkBench reports a range of statistics on load time that are
of limited interest at this stage.

#### Request Phase

Now you can do some benchmarking.
Run the request phase using the below command:

    ./bin/linkbench -c config/MyConfig.properties -r

LinkBench will log progress to the console, along with statistics.
Once all requests have been sent, or the time limit has elapsed, LinkBench
will notify you of completion:

    REQUEST PHASE COMPLETED. 25000000 requests done in 2266 seconds.
      Requests/second = 11029

You can also inspect the latency statistics. For example, the following line tells us the mean latency
for link range scan operations, along with latency ranges for median (p50), 99th percentile (p99) and
so on.

    GET_LINKS_LIST count = 12678653  p25 = [0.7,0.8]ms  p50 = [1,2]ms
                   p75 = [1,2]ms  p95 = [10,11]ms  p99 = [15,16]ms
                   max = 2064.476ms  mean = 2.427ms

[Back to content](#content)

### Using Docker image

First, build the image of testing container.

    docker build -t `name_of_image` .

Then, launch container:

    docker run -it -v `path_to_logs_directory`:/log --name `name_of_container` `name_of_image`

After that you can easily send results of benchmark to microb using export.py script.
    python export.py auth.conf `path_to_logs_directory`/linkbench.log linkbench linkbench

If you want the container to send results on its own, you need to have directory with the auth.conf and run following line:

    docker run -it -v *path_to_logs_directory*:/log -v *path_to_credentials*/:/credentials/ --name `name_of_container` `name_of_image`
[Back to content](#content)

# Instructions(Rus)
-------------
### Необходимые установки 
- - -

Инструкции предполагают, что у Вы пользуетесь UNIX-подобной системой

**Java**: Вам понадобится Java 7+ окружение.  LinkBench по умолчанию
      пользуется версией Java, прописанной в PATH. Вы можете исправить это, 
      установив JAVA\_HOME переменную окружения на директорию содержащую,
      желаемую версию Java. Также Вам потребуется Java JDK, чтобы скомпилировать проект. 

**Maven**: Чтобы запустить Linkbench, Вам необходимо иметь Apache Maven build tool. Если
    он еще не установлен, это можно его установить по этой ссылке http://maven.apache.org .

**Tarantool**: Чтобы запустить Linkbench для базы данных Tarantool, Вам нужно установить ее. 
    github страница проекта: https://github.com/tarantool/tarantool, официальный сайт проекта: http://tarantool.org

[К содержанию](#content)

### Загрузка и сборка LinkBench
----------------------------
Сначала скачайте исходный код

    git clone git@github.com:facebook/linkbench.git

Перейдите в директорию и соберите Linkbench

    cd linkbench
    mvn clean package

Чтобы пропустить тестирование медленный тестов (которые идут довольно долго), выполните

    mvn clean package -P fast-test

Чтобы пропустить все тесты:

    mvn clean package -DskipTests

Если сборка прошла успешно, Вы должны увидеть подобное сообщение:

    BUILD SUCCESSFUL
    Total time: 3 seconds

Если сборка завершилась с ошибками, Вам, возможно понадобится сконфигурировать Maven,
например, использовать proxy.  Пример Maven proxy конфигурации показан здесь:
http://maven.apache.org/guides/mini/guide-proxies.html

Теперь Вы можете воспользоваться LinkBench из командной строки:

    ./bin/linkbench

Выполнение этой строки без аргументов вернут короткое сообщение с подсказкой:

    Did not select benchmark mode
    usage: linkbench [-c <file>] [-csvstats <file>] [-csvstream <file>] [-D
           <property=value>] [-L <file>] [-l] [-r]
     -c <file>                       Linkbench config file
     -csvstats,--csvstats <file>     CSV stats output
     -csvstream,--csvstream <file>   CSV streaming stats output
     -D <property=value>             Override a config setting
     -L <file>                       Log to this file
     -l                              Execute loading stage of benchmark
     -r                              Execute request stage of benchmark

[К содержанию](#content)

### Использование Linkbench с Tarantool
-------------
В этом разделе мы опишем процесс запуска Tarantool и Lenckbench.

####Запуск Tarantool 

Вся настройка сервера прописана в директории src/tarantool_scripts.  
Перейдем туда и запустим сервер:
    
    # предполагаем, Вы находитесь в директории linkbench
    cd src/tarantool_scripts
    tarantool app.lua &

Эти строчки запустят демон Tarantool. Вы можете поменять настройки в app.lua файле.
по умолчанию  Tarantool сервер здесь запускается с  "движком" 'vynil' и кладетлоги  в tarantol.log.

[К содержанию](#content)

#### Configuration Files

LinkBench требует несколько файлов конфигурации.
Перед запуском бенчмарка Вы можете скопировать пример конфигурационного файла:

    cp config/LinkConfigTarantool.properties config/MyConfig.properties

Откроем MyConfig.properties.  Как минимум, Вам понадобится заполнить
настройки под *Tarantool Connection Information*:  сервер, имя пользователя, пароль, порт.

    # Tarantool connection information
    host = localhost
    user = linkbench
    password = your_password
    port = 3306
    dbid = linkdb

Если Вы хотите поменять масштаб бенчмарка (увеличить количество загружаемых данных), Вам нужно обратить внимание на настройку maxid1 в файле FBWorkload.properties.
Она контролирует число узлов заргужаемых в базу данных.

      # start node id (inclusive)
      startid1 = 1

      # end node id for initial load (exclusive)
      # With default config and MySQL/InnoDB, 1M ids ~= 1GB
      maxid1 = 10000001

[К содержанию](#content)

#### Загрузка данных

Загрузим данные с помощью нашего конфигурациооного файла:

    ./bin/linkbench -c config/MyConfig.properties -l

В случае успешного завершения увидим:

    LOAD PHASE COMPLETED.  Loaded 10000000 nodes (Expected 10000000).
      Loaded 47423071 links (4.74 links per node).  Took 620.4 seconds.
      Links/second = 76435

#### Фаза нагрузки

Запустим фазу нагрузки следующей комндой:

    ./bin/linkbench -c config/MyConfig.properties -r

В итоге получим подобное сообщение:

    REQUEST PHASE COMPLETED. 25000000 requests done in 2266 seconds.
      Requests/second = 11029

[К содержанию](#content)

## Simply way

You can launch benchmark without configuring, just downloading and launching script from gist:

    #assuming you are in linkbench directory
    git clone https://gist.github.com/IlyaMarkovMipt/fdc32320f5b9a4cb1d884450117bc689 script
    mv script/script.sh .
    chmod +x script.sh
    ./script 1

Parameter X = 1 of script is number of config file (LinkConfigTarantoolX.properties)

[Content](#content)

# Original Readme
[Original Readme](README_original.md)


