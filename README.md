# Static Website Deployment on AWS with Namecheap Domain and CI/CD

## Objective
This project demonstrates the steps to deploy a static website using Amazon S3, Route 53, CloudFront, and GitHub Actions for CI/CD, with a Namecheap domain.

---

## Project Steps

### 1. Domain Registration on Namecheap
1. Register your domain on [Namecheap](https://www.namecheap.com/).
2. Ensure you have access to manage DNS settings to integrate with AWS Route 53.

   ![Domain Registration Screenshot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/Domain-10-10-2024_10_35_AM.png)

---

### 2. Setting Up Amazon S3 Buckets
1. Go to the **AWS Management Console** > **S3**.
2. Create two buckets:
   - `example.com` - Main bucket to host website files.
   - `www.example.com` - Redirect bucket to route traffic to `example.com`.
   
3. Configure bucket settings:
   - Make both buckets **public** and enable **static website hosting**.
   - In the `www.example.com` bucket, set it to **redirect to `example.com`**.
   
4. Upload `index.html` to `example.com` bucket as the homepage.

   ![S3 Buckets Screenshot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/s3%20buckets.png)

---

### 3. Setting Up AWS Route 53
1. In **Route 53**, create a new **hosted zone** for `example.com`.
2. Add DNS records:
   - **A record** for `example.com` pointing to the S3 bucket.
   - **CNAME record** for `www.example.com` pointing to `example.com`.

   ![Route 53 Screenshot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/route53.png)

---

### 4. Updating Name Servers
1. Go to **Namecheap** and navigate to your domain’s **DNS settings**.
2. Change the **name servers** to the AWS Route 53 name servers provided in the hosted zone.

---

### 5. Generating SSL Certificates
1. In **AWS Certificate Manager**, request a new certificate for `example.com` and `www.example.com`.
2. Generate and validate **CNAME records**:
   - Add these CNAME records to your Namecheap DNS settings to complete the SSL validation.

   ![SSL Certificate Screenshot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/ACM.png)

---

### 6. Applying CloudFront CDN
1. In **Amazon CloudFront**, create a new distribution with the S3 bucket (`example.com`) as the origin.
2. Configure:
   - Enable **SSL** and set to redirect HTTP to HTTPS.
   - Set up custom error pages if needed.
   
3. Wait for the CloudFront distribution to deploy.

   ![CloudFront Distribution Screenshot](https://github.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/blob/main/static-website-s3/Screenshots/cloudfront.png)

---

### 7. Setting Up CI/CD with GitHub Actions

#### Step 1: Create AWS Access Keys
1. In AWS, create an IAM user with the necessary permissions for S3.
2. Generate an **Access Key ID** and **Secret Access Key**.
3. Add these keys to GitHub as secrets:
   - Go to **Settings** > **Secrets and variables** > **Actions** > **New repository secret**:
     - `AWS_ACCESS_KEY_ID`
     - `AWS_SECRET_ACCESS_KEY`

#### Step 2: Create GitHub Actions Workflow
1. In your repository, create `.github/workflows/main.yml`.
2. Add the following code for CI/CD:

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
         uses: actions/checkout@v2

       - name: Set up AWS CLI
         uses: aws-actions/configure-aws-credentials@v2
         with:
           aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
           aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
           aws-region: eu-central-1

       - name: Sync files to S3
         run: |
           aws s3 sync . s3://example.com --exclude ".git/*" --delete

---

### 8. Testing the Setup
1. **Access the Website**: Visit `https://example.com` and `https://www.example.com` to ensure the website is accessible over HTTPS.
2. **CloudFront Verification**: Confirm that requests are being served through CloudFront by checking the response headers. 

---

## Conclusion
This setup provides a full deployment pipeline for a static website using Amazon S3, Route 53, CloudFront, and GitHub Actions for CI/CD. The documentation covers each necessary step, from domain setup and SSL configuration to testing and verification. This project showcases both your technical skills and your understanding of AWS deployment best practices.

---

## Common Mistakes and Solutions
Below are some issues encountered during setup and their resolutions:

- **SSL Configuration**:
  - Certificates must be generated in a global region for use with CloudFront. Regional certificates may not be visible for CloudFront, causing SSL errors.
  
- **Bucket Setup**:
  - Only one S3 bucket should serve as the primary content host. Any additional buckets should be configured solely for traffic redirection.
  
- **CloudFront Origin Setup**:
  - When setting up the CloudFront origin, avoid configuring multiple origins or leaving the root directory field populated, as this can cause unnecessary redirections.

---

## Future Improvements
Consider adding logging and monitoring for your site using Amazon CloudWatch and implementing version control for bucket content. This will further enhance your website’s stability and provide insights into traffic and performance.

---
## Challenges I Faced and How I Solved Them

During the setup of my AWS static website, I encountered several challenges. Here’s a summary of the mistakes I made, along with the solutions I implemented to overcome them.

### 1. SSL Configuration
- **Challenge**: I created SSL certificates in the Frankfurt Region, which I could not locate when using CloudFront.
- **Solution**: I realized that CloudFront is a global service. To resolve this, I ensured that all resources pointing to CloudFront have public access and are configured in a global context.

### 2. Bucket Configuration
- **Challenge**: I initially created two identical S3 buckets containing the same files.
- **Solution**: I learned that there should only be one main bucket for serving content, and a separate bucket should exist solely for redirecting traffic. I consolidated my files into a single primary bucket.

### 3. Static Website Hosting
- **Challenge**: I failed to configure the S3 buckets for static website hosting.
- **Solution**: I updated the bucket settings to enable static website hosting, which is essential for serving web content directly from S3 and ensuring proper URL handling and access.

### 4. CloudFront Origin Setup
- **Challenge**: I mistakenly set up two origins in my CloudFront configuration instead of just one. Additionally, I populated the root directory field, which led to unnecessary redirections.
- **Solution**: I revised the CloudFront configuration to use a single origin and left the root directory field blank to avoid misrouting.

These experiences not only helped me understand the intricacies of AWS services but also improved my troubleshooting and configuration skills.


## Project Screenshots

| Step       | Description                        | Screenshot |
|------------|------------------------------------|------------|
| S3 Buckets | Initial setup of two S3 buckets    | ![S3 Buckets Screenshot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/s3%20buckets.png) |
| Route 53   | Configuring hosted zone and records | ![Route 53 Screenshot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/route53.png) |
| SSL Cert   | SSL Certificate creation and validation | ![SSL Certificate Screenshot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/ACM.png) |
| CloudFront | Setting up CloudFront distribution | ![CloudFront Distribution Screenshot](https://github.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/blob/main/static-website-s3/Screenshots/cloudfront.png) |
| CI/CD      | GitHub Actions workflow for CI/CD | ![GitHub Actions Workflow Screenshot](https://raw.githubusercontent.com/tanviralamcse/Launching-a-Static-Website-on-Amazon-S3/refs/heads/main/static-website-s3/Screenshots/gitworkflows.png) |

---

