/*
Assignment:

1. Create 2 servers and attach security group with port 22 and 80.
2. Server-1 on us-east-1a and server2 on us-east-1b
3. Servers need to deploy application (use remote exec concept)
4. Create a load balancer and attach the servers
5. Print output of load balancer DNS from output block.

*/


provider "aws" {
  alias  = "us_east_1"
  region = "us-east-1"
}

provider "aws" {
  alias  = "us_west_1"
  region = "us-west-1"
}

resource "aws_security_group" "web_sg_east" {
  provider    = aws.us_east_1
  name        = "web-access"
  description = "Allow SSH and HTTP traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
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

resource "aws_security_group" "web_sg_west" {
  provider    = aws.us_west_1
  name        = "web-access-west"
  description = "Allow SSH and HTTP traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
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

resource "aws_instance" "server_1" {
  provider      = aws.us_east_1
  ami          = "ami-04aa00acb1165b32a" # Example AMI, update as needed
  instance_type = "t2.micro"
  security_groups = [aws_security_group.web_sg_east.name]

  tags = {
    Name = "Server-1-East"
  }
}

resource "aws_instance" "server_2" {
  provider      = aws.us_west_1
  ami          = "ami-04acda42f3629e02b" # Example AMI, update as needed
  instance_type = "t2.micro"
  security_groups = [aws_security_group.web_sg_west.name]

 tags = {
    Name = "Server-2-West"
  }
}


###############################################################

provider "aws" {
  region = "us-east-1"
  alias  = "east"
}

provider "aws" {
  region = "us-west-1"
  alias  = "west"
}

resource "aws_elb" "my_elb_east" {
  provider           = aws.east
  name               = "my-load-balancer-east"
  availability_zones = ["us-east-1a", "us-east-1b"]

  listener {
    instance_port     = 80
    instance_protocol = "HTTP"
    lb_port           = 80
    lb_protocol       = "HTTP"
  }

  health_check {
    target              = "HTTP:80/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }

  instances = [aws_instance.server_1.id] #, aws_instance.server_2.id]

  tags = {
    Name = "MyELB-East"
  }
}

resource "aws_elb" "my_elb_west" {
  provider           = aws.west
  name               = "my-load-balancer-west"
  availability_zones = ["us-west-1a", "us-west-1c"]

  listener {
    instance_port     = 80
    instance_protocol = "HTTP"
    lb_port           = 80
    lb_protocol       = "HTTP"
  }

  health_check {
    target              = "HTTP:80/"
    interval            = 30
    timeout             = 5
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }

  instances = [aws_instance.server_2.id] #, aws_instance.web4.id]

  tags = {
    Name = "MyELB-West"
  }

}

output "load_balancer_dns_east" {
  value = aws_elb.my_elb_east.dns_name
}

output "load_balancer_dns_west" {
  value = aws_elb.my_elb_west.dns_name
}

