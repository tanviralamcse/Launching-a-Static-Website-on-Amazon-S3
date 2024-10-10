# My Website Setup Experience

In this document, I would like to share my experience setting up a website using AWS services with a Namecheap domain. Below are the steps I took to successfully launch my website.

## Step 1: Domain Registration with Namecheap

1. **Register a Domain**: 
   - I started by registering the domain `example.com` on [Namecheap](https://www.namecheap.com/).
   - I had an old domain. I used it. 
   - (you can buy from anywhere or directly from Route53)
   - or you can transfer your domain
![ScreenShot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/Domain-10-10-2024_10_35_AM.png)
## Step 2: Creating S3 Buckets

1. **Create Buckets**:
   - After registering the domain, I logged into the AWS Management Console and navigated to **S3**.
   - I created two S3 buckets:
     - `example.com`
     - `www.example.com`

2. **Configure Bucket Settings**:
   - I set both buckets to be **public** and enabled static website hosting.
   - Additionally, I configured the `www.example.com` bucket to redirect to `example.com`.

3. **Upload `index.html`**:
   - I uploaded my `index.html` file to the `example.com` bucket, which would serve as the homepage for my website.
![ScreenShot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/s3%20buckets.png)
## Step 3: Setting Up AWS Route 53

1. **Create a Hosted Zone**:
   - I then navigated to **Route 53** in the AWS Management Console and created a new hosted zone for `example.com`.

2. **Add Records**:
   - I added an A record pointing to my S3 bucket for `example.com` and a CNAME record for `www.example.com`.
![ScreenShot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/route53.png)
## Step 4: Updating Name Servers

1. **Update Name Servers**:
   - I accessed the domain settings in Namecheap and changed the name servers to the AWS Route 53 name servers, which I found in the hosted zone for my domain.

## Step 5: Generating SSL Certificates

1. **Request SSL Certificates**:
   - I navigated to **AWS Certificate Manager** to request a new public certificate for `example.com` and `www.example.com`.

2. **Generate CNAME Records**:
   - After requesting the certificate, AWS provided me with CNAME records for validation.

3. **Validate DNS Using CNAME Records**:
   - I logged into Namecheap and added the provided CNAME records to the DNS settings for my domain.
![ScreenShot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/ACM.png)
## Step 6: Applying CloudFront CDN

1. **Create a CloudFront Distribution**:
   - Next, I created a new CloudFront distribution in **Amazon CloudFront**, setting the origin to point to my S3 bucket (`example.com`).

2. **Configure Settings**:
   - I enabled **SSL** and set the default behavior to redirect HTTP to HTTPS. Additionally, I set up custom error pages as needed.

3. **Deploy Distribution**:
   - I waited for the CloudFront distribution to deploy, which took some time for the changes to propagate.
  
![ScreenShot](https://github.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/blob/main/static-website-s3/Screenshots/cloudfront.png)

## Steps to Set Up CI/CD

### 1. Create AWS Access Keys:

1. In the AWS Console, created an IAM user with `AdministratorAccess` or relevant permissions for managing S3.
2. Generate an **Access Key ID** and **Secret Access Key** for this user.
3. Store these keys as secrets in your GitHub repository:
   - Go to **Settings** > **Secrets and variables** > **Actions** > **New repository secret** and add the following:
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`

### 2. Create GitHub Actions Workflow:

1. In the `.github/workflows` directory, create a `main.yml` file to define the CI/CD pipeline.
2. This file contains instructions for checking out the repository code, setting up AWS credentials, and syncing the files with the S3 bucket.

#### Here's the workflow file (`main.yml`):

```yaml
name: CI/CD Pipeline for AWS S3

on:
  push:
    branches:
      - main  # Trigger the workflow on pushes to the 'main' branch

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2  # Check out the code from the repository

    - name: Set up AWS CLI
      uses: aws-actions/configure-aws-credentials@v2  # Set up AWS credentials
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}  # Access Key from GitHub Secrets
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}  # Secret Key from GitHub Secrets
        aws-region: eu-central-1  # Specify the AWS region

    - name: Sync files to S3
      run: |
        aws s3 sync . s3://BUCKET_NAME --exclude ".git/*" --delete  # Sync files to the S3 bucket, excluding .git files
```
### 3. Configure AWS S3 Bucket:
   - Setup S3 bucket as Static Website Hosting.
   - The S3 bucket name used in the workflow (s3://BUCKET_NAME) must match your actual bucket name.
![ScreenShot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/gitworkflows.png)
## Step 7: Testing the Setup

1. **Access the Website**:
   - Once everything was set up, I visited `https://example.com` and `https://www.example.com` to verify that the website was accessible and using HTTPS.

2. **Check CloudFront**:
   - I ensured that requests were being served through the CloudFront CDN by checking the headers.

## Conclusion

Through this process, I successfully set up a website using a Namecheap domain, hosted on AWS S3, secured with SSL, and served via CloudFront. I hope my experience helps others who are looking to achieve similar goals. Feel free to update this documentation as you make further changes or improvements to your setup.

## Mistakes
  - ## SSL configuration
    - I created SSL certificates in Frankfurt Region, and for that I was unable to locate them while Using CloudFront. 
      - then I realized that, CloudFront is global. and everything we point to must have public access or setup globally. 
  - ## Bucket Configuration: 
      - I initially created two identical S3 buckets containing the same files. In hindsight, I realized that there should only be one main bucket for serving content and a separate bucket solely for redirecting traffic.

  - ## Static Website Hosting
      - I did not configure the S3 buckets for static website hosting. This setup is essential for serving web content directly from S3, ensuring proper URL handling and access.

  - ## CloudFront Origin Setup
      - In my CloudFront configuration, I mistakenly set up two origins instead of just one. Additionally, I left the root directory field populated instead of leaving it blank, which led to unnecessary redirections.
