# CloudFront Distribution with S3 Origin – Secure Implementation

## Overview

This implementation demonstrates setting up a secure static content delivery system using Amazon S3 and CloudFront, with all S3 content accessible only through CloudFront. The setup uses Origin Access Control (OAC) / Origin Access Identity (OAI) to restrict access, and all steps are verified with screenshots (1.jpg to 24.jpg).

Amazon CloudFront is a fast content delivery network (CDN) service that securely delivers data, videos, applications, and APIs to customers globally with low latency and high transfer speeds. By integrating CloudFront with Amazon S3, we can serve static website content efficiently while maintaining strict security controls that prevent direct access to the S3 bucket.

This project follows AWS best practices for securing S3 origins and demonstrates the complete workflow from bucket creation to validation of CloudFront distribution functionality.

---

## Architecture Overview

The architecture consists of the following components:

- **Amazon S3 Bucket**: Serves as the origin for storing static content (HTML, CSS, JS, images)
- **Amazon CloudFront**: CDN distribution that caches and delivers content globally
- **Origin Access Control (OAC)**: Ensures only CloudFront can access the S3 bucket
- **S3 Bucket Policy**: Restricts all direct access, allowing only authorized CloudFront service principal
- **HTTPS Enforcement**: Redirects all HTTP requests to HTTPS for secure delivery

---

## Steps Performed

### 1. S3 Bucket Creation and Initial Setup

The first step involved creating a new Amazon S3 bucket to host the static website content. The bucket was configured with appropriate regional settings and initial configurations.

**Actions taken:**
- Created S3 bucket with name: `www.demo-cdn.com`
- Selected the appropriate AWS region (ap-south-1 - Mumbai)
- Reviewed bucket settings and confirmed creation
- Accessed the bucket console to verify successful creation
- Checked bucket properties and overview details

**Screenshots:**
- `1.jpg` - S3 Bucket Creation console with bucket name input
- `2.jpg` - S3 Bucket Console Overview showing bucket details
- `7.jpg` - Additional S3 bucket setup views and configuration

**Why this matters:** Proper bucket naming and regional selection are crucial for optimal performance and compliance with organizational policies. The bucket name must be globally unique and follow DNS naming conventions.

---

### 2. Content Upload

After creating the bucket, static website content was uploaded. This includes the main HTML file and any associated assets.

**Actions taken:**
- Prepared static content files (index.html and related assets)
- Uploaded `index.html` to the root of the S3 bucket
- Uploaded additional static files (CSS, JavaScript, images if applicable)
- Verified successful upload of all files
- Checked file permissions and metadata
- Confirmed file accessibility within S3 console

**Screenshots:**
- `8.jpg` - Static Content Upload interface showing upload progress
- `10.jpg` - S3 File List displaying uploaded files
- `11.jpg` - Uploaded Files Verification with file details

**Why this matters:** Ensuring all content is properly uploaded before configuring CloudFront prevents issues during distribution setup. All files should be verified for correct MIME types and metadata.

---

### 3. Public Access Block and S3 Bucket Policy

To secure the bucket and ensure content is only accessible through CloudFront, public access was blocked and a restrictive bucket policy was implemented.

**Actions taken:**
- Enabled "Block all public access" settings on the S3 bucket
- Confirmed that all four public access block settings are enabled
- Navigated to bucket policy section
- Created a bucket policy that allows only CloudFront service principal access
- Applied the policy with condition checking for specific CloudFront distribution ARN
- Verified policy syntax and confirmed no public access is allowed

**Bucket Policy Applied:**
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

**Screenshots:**
- `12.jpg` - Public Access Block Settings showing all blocks enabled
- `13.jpg` - Bucket Policy Console with policy editor
- `14.jpg` - Policy JSON Content displayed in editor
- `15.jpg` - Confirmed Policy Statement after successful application

**Why this matters:** This configuration is critical for security. Without blocking public access and implementing the correct bucket policy, your S3 content could be accessed directly, bypassing CloudFront and exposing your origin server. The policy ensures that only requests authenticated by CloudFront can retrieve objects.

---

### 4. CloudFront Distribution Creation

With the S3 bucket secured, the next step was creating a CloudFront distribution to serve the content globally.

