TwitterGoggles
==============

A Python 3.3 adaptation of the PHP program "Twitter Zombie", originally developed for the Twitter Search API 
version 1.0. This new project is built for the Twitter Search API version 1.1.

Version Notes
-------------
libbyh/TwitterGoggles is now on version 2.0. It is no longer compatible with [pmaconi/TwitterGoggles](http://github.com/pmaconi/TwitterGoggles) because of changes to the schema and the data stored (this version captures account creation date). If you update your version of [TwitterGoggles.py](TwitterGoggles.py), you must also use the updated database schema. You can update your current schema using [config/update_schema_v2.sql](config/update_schema_v2.sql).

Recent Changes
------------
- added account created_at date to db and script so we know how old these Twitter accounts are
- additional indexes in [config/schema.sql](schema.sql) to speed up queries
- allow user to choose a settings file - use case: run a single TwitterGoggles codebase that populates more than one database
- allow user to cache/store JSON returned before parsing into MySQL - use case: you really love keeping "raw" data or you want to parse it differently later
- see related project for analyzing data collected/parsed by TwitterGoggles: [Analyze Tweets from TG](https://github.com/casmlab/analyze-tweets-from-tg)

Dependencies
------------
- mysql-connector-python
- requests
- requests-oauthlib

Setup and Installation
----------------------
1. Install Python 3.3 on the computer you use.  Recognize that many standard installations of Python are
   currently 2.x, and you may need to install Python 3.3 as well.  To execute with Python3, you type ```python3```
2. Install the dependencies
    1. Make sure you have "pip" installed on your system (this is a package manager for Python3)
    2. From a command prompt, type:
```
pip install mysql-connector-python requests requests-oauthlib
```
3. if pip is not installed, issue these commands:
```wget http://pypi.python.org/packages/source/p/pip/pip-1.1.tar.gz#md5=62a9f08dd5dc69d76734568a6c040508```
    	 
	```tar -xvf pip*.gz```

	```cd pip*```

	```sudo python setup.py install```
3. Build database
   1. Create empty database
	2. Create new user for db or grant access to an existing user
	3. Run [config/schema.sql](config/schema.sql)
4. Set database (and optional: JSON file storage) config settings (see [config/example_settings.cfg](config/example_settings.cfg))
5. Save that file as config/settings.cfg (or something else and use the ```-s``` arg when you call the script)
5. Add your OAuth credentials to the oauth table
	1. Get OAuth credentials by setting up an Application at [Twitter's Developers site](https://dev.twitter.com/)
	2. EXAMPLE: 
<blockquote>
INSERT INTO \`oauth\` (\`oauth_id\`, \`name\`, \`consumer_key\`, \`consumer_secret\`, \`access_token\`, \`access_token_secret\`) VALUES (1, 'a name you can remember', 'consumer_key', 'consumer_secret', 'access_token', 'access_token_secret');
</blockquote>
5. Add your job(s) to the job table
	* job_id: an INT you can choose
	* state: an indication of how frequently the collection will occur, in minutes; must be 1 or greater to run at all
	* zombie_head: an INT, you'll use this to identify the head when you call TwitterGoggles
	* since_id_str: can be blank for new jobs
	* query: the "q=params" part of a Twitter Search Query (see https://dev.twitter.com/docs/using-search)
	** 'q=' is not required
	* description: a note to yourself about what this job does, will print in verbose mode 
	* last_count: NULL for new jobs
	* last_run: NULL for new jobs
	* analysis_state: 0 for new jobs
	* oauth_id: set to match the ID of the oauth credentials you just added 
   * EXAMPLE: 
<blockquote>
INSERT INTO \`job\` (\`job_id\`, \`state\`, \`zombie_head\`, \`since_id_str\`, \`query\`, \`description\`, \`last_count\`, \`last_run\`, \`analysis_state\`, \`oauth_id\`) VALUES (3, 1, 2, X'30', 
'q=from%3Alibbyh%20OR%20from%3Asgoggins', 'Libby\'s example job', NULL, NULL, 0, 1);
</blockquote>

Usage
-----
```
usage: TwitterGoggles.py [-h] [-v] [-d DELAY] [-c CACHE] [-s SETTINGS] head

positional arguments:
  head				Specify the head # (zombie_head in the job table)

optional arguments:
  -h, --help		Show this help message and exit
  -v, --verbose		Show additional logs
  -d, --delay		Delay execution by DELAY seconds
  -c, --cache		Cache the JSON returned before parsing it into MySQL
  -s, --settings	Full path to the settings config file with MySQL and Data param
```

Unix Cron Example
-----------------
```
*/1 * * * * /usr/local/bin/python3 /home/TwitterGoggles.py -v -d 2 1 >> ~/log/zombielog-head-1-1.txt
*/1 * * * * /usr/local/bin/python3 /home/TwitterGoggles.py -v -d 17 2 >> ~/log/zombie-head-2-1.txt
*/1 * * * * /usr/local/bin/python3 /home/TwitterGoggles.py -v -d 33 3 >> ~/log/zombielog-head-3-1.txt
*/1 * * * * /usr/local/bin/python3 /home/TwitterGoggles.py -v -d 47 4 >> ~/log/zombielog-head-4-1.txt
```


