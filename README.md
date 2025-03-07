# Blog-2 - Generate and evaluate images in Amazon Bedrock with Amazon Titan Image Generator G1 v2 and Anthropic Claude 3.5 Sonnet 

# What is it about ?
Generating and evaluating images in Amazon Bedrock with Amazon Titan Image Generator G1 v2 and Anthropic Claude 3.5 Sonnet

# Requirements
An AWS account to create and manage the necessary AWS resources for this solution

Amazon Nova Canvas and Anthropic Claude 3.5 Sonnet models enabled on Amazon Bedrock in AWS Region us-east-1

The IAM user access key and secret key to configure the AWS CLI and permissions

AWS CLI installed and configured 

Python version 3.8 with IDE

The latest Boto3 library


# Steps/Actions

# Steps on AWS Management Console
Sign in to the AWS Management Console as an IAM administrator or appropriate IAM user.

Create a Stack to deploy the YAML file that contains contains the infrastructure, including AWS Identity and Access Management (IAM)  users, policies, API methods, the S3 bucket, and the Lambda function code. provided in the CloudFormation template.

# Steps to Create a Stack
First remove the ReservedConcurrentExecutions line in all of it's two occurance in the provided YML file in Launch Stack.
Then Create a S3 Bucket that will hold this YML file.

Choose Create Stack in CloudFormation to create a new stack.

provide the S3 object url of the YML file and choose next.

In the Parameters section, enter the following:
A name for the new S3 bucket that will receive the images (for example, image-gen-your-initials)
The name of an existing S3 bucket where access logs will be stored.
A token that you will use to authenticate your API (a string of your choice)

After entering the parameters, choose Next.
Choose Next again.

Acknowledge the creation of IAM resources and choose Submit.

When the stack status is CREATE_COMPLETE, navigate to the Outputs tab and find the API information. Copy the ApiId, the ApiUrl and ResourceId to a safe place and continue to test.

