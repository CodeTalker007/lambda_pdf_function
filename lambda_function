import jinja2
import pdfkit
import boto3
from botocore import client
from datetime import datetime
import requests

def lambda_handler(event, context):
    
    # Create a Boto3 session with the provided credentials
    session = boto3.Session(
        aws_access_key_id="AWS_KEY_ID",
        aws_secret_access_key="AWS_SECRET_ID"
    )

    # Getting these values from SQS you can use you as you need
    apiToken = event['Records'][0]['messageAttributes']['access_token']['stringValue']
    url = event['Records'][0]['messageAttributes']['url']['stringValue']
    webhookUrl = event['Records'][0]['messageAttributes']['webhookUrl']['stringValue']

    # Getting Data from Api endpoint
    headers = {'Accept': 'application/json', 'Authorization': 'Bearer ' + apiToken}
    response = requests.get(url, headers=headers)
    data = response.json()
    
    today_date = datetime.today().strftime("%d %b, %Y")
    context = {'data': data, 'today_date': today_date}

    template_loader = jinja2.FileSystemLoader('./')
    template_env = jinja2.Environment(loader=template_loader)
    
    # You can add you html/jinja code in your html template file
    template = template_env.get_template('template.html')
    output_text = template.render(context)

    # You can also add you style sheet while generating PDF
    config = pdfkit.configuration(wkhtmltopdf='/opt/bin/wkhtmltopdf')
    the_pdf = pdfkit.from_string(output_text,
                    configuration=config, css='style.css')

    # Use the session to interact with AWS services
    s3 = session.resource('s3')
    key = "pdfreport/"+data['file_name']
    BUCKET = "BUCKET_NAME"

    bucket = s3.Bucket(BUCKET)
    s3_object = bucket.Object(key)

    # Upload the PDF content to the S3 object
    s3_object.put(Body=the_pdf)
    
    # Payload for the POST request (data to be sent)
    payload = {
        "path": key
    }

    # Calling Webhook to store my S3 Path 
    headers = {
        "Authorization": f"Bearer {apiToken}",
        "Content-Type": "application/x-www-form-urlencoded"
    }
    
    # Send the POST request
    response = requests.post(webhookUrl, data=payload, headers=headers)

    # Check the response status code
    if response.status_code == 200:
        print("POST request was successful" )
    else:
        print(f"POST request failed with status code: {response.status_code}")

    return key
