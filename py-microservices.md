# Getting Started with Python Microservices with Docker on Linux | Ubuntu 16.04 LTS | For Complete Beginners 

 _(author: Shubham Mishra, Merilytics Inc.)_ 

[![](http://www.keysolutions.com.pe/wp-content/uploads/2017/02/keyLogoPython-150x120.png)](https://www.docker.com)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[![](https://www.docker.com/sites/default/files/vertical_small.png)](https://www.docker.com)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[![N|Solid](http://1.bp.blogspot.com/-lgzZ7YBRO-Q/VJqGdotLkPI/AAAAAAAAAHo/qHriDc2Ff8s/w150-h120-c/canonical-ubuntu-logo-1.jpg)](https://www.docker.com) 

Modern large scale application development has the developers got inclined towards microservices architecture wherein the application is broken down to multiple independent or independently hosted (micro)services. Though extremely tempting to develop in this pattern it shifts a major load from development to devOps. In an environment where quick development, deployment and continuous integration is the demand of business its required to have your weaponize your developers with the tools that can sort the grand problems of deployment, packaging, code management and versioning. Though microservices architecture gives the freedom to have a polyglot code base, to make your development faster its advised to use language more adept towards quick shipping like Ruby, Python or Go. 

We'll be using python 3.6 for our development. The tool to modularize and manage our microservices shall be the now famous Docker. We'll be working on a Ubuntu Linux version 16.04, though any older version up to 12 should work following the same example. 

#### Containerization for Microservices 

Since the use case of microservices is such that every microservice can be created using a completely different language, technology, version, environment, etc. its imperative to have virtual isolation for all the entities that become part of our microservice application eco-system. 

#### Docker and Docker Compose 

Docker and Docker Compose are containerization engine and packaging engine respectively provided by docker. What we mean by that is docker engine is used to create and run individual containers with configurations as required by user. Docker Compose uses a single configuration file to contain information on all containers we'll be using for running our application thats made of all the individual containers. 

To install [Docker](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/) and [Docker Compose]( on your Linux machine I'd advice to follow the [official docker blog instruction linked here](https://docs.docker.com/engine/installation/linux/docker-ce/ubuntu/). Make sure you install Docker before [Docker Compose](https://docs.docker.com/compose/install/) and unless you are an enterprise customer install the community edition of docker (docker-ce). 


#### Setting up a development environment 

It's important to use the best available development tools for a pattern that inherently is so fragmented else developers will end up switching between windows and terminals to get even th first thing done. Thankfully we now have open source version of [Visual Studio Code](https://code.visualstudio.com/) which comes with a good community and large number of plugins to pretty much sort all our development needs. Visual Studio Code (VSCode) comes with pretty interface, file and extensions explorer, and integrated terminal which will be extremely helpful developing with docker containers. [Download](https://code.visualstudio.com/download) and [install](https://code.visualstudio.com/docs/setup/linux) visual studio code from the respective links. 

Now install the following extensions offered for free: 

- Python by Don Jayamanne 
- Docker by Microsoft 
- Docker Explore by Jun Huan 

These extensions are going to be extremely helpful in developing the application in a GUI interface. 

#### Docker Images 

Docker Images are metadata and data which is replicated everytime that image is run on a docker container. It's comparable to Operating System on a Machine where Docker Container is the machine and Image is the Operating System but this analogy is just to understand the relation and nothing more. We'll be using docker images to spin up our application server, API gateways, databases, caches, etc. A container can technically run multiple images but its advised against. 

So how we use docker image is by packing all configuration and properties that module needs before even starting that machine up. Once packed the docker machine would use the same configurations while strating up. 

Docker image configurations are written in special file always named `dockerfile` with no extentions. An example dockerfile is shown below, same which will be used later in creating an application server image for our demonstration. 

```dockerfile 
FROM python:3.6 
ADD . /home/web 
WORKDIR /home/web 
RUN apt-get update 
RUN apt install net-tools 
RUN python -m pip install -r requirements.txt 
CMD ["bash"] 
``` 

Each line of the dockerfile has information on configuring the particular docker image. It's to be noted that it's not always necessary to have a dockerfile to run an image on a container. We can `pull` someone else's image and run it on our docker container just as easily. But more about that in a subsequent section on Docker Compose file. 

Coming back to explaining dockerfile. Every line in dockerfile is a comfiguration inherent to that image and remains the same unless the image is rebuilt with changed configurations or changed files used by the configurations. 

`FROM` keyword followed by space and `parent-image-name:version` tells docker of the base image onto which to build the current image on (to read more about how to create parent or base image follow the [link here](https://docs.docker.com/engine/userguide/eng-image/baseimages/)). Here python:3.6 is an image pulled directly from [docker hub](https://hub.docker.com/) is an image of Ubuntu pre-installed with python 3.6 on it. 

`ADD` keyword followed by the two arguments separated by space lists the source location on the drive and its replica to be moumted including all the files in that location on the image. The example `ADD . /home/web` takes everything in the current directory and places it on the `/home/web` on the image container. 

`WORKDIR` keyword followed by single path arguments sets the active working directory available run operations/commands in. 

`RUN` keyword is used to run shell commands for the version of OS running on the container. 

`CMD` keyword can be used only once to run the command or application that the container is supposed to run on boot-up. In our case we used a little trick to run the **gunicorn** application server after attaching our *volume* instead of running at startup. This allowed us to modify our code within the volume during runtime. This gives quick modification and execution flexibility to our development. I just dropped some words like **gunicorn** and *volume* out of nowhere and I am aware, don't fret they'll be explained in subsequent development sections.

#### Application Development 

Now that we have some understanding of spinning up docker images. Let's get our hands dirty into actual application development. The folder structure can look something like th following, and it may vary from design to design, for our **flask** application we'll follow the following. 

``` 
.  
appdirectory 
+-- README.md
+-- docker-compose.yaml 
+-- proxy 
| +-- dockerfile 
| +-- proxy.conf 
+-- web 
| +-- templates 
| | +-- home.html 
| +-- wsgi.py 
| +-- run.py 
| +-- dockerfile 
| +-- requirements.txt 
| +-- classes.py 
``` 

Starting off with our `wsgi.py` script. Its the entrypoint to our application as we are running the application as **WSGI** app on **gunicorn**. Now what are **Flask**, **gunicorn** and **WSGI**. Flask is simple microframework to create python services that can be bound to a port. Its an open-source easy to learn framework to get started quick. Alternatives to Flask are **Falcon**, **Tornado**, etc. Now these framework ar packaged as a library for python and are shipped with in-the-box development server which must not be used in production because they are single threaded services and cannot handle multiple requests. Comes in the **gunicorn** and **uWSGI** like HTTP servers to run to solve the developer's nightmare of running python applications on a server. Python applications are given WSGI interface using gunicorn library through simple script like used in the project named as `wsgi.py`: 

```python 
from run import app 
if __name__ == "__main__": 
    app.run() 
``` 

Instead of running the application directly on the **gunicorn** server we'll be running this interface using the following syntax  

```bash 
gunicorn --bind 0.0.0.0:80 wsgi:app 
``` 

on the terminal, in our case it becomes the post build syntax and placed `docker-compose.yaml` file. 

Now let's take a look at our application which is created in `run.py` 
```python 
from flask import Flask, render_template, redirect
from pymongo import MongoClient
from classes import *

# config system
app = Flask(__name__)
# app.config.update(dict(SECRET_KEY='yoursecretkey'))
client = MongoClient(host='dstmongo', port=27017)
db = client.TaskManager

if db.settings.find({'name': 'task_id'}).count() <= 0:
    print("task_id Not found, creating....")
    db.settings.insert_one({'name':'task_id', 'value':0})

def updateTaskID(value):
    task_id = db.settings.find_one()['value']
    task_id += value
    db.settings.update_one(
        {'name':'task_id'},
        {'$set': {
                'value':task_id
                }
            })

def createTask(form):
    title = form.title.data
    priority = form.priority.data
    shortdesc = form.shortdesc.data
    task_id = db.settings.find_one()['value']
    
    task = {'id':task_id, 'title':title, 'shortdesc':shortdesc, 'priority':priority}

    db.tasks.insert_one(task)
    updateTaskID(1)
    return redirect('/')

def deleteTask(form):
    key = form.key.data
    title = form.title.data

    if(key):
        print(key, type(key))
        db.tasks.delete_many({'id':int(key)})
    else:
        db.tasks.delete_many({'title':title})

    return redirect('/')

def updateTask(form):
    key = form.key.data
    shortdesc = form.shortdesc.data
    
    db.tasks.update_one(
        {"id": int(key)},
        {"$set":
            {"shortdesc": shortdesc}
        }
    )

    return redirect('/')

def resetTask(form):
    db.tasks.drop()
    db.settings.drop()
    db.settings.insert_one({'name':'task_id', 'value':0})
    return redirect('/')

@app.route('/', methods=['GET','POST'])
def main():
    # create form
    cform = CreateTask(prefix='cform')
    dform = DeleteTask(prefix='dform')
    uform = UpdateTask(prefix='uform')
    reset = ResetTask(prefix='reset')

    # response
    if cform.validate_on_submit() and cform.create.data:
        return createTask(cform)
    if dform.validate_on_submit() and dform.delete.data:
        return deleteTask(dform)
    if uform.validate_on_submit() and uform.update.data:
        return updateTask(uform)
    if reset.validate_on_submit() and reset.reset.data:
        return resetTask(reset)

    # read all data
    docs = db.tasks.find()
    data = []
    for i in docs:
        data.append(i)

    return render_template('home.html', cform = cform, dform = dform, uform = uform, \
            data = data, reset = reset)

if __name__=='__main__':
    app.run(debug=True)

``` 

Aside from the fact that this application has Controller and View routing it has Model in the `classes.py` file which utlizes **Flask-WTF** library to create Models for frontend form data, we have connection to **MongoDB** database running on port `27017` in a container named `dstmongo` (which we'll see how to spin out in our docker-compose section). 
The file `classes.py` can be seen here serves as the Model for our application and contains field mapping for the **MongoDB** collection document field.
```python
from flask_wtf import FlaskForm
from wtforms import TextField, IntegerField, SubmitField

class CreateTask(FlaskForm):
    title = TextField('Task Title')
    shortdesc = TextField('Short Description')
    priority = IntegerField('Priority')
    create = SubmitField('Create')

class DeleteTask(FlaskForm):
    key = TextField('Task ID')
    title = TextField('Task Title')
    delete = SubmitField('Delete')

class UpdateTask(FlaskForm):
    key = TextField('Task Key')
    shortdesc = TextField('Short Description')
    update = SubmitField('Update')

class ResetTask(FlaskForm):
    reset = SubmitField('Reset')
```
Each class in this script refers to a CRUD action to be done on the document, while each field is a reference to all the fields that that are affected by the actions. 

We have already seen the Controller and Model part of our app, next comes the View which we'll keep simple just by an html page which is dynamically populated by use of double curly braces like this `{{<dynamic content here> }}` . The html file is kept in a folder named `templates` 



[Work In Progress] 

 
