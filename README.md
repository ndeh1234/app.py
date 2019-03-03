# app.py

from flask import Flask,g,jsonify, render_template,request
import requests
from peewee import *
from playhouse.shortcuts import model_to_dict
from flask_sqlalchemy import SQLAlchemy


#Configuring database file name
#DATABASE = 'chainsaw.db'

#define app using Flask
app = Flask(__name__)

app.config['DEBUG'] = True
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///weather.db'
db = SQLAlchemy(app)

class City (db.model):
    id = db.column(db.Integer, primary_key = True)
    name = db.column(db.String(50),nullable = False)

@app.route('/', methods = ['POST','GET'])
def index(request):
    if request.methods == 'POST':
        new_city = request.form.get('city')
        if new_city:
            new_city_object = City (name = new_city)
            db.seession.add(new_city_object)
            db.session.commit()

    cities = City.query.all()




    url = 'http://api.openweathermap.org/data/2.5/weather?q={}&units=imperial&'\
          'APPID=17959c71c1522324a3b9e201a9ce81a6'
    city = 'Minneapolis'

    weather_data = []

    for city in cities:


      r = request.get(url.format(city)).json()


    city_weather = {'city': city.name,
               'temperature': r ['main']['temp'],
               'description': r['weather'][0]['description'],
               'wind speed': r['wind']['speed'],
               'icon': r['weather'][0]['icon'],
                }

    weather_data.append(city_weather)

    return render_template('weather.html', weather_data = weather_data)




# Start the web app running on port 5000 in debug mode
if __name__ == '__main__':
    app.run()(debug=True, port = 5000)



