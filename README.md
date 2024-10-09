# DiscordBotReceiver--EC2

## Overview

This README provides a step-by-step guide to setting up an EC2 instance and creating a fully functioning Discord bot that connects to the Discord gateway and sends messages to an AWS SQS/FIFO queue.

## Table of Contents
- [Step 1: Launch an EC2 Instance](#step-1-launch-an-ec2-instance)
- [Step 2: Set Up the EC2 Instance](#step-2-set-up-the-ec2-instance)
- [Step 3: Clone the Repository](#step-3-clone-the-repository)
- [Step 4: Set Up the Node.js Development Environment](#step-4-set-up-the-nodejs-development-environment)
- [Step 5: Set Up AWS SQS FIFO Queue](#step-5-set-up-aws-sqs-fifo-queue)
- [Step 6: Open the Discord Channel and Send a Test Message](#step-6-open-the-discord-channel-and-send-a-test-message)

## Step 1: Launch an EC2 Instance

1. **Log in to AWS Management Console**:
   - Go to the EC2 Dashboard and click on **Launch Instance**.

2. **Choose an Amazon Machine Image (AMI)**:
   - Select **Amazon Linux 2** (it is free-tier eligible and suitable for your requirements).

3. **Choose an Instance Type**:
   - Select **t2.micro** instance, as this is a low-cost instance that qualifies for free-tier usage. This instance is sufficient for a bot that isn't resource-intensive.

4. **Configure Instance Details**:
   - Ensure you choose the correct VPC and subnet.
   - **Auto-assign Public IP**: Select **Enable**.
   - **Shut down when not in use**: Create an IAM role for the instance that allows it to be stopped automatically when not in use. You can manage shutdown policies using Lambda or CloudWatch later.

5. **Add Storage**:
   - Stick with the default **8 GB (General Purpose SSD)** or whatever you feel is suitable for the bot's temporary logs and files.

6. **Configure Security Group**:
   - Create a security group that allows inbound traffic on:
     - Port **443** (for WebSocket connections) or **80** if not using HTTPS.
     - Port **22** for SSH access.
   - You can leave all outbound traffic open.

7. **Assign an Elastic IP**:
   - After launching, navigate to the **Elastic IPs** section in the EC2 dashboard.
   - Allocate a new Elastic IP and associate it with your EC2 instance. This will ensure that the EC2 instance has a static IP address, regardless of restarts.

8. **Launch the Instance**:
   - Click **Launch** and select or create a new key pair to connect to your instance.
   - Launch the instance and wait for it to be initialized.

## Step 2: Set Up the EC2 Instance

1. **SSH into your EC2 instance**:
   - Use the key pair you created to SSH into the instance:
     ```bash
     ssh -i "your-key.pem" ec2-user@your-ec2-ip-address
     ```

2. **Update your instance and install necessary software**:
   - Run the following commands to update your instance and install Node.js and npm (necessary for running the bot):
     ```bash
     sudo yum update -y
     sudo amazon-linux-extras install epel -y
     sudo yum install nodejs npm -y
     ```

3. **Install AWS CLI (optional)**:
   - If you want to interact with AWS services (like SQS) via the command line, you can install the AWS CLI:
     ```bash
     sudo yum install aws-cli -y
     aws configure
     ```

## Step 3: Clone the Repository

1. **Install Git**:
   - If Git is not installed, run:
     ```bash
     sudo yum install git -y
     ```

2. **Clone the Repository**:
   - Clone your Discord bot repository:
     ```bash
     git clone https://github.com/yourusername/DiscordBotReceiver--EC2.git
     cd DiscordBotReceiver--EC2
     ```

## Step 4: Set Up the Node.js Development Environment

1. **Install Dependencies**:
   - Run the following command to install the necessary packages:
     ```bash
     npm install
     ```

2. **Create a `.env` File**:
   - Create a `.env` file in the root directory and add your environment variables (e.g., AWS credentials, bot tokens).

3. **Run the Application**:
   - To build the project, run:
     ```bash
     npm run build
     ```
   - Start the application:
     ```bash
     npm run start
     ```

4. **Development Mode** (optional):
   - If you want to run the application in development mode with live reloading, use:
     ```bash
     npm run dev
     ```

## Step 5: Set Up AWS SQS FIFO Queue

1. **Go to the SQS Dashboard**:
   - In AWS, navigate to **Simple Queue Service (SQS)** and click **Create Queue**.

2. **Create a FIFO Queue**:
   - **Queue Name**: Choose a name for your queue (make sure it ends with `.fifo`).
   - **Message Grouping**: Ensure you use message group IDs to avoid any potential processing issues, as FIFO queues are strict with order.
   - **Deduplication**: Either use content-based deduplication or generate your own message deduplication IDs.
   - Note down the **Queue URL** as it will be required when sending messages from your bot. Please update Queue URL in the sqsService.ts

## Step 6: Open the Discord Channel and Send a Test Message

1. **Open Discord**:
   - Open your Discord application or web client and navigate to the channel where your bot is present.

2. **Send a Test Message**:
   - In the Discord channel, type a test command that your bot is programmed to respond to (e.g., `!test`). This should trigger the bot to send a message to the SQS queue.

3. **Verify SQS Queue**:
   - Go back to the AWS SQS Dashboard and check the messages in your FIFO queue to confirm that the message was sent successfully.

---
