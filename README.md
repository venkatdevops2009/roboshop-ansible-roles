# RoboShop Ansible Roles

A comprehensive Ansible playbook and roles collection for deploying and managing the RoboShop microservices application infrastructure on AWS using Infrastructure as Code.

## 📋 Project Overview

This repository contains Ansible playbooks and roles for automating the deployment of RoboShop, a complete e-commerce microservices application. It handles infrastructure creation, configuration management, and orchestration of multiple services including databases, message queues, and application services.

---

## 📁 Repository Structure

```
roboshop-ansible-roles/
├── ansible.cfg                 # Ansible configuration file
├── inventory.ini              # Inventory file with host definitions
├── roboshop.yaml              # Main playbook for service configuration
├── instance-creation.yaml     # Playbook for EC2 instance and Route53 management
├── group_vars/                # Group variables directory
├── roles/                      # Ansible roles for each component
│   ├── common/                # Common configuration role
│   ├── mongodb/               # MongoDB database role
│   ├── redis/                 # Redis cache role
│   ├── mysql/                 # MySQL database role
│   ├── rabbitmq/              # RabbitMQ message broker role
│   ├── catalogue/             # Catalogue microservice role
│   ├── user/                  # User microservice role
│   ├── cart/                  # Cart microservice role
│   ├── shipping/              # Shipping microservice role
│   ├── payment/               # Payment microservice role
│   └── frontend/              # Frontend application role
└── .gitignore                 # Git ignore file
```

---

## 🔑 Key Components

### 1. **Configuration Files**

#### `ansible.cfg`
Main Ansible configuration file that specifies:
- Inventory file location: `./inventory.ini`
- Log file path: `/var/log/roboshop/ansible.log`

#### `inventory.ini`
Defines all hosts and groupings for the infrastructure:
- **local**: Localhost for local execution
- **mongodb**: MongoDB database server
- **redis**: Redis cache server
- **mysql**: MySQL database server
- **rabbitmq**: RabbitMQ message broker
- **catalogue**: Catalogue microservice
- **user**: User microservice
- **cart**: Cart microservice
- **shipping**: Shipping microservice
- **payment**: Payment microservice
- **frontend**: Frontend application

All services are mapped to `piridishop.shop` domain.

### 2. **Playbooks**

#### `roboshop.yaml` - Main Configuration Playbook
```yaml
# ansible-playbook roboshop.yaml -e "component=mongodb"
- name: configure "{{ component }}" server
  hosts: "{{ component }}"
  become: yes
  roles:
  - "{{ component }}"
```

**Usage**: Dynamically configures any component by passing the component name as a variable.

**Example Commands**:
```bash
# Deploy MongoDB
ansible-playbook roboshop.yaml -e "component=mongodb"

# Deploy Redis
ansible-playbook roboshop.yaml -e "component=redis"

# Deploy Frontend
ansible-playbook roboshop.yaml -e "component=frontend"
```

#### `instance-creation.yaml` - Infrastructure Management Playbook
Handles AWS EC2 and Route53 resource management:

**Supported Actions**:
- **create**: Provision EC2 instances and DNS records
- **delete**: Destroy EC2 instances and DNS records

**Key Features**:
- Creates 10 micro instances (t3.micro) across all components
- Automatically creates Route53 DNS records for each service
- Maintains service discovery with private IP addresses
- Public IP for frontend access
- Supports security group management

**Instances Managed**:
- mongodb, redis, mysql, rabbitmq
- catalogue, user, cart, shipping, payment
- frontend

**Configuration Variables**:
```yaml
instances: ["mongodb", "redis", "mysql", "rabbitmq", "catalogue", "user", "cart", "shipping", "payment", "frontend"]
image_id: ami-0220d79f3f480ecf5        # AMI ID (customizable)
domain_name: piridishop.shop           # Replace with your domain
action: create                          # or "delete"
region: us-east-1
instance_type: t3.micro
```

### 3. **Ansible Roles**

