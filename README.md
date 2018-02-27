# Chatbot


The logic is very simple - the ChatBot stores bag-of-word associations for responses to a previous sentence and uses this to try to match future responses. 

This is not aimed at any real practical task, it is just a framework for experimenting with and developing further.


## Files and Components ##

**Core**
+ `chatbot.py` - main ChatBot library 
+ `utils.py` - generic function utilities used by ChatBot (config, DB conn etc) 
+ `botserver.py` - Multi-Threaded server to allow multiple clients to connect to the ChatBot via network sockets
+ `simpleclient.py` - Simple network sockets client to connect to `botserver`

**Setup and Test**
+ `config.ini` - sample config file to copy into the ./config directory
+ `pwdutil.py` - store an encoded password for connecting to the database schema
+ `setupDatabase.py` - drop and recreate the database tables (existing data gets lost)
+ `pingDB.py` - test the database configuration: create test table, insert data,  query it, drop the test table.
+ `dataDump.py` - Dump out a database table in CSV format
+ `dataLoad.py` - Load a database table in CSV format

## Install and Setup ##

Within source-code working directory, create the following sub-directories:

`./config`  
`./dump`  

### MySQL Database Setup ###

**Database User**
Create a dedicated database for the SimpleBot data-store.

As `root` user in MySQL, create a dedicated MySQL user for the SimpleBot to connect to the datastore:

```
CREATE USER 'simplebot'@'localhost' IDENTIFIED BY '<myPassword123>';  

GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP on simplebot.* 
   TO 'simplebot'@'localhost';

GRANT CREATE TEMPORARY TABLES  on simplebot.* 
   TO 'simplebot'@'localhost';
``` 





**Initialisation of Database Password for ChatBot**

Password authentication details for the ChatBot to access the MySQL database are stored locally.  To avoid storing the actual password in clear-text the password is stored in encoded form.

A separate file called
`./config/.key` 
must be created in the config sub-directory with a single key phrase in it and no other text or characters - i.e.

```
$ echo "thisIsSecret" > ./.key
$ chmod 400 .key
```
(this is NOT the database password).  Set appropriate file permissions on this file.

When the `.key` file is in place, initialise password access to the remote MySQL database using 

`python pwdutil.py -s password`

where "password" is the appropriate real password for the ChatBot MySQL database user account (so this is not the key phrase set above).

NOTE - the approach taken here is not very secure.  It is better than storing the database password in clear text, but is still very limited and this configuration is not suitable for sensitive data.  

**Test Database Configuration**
A simple `pingDB.py` script is provided to test database access and base functionality.

The script connects as the configured mySQL user, creates a table "bot_test_tab", inserts a row, selects it back out and then drops the table.

Expected output is below:

```
$ python ./pingDB.py
C:\Python35\lib\site-packages\pymysql\cursors.py:323: Warning: (1051, "Unknown table 'simplebot.bot_test_tab'")
  self._do_get_result()
execute drop_test_tab Args: None Response 0
execute create_test_tab Args: None Response 0
execute insert_test_tab Args: ('a', 1) Response 1
execute select_test_tab Args: 1 Response 1
execute drop_test_tab Args: None Response 0
```

**Setup the Database Schema**

Run the python script `setupDatabase.py` to create the database schema.  EG:

```
$ python setupDatabase.py
Configuring Tables for database configuration:
        Server: localhost
        DB-User: simplebot
        DB-Name: simplebot
Continue? [Y/n] y
Connecting to database... connected.

Creating words table:
DROP TABLE IF EXISTS words
CREATE TABLE words ( hashid VARCHAR(16) COLLATE utf8_general_ci UNIQUE, word VARCHAR(64) COLLATE utf8_general_ci UNIQUE )
...
(etc,etc)
...
Done.

```


## Run the ChatBot ##

#### Local Single-User Chatbot ####
The ChatBot can be started in single-user mode as follows:

```
 python chatbot.py
```

#### Multi-Threaded Client-Server ChatBot ####

The chatbot can be started with a multi-threaded server scheduler (`botserver.py`) that listens for connections on a TCP port.  This is a very simple "bare-bones" multi session framework with no authentication and just relying on TCP sockets for connection.

Remote TCP Socket Connection requests are given a thread and their own session connection.  

The botserver gives each session a connection to the shared database server.  

**Start the server**

Before starting the server, check the section in the `./config/config.ini` file for configuring the server, EG:

```
[Server]
listen_host: 0.0.0.0
tcp_socket: 9999
listen_queue: 10
```

The default settings are fine for most cases.  To start the server run  

`nohup python3.5 botserver.py & `


**Connect a remote client**

To connect a remote client, make sure the port the server is listening to is allowed through the firewall (or use SSH tunneling).

Copy the `simpleclient.py` file to the remote client host.  

Start the client by running:  
   
```python simpleclient.py -a 192.168.0.99 -p 9999```  
(where the ChatBot Server host IP address or host-alias is supplied for the "-a" arg and ChatBot Server Port is supplied for the -p arg)


