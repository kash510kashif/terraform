provider "aws" {
  region     = "us-west-2"
  access_key = "
  secret_key = "
}
###############Creting VPC##############
resource "aws_vpc" "my_vpc" {
  cidr_block = "10.0.0.0/16"

}
###############creating subnet##########
resource "aws_subnet" "my_subnet" {
  vpc_id            = aws_vpc.my_vpc.id
  cidr_block        = "10.0.0.0/24"
  availability_zone = "us-west-2"

 
}
################Internet Gateway#################
resource "aws_internet_gateway" "gw" {
  vpc_id = aws_vpc.my_vpc.id
}

###########Routing Table##########
resource "aws_route_table" "rt" {
vpc_id = aws_vpc.my_vpc.id 
route = []
}
###############Route#####################
resource "aws_route" "route" {
  route_table_id            = aws_route_table.rt.id 
  destination_cidr_block    = "0.0.0.0/0"
  gateway_id = aws_internet_gateway.gw.id 

}  
#################Creating security group################
resource "aws_security_group" "sg" {
  name        = "allow_all"
  description = "Allow TLS inbound traffic"
  vpc_id      =  aws_vpc.my_vpc.id

  ingress {
    description      = "inbound"
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = null
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = null
        }
  
  tags = {
    Name = "allow_tls"
  }
}
##########Route table association##############
resource "aws_route_table_association" "a" {
  subnet_id      = aws_subnet.my_subnet.id 
  route_table_id = aws_route_table.rt.id    
}
###############Creating instance#############
resource "aws_instance" "ec" {
    subnet_id=aws_subnet.my_subnet.id
    ami = "ami-0a5899928eba2e7bd"
    instance_type = "t2.micro"
  
}


