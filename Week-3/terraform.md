# HashiCorp Terraform 

HashiCorp Terraform is an Infrastructure as Code (IaC) tool that allows you to define cloud and on-premises resources in human-readable configuration files. These files can be versioned, reused, and shared. Terraform supports a consistent workflow for provisioning and managing infrastructure throughout its lifecycle, from low-level components like compute and storage to high-level components like DNS and SaaS features.

Terraform interacts with cloud platforms and services through their APIs using providers. These providers enable Terraform to manage resources on virtually any platform with an accessible API. The Terraform community has developed thousands of providers, which are available in the Terraform Registry. 

**Core Terraform Workflow:**
1. **Write:** Define resources across multiple providers in configuration files.
2. **Plan:** Terraform generates an execution plan describing the proposed changes to the infrastructure.
3. **Apply:** On approval, Terraform executes the plan, respecting resource dependencies to create, update, or destroy resources in the correct order.

#### **Why Use Terraform?**
- **Manage Any Infrastructure:** Terraform can manage resources across a wide range of platforms using providers available in the Terraform Registry.
- **Track Infrastructure:** Terraform uses a state file to track your infrastructure, acting as the source of truth and allowing you to preview and approve changes before applying them.
- **Automate Changes:** Terraform's declarative configuration files describe the desired state of your infrastructure. Terraform handles the logic of achieving this state, allowing for efficient resource provisioning.
- **Standardize Configurations:** Terraform supports reusable configuration components called modules, which promote best practices and save time.
- **Collaborate:** Configuration files can be committed to version control systems (VCS), enabling team collaboration. HCP Terraform provides additional features like secure state storage, role-based access controls, and a private module registry.

#### **Installation**
Terraform can be installed using Homebrew on macOS:
```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

#### **Terraform Keywords**
- **Provider:** Providers enable Terraform to interact with various platforms. They can be found in the Terraform Registry and are developed by HashiCorp, platform maintainers, or the community.
  
- **Module:**
  - **Root Module:** Every Terraform configuration has a root module consisting of the resources defined in the `.tf` files in the main directory.
  - **Child Modules:** Modules can call other modules to include their resources, referred to as child modules. These can be sourced from the local filesystem or public/private registries.
  
- **State:** Terraform maintains a state file (`terraform.tfstate`) to map real-world resources to your configuration. This state file is critical for tracking infrastructure changes and ensuring the desired state matches the actual infrastructure.

- **Variables:**
  - **Input Variables:** Allow customization of modules without altering the module's source code, making configurations reusable across different environments. Input variables are defined using the `variable` block and accessed as `var.<NAME>`.
  - **Output Variables:** Output values are declared using the `output` block and are used to export information from a module. These values are displayed after a Terraform apply operation.

# Case I Terraform EC2 Case

    Create a Terraform project and apply it to your own AWS Environment

    Application
    Create a static web page from your own EC2 Instance (t2.micro)

    Case Rules
        You must use Terraform AWS Provider.
        You should write your own module.
        You should add bootstrapping to your module.
        Don't do a manual process on your EC2 Instance
        Everyone must access your Application
        You can personalize the application
        You must add screenshots and readme files to your repository
        Some files are important for the GitLab repository. (ex .tfstate)
        Don't add your own AWS credentials to your GitLab Repository

## Terraform
I create a Terraform structure same as the hands on.

## nginx 
I use following commands to change the ```index.html``` of the ```nginx```:

```
sudo su 
nano /var/www/html/index.nginx-debian.html
systemctl restart nginx
```

