+++
title = "How to Host Multiple Static Websites Under One S3 Bucket"
date = 2024-11-03T00:26:30+01:00
draft = false
summary = "Article explains how to host multiple static websites under a single AWS S3 bucket, accessible via different subdomains. It uses AWS services like S3, CloudFront, Lambda@Edge, and ACM (AWS Certificate Manager) to achieve this setup."
+++

Hello and welcome to my blog! Today, I’ll guide you through the process of hosting multiple static websites on different subdomains under a single S3 bucket. I implemented this for my postcard [project](https://www.postcard-gift.site/), and I’m excited to share the steps with you.

Here’s the result we aim to achieve. The first website will be accessible at the subdomain **alan.postcard-gift.site**:

{{< figure src="first-site-subdomain.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="First example of the website">}}

And the second website will be accessible at **lumberj3ck.postcard-gift.site**:

{{< figure src="second-site-subdomain.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Second example of the website">}}

Initially, I considered using the Vercel API for this purpose. However, I realized it might not be suitable due to potential usage limits. Since Vercel is essentially a wrapper around AWS, I decided it would be more cost-effective to use AWS directly, eliminating overhead costs. Moreover, because our use case doesn’t require features like website building or resource packaging, AWS is the ideal solution.

### Architecture

The architecture for our setup will look like this:

{{< figure src="lambdaS3.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="S3-and-Lambda-Edge architecture">}}

The magic lies in the combination of CloudFront and a Lambda@Edge function, which ensures that each URL correctly maps to the corresponding S3 folder.

### Steps to Set Up

#### 1. Create an S3 Bucket

First, create an S3 bucket without enabling public access. Since CloudFront will handle access to the bucket internally, there’s no need to make the bucket publicly accessible. 

{{< figure src="create-bucket.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Create bucket">}}

Next, enable **Static Website Hosting** in the bucket properties.

{{< figure src="enable-static-website-hosting.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Enable static website hosting">}}

Configure the following options:

{{< figure src="static-website-options.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Static website hosting options">}}

Now, upload your two static websites into the S3 bucket, ensuring that the folder names match the desired subdomains.

{{< blockquote >}}
**Note**: Folder names should correspond to subdomains. For example, the folder **alan** matches the subdomain **alan.postcard-gift.site**.
{{< /blockquote >}}

{{< figure src="two-websites-in-s3.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Two websites in S3">}}

#### 2. Set Up AWS Certificate Manager

Next, create an SSL certificate in AWS Certificate Manager for your domain. This ensures secure HTTPS access to your CloudFront distribution.

{{< blockquote >}}
**Note**: A single certificate can include multiple domains, which is useful if you’re hosting several subdomains.
{{< /blockquote >}}

{{< figure src="acm-domains.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="ACM domains">}}

Add the generated CNAME records to your DNS provider to validate domain ownership. If your DNS provider doesn’t support this, you can use Cloudflare, as I did.
{{< blockquote >}}
**Note**: I had an isssue with **NameCheap** they didn't allow me to insert CNAME value from **AWS Certificate manager**, that's why I used **cloudflare** instead. The process of moving to manage your domain name in cloudflare is pretty straighforward.
{{< / blockquote >}}

{{< figure src="cloudflare-dns-settings.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Cloudflare DNS settings">}}

#### 3. Configure CloudFront

Create a new CloudFront distribution with your S3 bucket as the origin. Use the bucket’s default endpoint instead of the static website hosting endpoint.
Since we don't want to leave bucket public, we need to use {{<  emphasize >}}Origin Access Control{{< / emphasize >}} for origin access, aws will generate bucket policy which we must apply 

{{< figure src="cloudfront-creation.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="CloudFront creation">}}

Set the **Cache Policy** to **CachingOptimized**:

{{< figure src="cache-policy-for-cloudfront.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Cache policy for CloudFront">}}

Add your subdomains as CNAMEs and select the SSL certificate created in AWS Certificate Manager.

{{< figure src="distribution-alternative-names.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Distribution alternative names">}}

#### 4. Configure S3 Bucket Policy

Update the S3 bucket policy with the one generated by CloudFront to allow access to the bucket.

{{< figure src="s3-policy.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="S3 bucket policy">}}

#### 5. Create a Lambda@Edge Function

Finally, create a Lambda@Edge function to handle URL routing. This function modifies requests to point to the correct S3 folder based on the subdomain.

{{< figure src="lambda-function-creation.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Lambda function creation">}}

{{< blockquote >}}
By default aws lambda must have a permissions for logging to the **CloudWatch**, in my case I have had a problem with that and lambda couldn't create proper logs to the CloudWatch, so I created **new role** for lambda function. You can check permissions at the configuration tab of your lambda. 
{{< / blockquote >}}


Here’s the JavaScript code for the Lambda@Edge function:

{{< highlight js "linenos=table,hl_lines=,linenostart=1" >}}
export const handler = async (event) => {
    // here our handler accepts parameter --> event
    const request = event.Records[0].cf.request;
    // logs are accesible in AWS CloudWatch
    console.log('Original request:', JSON.stringify(request, null, 2));
    const headers = request.headers;
    // Access the domain to which request has been made
    const host = headers.host[0].value;
    console.log('Host:', host);

    // Extract the domain parts
    const parts = host.split('.');
    let subdomain, folderName;
    
    // Parsing s3 bucket folders correctly
    if (parts.length > 2) {
        subdomain = parts[0];
        folderName = parts[1];
    } else {
        subdomain = '';
        folderName = parts[0];
    }

    console.log('Subdomain:', subdomain);
    console.log('Folder name:', folderName);

    // Hardcoded name of S3 bucket
    // Update!
    const bucketName = 'valetine-postcard-websites';

    // Modify the origin of request
    request.origin = {
        s3: {
            authMethod: 'none',
            domainName: `${bucketName}.s3.eu-central-1.amazonaws.com`,
            path: ''
        }
    };

    // Modify the URI to include the folder and subdomain path
    if (request.uri === '/' || request.uri.endsWith('/')) {
        request.uri = `/${folderName}/${subdomain}/index.html`;
    } else {
        request.uri = `/${folderName}/${subdomain}${request.uri}`;
    }

    console.log('Modified request:', JSON.stringify(request, null, 2));
    // now request goes to the correct folder of our s3 
    return request;
};
{{< /highlight >}}

Deploy the function to Lambda@Edge and associate it with the appropriate CloudFront event you need to select {{< emphasize >}} Viewer request {{< / emphasize >}}.

{{< figure src="deploy-to-lambda-edge.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Deploy to Lambda@Edge">}}


### Conclusion
Once everything is set up, your subdomains will correctly route to their respective folders in the S3 bucket. If you followed this guide step by step, you should now have a scalable and cost-effective way to host multiple static websites on AWS.