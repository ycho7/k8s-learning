# For this setup 

Cloud Platform: GCP 
VM spec: e2-standard-4 (4 vCPUs, 16 GB Memory)
Storage: 50GB

gcloud init
gcloud compute instances list
gcloud compute ssh <vm-name> --zone=<zone>
setup %USERPROFILE% for the .ssh profile so 
```
Host ycho-gcp
    HostName <IP>
    User <Username>
    IdentityFile <user_profile>/.ssh/google_compute_engine
    IdentitiesOnly yes
```

Install k3s



