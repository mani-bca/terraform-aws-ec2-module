# terraform-aws-ec2-modules


```bash
#!/bin/bash

# Update system packages
echo "Updating system packages..."
apt-get update -y
apt-get upgrade -y

# Install required dependencies
echo "Installing dependencies..."
apt-get install -y apt-transport-https ca-certificates curl software-properties-common

# Install Docker
echo "Installing Docker..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update -y
apt-get install -y docker-ce docker-ce-cli containerd.io

# Install Docker Compose
echo "Installing Docker Compose..."
curl -L "https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Start Docker service
echo "Starting Docker service..."
systemctl start docker
systemctl enable docker

# Create directory for DevLake
echo "Setting up DevLake..."
mkdir -p /opt/devlake
cd /opt/devlake

# Download DevLake docker-compose file
curl -LO https://raw.githubusercontent.com/apache/incubator-devlake/main/docker-compose.yml

# Modify the configuration to expose port 8080
sed -i 's/8080:8080/8080:8080/g' docker-compose.yml

# Start DevLake
echo "Starting DevLake services..."
docker-compose up -d

echo "Installation complete. DevLake should be available on port 8080."
```

To include this in your Terraform EC2 resource, you can use the following configuration:

```hcl
resource "aws_instance" "devlake_server" {
  ami           = "ami-0c55b159cbfafe1f0"  # Replace with recent Ubuntu AMI ID
  instance_type = "t2.medium"              # Recommended at least 2GB RAM for DevLake
  
  vpc_security_group_ids = [aws_security_group.devlake_sg.id]
  
  user_data = <<-EOF
#!/bin/bash

# Update system packages
echo "Updating system packages..."
apt-get update -y
apt-get upgrade -y

# Install required dependencies
echo "Installing dependencies..."
apt-get install -y apt-transport-https ca-certificates curl software-properties-common

# Install Docker
echo "Installing Docker..."
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update -y
apt-get install -y docker-ce docker-ce-cli containerd.io

# Install Docker Compose
echo "Installing Docker Compose..."
curl -L "https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# Start Docker service
echo "Starting Docker service..."
systemctl start docker
systemctl enable docker

# Create directory for DevLake
echo "Setting up DevLake..."
mkdir -p /opt/devlake
cd /opt/devlake

# Download DevLake docker-compose file
curl -LO https://raw.githubusercontent.com/apache/incubator-devlake/main/docker-compose.yml

# Start DevLake
echo "Starting DevLake services..."
docker-compose up -d

echo "Installation complete. DevLake should be available on port 8080."
EOF

  tags = {
    Name = "DevLake Server"
  }
}

# Security group to allow traffic to port 8080
resource "aws_security_group" "devlake_sg" {
  name        = "devlake_security_group"
  description = "Allow DevLake traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Restrict this to your IP for production
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Restrict this to your IP for production
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Make sure to:
1. Replace the AMI ID with a recent Ubuntu AMI ID for your target region
2. Consider using a larger instance type if you'll be processing a lot of data in DevLake
3. Restrict the security group CIDR blocks to specific IPs in a production environment

This script will install Docker, download the DevLake docker-compose file, and start DevLake with port 8080 exposed.
