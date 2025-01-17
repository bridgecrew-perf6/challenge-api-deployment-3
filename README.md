# Challenge-api-deployment
The real estate company ['ImmoEliza'](https://immoelissa.be/) was really happy about our previously made regression model. They would like you to create an API to let their web-devs create a website around it.

# Description (why)
Housing prices prediction is an essential element for an economy. Analysis of price ranges influence both sellers and buyers.<br> 
The API is stored on [Heroku](http://roberta-eliza.heroluapp.com/) and is documented [here](https://github.com/FrancescoMariottini/challenge-api-deployment/blob/main/Documentation.md).

Our [previous project](https://github.com/FrancescoMariottini/Belgium-prices-prediction/settings) created a Linear Regression model to estimate the price of the house accurately with the given features. Within this project an online API to run the price prediction is made. 

An overview of the project is available as the [Google presentation](https://docs.google.com/presentation/d/1Q2EqDDF23fEurSE9UoajGYLVUUjL3RoalAZpDN98q6E/edit#slide=id.p2). 

# Target audience (to whom)
['ImmoEliza'](https://immoelissa.be/) web developers to receive price prediction based on input values provided by a user.

# Objectives (what)
1. Be able to deploy a machine learning model.
2. Be able to create a Flask API that can handle a machine learning model.
3. Deploy an API to Heroku with Docker.

Project evalution is based on the compliance to the following criteria:

|Criteria|Indicator|
|---|---| 
|Is complete|Your API works.|	
||Your API is wrapped in a Docker image.|
||Pimp up the readme. (what, why, how, who).|
||Your model predict.|
||Your API is deployed on Heroku.|
|Is good |The repo doesn't contain unnecessary files.|
||You used typing.|
||The presentation is clean.|
||The web-dev group understood well how your API works.|

# Development (how)
Input requirements for the web developers were initially agreed with the other Becode teams. Then the team agreed on how to fulfill and split the required development steps among its members. Step 0 (team management) was added.

0. Team organization
1. Project preparation
2. Scrapping immoweb website for updates
3. Cleaning dataset for database storing
4. Set up of an online database
5. Querying the online database
6. Pre-processing pipeline
7. Fit your data!
8. Create your API
9. Create a Dockerfile to wrap your API
10. Deploy your Docker image in Heroku
11. Document your API

For step 2 and 3 [source files](https://github.com/FrancescoMariottini/Belgium-prices-prediction/tree/main/source) from the previous project were used as basis after adapting them in in line with the JSON input requirements.

## Input requirements
Hereby follow the agreed requirements for the json to be provided by the web developers:

```json
{
"data": {
"area": int,
"property-type": "APARTMENT" | "HOUSE" | "OTHERS",
"rooms-number": int,
"zip-code": int,
"land-area": Optional[int],
"garden": Optional[bool],
"garden-area": Optional[int],
"equipped-kitchen": Optional[bool],
"full-address": Optional[str],
"swimmingpool": Opional[bool],
"furnished": Opional[bool],
"open-fire": Optional[bool],
"terrace": Optional[bool],
"terrace-area": Optional[int],
"facades-number": Optional[int],
"building-state": Optional["NEW" | "GOOD" | "TO RENOVATE" | "JUST RENOVATED" | "TO REBUILD"]
}
}
```
Output requirements were not strictly fixed but rather delegated to each team after sharing a reference template:

```json
response = {
  prediction: {
    price: int,
    test_size: int,
    median_absolute_error: float,
    max_error: float,
    percentile025: float,
    percentile975: float
  },
  error: str
}
```
A HTTP status code is also provided in case of error.

## Step 0: Team organization ##
In the introduction meeting it emerged that team members were looking for an organisation to avoid overlaps and meet project requirements without rushing for it.

A Trello board (Kanban template) was organised and team members agreed on how to split the required development steps. For each step was responsible person was chosen, support could be provided if requested. A trello list with general useful links, guidelines and tips was provided.

Morning meeting were scheduled to check the status of the project and plan the day activites. Afternon meetings were scheduled at lunch if morning tasks were completed to set up new goal.

An interim review was set up on 4/12/20 to upload an already working version before refining it. 

### Highlights ###
Organisation and coordination after deploying the first deployment on Heroku on 4/12/2020 posed a challenge since many small improvements had to be tested which were overlapping.

## Step 1: Project preparation ##
A repository was prepared to fullfill the required requirements: 
1. Create a folder to handle your project.
1. Create a file app.py that will contain the code for your API.
1. Create a folder preprocessing that will contain all the code to preprocess your data.
1. Create a folder model that will contain your model.
1. Create a folder predict that will contain all the code to predict a price.

All these main folders, exclusively dedicated to the api, were created in a **source** folder. Additional folders (**assets**, **data**, **docs** and **outpus**) were created for the project.

### Highlights ###
Even with git connection problems it was still possible to push for ordered changes in the repository through github.
It wasn't initially clear how to split the code between the model creation/evaluation and the API service thus some additional time was spent on reconverting the files structure after.

## Step 2: Scrapping immoweb website for updates ##
The code from our [immoweb scraping challenge](https://github.com/FrancescoMariottini/challenge-collecting-data) at becode was integrated to run inside a docker. It is based on the [Selenium Webdriver](https://pypi.org/project/selenium/). We had to change it to work with firefox rather than chromium, because the headless mode of chromium returned anti-bot captcha pages and firefox did not.

## Step 3: Cleaning dataset for database storing ##
### Highlights ###

## Step 4: Set up of an online database ## 
### Highlights ###

## Step 5: Querying the online database ## 
### Highlights ###

## Step 6: Pre-processing pipeline ##
The DataFeatures class in validation.py serves to validate if the client stuck to the required json schema and stayed within realistic value ranges. It has operations for returning human readable errors and returning a corrected json, where for example "0" is evaluated as integer 0. The class is based on the [marshmallow](https://github.com/marshmallow-code/marshmallow) library.

The file cleaning_data.py contains all the code used to preprocess the validated request received to predict a new price. Optional values not provided by the client but needed by our model are imputed to be 0 (ex. does not have swimmingpool) or 1 (ex. garden-area is 0). We use 1 to indicate a zero area value because the model feature is actually log(area).

### Highlights ###
The pre-processing was split into two distinguished step, the validation of the request and then the formatting of the values after to comply with the model requirements.

## Step 7: Fit your data! ##
In the predict folder a file prediction.py contains all the code used to predict a new house's price. The file contains a function predict() that takes the preprocessed data as an input and returns a price as output.

### Highlights ###
Instead of providing only a single model, one model for property-type was provided. Models performance were tested only once and then stored as csv in the model folder to be retrieved later.

## Step 8: Create your API ##
In the app.py file, the Flask API contains:
* A route at / that accept:
    * GET request and return "alive" if the server is alive.
* A route at /predict that accept:
    * POST request that receives the data of a house in json format.
    * GET request returning a string to explain what the POST expect (data and format).
    
The complete documentation about the API is available [here](https://github.com/FrancescoMariottini/challenge-api-deployment/blob/main/Documentation.md).
    
### Highlights ###

## Step 9: Create a Dockerfile to wrap your API ##
To deploy the API Docker was used.
The Dockerfile created an image with Ubuntu and Python 3.8 plus all the required dependencies for the created code:
library|version
click|7.1.2
Flask|1.1.2
gunicorn|20.0.4
itsdangerous|1.1.0
Jinja2|2.11.2
MarkupSafe|1.1.1
marshmallow|3.9.1
numpy|1.19.4
pandas|1.1.4
python-dateutil|2.8.1
pytz|2020.4
six|1.15.0
Werkzeug|1.0.1

### Highlights ###
First we had to find a base Dockerimage. While we found an existing image with ubuntu and python 3.8 already installed, it was too big (1.2 Go)
so we opted in the end to start from the latest official ubuntu image that already had python 3.8 too, installing python3-pip ourselves.

## Step 10: Deploy your Docker image in Heroku ##
Heroku allowed to push the docker container on their server and to start it (more information [here](https://github.com/becodeorg/BXL-Bouman-2.22/tree/master/content/05.deployment/4.Web_Application)).

### Highlights ###
Making log messages available in the heroku web UI requird special attention. It turned out python print statements by themself were not reflected and stdout additionally needed flushing.

## Step 11: Document your API ##
API is documented [here](https://github.com/FrancescoMariottini/challenge-api-deployment/blob/main/Documentation.md).

### API FAQ ##
Hereby follow the answers to the main questions about the API.

**What routes are available? With which methods?**<br>
We have 2 routes available you'll find more information on them [here](https://github.com/FrancescoMariottini/challenge-api-deployment/blob/new/Documentation.md#alive)<br>

**What kind of data is expected (How should they be formatted?**<br>
Here's a [link](https://github.com/FrancescoMariottini/challenge-api-deployment/blob/new/Documentation.md#request-entity) to the expect data entity.
Some fields are mandatory and some must apply some validation conditions.<br>

**What is the output of each route in case of success?**<br>
Here's a [link](https://github.com/FrancescoMariottini/challenge-api-deployment/blob/new/Documentation.md#return-entity) to more deatail about about the return entity<br>

**What is the output in case of error?**<br>
Here's a [link](https://github.com/FrancescoMariottini/challenge-api-deployment/blob/new/Documentation.md#errors) to all the possible errors <br>

# Conclusions
First release of the API was made easier by using information from the previous project and by splitting the work among all the team members.
Additional small improvements posed a challenge since their impact had to be verifed in all the different steps involving different platforms/tools (comparing to a simpler code integration). 
Extension for database storing and update was hampered by the reduced working time due to unplanned Becode events and team members' personal reasons.

# Future steps
Project is considered concluded and no additional work is not foreseen. However a few possible improvements on the modelling part are hereby suggested to improve its accuracy:
1. scrapping more data online including also other key parameters (e.g. building construction year)
1. make full use of other available reliable datasets (e.g. official statistics) to improve the model
1. replace linear regression (degree equal to one) with a polynomial when relevant
1. use logaritmic transformations only when relevant

# Timeline (when): 
- 2/12/2020 (start)
- 9/12/2020 (code deliverable)
- 11/12/2020 (extension for database storing and update). Only 5 person-days available from 10/12/20 to 11/12/20 due to unplanned Becode events and team members' personal reasons.
- 14/12/2020 (presentation deliverable)


# Team:
[Ankita Haldia](https://www.linkedin.com/in/ankitahaldia/)
[Francesco Mariottini](https://www.linkedin.com/in/francescomariottini/)
[Opaps](https://www.linkedin.com/in/opapsditudidi/)
[Philippe](https://www.linkedin.com/in/phfimmers/)
