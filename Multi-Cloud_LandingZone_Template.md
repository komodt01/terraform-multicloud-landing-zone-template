##########################################
# AWS LANDING ZONE TEMPLATE
# Folder: /aws
##########################################

##########################################
# aws/main.tf
##########################################

terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.region
}

# VPC
resource "aws_vpc" "this" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(var.tags, {
    Name = "${var.project_name}-vpc"
  })
}

resource "aws_internet_gateway" "this" {
  vpc_id = aws_vpc.this.id

  tags = merge(var.tags, {
    Name = "${var.project_name}-igw"
  })
}

resource "aws_subnet" "public" {
  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.public_subnet_cidr
  map_public_ip_on_launch = true
  availability_zone       = var.az

  tags = merge(var.tags, {
    Name = "${var.project_name}-public-subnet"
    Tier = "public"
  })
}

resource "aws_subnet" "private" {
  vpc_id            = aws_vpc.this.id
  cidr_block        = var.private_subnet_cidr
  availability_zone = var.az

  tags = merge(var.tags, {
    Name = "${var.project_name}-private-subnet"
    Tier = "private"
  })
}

resource "aws_route_table" "public" {
  vpc_id = aws_vpc.this.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.this.id
  }

  tags = merge(var.tags, {
    Name = "${var.project_name}-public-rt"
  })
}

resource "aws_route_table_association" "public" {
  subnet_id      = aws_subnet.public.id
  route_table_id = aws_route_table.public.id
}

# Logging
resource "aws_cloudwatch_log_group" "vpc_flow_logs" {
  name              = "/${var.project_name}/vpc-flow-logs"
  retention_in_days = var.log_retention_days
  tags              = var.tags
}

resource "aws_iam_role" "vpc_flow_logs" {
  name = "${var.project_name}-vpc-flow-logs-role"
  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Principal = { Service = "vpc-flow-logs.amazonaws.com" }
      Action    = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy" "vpc_flow_logs" {
  name = "${var.project_name}-vpc-flow-logs-policy"
  role = aws_iam_role.vpc_flow_logs.id

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect = "Allow"
      Action = [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "logs:DescribeLogGroups",
        "logs:DescribeLogStreams"
      ]
      Resource = "*"
    }]
  })
}

resource "aws_flow_log" "vpc" {
  log_destination      = aws_cloudwatch_log_group.vpc_flow_logs.arn
  log_destination_type = "cloud-watch-logs"
  traffic_type         = "ALL"
  vpc_id               = aws_vpc.this.id
  iam_role_arn         = aws_iam_role.vpc_flow_logs.arn
  tags                 = var.tags
}

# IAM Example
resource "aws_iam_role" "readonly_role" {
  name = "${var.project_name}-readonly-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [{
      Effect    = "Allow"
      Principal = { AWS = var.trusted_admin_arn }
      Action    = "sts:AssumeRole"
    }]
  })
}

resource "aws_iam_role_policy_attachment" "readonly_attach" {
  role       = aws_iam_role.readonly_role.name
  policy_arn = "arn:aws:iam::aws:policy/ReadOnlyAccess"
}


##########################################
# aws/variables.tf
##########################################

variable "project_name" { type = string }
variable "region" { type = string default = "us-east-1" }
variable "vpc_cidr" { type = string default = "10.0.0.0/16" }
variable "public_subnet_cidr" { type = string default = "10.0.1.0/24" }
variable "private_subnet_cidr" { type = string default = "10.0.2.0/24" }
variable "az" { type = string default = "us-east-1a" }
variable "log_retention_days" { type = number default = 30 }
variable "trusted_admin_arn" { type = string }
variable "tags" {
  type = map(string)
  default = {
    environment         = "dev"
    owner               = "example-owner"
    cost_center         = "example-cost-center"
    data_classification = "internal"
  }
}


##########################################
# aws/outputs.tf
##########################################

output "vpc_id" { value = aws_vpc.this.id }
output "public_subnet_id" { value = aws_subnet.public.id }
output "private_subnet_id" { value = aws_subnet.private.id }
output "readonly_role_arn" { value = aws_iam_role.readonly_role.arn }


##########################################
# aws/terraform.tfvars.example
##########################################

project_name        = "example-landingzone"
region              = "us-east-1"
vpc_cidr            = "10.0.0.0/16"
public_subnet_cidr  = "10.0.1.0/24"
private_subnet_cidr = "10.0.2.0/24"
az                  = "us-east-1a"
trusted_admin_arn   = "arn:aws:iam::123456789012:user/example-admin"


##########################################
# AZURE LANDING ZONE TEMPLATE
# Folder: /azure
##########################################

##########################################
# azure/main.tf
##########################################

