<p align="center">
  <img src="https://tryhackme.com/_next/image?url=https%3A%2F%2Fcdn-images.tryhackme.com%2Froom-icons%2F5dbea226085ab6182a2ee0f7-1785251342649&w=96&q=75">
</p>

# Complimentary


## FINAL STEP
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
> The console prints the DynamoDB records of customers; then you can find your lab

# Congratulations on your Exploration 🎉
