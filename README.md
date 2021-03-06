# WeatherServer

Full Stack Embedded 2016 features a cheap weather station. This Django app 
serves the observed weather data in human and machine readable formats.

Please refer to ``LICENSE`` if you want to copy or use this software.

## Authors
The members of Full Stack Embedded 2016 are, in alphabetical order:

* Frederic Afadjigla
* Nico Caspari
* William Herndon
* Daniel Lee
* Ruediger Reiter
* Gregor Schnee

This component was authored mainly by [Daniel Lee](erget2005@gmail.com) and 
[Gregor Schnee](schneegor@gmail.com).

## Required/suggested packages apt-get packages
Use "sudo apt-get install <package-name>[, <package-name>]"
### Required:
* python3
* python3-pip (required for setup.py to run)

## Suggested for development setup:
* apache2
* libapache2-mod-python
* mysql-server
* python3-pandas

## Suggested packages pip packages
### Use "sudo pip3 install <package-name>[, <package-name>]":
* Django
* MySQL-python ?

## Required setup
sudo python3 setup.py install


## Using test data
Test data is currently a hack because currently no actual data can enter the 
database - the weather station hardware does not exist yet. This section will
be removed in the future.

### Creating and importing test data
Test data can be created by executing
``show_weather/utils/create_mock_dataseries.py``. This creates a file, 
``data_series.csv``, in the current working directory:

```
me@host:~/WeatherServer/weather_server/show_weather/utils/> python3 create_mock_dataseries.py
```

This generates a randomized series of observations, stored as a CSV in 
``weather_server/show_weather/utils``. You can load this into the database 
using ``weather_server/show_weather/utils/import_generated_series.py``. This 
is slightly complicated because it uses Django's ORM, so it has to either be 
started inside a Django shell, which takes care of all the bookkeeping 
legwork for you (establishing DB connection, etc.) or you have to do Django's
administrative work for it. This example reduces the workload by using a 
Django shell, but it's a bit esoteric for the same reason - the Django shell 
is started, then the script is loaded as a string and executed inside that 
environment. Don't worry, I washed my hands after writing it and it will soon
die.

``weather_server/show_weather/utils/import_generated_series.py`` creates an 
observing station and saves all of the observations as belonging to that 
station. It also expects the sample data series to be stored inside the 
``utils`` folder.

```
me@host:~/WeatherServer/weather_server> python3 manage.py shell
Python 3.4.1 (default, May 23 2014, 17:48:28) [GCC] on linux
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>> with open("show_weather/utils/import_generated_series.py") as script:
...     exec(script.read())
```

Lots of warnings follow because the ``datetime`` object used is timezone 
naive. Now your database is full of observations for my apartment, at the 
intersection of Constitution Hill, The Mall and Birdcage Walk.

### Examining test data
If you haven't done so already, you'll need to set up your Django application.
Set up the tables in your database and a super user so that you can use the 
admin page as follows:


```
me@host:~/WeatherServer/weather_server> # Setup database
me@host:~/WeatherServer/weather_server> python3 manage.py makemigrations
me@host:~/WeatherServer/weather_server> python3 manage.py migrate
me@host:~/WeatherServer/weather_server> # Setup super user - follow prompts
me@host:~/WeatherServer/weather_server> python3 manage.py createsuperuser
me@host:~/WeatherServer/weather_server> python3 manage.py runserver
```

Now your development server is running. If you navigate to 
``localhost:8000/admin/`` you can login to the admin page and see the entries
in your database.

## API for viewing data
Currently, only CSVs are served as raw data. The following endpoints are 
supported:

* ``/data/csv/stations/`` - return a CSV of known station metadata
* ``/data/csv/$station_id/$start_date - $end_date`` - return a CSV of 
observations made at the station with ``$station_id`` between the dates 
(inclusively) specified. Dates are to be formated as ``YYYY-MM-DD HH:MM``.