The `roles/` directory contains modular, reusable roles for each component:

#### **Infrastructure Roles**:
- **common**: Base configuration, dependencies, and common setup
- **mongodb**: NoSQL database installation and configuration
- **redis**: In-memory cache installation and configuration
- **mysql**: Relational database installation and configuration
- **rabbitmq**: Message broker installation and configuration

#### **Application Roles**:
- **catalogue**: Product catalogue microservice
- **user**: User management microservice
- **cart**: Shopping cart microservice
- **shipping**: Shipping management microservice
- **payment**: Payment processing microservice
- **frontend**: Web frontend application

---

## 🚀 Quick Start Guide

### Prerequisites
- AWS Account with appropriate IAM permissions
- Ansible installed (version 2.9+)
- AWS CLI configured with credentials
- SSH key pair for EC2 instances
- Valid domain name (or update `piridishop.shop` in configs)

### Step 1: Clone the Repository
```bash
git clone https://github.com/venkatdevops2009/roboshop-ansible-roles.git
cd roboshop-ansible-roles
```

### Step 2: Configure Your Environment
Edit `instance-creation.yaml` to customize:
```yaml
domain_name: your-domain.com           # Replace with your domain
image_id: ami-xxxxxxxx                 # Update to your region's AMI
region: us-east-1                      # Change if needed
```

### Step 3: Update Inventory
Edit `inventory.ini` to replace `piridishop.shop` with your domain:
```ini
[mongodb]
mongodb.your-domain.com

[redis]
redis.your-domain.com
# ... update all other services
```

### Step 4: Create Infrastructure
```bash
# Create EC2 instances and Route53 records
ansible-playbook instance-creation.yaml -e "action=create"
```

### Step 5: Configure Services
Deploy each component in dependency order:

```bash
# Step 1: Deploy MongoDB
ansible-playbook roboshop.yaml -e "component=mongodb"

# Step 2: Deploy Redis
ansible-playbook roboshop.yaml -e "component=redis"

# Step 3: Deploy MySQL
ansible-playbook roboshop.yaml -e "component=mysql"

# Step 4: Deploy RabbitMQ
ansible-playbook roboshop.yaml -e "component=rabbitmq"

# Step 5: Deploy Catalogue Service
ansible-playbook roboshop.yaml -e "component=catalogue"

# Step 6: Deploy User Service
ansible-playbook roboshop.yaml -e "component=user"

# Step 7: Deploy Cart Service
ansible-playbook roboshop.yaml -e "component=cart"

# Step 8: Deploy Shipping Service
ansible-playbook roboshop.yaml -e "component=shipping"

# Step 9: Deploy Payment Service
ansible-playbook roboshop.yaml -e "component=payment"

# Step 10: Deploy Frontend
ansible-playbook roboshop.yaml -e "component=frontend"
```

---

## 🔧 Advanced Usage

### Running Specific Playbooks

#### Deploy Single Component
```bash
ansible-playbook roboshop.yaml -e "component=redis"
```

#### Deploy with Verbose Output
```bash
ansible-playbook roboshop.yaml -e "component=mongodb" -v
```

#### Deploy with Additional Variables
```bash
ansible-playbook roboshop.yaml -e "component=mysql" -e "other_var=value"
```

### Infrastructure Management

#### Create All Infrastructure
```bash
ansible-playbook instance-creation.yaml -e "action=create"
```

#### Delete All Infrastructure
```bash
ansible-playbook instance-creation.yaml -e "action=delete"
```

#### Deploy Specific Hosts
```bash
ansible-playbook roboshop.yaml -e "component=catalogue" -i inventory.ini -l catalogue
```

---

## 📊 Service Architecture

### Dependency Order
```
MongoDB, Redis, MySQL, RabbitMQ (Infrastructure)
    ↓
Catalogue, User (Core Services)
    ↓
Cart, Shipping (Dependent Services)
    ↓
Payment (Dependent Services)
    ↓
Frontend (Presentation Layer)
```

