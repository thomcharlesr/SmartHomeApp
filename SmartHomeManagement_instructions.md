# Smart Home Management - Lab Instructions
The SmartHomeApp is a cloud-based solution designed to enable real-time control, monitoring, and logging of smart home devices. Below are the organized steps taken to build the backend of the app using AWS services.

**1. Database: Amazon DynamoDB for Data Storage**

Amazon DynamoDB was chosen as the primary storage solution to replace the originally planned Amazon RDS. DynamoDB offers a fast, scalable, and managed NoSQL database that allows for efficient data management without the complexities of a relational database.

Step 1.1: Create DynamoDB Tables
UserSettings Table: Stores individual device states and configurations, making it easy to update and track each device in a user’s environment.
Table Name: UserSettings
Partition Key: UserID (String)
Sort Key: DeviceID (String)
DeviceLogs Table: Serves as a dedicated log repository to record every action or change in device state, providing a historical log separate from the main settings.
Table Name: DeviceLogs
Partition Key: LogID (String)
Sort Key: DeviceID (String)
Auto-scaling is enabled for both tables to manage read/write capacity automatically.

**2. Backend Logic: AWS Lambda for Device Control**

AWS Lambda functions were used to implement backend logic for handling device state updates and notifications. This allows the app to execute actions and maintain device states in real-time.

Step 2.1: Create Lambda Functions

Function Name: UpdateDeviceState
Runtime: Python 3.9
Permissions: Attach the LabRole to enable Lambda access to DynamoDB.
Code Summary:
Retrieves DeviceID and DeviceState from incoming events.
Updates the DeviceState in the UserSettings table in DynamoDB.
Sends a notification to users regarding the updated device state.
Environment Variables:

Configure necessary environment variables such as DB_NAME (UserSettings), SNS_TOPIC_ARN, and placeholders for DB_HOST, DB_USER, and DB_PASS (though not needed, they avoid potential code errors).

**3. Frontend Communication: AWS IoT Core for Device Communication**

AWS IoT Core facilitates real-time device communication, enabling seamless interactions between the app and IoT-enabled smart home devices.

Step 3.1: Set Up IoT Core and Policies

Create an IoT Policy to define permissions for device communication actions such as iot:Connect, iot:Publish, iot:Subscribe, and iot:Receive.
Attach the IoT Policy to the device certificate to allow authorized communication between the device and AWS IoT Core.
Step 3.2: Register IoT Devices (Things)

Register each smart home device as a "Thing" in AWS IoT Core and assign the policy.
Generate device certificates and keys for secure communication between the devices and AWS IoT.
Step 3.3: Configure an IoT Rule

Set up an IoT Rule to capture messages from devices on specific MQTT topics (e.g., device/update) and trigger the UpdateDeviceState Lambda function.
SQL Statement: SELECT * FROM 'device/update'
Action: Invoke the UpdateDeviceState Lambda function, allowing device state updates to be synchronized with the backend database.
4. Notifications: Amazon SNS for User Alerts
Amazon Simple Notification Service (SNS) was used to send notifications to users about device status changes, ensuring timely updates for critical device activities.

**Step 4.1: Set Up SNS Topic**

Topic Name: DeviceNotification
Create a new SNS topic for device notifications and configure subscriptions for user notifications (e.g., via email or SMS).
Add the SNS Topic ARN as an environment variable (SNS_TOPIC_ARN) in the Lambda function for easy integration.
Lambda Integration with SNS:

After each device state update, the Lambda function sends a notification to the SNS topic, informing users of the latest device status.

**5. Backend Logging: Use DynamoDB for Log Storage Instead of S3**

Due to role-based limitations, Amazon DynamoDB was selected as an alternative to Amazon S3 for log storage. DynamoDB offers seamless integration with Lambda and requires no additional permissions, making it an efficient choice for storing event logs.

Step 5.1: Set Up DynamoDB Table for Logging

Table Name: LambdaLog
Partition Key: LogID (String)
Step 5.2: Modify Lambda Function for Logging

Update the Lambda function to write event logs to the LambdaLog table in DynamoDB after each device state change.
Log details include LogID, Timestamp, DeviceID, and DeviceState, providing a reliable record of device activities.

**6. Monitoring and Logging: Amazon CloudWatch for System Monitoring**

Amazon CloudWatch was used to monitor and log the execution of Lambda functions, providing insights into application performance and event handling.

Step 6.1: Configure CloudWatch for Lambda Monitoring

CloudWatch Logs track every Lambda invocation, including the UpdateDeviceState function, allowing you to review event flow, monitor errors, and check system health.

Step 6.2: Testing and Verifying Logs in CloudWatch

Lambda Execution Verification: View Lambda execution logs in CloudWatch to confirm successful processing of messages and database updates.
SNS Notification Check: Verify that SNS notifications were sent by viewing CloudWatch logs for each Lambda run.
Device Communication Testing: Subscribe to MQTT topics in AWS IoT Core and publish test messages to ensure that device updates trigger Lambda functions and generate corresponding logs.

**Summary**
By following these steps, the SmartHomeApp backend was built with a robust architecture for real-time device control, data storage, notification management, and monitoring. With the integration of AWS services like DynamoDB, Lambda, IoT Core, SNS, and CloudWatch, this solution provides a secure, scalable, and responsive foundation for smart home management.

**Project Achievements:**

Device Control: Instantaneous device updates and settings management.
Data Storage: Reliable storage and retrieval of user/device configurations and logs.
User Notifications: Timely alerts on device status changes.
Logging and Monitoring: Detailed tracking of events, system health, and error management.