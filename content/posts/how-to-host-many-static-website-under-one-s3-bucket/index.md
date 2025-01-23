+++
title = 'How to Host Many Static Website Under One S3 Bucket'
date = 2024-11-03T00:26:30+01:00
draft = false
summary = 'This is how you can store several static websites under one s3 bucket accessing them via different subdomains.'
+++


Hello and welcome to my blog, today I'm going to tell you, how I have done uploading of static websites at different subdomains for my postcard [project](https://www.postcard-gift.site/). 

This is the result we want to achieve at the end, the first website with subdomain **alan.postcard-gift.site**.
{{< figure src="first-site-subdomain.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="First example of the website">}}

And the second website with subdomain **lumberj3ck.postcard-gift.site**.  

{{< figure src="second-site-subdomain.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Second example of the website">}}

I had to find an easy way to store and publish many websites at the same place firstly I decided to move on with vercel API, but it didn't work out because I was afraid to quickly hit the limits of vercel api, at the end of the day vercel is a wrapper around aws, it would be cheaper to use aws ourselves without overheads, 
especially when we don't have to build and package our website resourses thus don't even need most of the features of vercel.
### Architecture

Schema of our architecture will look like this:

{{< figure src="lambdaS3.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="S3-and-lambda-edge-arhitecture">}}

The most magic is done by cloudfront and Lambda Edge function, which actually is responsible for the correct url recognition and pointing to the right s3 folder.


Let's dive deep into the process of setting up. First thing first we need to create S3 bucket. 

{{< figure src="create-bucket.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="create-bucket">}}

We need to create a default s3 bucket without any specific configuration and public access to bucket is don't needed, because cloudfront can access s3 bucket internally, bucket don't need to be publically accesible. Next we need to enable static website hosting inside of our bucket properties.


{{< figure src="enable-static-website-hosting.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="enable-static-website-hosting">}}

The following options you need to enable:

{{< figure src="static-website-options.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="static-website-options">}}

Now we are ready to push our two pages to s3 bucket.


{{< blockquote >}}
**Note**: Notice the folders name are the same as subdomain. **alan** folder is corresponds to the **alan.postcard-gift.site**, because that exactly we are going to understand where each request must be routed to.
{{< / blockquote >}}


{{< figure src="two-websites-in-s3.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="Second example of the website">}}

### AWS Certificate manager

Now we need to go to the AWS Certificate manager and create a certificate for domain,  which our cloudfront distribution will be accesible from.
{{< blockquote >}}
**Note**: In one certificate you can have multiple domains, that what we want in case if our cloudfront distribution has to be accesible from multiple domains. 
{{< / blockquote >}}

{{< figure src="acm-domains.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="acm-domains">}}

Values which are generated to prove that this domain is owned by you must be pasted in your dns provider. 

{{< blockquote >}}
**Note**: I had an isssue with **NameCheap** they didn't allow me to insert CNAME value from **AWS Certificate manager**, that's why I used **cloudflare** instead. The process of moving to manage your domain name in cloudflare is pretty straighforward you can find a guide in youtube.
{{< / blockquote >}}

This is how is looks like in my cloudflare dns settings.

{{< figure src="cloudflare-dns-settings.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="cloudflare-dns-settings">}}

After that done we are ready to move on with cloudfront distribution.

### Cloudfront

We need to start with creating new cloudfront distribution, the origin of our distribution will be our s3 bucket, however we don't want to use the endpoint for serving s3 as a website and want to use default s3 bucket's endpoint.
Also since we don't want to leave bucket public, we need to use second option for origin access, aws will generate bucket policy which must use. 

{{< figure src="cloudfront-creation.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="cloudfront creation">}}

Cache policy must be set to CachingOptimized.

{{< figure src="cache-policy-for-cloudfront.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="cache-policy-for-cloudfront">}}

Sine I want to make my distribution to be accesible from two following domains I added them as a CNAMES, you can keep one if you want to have one domain name.
Inside of ssl certificate field you must select, certificate which we created inside of a [ACM](#aws-certificate-manager).
If you decided to use two domains, ssl certificate must be covering both of them.

{{< figure src="distribution-alternative-names.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="distribution-alternative-names">}}

All of the other settings might be left at it's default state.

After we can safely create cloudfront distribution, it will generate for policy for the s3 bucket, which we need to copy and paste into s3 bucket policy

### S3 Bucket Policy

This is how it looks like in my s3 bucket permissions tab.

{{< figure src="s3-policy.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="s3-policy">}}

Now when we are done with cloudfront and s3 bucket we need to go to the last, but no the least part of architecture Lambda@Edge.

### Lambda@Edge

Lambda Edge is basically default lambda which will be triggered on the special event on cloudfront distribution. When the request comes to cloudfront distribution, this lambda edge function is going to be called, we can specify at which event this function will be called, each event describes the information available for the lambda function, since we would need to mutate the request object and overwrite s3 bucket object path, it is extermely important.

Since we are going to implement this function in javascript, the runtime must be Node.js, but you can implement function in any language.

{{< figure src="lambda-function-creation.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="lambda-function-creation.png">}}

By default aws lambda must have a permissions for logging to the **CloudWatch**, in my case I have had a problem with that and lambda couldn't create proper logs to the CloudWatch, so I created **new role** for lambda function. 

The Lambda@Edge function code is bellow:

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

If you are following guide change the hardcoded s3 bucket name in your lambda. When you created your function which modifies request, we are almost done and ready to deploy function.

Now go to lambda actions tab 

{{< figure src="lambda-actions.png" style="min-width: 250px; max-width:800px; margin:0;" width="100%" alt="lambda actions">}}