**Actions taken:**
- Navigated to CloudFront console and initiated distribution creation
- Selected "Web Distribution" type
- Configured origin settings with S3 bucket domain
- Used the recommended S3 origin format: `www.demo-cdn.com.s3.ap-south-1.amazonaws.com`
- Chose appropriate origin protocol policy
- Set up default root object as `index.html`
- Configured SSL/TLS certificate settings
- Selected price class based on geographic requirements

**Screenshots:**
- `16.jpg` - CloudFront Distribution Creation wizard first page
- `17.jpg` - Origin URL Selection showing S3 bucket endpoint
- `18.jpg` - OAI/OAC Creation Confirmation dialog

**Why this matters:** Proper distribution configuration ensures optimal content delivery. The origin domain name format is crucial - using the full S3 REST API endpoint allows CloudFront to properly communicate with S3.

---

### 5. Origin Access Control (OAC/OAI) Configuration

Origin Access Control is the modern replacement for Origin Access Identity and provides better security and functionality for S3 origins.

**Actions taken:**
- Created a new Origin Access Control (OAC) within CloudFront distribution settings
- Named the OAC appropriately for identification
- Selected signing behavior (sign requests)
- Associated OAC with the S3 origin
- AWS automatically generated the required S3 bucket policy
- Copied the generated policy to S3 bucket policy editor
- Verified OAC was properly attached to distribution origin
- Confirmed policy update in S3 bucket

**Screenshots:**
- `3.jpg` - CloudFront - OAI/OAC Setup interface
- `4.jpg` - Origin Access Selection dropdown and options
- `5.jpg` - Attaching Policy to S3 bucket process
- `19.jpg` - Final Origin Setup confirmation with OAC enabled

**Why this matters:** OAC is essential for secure S3 origins. It ensures that CloudFront signs requests to S3 using AWS Signature Version 4 (SigV4), providing stronger security than the legacy OAI. The bucket policy works in conjunction with OAC to authenticate CloudFront's requests.

---

### 6. CloudFront Cache Behavior and Distribution Settings

Cache behavior settings determine how CloudFront handles requests, caches content, and forwards requests to the origin.

**Actions taken:**
- Configured default cache behavior settings
- Set Viewer Protocol Policy to "Redirect HTTP to HTTPS"
- Allowed HTTP methods: GET, HEAD (read-only for static content)
- Enabled automatic compression for supported file types
- Selected appropriate Cache Policy (CachingOptimized or custom)
- Configured query string forwarding behavior
- Set cookie forwarding policy
- Enabled CloudFront logs (optional but recommended)
- Reviewed all distribution settings before deployment
- Initiated distribution deployment
- Monitored deployment status until "Deployed" state reached

**Screenshots:**
- `6.jpg` - Distribution Status showing deployment progress
- `9.jpg` - CloudFront Settings overview page
- `20.jpg` - Distribution Details showing configuration summary
- `21.jpg` - Cache Behavior Settings with all options visible
- `22.jpg` - Distribution Deployed confirmation with status "Enabled"

**Why this matters:** Cache behavior directly impacts performance and cost. Proper configuration ensures:
- Content is served securely over HTTPS
- Frequently accessed content is cached at edge locations
- Origin requests are minimized, reducing costs and improving speed
- Compression reduces bandwidth usage

---

### 7. Testing and Validation

The final and most critical step is validating that the entire setup works correctly and securely.

**Testing performed:**

#### A. CloudFront Content Delivery Test
- Accessed content via CloudFront domain: `https://dxxxxxxxxxxxx.cloudfront.net/index.html`
- Verified that content loads successfully
- Checked that all assets (CSS, JS, images) load correctly
- Confirmed page renders properly in browser

#### B. Response Headers Analysis
- Opened browser Developer Tools (Network tab)
- Examined HTTP response headers for the CloudFront request
- Verified presence of CloudFront-specific headers:
  - `via: 1.1 cloudfront` - Confirms request went through CloudFront
  - `x-cache: Miss from cloudfront` - First request (not in cache)
  - `x-amz-cf-pop: BOM50-C1` - CloudFront edge location serving content
  - `x-amz-cf-id` - Unique request identifier
- Made second request to same URL
- Verified cache hit: `x-cache: Hit from cloudfront` - Content served from cache

#### C. Cache Performance Validation
- Compared response times between cache miss and cache hit
- Confirmed significantly faster response on cached content
- Verified `age` header showing time content has been in cache
- Tested cache invalidation (optional)

