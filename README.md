# mysql_layer_aws_layer_python

#MYSQL AWS LAMBDA Layer ready to import in your AWS lambda code!

In order to use, you need to upload the layer into your AWS Layers.

Go to the AWS Management Console and navigate to the Lambda service.
Click on the "Layers" option in the left-hand menu.
Click on the "Create Layer" button.
Enter a name for your layer and select the "Upload a .zip file" option.
Upload the .zip file containing your layer code and dependencies.
Specify a compatible runtime for your layer (e.g. Python 3.8).
Click on the "Create" button to create your layer.

Once your layer is created, you can add it to your Lambda function by going to the "Layers" section of your function's configuration page and clicking on the "Add a layer" button.
Select your layer from the list of available layers and click on the "Add" button.
Save your changes to the function configuration.

#HOW TO USE IN YOUR CODE:

import mysql.connector

#PYTHON EXAMPLE:

```
import boto3
import logging
import mysql.connector
import os
import time
import datetime
import json
from urllib.parse import urlsplit, parse_qs

logger = logging.getLogger()
logger.setLevel(logging.INFO)

rds_client = boto3.client('rds')

#SECRET MANAGER GET_DB_KEY
def generateToken():
    logging.info('Generate database token...')
    try:
        token = rds_client.generate_db_auth_token(
            DBHostname=os.environ['DB_HOST'],
            Port=3306,
            DBUsername=os.environ['DB_USER'],
            Region=os.environ['AWS_REGION'],
            )
        logging.info('Token successfully obtained. Start complete.')
        return token
    #❌TOKEN_ERROR
    except mysql.connector.Error as err:
    logger.error(err.msg)
    error = "Connection error"
    return log_err(error)

#RDS_CONNECTION
def make_connection():
database_token = generateToken()
    #CHECK_TOKEN_VALID_DATE
    return mysql.connector.connect(
        host=os.environ['DB_HOST'],
        port=3306,
        database=os.environ['DB_NAME'],
        user=os.environ['DB_USER'],
        password=database_token)

#LAMBDA_HANDLER
def lambda_handler(event, context):
    context.callbackWaitsForEmptyEventLoop = False
    logging.info('Connecting to database...')

    #CONNECT
    try:
        cnx = make_connection()
        logging.info('Connected!!')
        cursor = cnx.cursor(dictionary=True)

        #HERE IS YOUR SQL COMMAND LIST

        #CLOSE_DB_CONNECTION_AFTER SQL COMMANDS
        cnx.close()
        #✅SUCCESS
        result = {
            "message": "API WAS SUCCESS",
        }
        return success(result)


    #❌DB_CONNECTION_ERROR
    except mysql.connector.Error as err:
        return log_err(err.msg)
    finally:
        try:
            cnx.close()
        except:
            pass

#ERROR_MESSAGE
def log_err(errmsg):
    logger.error(errmsg)
    return {"body": json.dumps({"error": errmsg}), "headers": {'Content-Type': 'application/json',
    'Access-Control-Allow-Origin':'\*'}, "statusCode": 400,
    "isBase64Encoded": "false"}

#SUCCESS_MESSAGE
def success(result):
    return {'body': json.dumps({"result": result}), 'headers': {'Content-Type': 'application/json',
    'Access-Control-Allow-Origin':'\*'}, 'statusCode': 200,
    'isBase64Encoded': 'false'}
```
