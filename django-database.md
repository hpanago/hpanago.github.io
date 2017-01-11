#Error: OperationalError at /admin/login/ unable to open database file 
#Django version: 1.9.2

##When using sqlite this could turn out to be a common error, especially when attempting to login to the admin panel.

There are 3 reasons for this:

1. Your sqlite database file is not writable or it is not owned by apache's group (www-data).
2. Your sqlite database's file parent directory is not owned by the group www-data or not writable by it.
3. You might need to change the NAME of the DATABASES in your settings.py and use the full path to the file.

In order to solve this you can run the following commands:

```chown -R :www-data djangoProjectFolder```

```chmod 755 djangoProjectFolder/bd.sqlite3```

and for the syntax in your settings.py you can follow the official documentation which can be found [here](https://docs.djangoproject.com/en/1.10/ref/settings/#name).