#### D. S3 Direct Access Restriction Test
- Attempted to access S3 object directly using S3 URL:
  ```
  https://www.demo-cdn.com.s3.ap-south-1.amazonaws.com/index.html
  ```
- Confirmed response shows "Access Denied" error
- Verified XML error message from S3:
  ```xml
  <Error>
    <Code>AccessDenied</Code>
    <Message>Access Denied</Message>
  </Error>
  ```
- This confirms bucket policy is working correctly and S3 content is properly secured

**Screenshots:**
- `23.jpg` - Testing CloudFront Domain showing successful content delivery in browser
- `24.jpg` - Cache and Access Headers Verification in Developer Tools showing response headers

**Why this matters:** Testing validates the entire implementation. It confirms:
- CloudFront is successfully delivering content
- Caching is working, improving performance and reducing costs
- S3 bucket is secure and not directly accessible
- Only CloudFront can retrieve content from the origin

### CloudFront Distribution Configuration
- **Distribution Domain**: `dxxxxxxxxxxxx.cloudfront.net`
- **Origin**: S3 bucket (`www.demo-cdn.com`)
- **Origin Protocol**: HTTPS only
- **Viewer Protocol Policy**: Redirect HTTP to HTTPS
- **Allowed HTTP Methods**: GET, HEAD
- **Cache Policy**: CachingOptimized
- **Origin Access**: Origin Access Control (OAC)
- **Compression**: Enabled
- **IPv6**: Enabled

### S3 Bucket Configuration
- **Bucket Name**: `www.demo-cdn.com`
- **Region**: ap-south-1 (Mumbai)
- **Public Access**: Blocked (all four settings)
- **Bucket Policy**: CloudFront service principal only
- **Versioning**: Disabled
- **Encryption**: Server-side encryption enabled (optional)

### Security Measures
1. **Origin Access Control (OAC)**: Modern authentication method replacing legacy OAI
2. **Bucket Policy**: Restricts access to specific CloudFront distribution ARN
3. **Public Access Block**: All four settings enabled to prevent accidental public exposure
4. **HTTPS Only**: No unencrypted HTTP traffic allowed
5. **Signed Requests**: CloudFront signs all requests to S3 using AWS SigV4
---

## Cost Considerations

- **S3 Storage**: Minimal cost for storing static files
- **S3 Requests**: Significantly reduced due to CloudFront caching
- **CloudFront Data Transfer**: First 1TB/month free tier (new AWS accounts)
- **CloudFront Requests**: Charged per 10,000 requests (HTTP/HTTPS)
- **No Charge**: Data transfer from S3 to CloudFront

**Cost Optimization Tips:**
- Use appropriate cache TTL to maximize cache hit ratio
- Enable compression to reduce data transfer
- Choose price class based on geographic distribution of users
- Monitor CloudFront metrics to optimize cache behavior

---

## Troubleshooting Guide

### Issue: Content not loading from CloudFront
**Solution:** 
- Check distribution status is "Deployed"
- Verify origin settings point to correct S3 bucket
- Ensure OAC is properly configured
- Check CloudFront logs for errors

### Issue: Access Denied from CloudFront
**Solution:**
- Verify S3 bucket policy includes correct CloudFront distribution ARN
- Ensure OAC is created and attached to origin
- Check that bucket policy allows `s3:GetObject` action
- Verify condition in bucket policy matches distribution ARN

### Issue: Cache not working
**Solution:**
- Check cache behavior settings
- Verify Cache-Control headers from origin
- Use CloudFront invalidation to clear cache
- Review cache hit ratio in CloudFront metrics

### Issue: Direct S3 access still works
**Solution:**
- Verify all public access block settings are enabled
- Check bucket policy doesn't have unintended Allow statements
- Ensure no bucket ACLs grant public access
- Review IAM policies that might grant access

---
## Conclusion

This implementation successfully demonstrates a secure, scalable, and cost-effective solution for delivering static content using Amazon S3 and CloudFront. The setup ensures that all content is served through CloudFront's global CDN network while maintaining strict security controls that prevent direct access to the origin S3 bucket.

All steps have been thoroughly documented with 24 screenshots covering every aspect from initial bucket creation through final validation testing. The implementation follows AWS best practices and is ready for production use.
