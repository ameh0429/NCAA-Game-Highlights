# NCAA-Game-Highlights
This project uses RapidAPI to obtain NCAA game highlights using a Docker container and uses AWS Media Convert to convert or enhance the media file.
## File Overview
- **config.py:** This script Imports necessary environment variables and assigns them to Python variables, providing default values where appropriate. This approach allows for flexible configuration management, enabling different settings for various environments (e.g., development, staging, production) without modifying the source code.

- **fetch.py:** Establishes the date and league that will be used to find highlights. I am using NCAA in this project because it's included in the free version. This will fetch the highlights from the API and store them in an S3 bucket as a JSON file (basketball_highlight.json)

- **process_one_video.py:** Connects to the S3 bucket and retrieves the JSON file. Extracts the first video URL from within the JSON file. Downloads the video file from the internet into the memory using the requests library. Saves the video as a new file in the S3 bucket under a different folder (videos/) Logs the status of each step.

- **mediaconvert_process.py:** Creates and submits a MediaConvert job Uses MediaConvert to process a video file - configures the video codec, resolution and bitrate. Also configured the audio settings Stores the processed video back into an S3 bucket.

- **run_all.py:** Runs the scripts in a chronological order and provides buffer time for the tasks to be created.

- **.env file:** stores all the environment variables, these are variables that we don't want to hardcode into our script.

- **Dockerfile:** Provides the step-by-step approach to build the image.

- **Terraform Scripts:** These scripts are used to created resources in AWS in a scalable and repeatable way (Not included in this project)

## Prerequisites 
Before running the scripts, ensure you have the following:
- Create Rapidapi Account to access highlight images and videos https://rapidapi.com/highlightly-api-highlightly-api-default/api/sport-highlights-api/playground/apiendpoint_16dd5813-39c6-43f0-aebe-11f891fe5149
- Verify prerequisites are installed e.g. Docker, AWS CLI, Python3
- Retrieve AWS Account ID

## Architecture Diagram

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/zu1xhmcwnt289rd4ntyt.png)

## Project Structure

```
src/
├── Dockerfile
├── config.py
├── fetch.py
├── mediaconvert_process.py
├── process_one_video.py
├── requirements.txt
├── run_all.py
├── .env
├── .gitignore
```

## Setup Instruction
#### Clone the repository

```
git clone https://github.com/ameh0429/NCAA-Game-Highlights.git
cd NCAA-Game-Highlights
```

#### Add API Key to AWS Secrets Manager

```
aws secretsmanager create-secret \
    --name my-api-key \
    --description "API key for accessing the Sport Highlights API" \
    --secret-string '{"api_key":"YOUR_ACTUAL_API_KEY"}' \
    --region us-east-1
```

#### Create an IAM role or user
This will give the user or role necessary permission to interact with the services used in the project e.g. AmazonS3FullAccess, MediaConvertFullAccess and AmazonEC2ContainerRegistryFullAccess. The thrust policy should be updated:

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": [
          "ec2.amazonaws.com",
          "ecs-tasks.amazonaws.com",
          "mediaconvert.amazonaws.com"
        ],
        "AWS": "arn:aws:iam::<"your-account-id">:user/<"your-iam-user">"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

```

#### Create S3 Bucket

```
aws s3api create-bucket --bucket ameh-video721 --region us-east-1
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/vzjeadqp09e10cor5921.png)

#### Update .env file
This is where we put our sensitive details so that they will not be hardcoded in the repository. Also secure it by running `chmod 600 .env` to prevent unauthorize access to the .env file. Also add the file to .gitignore to prevent it from being committed to version control.
- RapidAPI_KEY: Ensure that you have successfully created the account and select "Subscribe To Test" in the top left of the Sports Highlights API
- AWS_ACCESS_KEY_ID=your_aws_access_key_id_here
- AWS_SECRET_ACCESS_KEY=your_aws_secret_access_key_here
- S3_BUCKET_NAME=your_S3_bucket_name_here
- MEDISCONVERT_ENDPOINT=https://your_mediaconvert_endpoint_here.amazonaws.com
```
aws mediaconvert describe-endpoints
```
- MEDIACONVERT_ROLE_ARN=arn:aws:iam::your_account_id:role/HighlightProcessorRole

#### Locally Build & Run the Docker Container

```
docker build -t highlight-processor
```
![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/kvipros9hayjyn3v2180.png)

```
docker run --env-file .env highlight-processor
```

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/59082p6iv9rwq6mehbkk.png)

This will run fetch.py, process_one_video.py and mediaconvert_process.py and the following files should be saved in your S3 bucket:

#### Confirm video upload in S3

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q0ztk6y2199cg718lvbe.png)

#### Open the Basketball Highlight JSON folder

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/sfdi2zuzfx9lytqupg4v.png)

#### Open the processed video folder
This video quality has been enhanced by AWS Media Convert. 

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/hg8bcr032vjlgv9tnb2k.png)

## What we Learned
- Working with Docker and AWS Services
- Identity Access Management (IAM) and least privilege
- How to enhance media quality

## Future Enhancements
- Using Terraform to enhance the Infrastructure as Code (IaC)
- Increasing the amount of videos process and converted with AWS Media Convert.
