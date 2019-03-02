# app.py

from flask import Flask,g,jsonify, render_template
import requests
from peewee import *
from playhouse.shortcuts import model_to_dict


#Configuring database file name
DATABASE = 'chainsaw.db'

#define app using Flask
app = Flask(__name__)

#Connect to the database
database = SqliteDatabase(DATABASE)

#Model class
class Chainsaw(Model):
    name = CharField()
    country = CharField()
    catches = IntegerField()

    class Meta:
        database = database

# Create table for model
database .create_tables([Chainsaw])

@app.before_request
def before_request():
    g.db = database
    g.db.connect()

@app.after_request
def after_request(response):
    g.dbclose()
    return response

# GET one record by id
@app.route('/chainsaw_API/chainsaw/<catcher_id>')
def get_by_id(catcher_id):
    try:
        c = Chainsaw.get_by_id(catcher_id)
        return jsonify(model_to_dict(c))
    except DoesNotExist:
        return 'Not found', 404

#GET all records by sending a GET request to chainsaw_API/chainsaw
@app.route('/chainsaw_API/chainsaw')
def returnAll():
    res = Chainsaw.select()

    return jsonify([model_to_dict(c) for c in res])



# POST to create a new record
@app.route('/chainsaw_API/chainsaw', methods=['POST'])
def addOne():
    with database.atomic():
        c = Chainsaw.create(**requests.form.to_dict())
        return jsonify(model_to_dict(c)), 201 # 201 status means resource created


# PATCH to modify an existing record
@app.route('/chainsaw_API/chainsaw/<catcher_id>', methods=['PATCH'])
def editOne(catcher_id):
    with database.atomic():
        Chainsaw.update(**requests.form.to_dict())\
        .where(Chainsaw.id == catcher_id).execute()\
        .execute()
        return 'ok', 200 # 200 means ok, request successful




# Start the web app running on port 5000 in debug mode
if __name__ == '__main__':
    app.run()(debug=True, port = 5000)



