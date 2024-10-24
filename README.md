# terraform_ec2_vpc
# terraform-sample

# Write a simple Terraform script to:

# Deploy an EC2 instance in a VPC.
# Attach the instance to an Elastic Load Balancer (ELB).
# Use CloudWatch for monitoring CPU and disk space, triggering auto-scaling when thresholds are exceeded.

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  tags = {
    Name = "TerraformWebServer"
  }
}

resource "aws_elb" "main" {
  name               = "main-elb"
  availability_zones = ["us-west-2a"]

  listener {
    instance_port     = 80
    instance_protocol = "HTTP"
    lb_port           = 80
    lb_protocol       = "HTTP"
  }

  instances = [aws_instance.web.id]
}

resource "aws_cloudwatch_metric_alarm" "high_cpu" {
  alarm_name          = "high_cpu_alarm"
  comparison_operator = "GreaterThanOrEqualToThreshold"
  evaluation_periods  = 2
  metric_name         = "CPUUtilization"
  namespace           = "AWS/EC2"
  period              = 120
  statistic           = "Average"
  threshold           = 70

  alarm_description = "This metric monitors EC2 CPU usage"
  dimensions = {
    InstanceId = aws_instance.web.id
  }
}
