# Terraform destroy -target – Destroying a Single Resource

---

# 1️⃣ Can We Destroy a Single Resource Using Terraform?

Yes.

Terraform allows destroying a specific resource using the `-target` flag without affecting the entire environment.

Example scenario:
You have multiple Azure resources:
- Resource Group
- VNet
- Subnet
- NSG
- 2 Virtual Machines

You want to destroy only **one VM** without touching other infrastructure.

---

# 2️⃣ Command Syntax

To destroy a specific resource:

```bash
terraform destroy -target=<resource_address>
````

For an Azure VM example:

```bash
terraform destroy -target=azurerm_linux_virtual_machine.vm1
```

Terraform will:

* Destroy only that VM
* Update the state file accordingly
* Leave other resources untouched

---

# 3️⃣ What Happens Internally?

Let’s break it down technically.

---

## 🔹 Before Destroy

```
.tf  ==  state  ==  real infrastructure
```

State file contains:

* VM1 ID
* VM2 ID
* VNet ID
* etc.

---

## 🔹 During `terraform destroy -target`

Terraform:

1. Reads `.tf` configuration
2. Reads state file
3. Identifies the targeted resource
4. Builds dependency graph
5. Destroys only that resource
6. Removes that resource entry from state file

---

## 🔹 After Destroy

### Real Infrastructure:

* VM1 → Deleted
* Other resources → Unchanged

### State File:

* Entry for VM1 → Removed
* Other entries → Remain

### Configuration (.tf):

* No change
* VM1 block still exists in `.tf`

Important:
Terraform does NOT modify `.tf` files.

---

# 4️⃣ What Happens If You Run `terraform plan` After Target Destroy?

Because the VM block still exists in `.tf`:

```
.tf  !=  state  !=  real infrastructure
```

Terraform will detect:

* Resource exists in configuration
* Resource does not exist in state
* Resource does not exist in real infra

Terraform will plan to recreate it:

```bash
terraform plan
```

Output will show:

```
+ create azurerm_linux_virtual_machine.vm1
```

---

# 5️⃣ How to Recreate the Deleted VM?

Simply run:

```bash
terraform apply
```

Terraform will:

* Create the VM again
* Add its new ID to the state file
* Align configuration, state, and real infrastructure

---

# 6️⃣ What Happens to State and Configuration?

## 🔹 During Destroy

| Component  | What Happens              |
| ---------- | ------------------------- |
| Real Infra | Targeted resource deleted |
| State File | Targeted resource removed |
| .tf File   | No change                 |

---

## 🔹 During Recreate

| Component  | What Happens           |
| ---------- | ---------------------- |
| Real Infra | Resource created again |
| State File | New resource ID added  |
| .tf File   | No change              |

---

# 7️⃣ Important Warning About -target

HashiCorp does NOT recommend routine use of `-target`.

Why?

* It bypasses full dependency graph planning
* Can cause partial infrastructure drift
* May lead to inconsistent environments if misused

Use it:

* For emergency fixes
* For controlled partial rebuilds
* For troubleshooting

Avoid using it in CI/CD pipelines regularly.

---

# 8️⃣ Advanced Scenario (Dependencies)

If VM depends on:

* NIC
* Disk
* Public IP

Terraform may also destroy dependent child resources automatically if required.

Terraform never violates dependency graph integrity.

---

# 9️⃣ Summary (Short Version)

## 🔹 Can we destroy a single resource using Terraform?

Yes. Using:

```bash
terraform destroy -target=<resource_address>
```

Example:

```bash
terraform destroy -target=azurerm_linux_virtual_machine.vm1
```

---

## 🔹 What happens internally?

* Terraform deletes only the targeted resource.
* Removes that resource from state file.
* Leaves configuration untouched.
* Other resources remain unaffected.

---

## 🔹 How to recreate it?

Since the resource block still exists in `.tf`, running:

```bash
terraform apply
```

will recreate it and update the state file.

---

# 🔟 One-Line Answer

> Yes, Terraform allows destroying a specific resource using `-target`. It deletes the resource from real infrastructure and removes it from the state file while leaving the configuration unchanged. Running `terraform apply` later recreates it.

---

# 🔥 Key Takeaway

`.tf` defines desired state.
State file tracks managed objects.
`-target` modifies infrastructure and state selectively — but not configuration.

```
