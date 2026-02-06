**GCP login**



gcloud auth application-default login



$env:Path = "C:\\terraform;" + $env:Path



Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass



terraform plan -var "project\_id=skybyte-485704"





**Get Policy**





gcloud projects get-iam-policy skybyte-485704 --flatten="bindings\[].members" --filter="bindings.role:roles/iap.tunnelResourceAccessor"





**Set Policy**



gcloud projects add-iam-policy-binding skybyte-485704 --member="user:YOUR\_EMAIL" --role="roles/iap.tunnelResourceAccessor"





**Check firewall rule network**



gcloud compute firewall-rules describe allow-iap-ssh







$env:GOOGLE\_APPLICATION\_CREDENTIALS = "C:\\Users\\akrra\\Downloads\\prod-testzeus-hard-285a60594ea5.json"

