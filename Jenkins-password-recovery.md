# 🔐 Jenkins Password Recovery: From Panic to Access in Minutes

![Cover](https://cdn.hashnode.com/res/hashnode/image/upload/v1747974098712/1e57bf96-e7ca-4cc7-af7f-ffb38ff634d6.png)

## ❓ What If You Forget Your Jenkins Password?

Let’s talk about a situation we don’t usually think about…
What if you forget your **Jenkins password**? Or worse — what if **the Jenkins admin** forgets theirs?

---

## 🧪 Real-Time Setup & Workflow Tips

1. **Default Admin**: Jenkins starts with a built-in `admin` account. Use the initial password from `/secrets/initialAdminPassword`.
2. **Create Your Own Admin Account**: Right after setup, create a user like `devops_admin` and disable the default `admin`.
3. **Role-based User Management**: Create accounts like `qa_user`, `dev_user`, and assign them folder-specific rights.
4. **Password Resets for Users**:

   * Admin goes to **Manage Jenkins → Manage Users**
   * Click user → **Configure → Reset Password**

---

## 🔑 Scenario 1: Regular User Forgot Password

* Admin resets it under **Manage Users**
* New password is shared securely
* User logs back in

## 🔐 Scenario 2: Admin Forgot Password

This is serious, but not the end.

### ✅ Real-World Best Practices

* Always have **at least 2 admin accounts**
* Store credentials in tools like **AWS Secrets Manager**, **1Password**, or **Vault**
* Don’t use the default `admin` account long-term

---

### 💥 Worst Case: No Admin Access Left

Just like breaking a car window in an emergency, here’s how to safely regain Jenkins admin access:

### 🔓 Reset Jenkins Admin Password via `config.xml`

1. SSH into the Jenkins server (EC2 or VM)
2. Go to the Jenkins home dir:

```bash
cd /var/lib/jenkins/
```

3. Edit the config file:

```bash
sudo vim config.xml
```

4. Find:

```xml
<useSecurity>true</useSecurity>
```

Change it to:

```xml
<useSecurity>false</useSecurity>
```

5. Restart Jenkins:

```bash
sudo systemctl restart jenkins
```

6. Access Jenkins — login screen will be **skipped**
7. Navigate to:

   * **Manage Jenkins → Configure Global Security**
   * Re-enable security and use "Jenkins’ own user database"
8. Go to **Users**, reset the admin password
9. Re-enable security rules and authorization settings

![Bypass login screen](https://cdn.hashnode.com/res/hashnode/image/upload/v1747972690545/2c2028ae-b2f6-4f4f-b210-aa0486dba13b.png)

🎥 [Watch the video demo here](https://drive.google.com/file/d/1RsGyliP3Oyng8XwI6tlmQhEKIDU-k9UB/preview)

---

## 🙈 Forgot Your Username Too?

No worries. Run:

```bash
cd /var/lib/jenkins/users/
ls
```

Each folder = one user. That’s your username!

![Users Folder](https://cdn.hashnode.com/res/hashnode/image/upload/v1747973638479/711ff75b-4ed1-447f-bee8-8d8177fc7dd8.png)

---

## 🧠 Key Takeaways

* ❌ Stop using `admin` long-term — create named admin accounts
* 🔐 Store secrets in vaults, not Notepad!
* 🧍‍♀️ Keep multiple admins — no single point of failure
* 🔁 Practice resets in a test Jenkins setup
* 💾 Regular backups of `/var/lib/jenkins/`

> **Pro Tip:** Run quarterly "Jenkins password drills" just like fire drills 🚨

---

💬 Found this helpful? Connect with me on [**LinkedIn**](https://www.linkedin.com/in/bhavya-pasupuleti/) and let’s grow together in DevOps!

✨ Star the GitHub repo or share this post with your team!

📖 **Full Blog on Hashnode:** [https://devops-by-bhavya.hashnode.dev/](https://devops-by-bhavya.hashnode.dev/)

🔐 #jenkins #devops #cicd #security #passwordrecovery
