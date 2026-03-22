# Terraform Drift – Complete Explanation

## 1️⃣ Terraform Architecture Basics

Before understanding drift, we must understand how Terraform works internally.

Terraform works with three layers:

```

Terraform Configuration (.tf files)  →  Terraform State File  →  Real Infrastructure

```

### 🔹 1. .tf files (Configuration)
- Defines the **desired state**
- Written in HCL
- Example: instance type, tags, size, etc.

### 🔹 2. terraform.tfstate (State File)
- Terraform’s **source of truth**
- Stores the last known infrastructure state
- Maps resource addresses to real provider IDs

### 🔹 3. Real Infrastructure
- Actual AWS / Azure / GCP resources
- Managed via provider APIs

---

# 2️⃣ What is Terraform Drift?

## 🔍 Definition

**Terraform drift occurs when the real infrastructure differs from what is recorded in the Terraform state file.**

It usually happens when someone modifies infrastructure **outside Terraform**, such as:

- Changing instance type from AWS Console
- Modifying security group rules manually
- Updating resources via CLI
- Another automation tool changing infra

---

# 3️⃣ Understanding Drift with Comparison

## ✅ Normal Condition (No Drift)

```

.tf  ==  state  ==  real infrastructure

```

Everything is aligned.

---

## ❌ Drift Condition

```

.tf  ==  state
BUT
state  !=  real infrastructure

````

Example:

- `.tf` says EC2 instance type = `t3.micro`
- State file says = `t3.micro`
- Someone manually changes it to `t3.medium`
- Now real infra = `t3.medium`

This is Terraform drift.

---

# 4️⃣ How Terraform Detects Drift

When you run:

```bash
terraform plan
````

Terraform:

1. Refreshes state by calling provider APIs
2. Compares real infrastructure with stored state
3. Detects differences
4. Shows corrective actions

You will see something like:

```
~ instance_type = "t3.medium" -> "t3.micro"
```

Terraform wants to restore infrastructure to match configuration.

---

# 5️⃣ terraform refresh and refresh-only

## 🔹 Old Command

```bash
terraform refresh
```

* Updates only the state file
* Does NOT change infrastructure
* Does NOT change .tf files
* Deprecated in newer versions

---

## 🔹 Modern Recommended Way

```bash
terraform apply -refresh-only
```

This:

* Reads real infrastructure
* Updates state file only
* Makes no infrastructure changes

---

# 6️⃣ How to Fix Terraform Drift

There are two approaches:

---

## ✅ Option 1 – Enforce Terraform Configuration (Recommended)

If manual change was incorrect:

```bash
terraform apply
```

Terraform will revert infrastructure back to desired state.

---

## ✅ Option 2 – Accept Manual Changes

If manual change is valid and should be permanent:

1. Update state:

   ```bash
   terraform apply -refresh-only
   ```

2. Update `.tf` configuration to match real infra

3. Run:

   ```bash
   terraform plan
   ```

4. Ensure no changes are shown

Now everything is aligned again.

---

# 7️⃣ Important Clarification

Drift is about:

```
State file  vs  Real Infrastructure
```

It is NOT primarily about:

```
.tf file  vs  state file
```

Even if you never change `.tf`, drift can still happen.

---


## 🔹 What is Terraform Drift?

Terraform drift happens when the actual infrastructure differs from what is stored in the Terraform state file, usually due to manual or external changes outside Terraform.

## 🔹 How is drift detected?

By running:

```bash
terraform plan
```

Terraform refreshes state from the provider and compares differences.

## 🔹 How to fix drift?

* Run `terraform apply` to enforce configuration
  OR
* Run `terraform apply -refresh-only` and update `.tf` to accept changes

---

# 9️⃣ One-Line Answer

> Terraform drift occurs when real infrastructure changes outside Terraform cause a mismatch between the state file and the actual environment. Terraform detects it during plan by refreshing state and comparing it with configuration.

---

# 🔟 Key Takeaway

Terraform depends on the integrity of its **state file**.
Any external modification breaks that integrity and causes drift.
Best practice: Always manage infrastructure only through Terraform.

```

---
