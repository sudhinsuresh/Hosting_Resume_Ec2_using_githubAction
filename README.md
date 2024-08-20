# Hosting_Resume_Ec2_using_githubAction
# Push-to-EC2

This repository contains a CI/CD pipeline that automates the deployment of changes to an EC2 instance using GitHub Actions. The pipeline is triggered whenever changes are pushed to the `main` branch.

## Workflow Overview

The GitHub Actions workflow defined in this repository performs the following steps:

1. **Trigger on Push to Main Branch**: The workflow is triggered only when changes are pushed to the `main` branch.

2. **Checkout the Repository**: The workflow checks out the repository to obtain the latest files.

3. **Deploy to EC2**: 
   - The workflow connects to an EC2 instance using SSH.
   - It then deploys the latest files from the repository to the specified directory on the EC2 server.

4. **Execute Remote SSH Commands**:
   - The workflow updates the package list on the EC2 instance.
   - It installs and starts the Apache2 web server.
   - Finally, it moves the deployed files to the Apache server's root directory.

## GitHub Actions Configuration

Here is a summary of the GitHub Actions configuration used in this repository:

```yaml
# Trigger deployment only on push to main branch
on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy to EC2 on main branch push
    runs-on: ubuntu-latest

    steps:
      - name: Checkout the files
        uses: actions/checkout@v2

      - name: Deploy to Server 1
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.EC2_SSH_KEY }}
          REMOTE_HOST: ${{ secrets.HOST_DNS }}
          REMOTE_USER: ${{ secrets.USERNAME }}
          TARGET: ${{ secrets.TARGET_DIR }}

      - name: Executing remote ssh commands using ssh key
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST_DNS }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo apt-get -y update
            sudo apt-get install -y apache2
            sudo systemctl start apache2
            sudo systemctl enable apache2
            cd home
            sudo mv * /var/www/html
