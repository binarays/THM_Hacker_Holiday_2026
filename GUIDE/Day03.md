
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
