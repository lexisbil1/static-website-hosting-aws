# Deployment Guide — Static Website Hosting on AWS

This guide documents every step taken in the AWS Management Console to deploy the static website, from bucket creation to a live HTTPS site on a custom domain.

---

## 1. Create S3 Bucket

1. Sign in to the **AWS Management Console**
2. Navigate to **S3**
3. Click **Create bucket**
4. Enter a **Bucket name** (must be globally unique, e.g. `teamtheta-site`)
5. Select the **AWS Region** (e.g. `us-east-1`)
6. Under **Object Ownership**, leave **ACLs disabled** (recommended)
7. Under **Block Public Access settings for this bucket**, leave all four boxes **checked** (bucket stays private — access will go through CloudFront only)
8. Leave **Bucket Versioning** disabled (or enable if desired)
9. Leave **Default encryption** as **Amazon S3 managed keys (SSE-S3)**
10. Click **Create bucket**

---

## 2. Upload Website Files

1. Open the newly created bucket
2. Click **Upload**
3. Click **Add files** / **Add folder**
4. Select the website files (`index.html`, `error.html`, CSS, JS, images, etc.)
5. Scroll down and click **Upload**
6. Confirm all files appear in the bucket's **Objects** list

---

## 3. Block Public Access

1. Inside the bucket, go to the **Permissions** tab
2. Under **Block public access (bucket settings)**, click **Edit**
3. Confirm all four options are checked:
   - Block all public access
4. Click **Save changes**
5. Type **confirm** in the dialog box and click **Confirm**
6. Verify the bucket **Access** column on the S3 bucket list shows **Bucket and objects not public**

---

## 4. Create Origin Access Control (OAC)

1. Navigate to **CloudFront**
2. In the left sidebar, click **Origin access**
3. Click **Create control setting**
4. Enter a **Name** (e.g. `teamtheta-oac`)
5. Under **Signing behavior**, select **Sign requests (recommended)**
6. Under **Origin type**, select **S3**
7. Click **Create**

*(Note: OAC can also be created directly from the "Create distribution" screen when adding the origin — see Step 5.)*

---

## 5. Create CloudFront Distribution

1. In **CloudFront**, click **Create distribution**
2. Under **Origin domain**, select the S3 bucket created in Step 1
3. Under **Origin access**, select **Origin access control settings (recommended)**
4. Select the OAC created in Step 4 (or click **Create new OAC** here)
5. Click **Copy policy** when prompted — this generates the required S3 bucket policy
6. Under **Viewer protocol policy**, select **Redirect HTTP to HTTPS**
7. Under **Allowed HTTP methods**, leave as **GET, HEAD**
8. Under **Web Application Firewall (WAF)**, select **Do not enable security protections** (or enable if desired)
9. Under **Settings → Alternate domain name (CNAME)**, click **Add item** and enter the custom domain (e.g. `teamtheta.site`)
10. Under **Custom SSL certificate**, select the ACM certificate (created in Step 6 — if not yet issued, this distribution can be updated afterward)
11. Under **Default root object**, enter `index.html`
12. Click **Create distribution**
13. Go back to the **S3 bucket → Permissions → Bucket policy**
14. Click **Edit**, paste the policy copied in step 5, and click **Save changes** (this grants the CloudFront distribution read access to the bucket)

---

## 6. Request ACM Certificate

1. Navigate to **AWS Certificate Manager (ACM)**
2. **Important:** switch the region selector (top right) to **US East (N. Virginia) us-east-1** — CloudFront only accepts certificates issued in this region
3. Click **Request a certificate**
4. Select **Request a public certificate** → **Next**
5. Under **Fully qualified domain name**, enter the domain (e.g. `teamtheta.site`)
6. Click **Add another name to this certificate** and add `www.teamtheta.site` if needed
7. Under **Validation method**, select **DNS validation** (recommended)
8. Leave **Key algorithm** as default (RSA 2048)
9. Click **Request**

---

## 7. Validate Certificate

1. Open the newly requested certificate from the ACM dashboard
2. Under **Domains**, note the **CNAME name** and **CNAME value** provided for each domain
3. Click **Create records in Route 53** (available if the domain's hosted zone is already in Route 53)
4. Confirm the domain(s) to validate and click **Create records**
5. Wait for the **Status** column to change from **Pending validation** to **Issued** (can take a few minutes)

---

## 8. Configure Route 53

1. Navigate to **Route 53**
2. Click **Hosted zones**
3. Select the hosted zone for the domain (e.g. `teamtheta.site`) — create one first via **Create hosted zone** if it doesn't exist
4. Click **Create record**
5. Leave **Record name** blank (for the root domain) or enter `www` for a subdomain
6. Set **Record type** to **A**
7. Toggle **Alias** to **on**
8. Under **Route traffic to**, select **Alias to CloudFront distribution**
9. Choose the CloudFront distribution created in Step 5 from the dropdown
10. Leave **Routing policy** as **Simple routing**
11. Click **Create records**
12. (Optional) Repeat for an **AAAA** record if IPv6 support is desired

---

## 9. Test HTTPS

1. Wait for the CloudFront distribution status to change to **Deployed** (can take 5–15 minutes)
2. Open a browser and navigate to `https://teamtheta.site`
3. Confirm the site loads and the browser shows a valid padlock/HTTPS connection
4. Click the padlock icon and verify the certificate is issued by Amazon and matches the domain
5. Attempt to access the S3 bucket's direct URL or REST endpoint directly and confirm it returns **Access Denied** (verifying the bucket is not publicly reachable)
6. Test `http://teamtheta.site` and confirm it automatically redirects to `https://`

---

## Troubleshooting Notes

- If the site returns **403 Forbidden**: check that the S3 bucket policy was updated with the CloudFront-generated policy (Step 5) and that OAC is correctly attached to the origin.
- If the ACM certificate isn't selectable in CloudFront: confirm it was requested in **us-east-1**, not another region.
- If DNS changes aren't reflecting: allow time for propagation, or clear local DNS cache (`ipconfig /flushdns` on Windows).
- If CloudFront serves stale content after an update: create an **Invalidation** under the distribution's **Invalidations** tab with path `/*`.
