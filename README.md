# Static Website Hosting on AWS

## Project Overview

This project demonstrates how to securely host a static website on AWS using **Amazon S3**, **Amazon CloudFront**, **AWS Certificate Manager (ACM)**, and **Amazon Route 53**.

The website is served from a private Amazon S3 bucket through CloudFront using **Origin Access Control (OAC)**, secured with HTTPS using an ACM SSL/TLS certificate, and accessed through a custom domain managed by Route 53.

## Live Demo

🔗 [https://teamtheta.site](https://teamtheta.site)

## Project Objectives

- Host a static website on AWS
- Secure the website with HTTPS
- Configure a custom domain
- Improve performance using CloudFront
- Implement AWS security best practices by keeping the S3 bucket private

## AWS Services Used

| Service | Purpose |
|---|---|
| Amazon S3 | Stores static website files |
| Amazon CloudFront | Global CDN for caching and HTTPS |
| AWS Certificate Manager | Issues and manages SSL/TLS certificates |
| Amazon Route 53 | DNS management for the custom domain |

## Architecture

```
               User
                 │
                 ▼
          Amazon Route 53
                 │
                 ▼
      Amazon CloudFront
                 │
        HTTPS (ACM Certificate)
                 │
                 ▼
      Private Amazon S3 Bucket
```

<img width="602" height="577" alt="Architecture Diagram" src="https://github.com/user-attachments/assets/ef416e65-a17d-418b-a508-ebb99c411ad4" />

## Features

- Static website hosting
- HTTPS enabled
- Custom domain
- Global content delivery
- CloudFront caching
- Private S3 bucket
- Origin Access Control (OAC)
- Responsive website

## Project Structure

```
static-website-hosting-aws/
├── README.md
├── website/
├── docs/
├── screenshots/
└── architecture/
```

## Deployment Steps

### Step 1 — Clone the repository

```bash
git clone https://github.com/yourusername/static-website-hosting-aws.git
```

### Step 2 — Create the S3 bucket

Create a private S3 bucket to store the website files. Block all public access — CloudFront will access it via OAC instead.

```bash
aws s3 mb s3://your-bucket-name
```

### Step 3 — Upload website files

```bash
aws s3 sync ./website s3://your-bucket-name
```

### Step 4 — Request an SSL/TLS certificate (ACM)

In **AWS Certificate Manager**, request a public certificate for your custom domain (e.g. `teamtheta.site`). ACM certificates used with CloudFront must be requested in the **us-east-1** region.

Validate the certificate via DNS by adding the provided CNAME record in Route 53.

### Step 5 — Create the CloudFront distribution

- Set the S3 bucket as the origin
- Enable **Origin Access Control (OAC)** so only CloudFront can read from the bucket
- Attach the ACM certificate for HTTPS
- Add your custom domain as an alternate domain name (CNAME)
- Update the S3 bucket policy to allow access only from the CloudFront distribution

### Step 6 — Configure Route 53

Create an **A record (Alias)** in Route 53 pointing your domain to the CloudFront distribution.

### Step 7 — Verify

Visit your custom domain over HTTPS to confirm the site loads correctly and the S3 bucket remains inaccessible directly.

```bash
curl -I https://teamtheta.site
```

## Security Notes

- The S3 bucket has **no public access** — it is only reachable through CloudFront via OAC.
- HTTPS is enforced end-to-end using the ACM certificate.
- Bucket policy is scoped to allow requests only from the specific CloudFront distribution.

## Future Improvements

- Add CI/CD pipeline (e.g. GitHub Actions) to automate deployments on push
- Add CloudFront cache invalidation step to deployment workflow
- Add a WAF for additional security
- Add monitoring/alerts via CloudWatch

## Screenshots

Home Page
<img width="1632" height="892" alt="image" src="https://github.com/user-attachments/assets/d5640aeb-9462-413b-8386-4e256e0f8ee8" />

S3 Bucket
<img width="1542" height="572" alt="image" src="https://github.com/user-attachments/assets/aa4ffa4e-fb49-4d8f-b80d-00ebfb7c7012" />

CloudFront Distribution
<img width="1227" height="600" alt="image" src="https://github.com/user-attachments/assets/41b9ec25-8f16-4459-ba4f-8c1e3d571de3" />

Route53
<img width="1236" height="601" alt="image" src="https://github.com/user-attachments/assets/e34d3d79-e80e-4384-a5dd-1e8a262ba4b7" />

ACM Certificate
<img width="1062" height="536" alt="image" src="https://github.com/user-attachments/assets/74cc1667-bfbe-491e-8156-0bd99a73f0b9" />


HTTPS Working
<img width="1025" height="521" alt="image" src="https://github.com/user-attachments/assets/9341760c-8d54-4a05-b8c1-641b2c2cfc5c" />


## Author

**Alex Agyei**
🔗 [GitHub](https://github.com/lexisbil1) 
· [LinkedIn](https://linkedin.com/in/alex-agyei)

## License

This project is licensed under the [MIT License](LICENSE).
