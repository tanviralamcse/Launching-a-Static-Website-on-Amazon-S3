# Static Website Hosting on Amazon S3

## Project Overview
This project demonstrates how to host a static website using **Amazon S3**. The website consists of HTML files stored in an S3 bucket, and it is made publicly accessible via S3's static website hosting feature. Additionally, the website can be accelerated using **Amazon CloudFront** for content distribution.

### AWS Services Used
- **Amazon S3**: Storage and static website hosting.
- **Amazon CloudFront** (optional): Content delivery network (CDN) for faster loading across regions.
- **Amazon Route 53** (optional): Domain management and DNS configuration for the website URL.

---

## Project Setup

### 1. Create an S3 Bucket
1. Log in to your [AWS Management Console](https://aws.amazon.com/console/).
2. Navigate to **S3** and create a new bucket.
   - **Bucket name**: Choose a globally unique name (e.g., `my-website-example`).
   - **Region**: Select a region close to your user base.
3. In the **Properties** tab of the bucket, enable **Static Website Hosting**:
   - Set the index document as `index.html`.
   - (Optional) Set the error document as `error.html`.

4. Upload your website files (e.g., `index.html`, `style.css`, `images/`).
   - Ensure that `index.html` is correctly placed in the root directory.

### 2. Configure Bucket Permissions
1. Open the **Permissions** tab in the S3 bucket.
2. Update the **Bucket Policy** to allow public read access to your files. Here is an example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*"
    }
  ]
}
