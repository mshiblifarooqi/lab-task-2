# Terraform Setup Guide: Windows + AWS
## Complete Step-by-Step Instructions for Hands-On Lab

---

## Part 1: Install Terraform on Windows

### Step 1: Download Terraform

1. Visit: **https://www.terraform.io/downloads.html**
2. Scroll to "Windows (AMD64)" or your system architecture (32-bit or 64-bit)
3. Click to download the `.zip` file
4. You'll get a file like `terraform_1.X.X_windows_amd64.zip`

### Step 2: Extract the File

1. Right-click the downloaded `.zip` file
2. Select **Extract All...**
3. Extract to a folder like `C:\terraform` (create this folder if it doesn't exist)
4. After extraction, you should have `terraform.exe` in this folder

### Step 3: Add Terraform to System PATH

This allows you to use `terraform` command from anywhere in Command Prompt.

**Method A: Using GUI (Easier)**

1. Press **Windows Key + R** → Type `sysdm.cpl` → Press Enter
2. Click **Environment Variables** button (bottom right)
3. Under "System variables", click **New**
4. Variable name: `Path`
5. Variable value: `C:\terraform` (or wherever you extracted it)
6. Click **OK** on all windows
7. **Close Command Prompt completely and reopen it** (restart needed for changes to take effect)

**Method B: Using PowerShell (Advanced)**

Open PowerShell as Administrator and run:
```powershell
[Environment]::SetEnvironmentVariable("Path", "$env:Path;C:\terraform", "User")
```

### Step 4: Verify Installation

1. Open **Command Prompt** or **PowerShell**
2. Run this command:
   ```
   terraform --version
   ```
3. You should see output like:
   ```
   Terraform v1.6.0
   ```

✅ **If you see the version number, Terraform is installed correctly!**

---

## Part 2: Configure AWS Credentials

### Prerequisites
- AWS Account (Free Tier or paid)
- AWS IAM user with programmatic access (Access Key + Secret Access Key)

### Step 1: Create AWS IAM User (if you haven't already)

1. Login to AWS Console: **https://console.aws.amazon.com**
2. Search for **IAM** service
3. Click **Users** (left sidebar)
4. Click **Create User**
5. Username: `terraform-user` (or any name)
6. Click **Next**
7. Select **Attach policies directly**
8. Search for and select: **EC2FullAccess** (for this lab)
9. Click **Next** → **Create User**
10. Click on your newly created user
11. Click **Security Credentials** tab
12. Click **Create Access Key**
13. Select **Command Line Interface (CLI)**
14. Click the checkbox for confirmation
15. Click **Create Access Key**
16. **IMPORTANT: Copy and save your Access Key and Secret Access Key** (you'll only see it once!)

### Step 2: Store AWS Credentials on Windows

**Option A: Using Environment Variables (Recommended)**

1. Press **Windows Key + R** → Type `sysdm.cpl` → Press Enter
2. Click **Environment Variables**
3. Click **New** (under System variables or User variables)
4. Create the following variables:

   | Variable Name | Variable Value |
   |---|---|
   | `AWS_ACCESS_KEY_ID` | Your Access Key ID |
   | `AWS_SECRET_ACCESS_KEY` | Your Secret Access Key |
   | `AWS_DEFAULT_REGION` | `us-east-1` |

4. Click **OK** on all windows
5. **Restart Command Prompt/PowerShell for changes to take effect**

**Option B: Using AWS CLI Config File**

1. Open Command Prompt or PowerShell
2. Run:
   ```
   aws configure
   ```
3. Enter your credentials when prompted:
   ```
   AWS Access Key ID: [your-access-key-id]
   AWS Secret Access Key: [your-secret-access-key]
   Default region name: us-east-1
   Default output format: json
   ```

### Step 3: Verify AWS Credentials

Open Command Prompt/PowerShell and run:
```
aws sts get-caller-identity
```

You should see output like:
```json
{
    "UserId": "AIDAJ...",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/terraform-user"
}
```

✅ **If you see your Account ID, AWS credentials are configured correctly!**

---

## Part 3: Create Your First Terraform Project

### Step 1: Create Project Directory

1. Open Command Prompt or PowerShell
2. Create a folder for your project:
   ```
   mkdir C:\Users\YourUsername\terraform-lab
   cd C:\Users\YourUsername\terraform-lab
   ```

### Step 2: Initialize Terraform Project

1. Inside your project folder, run:
   ```
   terraform init
   ```

2. You should see output like:
   ```
   Terraform has been successfully configured!
   ```

3. This creates a `.terraform` folder (hidden) - don't delete it!

### Step 3: Verify Setup

Run:
```
terraform --version
aws sts get-caller-identity
```

Both should work without errors.

✅ **You're ready to write Terraform code!**

---

## Part 4: Quick Verification Checklist

Before moving to the lab, verify:

- [ ] Terraform installed (`terraform --version` shows version)
- [ ] AWS credentials configured (`aws sts get-caller-identity` shows your Account ID)
- [ ] Project folder created and initialized (`terraform init` completed successfully)
- [ ] Using Command Prompt or PowerShell from project folder

---

## Troubleshooting

### Problem: "terraform: command not found"
**Solution:** 
- Restart Command Prompt/PowerShell after adding PATH
- Verify Terraform is in your PATH: 
  ```
  echo %PATH%
  ```

### Problem: "Unable to locate credentials"
**Solution:**
- Verify AWS credentials with: `aws sts get-caller-identity`
- Check environment variables are set correctly
- Restart Command Prompt/PowerShell after setting variables

### Problem: "InvalidClientTokenId" error when running terraform
**Solution:**
- Your AWS Access Key ID or Secret is wrong
- Regenerate credentials in AWS IAM console
- Use fresh Access Key and Secret

### Problem: "NoCredentialProviders"
**Solution:**
- Credentials are not configured
- Try using `aws configure` command
- Or set environment variables manually

---

## Next Steps

Once setup is complete, proceed to: **"Lab Exercise: Create EC2 Instance with Terraform"**

See the Terraform code file for the EC2 creation exercise.
