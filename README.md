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

## Step 3: Setting Up AWS Route 53

1. **Create a Hosted Zone**:
   - I then navigated to **Route 53** in the AWS Management Console and created a new hosted zone for `example.com`.

2. **Add Records**:
   - I added an A record pointing to my S3 bucket for `example.com` and a CNAME record for `www.example.com`.

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

## Step 6: Applying CloudFront CDN

1. **Create a CloudFront Distribution**:
   - Next, I created a new CloudFront distribution in **Amazon CloudFront**, setting the origin to point to my S3 bucket (`example.com`).

2. **Configure Settings**:
   - I enabled **SSL** and set the default behavior to redirect HTTP to HTTPS. Additionally, I set up custom error pages as needed.

3. **Deploy Distribution**:
   - I waited for the CloudFront distribution to deploy, which took some time for the changes to propagate.

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
      - then I realized that, CloudFront is global. and everything we point to it must has public access or setup globally. 
  - ## Bucket Configuration: 
      - I initially created two identical S3 buckets containing the same files. In hindsight, I realized that there should only be one main bucket for serving content and a separate bucket solely for redirecting traffic.

  - ## Static Website Hosting
      - I did not configure the S3 buckets for static website hosting. This setup is essential for serving web content directly from S3, ensuring proper URL handling and access.

  - ## CloudFront Origin Setup
      - In my CloudFront configuration, I mistakenly set up two origins instead of just one. Additionally, I left the root directory field populated instead of leaving it blank, which led to unnecessary redirections.