terraform {
  required_version = ">= 1.5.0"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

provider "azurerm" { features {} }

resource "azurerm_resource_group" "rg" {
  name     = "${var.project_name}-rg"
  location = var.location
  tags     = var.tags
}

resource "azurerm_virtual_network" "vnet" {
  name                = "${var.project_name}-vnet"
  address_space       = [var.vnet_cidr]
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  tags                = var.tags
}

resource "azurerm_subnet" "public" {
  name                 = "${var.project_name}-public-subnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [var.public_subnet_cidr]
}

resource "azurerm_subnet" "private" {
  name                 = "${var.project_name}-private-subnet"
  resource_group_name  = azurerm_resource_group.rg.name
  virtual_network_name = azurerm_virtual_network.vnet.name
  address_prefixes     = [var.private_subnet_cidr]
}

resource "azurerm_log_analytics_workspace" "law" {
  name                = "${var.project_name}-law"
  location            = var.location
  resource_group_name = azurerm_resource_group.rg.name
  retention_in_days   = var.log_retention_days
  sku                 = "PerGB2018"
  tags                = var.tags
}

resource "azurerm_monitor_diagnostic_setting" "vnet_diag" {
  name                       = "${var.project_name}-vnetdiag"
  target_resource_id         = azurerm_virtual_network.vnet.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.law.id

  metric {
    category = "AllMetrics"
    enabled  = true
  }
}

resource "azurerm_role_assignment" "reader" {
  scope                = azurerm_resource_group.rg.id
  role_definition_name = "Reader"
  principal_id         = var.reader_principal_object_id
}


##########################################
# azure/variables.tf
##########################################

variable "project_name" { type = string }
variable "location" { type = string default = "eastus" }
variable "vnet_cidr" { type = string default = "10.10.0.0/16" }
variable "public_subnet_cidr" { type = string default = "10.10.1.0/24" }
variable "private_subnet_cidr" { type = string default = "10.10.2.0/24" }
variable "log_retention_days" { type = number default = 30 }
variable "reader_principal_object_id" { type = string }
variable "tags" {
  type = map(string)
  default = {
    environment         = "dev"
    owner               = "example-owner"
    cost_center         = "example-cost-center"
    data_classification = "internal"
  }
}


##########################################
# GCP LANDING ZONE TEMPLATE
# Folder: /gcp
##########################################

##########################################
# gcp/main.tf
##########################################

terraform {
  required_version = ">= 1.5.0"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 5.0"
    }
  }
}

provider "google" {
  project = var.project_id
  region  = var.region
}

resource "google_compute_network" "vpc" {
  name                    = "${var.project_name}-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "${var.project_name}-subnet"
  ip_cidr_range = var.subnet_cidr
  region        = var.region
  network       = google_compute_network.vpc.id

  log_config {
    aggregation_interval = "INTERVAL_5_MIN"
    flow_sampling        = 0.5
    metadata             = "INCLUDE_ALL_METADATA"
  }
}

resource "google_compute_firewall" "allow_ssh_internal" {
  name    = "${var.project_name}-allow-ssh"
  network = google_compute_network.vpc.name

  allows {
    protocol = "tcp"
    ports    = ["22"]
  }

  source_ranges = [var.allowed_ssh_cidr]
}

resource "google_project_iam_binding" "viewer" {
  project = var.project_id
  role    = "roles/viewer"
  members = ["user:${var.viewer_user_email}"]
}


##########################################
# gcp/variables.tf
##########################################

variable "project_id" { type = string }
variable "project_name" { type = string }
variable "region" { type = string default = "us-central1" }
variable "subnet_cidr" { type = string default = "10.20.0.0/24" }
variable "allowed_ssh_cidr" { type = string default = "10.20.0.0/16" }
variable "viewer_user_email" { type = string }


##########################################
# OCI LANDING ZONE TEMPLATE
# Folder: /oci
##########################################

##########################################
# oci/main.tf
##########################################

terraform {
  required_version = ">= 1.5.0"
  required_providers {
    oci = {
      source  = "oracle/oci"
      version = "~> 6.0"
    }
  }
}

provider "oci" {
  region = var.region
}

resource "oci_core_vcn" "vcn" {
  compartment_id = var.compartment_ocid
  cidr_block     = var.vcn_cidr
  display_name   = "${var.project_name}-vcn"
}

resource "oci_core_internet_gateway" "igw" {
  compartment_id = var.compartment_ocid
  display_name   = "${var.project_name}-igw"
  vcn_id         = oci_core_vcn.vcn.id
  enabled        = true
}

resource "oci_core_route_table" "public_rt" {
  compartment_id = var.compartment_ocid
  vcn_id         = oci_core_vcn.vcn.id
  display_name   = "${var.project_name}-public-rt"

  route_rules {
    destination       = "0.0.0.0/0"
    destination_type  = "CIDR_BLOCK"
    network_entity_id = oci_core_internet_gateway.igw.id
  }
}

resource "oci_core_subnet" "public" {
  compartment_id      = var.compartment_ocid
  vcn_id              = oci_core_vcn.vcn.id
  cidr_block          = var.public_subnet_cidr
  display_name        = "${var.project_name}-public-subnet"
  route_table_id      = oci_core_route_table.public_rt.id
  prohibit_public_ip_on_vnic = false
}

resource "oci_core_subnet" "private" {
  compartment_id      = var.compartment_ocid
  vcn_id              = oci_core_vcn.vcn.id
  cidr_block          = var.private_subnet_cidr
  display_name        = "${var.project_name}-private-subnet"
  prohibit_public_ip_on_vnic = true
}

resource "oci_logging_log_group" "lg" {
  compartment_id = var.compartment_ocid
  display_name   = "${var.project_name}-log-group"
}

##########################################
# oci/variables.tf
##########################################

variable "project_name" { type = string }
variable "compartment_ocid" { type = string }
variable "tenancy_ocid" { type = string }
variable "region" { type = string default = "us-phoenix-1" }
variable "vcn_cidr" { type = string default = "10.40.0.0/16" }
variable "public_subnet_cidr" { type = string default = "10.40.1.0/24" }
variable "private_subnet_cidr" { type = string default = "10.40.2.0/24" }
variable "readonly_group_name" { type = string }


