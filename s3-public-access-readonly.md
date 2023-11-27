# AWS S3 public access read only

Create an AWS S3 bucket policy that allows reading data only from a specific URL, you can use the `aws:Referer` condition key. Here's an example JSON policy that grants read access to objects in the S3 bucket only if the request is referred from a specific URL:

```json
{
  "Version": "2012-10-17",
  "Id": "ExamplePolicy",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*",
      "Condition": {
        "StringLike": {
          "aws:Referer": [
            "http://allowed-url.com/*",
            "https://allowed-url.com/*"
          ]
        }
      }
    }
  ]
}
```

In this example:

- `Effect`: Set to "Allow" to grant permission.
- `Principal`: Set to "*" to allow any AWS identity to perform the specified actions.
- `Action`: Set to "s3:GetObject" to allow reading objects from the bucket.
- `Resource`: Set to the ARN (Amazon Resource Name) of your S3 bucket and objects.
- `Condition`: Specifies the conditions under which the policy is in effect.
  - `StringLike`: Compares the specified string value against the actual value of `aws:Referer`.
  - `aws:Referer`: Contains an array of allowed referring URLs.

Replace `"your-bucket-name"` with your actual bucket name and adjust the allowed URL(s) in the `aws:Referer` array. Note that the `aws:Referer` condition is not foolproof because it can be easily spoofed, but it provides a basic level of protection.

Make sure to replace `"http://allowed-url.com/*"` and `"https://allowed-url.com/*"` with the actual URLs from which you want to allow access. If you have multiple allowed URLs, you can add more entries to the array.
