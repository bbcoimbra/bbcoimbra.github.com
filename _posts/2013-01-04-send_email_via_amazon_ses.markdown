---
layout: post
title: "Send emails via Amazon SES"
---

**NOTE**: You have to configure your domain to send emails into AWS SES service
before try to send emails. Also AWS\_ACCESS\_KEY and AWS\_SECRET\_KEY env vars have
to be present into your shell env.

```shell
#!/bin/bash 
 
date="$(date -R)"
priv_key="$AWS_SECRET_KEY"
access_key="$AWS_ACCESS_KEY"
signature="$(echo -n "$date" | openssl dgst -sha256 -hmac "$priv_key" -binary | base64 -w 0)"
auth_header="X-Amzn-Authorization: AWS3-HTTPS AWSAccessKeyId=$access_key, Algorithm=HmacSHA256, Signature=$signature"
endpoint="https://email.us-east-1.amazonaws.com/"
 
message="YOUR MESSAGE HERE"
action="Action=SendEmail"
source="Source=$SOURCE_EMAIL"
to="Destination.ToAddresses.member.1=$DEST_EMAIL"
subject="Message.Subject.Data=Send Test"
message="Message.Body.Text.Data=$message"
 
curl -v -X POST -H "Date: $date" -H "$auth_header" --data-urlencode "$message" --data-urlencode "$to" --data-urlencode "$source" --data-urlencode "$action" --data-urlencode "$subject"  "$endpoint"
```