### Network Configuration
- **Private Communication**: Services communicate using private IP addresses
- **External Access**: Frontend accessible via public IP
- **DNS Resolution**: All services discoverable via Route53 DNS records
- **Security Groups**: Separate security groups per component + common security group

---

## 📝 Inventory Structure

The `inventory.ini` file organizes hosts into logical groups:

```ini
[local]
localhost                           # Local machine for instance creation

[mongodb]
mongodb.piridishop.shop            # NoSQL Database

[redis]
redis.piridishop.shop              # Cache Layer

[mysql]
mysql.piridishop.shop              # Relational Database

[rabbitmq]
rabbitmq.piridishop.shop           # Message Queue

[catalogue]
catalogue.piridishop.shop          # Product Service

[user]
user.piridishop.shop               # User Service

[cart]
cart.piridishop.shop               # Cart Service

[shipping]
shipping.piridishop.shop           # Shipping Service

[payment]
payment.piridishop.shop            # Payment Service

[frontend]
frontend.piridishop.shop           # Web Frontend
```

---

## 🛠️ Customization

### Change Domain Name
1. Update `instance-creation.yaml`:
   ```yaml
   domain_name: your-domain.com
   ```

2. Update `inventory.ini`:
   ```ini
   [mongodb]
   mongodb.your-domain.com
   # ... update all services
   ```

### Change AWS Region
Update `instance-creation.yaml`:
```yaml
region: eu-west-1  # Or your preferred region
```

### Change Instance Type
Update `instance-creation.yaml`:
```yaml
instance_type: t3.small  # Or t3.medium for production
```

### Change AMI
Update `instance-creation.yaml`:
```yaml
image_id: ami-xxxxxxxx  # Get from your AWS region
```

---

## 🔐 Security Considerations

- Ensure EC2 instances have proper security group rules
- Use private IP addresses for internal communication
- Restrict frontend security group to necessary ports
- Store sensitive data in Ansible vault
- Use IAM roles for EC2 instance permissions
- Enable Route53 health checks for critical services
- Regular backup of database instances

---

## ❌ Cleanup

To remove all infrastructure:
```bash
ansible-playbook instance-creation.yaml -e "action=delete"
```

This will:
- Delete all EC2 instances
- Remove all Route53 DNS records
- Clean up security groups

---

## 📚 Additional Resources

- [Ansible Documentation](https://docs.ansible.com/)
- [AWS EC2 Module](https://docs.ansible.com/ansible/latest/collections/amazon/aws/ec2_instance_module.html)
- [Route53 Module](https://docs.ansible.com/ansible/latest/collections/amazon/aws/route53_module.html)
---

## 📞 Support & Contribution

For issues, questions, or contributions:
1. Open an issue on GitHub
2. Create a pull request with improvements
3. Follow Ansible best practices
4. Test playbooks thoroughly before deployment

---

## 📄 License

This project is provided as-is for educational and development purposes.

---

## ✍️ Author

**venkatdevops2009**

GitHub: [@venkatdevops2009](https://github.com/venkatdevops2009)

---

## 🔗 Related Projects

- [RoboShop DevOps](https://github.com/roboshop-devops)
- Ansible Role Collections
- AWS Infrastructure Automation

---

## 📋 Checklist for First-Time Deployment

- [ ] Fork/Clone repository
- [ ] Update domain name in both config files
- [ ] Update AMI ID for your AWS region
- [ ] Configure AWS credentials locally
- [ ] Test SSH connectivity
- [ ] Create VPC and security groups (if not using defaults)
- [ ] Run instance creation playbook
- [ ] Verify instances in AWS console
- [ ] Deploy components in order
- [ ] Test service connectivity
- [ ] Access frontend via domain name
- [ ] Monitor logs in `/var/log/roboshop/ansible.log`

---

**Last Updated**: July 2026
**Language**: Jinja (Ansible Templates)
**Status**: Active Development
