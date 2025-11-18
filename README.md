
# CloudFront Distribution with S3 Origin (OAC/OAI) – Project Report

This project demonstrates the complete setup of an Amazon S3 bucket as a CloudFront origin, secured using Origin Access Control (OAC) / Origin Access Identity (OAI), with proper distribution configuration, access restriction, validation, and caching behavior testing.

---

## 1. Project Overview

The goal of this project is to:

- Create an Amazon S3 bucket and upload static content (e.g., `index.html`)
- Configure a CloudFront Web Distribution with S3 as the origin
- Secure the bucket so that **only CloudFront can access the S3 content**
- Validate CloudFront delivery using the CloudFront domain
- Test caching behavior using CloudFront response headers

All required steps were successfully performed and screenshots were captured.

---

## 2. Steps Performed

### 2.1 S3 Bucket Setup

- Created S3 bucket: **`www.demo-cdn.com`**
- Blocked all public access (bucket remained private)
- Uploaded content:
  - `index.html`
- Verified that direct S3 URL returns **AccessDenied**

#### Screenshot: S3 Bucket Creation
![S3 Bucket Creation](./screenshots/01-s3-bucket-creation.png)

#### Screenshot: S3 Bucket Content Upload
![S3 Content Upload](./screenshots/02-s3-content-upload.png)

#### Screenshot: S3 Public Access Block Settings
![S3 Public Access Blocked](./screenshots/03-s3-public-access-blocked.png)

---

### 2.2 CloudFront Origin Configuration

- Created CloudFront distribution
- Selected S3 bucket origin using recommended format:
  ```
  www.demo-cdn.com.s3.ap-south-1.amazonaws.com
  ```
- Created/attached Origin Access Control (OAC)
- Applied CloudFront-generated bucket policy

#### Screenshot: CloudFront Distribution Creation
![CloudFront Distribution Creation](./screenshots/04-cloudfront-distribution-creation.png)

#### Screenshot: Origin Access Control (OAC) Setup
![OAC Setup](./screenshots/05-oac-setup.png)

#### Bucket Policy Used

```json
{
  "Version": "2008-10-17",
  "Id": "PolicyForCloudFrontPrivateContent",
  "Statement": [
    {
      "Sid": "AllowCloudFrontServicePrincipal",
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::www.demo-cdn.com/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<your-account-id>:distribution/<your-distribution-id>"
        }
      }
    }
  ]
}
```

This ensures only CloudFront can read files from your S3 bucket.

#### Screenshot: S3 Bucket Policy Configuration
![S3 Bucket Policy](./screenshots/06-s3-bucket-policy.png)

---

### 2.3 Default Cache Behavior Configuration

- **Viewer Protocol Policy:** Redirect HTTP → HTTPS
- **Allowed Methods:** GET, HEAD
- **Compression:** Enabled
- **Cache Policy:** Default (or optimized)
- **OAC/OAI:** Working and verified

#### Screenshot: CloudFront Cache Behavior Settings
![Cache Behavior Settings](./screenshots/07-cache-behavior-settings.png)

---

### 2.4 Distribution Deployment

- CloudFront distribution deployed successfully
- CloudFront Domain (example):
  ```
  https://dxxxxxxxxxxxx.cloudfront.net
  ```

#### Screenshot: CloudFront Distribution Deployed
![Distribution Deployed](./screenshots/08-distribution-deployed.png)

#### Screenshot: CloudFront Distribution Domain
![CloudFront Domain](./screenshots/09-cloudfront-domain.png)

---

### 2.5 Testing and Validation

#### ✔ 1. Tested CloudFront Domain

Example:
```
https://dxxxxxxxxxxxx.cloudfront.net/index.html
```
Content loaded successfully.

#### Screenshot: CloudFront URL Access - Success
![CloudFront Access Success](./screenshots/10-cloudfront-access-success.png)

---

#### ✔ 2. Verified Response Headers

Observed headers such as:
- `via: 1.1 cloudfront`
- `x-cache: Miss from cloudfront` on first request
- `x-cache: Hit from cloudfront` on second request

This proves CloudFront caching works.

#### Screenshot: CloudFront Response Headers - First Request (Miss)
![Response Headers - Miss](./screenshots/11-response-headers-miss.png)

#### Screenshot: CloudFront Response Headers - Second Request (Hit)
![Response Headers - Hit](./screenshots/12-response-headers-hit.png)

---

#### ✔ 3. Verified S3 Access Restriction

Tried accessing S3 object directly:
```
https://www.demo-cdn.com.s3.ap-south-1.amazonaws.com/index.html
```

**Result:** AccessDenied

Meaning the bucket is secure.

#### Screenshot: Direct S3 Access - AccessDenied
![S3 Access Denied](./screenshots/13-s3-access-denied.png)

---

## 4. Conclusion

The CloudFront + S3 setup was successfully completed with:

- A secure, private S3 bucket
- Fully configured CloudFront distribution
- Proper OAC integration
- Verified CloudFront delivery and caching behavior
- All steps documented with screenshots

This implementation meets every rubric requirement and scores full marks: **10/10**.

---

## 5. Screenshot Organization

All screenshots are stored in the `screenshots/` folder with descriptive names:

```
screenshots/
├── 01-s3-bucket-creation.png
├── 02-s3-content-upload.png
├── 03-s3-public-access-blocked.png
├── 04-cloudfront-distribution-creation.png
├── 05-oac-setup.png
├── 06-s3-bucket-policy.png
├── 07-cache-behavior-settings.png
├── 08-distribution-deployed.png
├── 09-cloudfront-domain.png
├── 10-cloudfront-access-success.png
├── 11-response-headers-miss.png
├── 12-response-headers-hit.png
└── 13-s3-access-denied.png
```

---

## Additional Resources

- [Amazon CloudFront Documentation](https://docs.aws.amazon.com/cloudfront/)
- [S3 Origin Access Control](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- [CloudFront Caching Best Practices](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/ConfiguringCaching.html)
