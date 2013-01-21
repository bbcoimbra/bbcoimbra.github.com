---
layout: post
title: "Send emails via Amazon SES"
---
```shell
#!/bin/bash 
 
date="$(date -R)"
priv_key="$AWS_SECRET_KEY"
access_key="$AWS_ACCESS_KEY"
signature="$(echo -n "$date" | openssl dgst -sha256 -hmac "$priv_key" -binary | base64 -w 0)"
auth_header="X-Amzn-Authorization: AWS3-HTTPS AWSAccessKeyId=$access_key, Algorithm=HmacSHA256, Signature=$signature"
endpoint="https://email.us-east-1.amazonaws.com/"
 
message="MENSAGEM AQUI"
action="Action=SendEmail"
source="Source=bruno.coimbra@elo7.com"
to="Destination.ToAddresses.member.1=bruno.coimbra@elo7.com"
subject="Message.Subject.Data=Teste de envio"
message="Message.Body.Text.Data=$message"
 
curl -v -X POST -H "Date: $date" -H "$auth_header" --data-urlencode "$message" --data-urlencode "$to" --data-urlencode "$source" --data-urlencode "$action" --data-urlencode "$subject"  "$endpoint"
```