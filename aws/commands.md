## How to make Continuous Deployment (CD) configuration.

#### Required

- Open SSH: run `Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'` on Power Shell and verify if all is installed, else install.

#### Using

- AWS Elastic Compute Cloud (EC2)
- Github Actions

#### 1. Setting up instance on EC2

1. Create an instance using the wanted OS (recommended is Linux).
2. In the last step, generate a key pair .pem file and download it.
3. Enable anywhere to access IPV4 and IPV6 in API_PORT
   1. Access EC2 > Instances > Security
   2. Click on Security Groups
   3. Edit inbound and outbound rules enabling anywhere access in API_PORT

#### 2. Access EC2 with SSH and .pem

1. Access EC2 > Instances > Your Instance > Connect > SSH Client
2. Copy the example and change the filename if necessary.
3. In the file folder, execute the command.

#### 3. Installing Docker

1. Update: `sudo yum update`
2. Install docker: `sudo yum install docker`
3. Install docker-compose:
   1. `wget https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)`
   2. `wget https://github.com/docker/compose/releases/latest/download/`
   3. `docker-compose-$(uname -s)-$(uname -m)`
   4. `sudo mv docker-compose-$(uname -s)-$(uname -m) /usr/local/bin/docker-compose`
   5. `sudo chmod -v +x /usr/local/bin/docker-compose`
4. Enable docker on boot: `sudo systemctl enable docker.service`
5. Start docker: `sudo systemctl start docker.service`
6. Setting up docker:
   1. `sudo groupadd docker`
   2. `sudo usermod -aG docker $USER`
   3. `newgrp docker`

#### 3. Setting up the repository

1. Clone: git clone https://github.com/mytattoo-org/api.git api
2. Cd: cd api
3. Setting up .env
   1. Create file: touch .env
   2. Edit file:
      1. vim .env
      2. **Press i to insert**
      3. **Paste env variables**
      4. **:wq to save**

#### 4. Set up a Github Action workflow and connect it to EC2.

1. Use EC2 access command to create Github repository secrets EC2_HOST and EC2_USER.
   - For example, command do access ec2: `ssh -i "mytattoo.pem" ec2-user@ec2-18-228-119-214.sa-east-1.compute.amazonaws.com`
     - secrets.EC2_HOST = ec2-18-228-119-214.sa-east-1.compute.amazonaws.com
     - secrets.EC2_USER = ec2-user
2. Make an EC2_PRIVATE_KEY secret by copying .pem content.
3. Create .github/workflows/main.yml

   ```
   name: CD

   on:
   push:
   branches:
     - master

   jobs:
   deploy:
   name: Deploy to EC2
   runs-on: ubuntu-latest
   steps:
     - uses: actions/checkout@v2
     - name: Checkout the files
       env:
         HOST: ${{ secrets.EC2_HOST }}
         USER: ${{ secrets.EC2_USER }}
         PRIVATE_KEY: ${{ secrets.EC2_PRIVATE_KEY }}
       run: |
         echo "$PRIVATE_KEY" > private_key && chmod 600 private_key
         ssh -o StrictHostKeyChecking=no -i private_key ${USER}@${HOST} '
           cd api &&
           git checkout master &&
           git pull origin master &&
           docker-compose up -d --build
         '

   ```

4. Push on master and test :rocket:
