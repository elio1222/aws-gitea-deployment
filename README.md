## Architecture Summary
This project deploys a self-hosted Gitea service using Docker on an AWS EC2 instance. To ensure data persistence beyond the container lifecycle, a 100GB EBS volume is formatted with ext4 and bind-mounted from the host directory (~/data) to the container's internal path (/data). The architecture includes a disaster recovery workflow where a Bash script automates the compression of the data directory. By running `./backup.sh`, I then upload it to an Amazon S3 bucket utilizing an IAM Instance Role called "clerk". 

In hindsight, using a 100GB EBS volume was overkill for this assignment but it was the default so why not.

## Step-By-Step Deployment Instructions
I am using my EC2 instance from Assignment 2, so no need for me to launch a new instance.\
\
One of the first things was create an EBS volume on the AWS console, I attached it. This YouTube video helped me set up EBS on the web: https://www.youtube.com/watch?v=VnO3Lz7Qr0U.
\
\
Then, I did the following commands:
```
sudo lsblk
sudo mkfs.ext4 /dev/nvme1n1
mkdir -p ~/data
sudo mount /dev/nvme1n1 ~/data
df -h
```
ALMOST FORGOT I ALSO CHANGED THE OWNER: `sudo chown -R $USER:$USER ~/data`\
\
Essentially, I created a filesystem for the EBS volume and then mounted it do my `~/data` directory. To ensure, mount persistently, I also configured the `/etc/fstab/` so the volume mounts at boot.\

After my EBS was mounted, I needed to create a Dockerfile to run Gitea. These are the commands I did step-by-step:
```
mkdir -p ~/gitea
cd gitea/
sudo touch Dockerfile
sudo nano Dockerfile

FROM gitea/gitea:1.22
VOLUME ["/data"]
EXPOSE 3000 22
```
Then I used Docker MAGIC to run Gitea
```
docker build -t my-gitea .
docker run -d -p 3000:3000 -v ~/data:/data --name gitea-container my-gitea
docker start gitea-container
```

After I set up Gitea's settings on the Web from my EC2 Instance IP address on 3000 port. I created a S3 Bucket and IAM role that has full S3 access and called it clerk. From that, I used the `backup.sh` provided in the assignment pdf and uploaded the backup to my S3 Bucket.

The Restore Procedures and whatnot are in the Evidence folder.

## Docker Commands Used
Docker **build** command: `docker build -t my-gitea .` \
Docker **run** command: `docker run -d -p 3000:3000 -v ~/data:/data --name gitea-container my-gitea`\
Docker **container** command: `docker start gitea-container`


