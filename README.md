# MTA-STS

## A guide to host mta-sts.txt with GitHub via Netlify, and deploy your policy in DNS

If MTA-STS is new to you, please read Google's documentation at
[Increase email security with MTA-STS and TLS reporting](https://support.google.com/a/answer/9261504) to familiarize with how to properly configure MTA-STS before proceeding. While Google's documentation is excellent, it does not get into how to host the mta-sts.txt file, and that's where this help guide comes in.

This repo has a separate directory for each MTA-STS mode (*testing, enforce, none*) with an mta-sts.txt file configured for the corresponding mode. By connecting your repo to a site in Netlify to host the mta-sts.txt file, you can easily change which mode you're in by modifying the *publish directory* setting in the Netlify site. Furthermore, this allows you to use one repo for one or more sites in Netflify, for each domain name added in your Google G Suite account with the same MX records.

For in-depth details how MTA-STS works you can review [SMTP MTA Strict Transport Security (MTA-STS)](https://tools.ietf.org/html/rfc8461) from IETF.

## 1. GitHub

1. Click to [create your own template copy of this repo](https://github.com/kenfraser/gsuite-mta-sts/generate), or click the green button near the top right called "Use this template" to perform the same action.
2. Now that you have your own copy, in each directory (*testing, enforce, none*), configure your mta-sts.txt file:
   + Verify the MX key/value pairs match what you have in DNS
   + Edit the max_age key/value pair to your preferred time in seconds (*between 1 day and 1 year*)

## 2. Netlify

1. Create a site:
   + Point to your new repo in GitHub
   + Set the publish directory to the MTA-STS mode you want to be in (*testing, enforce, none*)
   + After the site is created and deployed, verify you can get to the mta-sts.txt file by going to https://your-site-name.netlify.app/.well-known/mta-sts.txt, and that it's showing the correct mode that you entered as the publish directory.
2. Add your subdomain to the new site:
   + Domain Management > Add a Domain
   + Enter your subdomain mta-sts.yourdomain.com
   + Copy the default subdomain (*ex. your-site-name.netlify.app*), as you will need this for the next step to create a CNAME record in DNS.

## 3. DNS (*Point to Netlify*)

Create a CNAME record in DNS pointing to the site created in Netlify.

1. Go to your DNS management console (*ex. Cloudflare, Amazon Route 53, etc.*)
2. Create a new CNAME record for yourdomain.com

| Type  | Name    | Target                     |
| ----- | ------- | -------------------------- |
| CNAME | mta-sts | your-site-name.netlify.app |


 >
*Note:* If you manage DNS with Cloudflare, disable the proxy and set to DNS only (*uncheck the orange cloud*)

3. Go to https://yourdomain.com/.well-known/mta-sts.txt to verify the site is working. You may have to wait a bit for DNS to propogate.

## 4. Google G Suite

Create either a new mailbox, or an email alias to receive reports from other mail servers that support MTA-STS.

__Address examples:__
+ smtp.tls.reports@yourdomain.com
+ tls.report@yourdomain.com
+ mta.sts@yourdomain.com

*Optional:* If you're creating an email alias, you could create a custom filter in Gmail to label all incoming mail sent to the alias you created and make them easier to find. (*ex label. MTA-STS Reports*)

## 5. DNS (*Deploy policy*)

With all the above in place, it's time to deploy your policy. Head back to your DNS management console and create two TXT records.

__Important__: For the id value in the _mta-sts TXT record, replace the timestamp *1597582738053* with either the current timestamp or another unique value (*1–32 alphanumeric characters*). You will need to update this value every time a change to your published policy is made to tell other mail servers there is a newer policy. You can get the current time stamp by going to [Unix Time Stamp](https://www.unixtimestamp.com/) or whichever your preferred method is.

| Type | Name       | Value                                                  |
| ---- | ---------- | ------------------------------------------------------ |
| TXT  | _smtp._tls | v=TLSRPTv1;rua=mailto:smtp.tls.reports@yourdomain.com; |
| TXT  | _mta-sts   | v=STSv1;id=1597582738053;                              |

## 6. G Suite (*Verify policy is valid and published*)

In your G Suite Admin Console, go to [MTA-STS Configuration Diagnostics](https://admin.google.com/ac/apps/cs/diagnostic). If it shows "MTA-STS Configuration – Valid" you are good to go. If says "Invalid" go back and review any potential errors in your configuration.