![image_2025_03_05T07_07_20_658Z](https://github.com/user-attachments/assets/8ccc5bb3-9665-470a-99bf-ba64fd9aea85)

# Evaluating and Testing Images using Console

## Steps
On the API Gateway console, choose APIs in the navigation pane.

On the APIs list, choose BedrockImageGenEval.

In the Resources section, select the POST method below /generate-image.

Choose the Test tab in the method execution settings.

In the Request body section, enter the following JSON structure:
{ “prompt”:”your prompt” }

Choose Test.

![image_2025_03_05T07_07_20_668Z](https://github.com/user-attachments/assets/4b109ac7-0135-48b6-8d8c-dd55ecf19b38)

## OUTPUT
![image_2025_03_05T07_07_20_670Z](https://github.com/user-attachments/assets/8ced6c69-38a5-4e83-b32b-a039c9435816)


# Blog-2 - Additional 1 - Generating Images using Amazon Nova Canvas
## Create a python file
create  python file named main.py or any other name

## Creating virtual environment in python

Use virtual environment to maintain packages across different projects

```bash
python -m venv venv
```

Activate virtual Enviroment

```bash
venv\Script\activate
```

## Installation of Boto3

Installing boto3 on our virtual environment

```bash
pip install boto3
```

## Code

```python
import boto3
import json
import base64
import io
Import uuid
from PIL import Image

# Set up the Amazon Bedrock client
bedrock_client = boto3.client(
    service_name="bedrock-runtime",
    region_name="us-east-1"
)

# Define the model ID
model_id = "amazon.nova-canvas-v1:0"

# Prepare the input prompt
prompt = "a black cat in an alleyway with blue eyes."

# Create the request payload
body = json.dumps({
        "taskType": "TEXT_IMAGE",
        "textToImageParams": {
            "text": prompt
        },
        "imageGenerationConfig": {
            "numberOfImages": 1,
            "height": 1024,
            "width": 1024,
            "cfgScale": 8.0,
            "seed": 0
        }
    })

accept = "application/json"
content_type = "application/json"

# Invoke the Amazon Bedrock model
response = bedrock_client.invoke_model(
    modelId=model_id,
    body=body,
    accept=accept, 
    contentType=content_type
)

# Process the response
result = json.loads(response["body"].read())

base64_image = result.get("images")[0]
base64_bytes = base64_image.encode('ascii')
image_bytes = base64.b64decode(base64_bytes)

image = Image.open(io.BytesIO(image_bytes))
image.show()
unique_id = uuid.uuid4()
image.save(f"D:\Data-AI Blogs\Blog-2\generated-images\{unique_id}.png")

print(f"Generated Image is Stored Successfully at : D:\Data-AI\Blogs\Blog-2\generated-images\{unique_id}.png")
```

## Run File 

```bash
python [NAME_OF_FILE].py
```

## OUTPUT

Generated Image is Stored Successfully at : D:\Data-AI Blogs\Blog-2\generated-images\e4e392df-27a1-4fe2-a714-8ba9a68566b4.png

![610ef92b-b419-4a58-a3da-f5d62cff8265](https://github.com/user-attachments/assets/da7d2bb7-475c-46d8-8550-e56a7b49e696)


# Blog-2 - Additional 2 - Storing the Generated Images in Amazon S3 Bucket

## Code

```python
import boto3
import json
import base64
import io
import uuid
from PIL import Image

# Set up the Amazon Bedrock client
bedrock_client = boto3.client(
    service_name="bedrock-runtime",
    region_name="us-east-1"
)

# Define the model ID
model_id = "amazon.nova-canvas-v1:0"

# Prepare the input prompt
prompt = "create me an image of indian flag which should be held by cricketer ms dhoni in indian cricket team jersey on the moon"

# Create the request payload
body = json.dumps({
        "taskType": "TEXT_IMAGE",
        "textToImageParams": {
            "text": prompt
        },
        "imageGenerationConfig": {
            "numberOfImages": 1,
            "height": 1024,
            "width": 1024,
            "cfgScale": 8.0,
            "seed": 0
        }
    })

accept = "application/json"
content_type = "application/json"

# Invoke the Amazon Bedrock model
response = bedrock_client.invoke_model(
    modelId=model_id,
    body=body,
    accept=accept, 
    contentType=content_type
)

# Process the response
result = json.loads(response["body"].read())

base64_image = result.get("images")[0]
base64_bytes = base64_image.encode('ascii')
image_bytes = base64.b64decode(base64_bytes)

image = Image.open(io.BytesIO(image_bytes))
image.show()

s3 = boto3.client("s3")
id = uuid.uuid4()

bucket_name = "blog-2-bucket-for-image-storage"
object_path = f"generated-images/{id}.png"

response = s3.put_object(
            Bucket=bucket_name,
            Key=object_path,
            Body=image_bytes,
            ContentType='image/jpeg'
        )

object_url = f"https://{bucket_name}.s3.us-east-1.amazonaws.com/{object_path}"
print(f"Generated Image is Stored Successfully at S3 location : {object_url}")
```

## OUTPUT

Generated Image is Stored Successfully at S3 location : https://blog-2-bucket-for-image-storage.s3.us-east-1.amazonaws.com/generated-images/aca33f19-c5e3-464a-8e95-08093960887b.png

![image](https://github.com/user-attachments/assets/59b23328-94f1-42b5-838f-0aae78566441)

![image](https://github.com/user-attachments/assets/596aaf64-b936-4c93-b253-ad7a32f1de47)


# Blog-2 - Additional 3 - Modifying Generated Images using Amazon Titan Image Generator

## Code

```python
import boto3
import json
import base64
import io
import os
from PIL import Image
import uuid

# Initialize Amazon Bedrock client
bedrock_client = boto3.client("bedrock-runtime", region_name="us-east-1")

# Define the model ID
model_id = "amazon.titan-image-generator-v2:0"

# Step 1: Generate Initial Image
# Prepare the input prompt
initial_prompt = "Create me an image of a cricket player with bat in one hand which is held on the shoulder and helmet in another hand and the player should have wear blue jersey with team logo of mumbai indians"

# Create the request payload
payload = {
    "taskType": "TEXT_IMAGE",
    "textToImageParams": {"text": initial_prompt},
    "imageGenerationConfig": {
        "numberOfImages": 1,
        "height": 1024,
        "width": 1024,
        "cfgScale": 8.0,
        "seed": 0
    }
}

# Invoke the Amazon Titan Image Generator v2 model
response = bedrock_client.invoke_model(
    modelId=model_id,
    body=json.dumps(payload),
    accept="application/json",
    contentType="application/json"
)

# Process the response
result = json.loads(response["body"].read())

base64_image = result.get("images")[0]
base64_bytes = base64_image.encode('ascii')
image_bytes = base64.b64decode(base64_image)

initial_image = Image.open(io.BytesIO(image_bytes))
initial_image.show()

unique_id = uuid.uuid4()
os.mkdir(f"generated-and-modified-images/{unique_id}")

initial_image_path = f"generated-and-modified-images/{unique_id}/generated_image.png"
initial_image.save(initial_image_path)
print(f"Generated image saved at: {initial_image_path}")

# Step 2: Modifying the Generated Image (with an updated prompt)
modified_prompt = "Change the player's complete jersey color to yellow and team logo name to chennai super kings in the previously generated image."

with open(f"D:\Data-AI Blogs\Blog-2\generated-and-modified-images\{unique_id}\generated_image.png","rb") as image_file:
    base64_image = base64.b64encode(image_file.read()).decode("utf-8")

# Update Payload for Image Updation
payload = {
     "taskType": "IMAGE_VARIATION",
     "imageVariationParams": {
         "text": modified_prompt,
         "images": [base64_image],
         "similarityStrength": 1.0,  # Range: 0.2 to 1.0
     },
     "imageGenerationConfig": {
         "quality": "premium",
         "numberOfImages": 1,
         "height": 1024,
         "width": 1024,
         "cfgScale": 8.0
     }
}

response = bedrock_client.invoke_model(
    modelId=model_id,
    body=json.dumps(payload),
    accept="application/json",
    contentType="application/json"
)

result = json.loads(response["body"].read())

base64_modified_image = result.get("images")[0]
base64_bytes = base64_image.encode('ascii')
modified_image_bytes = base64.b64decode(base64_modified_image)

# Save and display the modified image
modified_image = Image.open(io.BytesIO(modified_image_bytes))
modified_image.show()

modified_image_path = f"generated-and-modified-images/{unique_id}/modified_image.png"
modified_image.save(modified_image_path)
print(f"Modified image saved at: {modified_image_path}")
```

## Output

Generated image saved at: generated-and-modified-images/6ba0e04c-def5-4efb-a710-83b35d2231f2/generated_image.png

![generated_image](https://github.com/user-attachments/assets/ceddb0bd-6393-4b27-a8da-03ea185de36f)

Modified image saved at: generated-and-modified-images/6ba0e04c-def5-4efb-a710-83b35d2231f2/modified_image.png

![modified_image](https://github.com/user-attachments/assets/697d98d9-b297-48b8-95ed-c16f4734fb96)
