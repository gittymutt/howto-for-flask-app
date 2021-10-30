How to set up a simple flask app on digitalocean


create a directory /var/www/myapp

create your venv there if necessary. there will be a /bin directory 
with the 'activate' file in there, as well as the activate_this.py file. 
Or you may have to use pyenv to get that file. Or you can activate the env
from the code?

You may need to install mod_wsgi 
https://flask.palletsprojects.com/en/2.0.x/deploying/mod_wsgi/

in the myapp directory, add the file myappfile.wsgi

#! /usr/bin/python3.6
import logging
import sys
activate_this = '/var/www/app/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/myapp/')
from myappfile import app as application
application.secret_key = 'somesecret'


also in the myapp directory add the file myappfile.py

from flask import Flask, request
from flask_cors import CORS
from pattern3.en import conjugate
# https://github.com/clips/pattern/wiki/pattern-en#conjugation
app = Flask(__name__)
cors = CORS(app)
app.debug = True

@app.route('/', methods = ['GET'])
def hello_world():
    subject = request.args.get('subject')
    verb = request.args.get('verb')
    forms = get_forms(subject, verb)
    return forms

def get_forms(subject, verb):
    baseform = conjugate(verb, "VB")
    thirdpers = conjugate(verb, "3sg")
    simplepast = conjugate(verb, "p")
    pastpart = conjugate(verb, "VBN")
    ingform = conjugate(verb, "VBG")
    return {'thirdpers': thirdpers,
      'simplepast': simplepast,
      'pastpart': pastpart,
      'ingform': ingform,
      'baseform': baseform,
      'subject': subject
    }


https://flask.palletsprojects.com/en/2.0.x/deploying/mod_wsgi/
in /etc/apache2/sites-available, add file myapp.conf

<VirtualHost *:80>
                ServerName 167.99.145.79
                ServerAdmin email@mywebsite.com
                WSGIDaemonProcess my_flask_app user=username group=groupname threads=5
                WSGIScriptAlias /app /var/www/app/my_flask_app.wsgi
                <Directory /var/www/app>
                  WSGIProcessGroup my_flask_app
                  WSGIApplicationGroup %{GLOBAL}
                  Require all granted
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                SetEnv TZ America/New_York
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


you can find the user and groupname with apachectl -S

at the prompt:
a2ensite myapp
systemctl reload apache2


check logs for problems /var/log/apache2/error.log


In WSGIScriptAlias /app /var/www/app/myappfile.wsgi
 
the /app means that you go to xxx.xx.xxx.xx/app


