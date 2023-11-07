# README.md for Deploying GPU-based Apps on AWS Elastic Beanstalk

This README guide outlines the major steps for deploying GPU-based applications using Amazon Elastic Beanstalk, including setting up IAM roles, EC2 instances with GPU support, CUDA drivers, and optional dependencies like FFmpeg.

## Prerequisites

Before proceeding, ensure you have an AWS account with the necessary permissions to create roles, instances, and Elastic Beanstalk environments.

## Step 1: Set Up IAM Role

1. Navigate to the IAM Management Console.
2. Create a new role with the following policies:
    - `AdministratorAccess-AWSElasticBeanstalk`
    - `AmazonEC2FullAccess`
    - `AmazonS3FullAccess`
    - `AWSHealthFullAccess`

## Step 2: Launch EC2 Instance

1. Go to the EC2 Dashboard.
2. Launch an instance with the following specifications:
    - AMI: `ami-099df472d83c1794b`
    - Instance Type: `g4dn.xlarge` for GPU support

## Step 3: Install CUDA Drivers

SSH into your EC2 instance and execute the following commands to install CUDA drivers:

```bash
sudo dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/fedora37/x86_64/cuda-fedora37.repo
sudo dnf clean all
sudo dnf -y install cuda-toolkit-12-3
sudo dnf -y module install nvidia-driver:latest-dkms
```

Verify the driver installation with the command:

```bash
nvidia-smi
```

## Step 4: Install FFmpeg (Optional)

Follow the instructions from this [GitHub repository](https://github.com/ranareehanaslam/Install-ffmpeg-on-AWS-Linux-AMI) to install FFmpeg on Amazon Linux.

```bash
sudo su -
cd /usr/local/bin
mkdir ffmpeg
cd ffmpeg
wget https://johnvansickle.com/ffmpeg/releases/ffmpeg-release-amd64-static.tar.xz
tar xvf ffmpeg-release-amd64-static.tar.xz
mv ffmpeg-6.0-amd64-static/ffmpeg .
ln -s /usr/local/bin/ffmpeg/ffmpeg /usr/bin/ffmpeg
exit
```

## Step 5: Install Additional Dependencies (Optional)

Execute the following commands to install optional dependencies:

```bash
sudo yum update -y
sudo yum install gcc-c++ cmake -y
sudo yum install python3-devel -y
sudo yum install boost boost-devel boost-python3 boost-python3-devel -y
sudo yum install openblas openblas-devel -y
sudo yum install lapack lapack-devel -y
sudo yum install libX11 libX11-devel -y
sudo yum install libpng libpng-devel -y
sudo yum install mesa-libGL -y
sudo yum install git -y
```

## Step 6: Create an EC2 Image

After setting up the instance, create an image (AMI) of the configured EC2 instance for later use in Elastic Beanstalk.

## Step 7: Set Up Elastic Beanstalk Environment

1. Navigate to the Elastic Beanstalk dashboard.
2. Create a new environment using the 'Web server environment' option.
3. Choose `Python` as the platform and select `Sample application`.
4. Configure the environment to use a combination of spot and on-demand instances for high availability and cost optimization.

### Configure Service Access

- Create and use a new service role or select the IAM role created earlier.

### Set Up Networking

- Choose the VPC and assign public IP addresses to the instances.
- Select the first two unique subnets.

### Configure Instance Options

- Choose the root volume type and size (e.g., 50GB Magnetic).
- Set the security group.
- Configure instance scaling options as needed, using both on-demand and spot instances.

### Launch Environment

- Insert the previously created AMI ID.
- Set scaling metrics.
- Review the configuration and launch the environment.

## Step 8: Deploy Your Application

Prepare your Flask or Python app for deployment:

1. Ensure you have a `requirements.txt` file for dependencies.
2. Name your Flask app `application.py` and initialize the Flask app as follows:

    ```python
    application = Flask(__name__)
    ```

3. Define routes with `@application.route('/', methods=['GET'])`.

Finally, upload your app to the Elastic Beanstalk environment and verify its GPU usage.

## Conclusion

After uploading your application and configuring the environment, wait for Elastic Beanstalk to set up the resources. Once ready, you can access your GPU-powered application hosted on AWS.
