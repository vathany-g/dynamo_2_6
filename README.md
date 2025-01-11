# dynamo_2_6
provider "aws" {
  region = "ap-southeast-1" # Replace with your preferred AWS region
}


resource "aws_dynamodb_table" "book_inventory" {
  name         = "thila-bookinventory"
  hash_key     = "ISBN"
  range_key    = "Genre"
  billing_mode = "PAY_PER_REQUEST"


  attribute {
    name = "ISBN"
    type = "S"
  }


  attribute {
    name = "Genre"
    type = "S"
  }
}


# IAM Role
resource "aws_iam_role" "dynamodb_access_role" {
  name = "thila-dynamodb-read-role"


  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Effect = "Allow",
        Principal = {
          Service = "ec2.amazonaws.com"
        },
        Action = "sts:AssumeRole"
      }
    ]
  })
}


# IAM Policy
resource "aws_iam_policy" "dynamodb_read" {
  name = "thila-dynamodb-read"


  policy = jsonencode({
    Version = "2012-10-17",
    Statement = [
      {
        Action   = ["dynamodb:List*", "dynamodb:Get*", "dynamodb:Scan"],
        Effect   = "Allow",
        Resource = "arn:aws:dynamodb:ap-southeast-1:255945442255:table/thila-bookinventory"
      }
    ]
  })
}


# Attach Policy to Role
resource "aws_iam_role_policy_attachment" "dynamodb_role_policy" {
  role       = aws_iam_role.dynamodb_access_role.name
  policy_arn = aws_iam_policy.dynamodb_read.arn
}


# IAM Instance Profile
resource "aws_iam_instance_profile" "dynamodb_profile" {
  name = "thila-dynamodb-profile"
  role = aws_iam_role.dynamodb_access_role.name
}


# EC2 Instance
resource "aws_instance" "dynamodb_reader" {
  ami           = "ami-0995922d49dc9a17d" # Replace with your preferred AMI ID
  instance_type = "t2.micro"


  subnet_id = "subnet-0417533960652300" # Replace with your Subnet ID


  vpc_security_group_ids = [
    aws_security_group.ec2_security_group.id
  ]


  iam_instance_profile = aws_iam_instance_profile.dynamodb_profile.name


  tags = {
    Name = "thila-dynamodb-reader"
  }
}


# Security Group
resource "aws_security_group" "ec2_security_group" {
  name        = "ec2-security-group"
  description = "Allow SSH and HTTPS traffic"
  vpc_id      = "vpc-036cf7926623c8d37" # Replace with your VPC ID


  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }


  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
