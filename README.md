# MTA-STS

## A serverless solution to host mta-sts.txt with GitHub via Netlify, and deploy your policy in DNS

This repo has a folder for each MTA-STS mode ([testing](testing/.well-known/mta-sts.txt), [enforce](enforce/.well-known/mta-sts.txt), [none](none/.well-known/mta-sts.txt)) with an mta-sts.txt file configured for the corresponding mode. By setting the publish directory in [netfliy.toml](netlify.toml) you can easily switch which mode you are in.

You can also use the one repo with multiple Netlify sites if you manage multiple domain names that have the same MX records in DNS. If that is the case you could either use the one netlify.toml file to change the mode for all domains in one go, or if you prefer to manage their modes separately, delete the netlify.toml file and configure the publish directory in the Netlify UI for each site.

### 1. GitHub

1. Click to [create your own template copy of this repo](https://github.com/kenfraser/gsuite-mta-sts/generate), or click the green button near the top right called "Use this template" to perform the same action.
2. Now that you have your own copy, in each directory ([testing](testing/.well-known/mta-sts.txt), [enforce](enforce/.well-known/mta-sts.txt), [none](none/.well-known/mta-sts.txt)), configure the mta-sts.txt file:
   - If needed, modify the `mx` key/value pairs so there is a match for each MX record in your DNS.
   - if needed, modify `max_age` to your preferred time in seconds (_between 1 day and 1 year_)
3. In [netfliy.toml](netlify.toml), set which directory / MTA-STS mode you want published to Netlify

### 2. Netlify

1. Create a site:
   - Point to your new repo in GitHub
   - After the site is created and deployed, verify that you can get to the mta-sts.txt file by going to `https://your-site-name.netlify.app/.well-known/mta-sts.txt`, and that it's showing the correct mode you entered as the publish directory in your netlify.toml file.
2. Add your subdomain to the new site:
   - Domain Management > Add a Domain
   - Enter your subdomain `mta-sts.yourdomain.com`
   - Copy the default subdomain (_ex._ `your-site-name.netlify.app`), as you will need this for the next step to create a CNAME record in DNS.

### 3. DNS (_Point to Netlify_)

Create the subdomain `mta-sts` and point it your site in Netlify.

1. Go to your DNS management console (_ex. Cloudflare, Amazon Route 53, etc._)
2. Create a new CNAME record for `yourdomain.com`

| Type    | Name      | Target                       |
| ------- | --------- | ---------------------------- |
| `CNAME` | `mta-sts` | `your-site-name.netlify.app` |

_Note:_ If you manage DNS with Cloudflare, disable the proxy and set to DNS only (_uncheck the orange cloud_)

3. Go to `https://yourdomain.com/.well-known/mta-sts.txt` to verify the site is working. You may have to wait a bit for DNS to propogate.

### 4. Google G Suite

Create either a new mailbox, or an email alias to receive reports from other mail servers that support MTA-STS.

**Address examples:**

- `smtp.tls.reports@yourdomain.com`
- `tls.report@yourdomain.com`
- `mta.sts@yourdomain.com`

_Optional:_ If you're creating an email alias, you could create a custom filter in Gmail to label all incoming mail sent to the alias you created and make them easier to find. (_ex label. MTA-STS Reports_)

### 5. DNS (_Deploy policy_)

With all the above in place, it's time to deploy your policy. Head back to your DNS management console and create two TXT records.

| Type  | Name         | Value                                                    |
| ----- | ------------ | -------------------------------------------------------- |
| `TXT` | `_smtp._tls` | `v=TLSRPTv1;rua=mailto:smtp.tls.reports@yourdomain.com;` |
| `TXT` | `_mta-sts`   | `v=STSv1;id=1597582738053;`                              |

**`_smtp._tls`**: Replace the email address with the one you created.

**`_mta-sts`**: Replace the timestamp _1597582738053_ with either the current timestamp or another unique value (_1–32 alphanumeric characters_). You will need to update this value every time a change to your published policy is made to tell other mail servers there is a newer policy. You can get the current time stamp by going to [Unix Time Stamp](https://www.unixtimestamp.com/) or whichever your preferred method is.

### 6. G Suite (_Verify policy is valid and published_)

In your G Suite Admin Console, go to [MTA-STS Configuration Diagnostics](https://admin.google.com/ac/apps/cs/diagnostic). If it shows "MTA-STS Configuration – Valid" you are good to go. If says "Invalid" go back and review any potential errors in your configuration.

## Resources

- Google G Suite Admin Help: [Increase email security with MTA-STS and TLS reporting](https://support.google.com/a/answer/9261504)
- IETF RFC 8461: [SMTP MTA Strict Transport Security (MTA-STS)](https://tools.ietf.org/html/rfc8461)
