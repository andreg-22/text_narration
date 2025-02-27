# AWS Text-to-Speech Project

## Overview
This project demonstrates how to convert text to speech using AWS services such as Lambda, Polly, S3, and IAM. The project involves creating an IAM role with the necessary permissions, a Lambda function to process the text and convert it to speech, and an S3 bucket to store the resulting audio files.

## Architecture
1. **IAM Role**: An IAM role was created to grant the Lambda function the necessary permissions to call Amazon Polly and put objects in an S3 bucket.
2. **Lambda Function**: A Lambda function was created using Node.js to convert text to audio (mp3) using Amazon Polly and store the output in an S3 bucket.
3. **Amazon Polly**: Used to convert the input text into speech.
4. **S3 Bucket**: An S3 bucket was created in the US East 1 region to store the converted mp3 files.

## Setup

### Step 1: Create IAM Role
1. Create an IAM role with the following policies:
    - `AmazonPollyFullAccess`
    - `AmazonS3FullAccess`
    - `AWSLambdaBasicExecutionRole`
2. Attach the role to the Lambda function.

### Step 2: Create S3 Bucket
1. Create an S3 bucket in the US East 1 region to store the mp3 files.

### Step 3: Create Lambda Function
1. Create a Lambda function using Node.js runtime.
2. Add the following code to the Lambda function:

```javascript
const { PollyClient, SynthesizeSpeechCommand } = require("@aws-sdk/client-polly");
const { S3Client, PutObjectCommand } = require("@aws-sdk/client-s3");
const { Readable } = require("stream");

const REGION = "us-east-1"; // Change to your AWS region

const polly = new PollyClient({ region: REGION });
const s3 = new S3Client({ region: REGION });

exports.handler = async (event) => {
    try {
        if (!event.text) {
            return {
                statusCode: 400,
                body: JSON.stringify({ message: "Missing text input" })
            };
        }

        const text = event.text;

        // Polly parameters
        const pollyParams = {
            Text: text,
            OutputFormat: "mp3",
            VoiceId: "Joanna"
        };

        // Generate speech using Polly
        const pollyCommand = new SynthesizeSpeechCommand(pollyParams);
        const data = await polly.send(pollyCommand);

        if (!data.AudioStream) {
            throw new Error("Polly did not return an AudioStream.");
        }

        // Convert Polly AudioStream into a buffer
        const audioBuffer = await streamToBuffer(data.AudioStream);

        // Generate a unique filename for S3
        const key = `audio-${Date.now()}.mp3`;

        // Upload to S3
        const s3Params = {
            Bucket: "polly-audio-files-for-project2.1",
            Key: key,
            Body: audioBuffer,
            ContentType: "audio/mpeg"
        };

        const s3Command = new PutObjectCommand(s3Params);
        await s3.send(s3Command);

        return {
            statusCode: 200,
            body: JSON.stringify({
                message: `The audio file has been stored as ${key}`,
                fileUrl: `https://${s3Params.Bucket}.s3.${REGION}.amazonaws.com/${key}`
            })
        };
    } catch (error) {
        console.error("Error:", error);
        return {
            statusCode: 500,
            body: JSON.stringify({ message: "Internal server error", error: error.message })
        };
    }
};

// Helper function to convert a stream to a buffer
const streamToBuffer = async (stream) => {
    const chunks = [];
    for await (const chunk of Readable.toWeb(stream)) {
        chunks.push(chunk);
    }
    return Buffer.concat(chunks);
};
```

### Step 4: Test the Setup
1. Trigger the Lambda function with a test event containing the text to be converted.
2. Verify that the mp3 file is uploaded to the S3 bucket.

## Conclusion
This project demonstrates how to use AWS Lambda, Polly, S3, and IAM to create a text-to-speech conversion service. By following the steps outlined above, you can set up a similar service in your AWS environment.

## License
This project is licensed under the MIT License.
