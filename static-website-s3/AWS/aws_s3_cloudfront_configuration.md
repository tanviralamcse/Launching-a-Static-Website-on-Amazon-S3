
# AWS S3 and CloudFront Configuration Guide for example.com

## 1. S3 Buckets Configuration

### Main Bucket: example.com
- **Purpose**: Serve static website content.
- **Bucket Name**: example.com
- **Static Website Hosting**: 
  - Enable static website hosting.
  - Hosting Type: **Host a static website**.
  - Index Document: `index.html`
  - Error Document: `404.html` (if applicable).

#### Policies:
- **Bucket Policy**:
  ```json
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Sid": "PublicReadGetObject",
              "Effect": "Allow",
              "Principal": "*",
              "Action": "s3:GetObject",
              "Resource": "arn:aws:s3:::example.com/*"
          }
      ]
  }
  ```

### 2. Redirect Bucket: www.example.com
- **Purpose**: Redirect traffic from `www.example.com` to `example.com`.
- **Bucket Name**: www.example.com
- **Static Website Hosting**:
  - Enable static website hosting.
  - Hosting Type: **Redirect requests**.
  - Redirect to: `example.com`
  - Protocol: **HTTP** or **HTTPS**.

### 3. CloudFront Configuration
- **CloudFront Distribution**:
  - **Origin Domain Name**: `example.com.s3-website.eu-central-1.amazonaws.com` (this should point to your main bucket's website endpoint).
  - **Viewer Protocol Policy**: Set to **Redirect HTTP to HTTPS** to enforce secure connections.
  - **Alternate Domain Names (CNAMEs)**: 
    - Add both `example.com` and `www.example.com`.

- **Default Root Object**: 
  - Set this to `index.html`.

### 4. Route 53 Configuration
- **DNS Records**:
  - **A Record for example.com**:
    - Type: A
    - Alias: Yes
    - Value: `xxxxxxx.cloudfront.net` (CloudFront Distribution).
  
  - **A Record for www.example.com**:
    - Type: A
    - Alias: Yes
    - Value: `xxxxxxx.cloudfront.net` (CloudFront Distribution).

### 5. Testing and Validation
- Once all configurations are done, you should:
  - Access `https://example.com` and `https://www.example.com` to ensure both URLs direct users correctly.
  - Verify that accessing `https://example.com` shows your `index.html` file without any redirect loops.
  - Test error pages (e.g., 404) to confirm they display correctly if set up.

### Summary of Best Practices:
- **Single Bucket for Content**: Use one main bucket for serving website content.
- **Use CloudFront**: For better performance and security.
- **Redirect Bucket**: If using a `www` version, it should redirect to the non-www version.
- **S3 Permissions**: Ensure bucket policies allow public access for reading the content.
