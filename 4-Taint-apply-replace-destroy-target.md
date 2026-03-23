# Terraform Resource Replacement – taint vs replace vs destroy -target

This document explains how to recreate a specific resource in Terraform and compares:

- `terraform taint`
- `terraform apply -replace=<resource>`
- `terraform destroy -target=<resource>` + `terraform apply`

Simple explanation. Technically accurate. Interview-ready.

---

# 1️⃣ Why Would You Need Resource Replacement?

Common real-world scenarios:

- VM boot failed
- Cloud-init script failed
- OS disk corrupted
- Provisioner partially executed
- Resource unhealthy but configuration is correct
- You want a clean rebuild without modifying `.tf`

In these cases, you want Terraform to destroy and recreate only one resource.

---

# 2️⃣ Method 1 – terraform taint (Legacy Approach)

## 🔍 What It Does

Marks a resource in the **state file** as tainted.

It does NOT destroy immediately.  
It only flags it for recreation during next `terraform apply`.

---

## Example

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcd1234"
  instance_type = "t3.micro"
}
````

### Step 1 – Mark as Tainted

```bash
terraform taint aws_instance.web
```

Output:

```
Resource instance aws_instance.web has been marked as tainted.
```

No infrastructure change yet.

---

### Step 2 – Apply

```bash
terraform apply
```

Plan Output:

```
-/+ resource "aws_instance" "web" {
      id = "i-0123456789"
    }

Plan: 1 to add, 0 to change, 1 to destroy.
```

Terraform:

* Destroys old instance
* Creates new instance
* Updates state with new ID

---

## What Happens Internally?

| Component  | Effect                            |
| ---------- | --------------------------------- |
| Real Infra | Destroy + Recreate (during apply) |
| State File | Marked tainted → Updated          |
| .tf File   | No change                         |

---

## Status

`terraform taint` is discouraged in modern Terraform.

Preferred alternative:

```
terraform apply -replace=<resource>
```

---

# 3️⃣ Method 2 – terraform apply -replace=<resource> (Recommended)

## 🔍 What It Does

Forces Terraform to replace a resource in a single command.

Example:

```bash
terraform apply -replace=aws_instance.web
```

---

## What Happens Internally?

Terraform:

1. Plans full dependency graph
2. Marks resource for replacement in memory
3. Shows destroy + create
4. Executes in one step

---

## Behavior

| Component  | Effect                |
| ---------- | --------------------- |
| Real Infra | Destroy + Recreate    |
| State File | Updated automatically |
| .tf File   | No change             |

---

## Why It Is Better

* No intermediate tainted state
* Cleaner workflow
* Safer in automation
* Fully dependency-aware

---

# 4️⃣ Method 3 – terraform destroy -target + terraform apply

## 🔍 What It Does

Immediately destroys a specific resource.

Example:

```bash
terraform destroy -target=aws_instance.web
```

Terraform deletes resource instantly.

---

## After Destroy

State file:

* Removes resource entry

Infrastructure:

* Resource gone

`.tf`:

* Still contains resource block

Now system looks like:

```
.tf != state != real infrastructure
```

---

## Recreate It

Run:

```bash
terraform apply
```

Terraform sees:

* Resource defined in .tf
* Missing in state
* Missing in real infra

It recreates it.

---

## Behavior

| Component  | During Destroy | During Apply |
| ---------- | -------------- | ------------ |
| Real Infra | Deleted        | Recreated    |
| State File | Removed        | Added back   |
| .tf File   | No change      | No change    |

---

# 5️⃣ Critical Comparison

## 🔥 When Does Destroy Happen?

| Method          | Destroy Timing    |
| --------------- | ----------------- |
| taint           | During next apply |
| apply -replace  | Same apply        |
| destroy -target | Immediately       |

---

## 🔥 State Impact

| Method          | State Modification          |
| --------------- | --------------------------- |
| taint           | Marks resource as tainted   |
| apply -replace  | No persistent taint flag    |
| destroy -target | Removes resource from state |

---

## 🔥 Dependency Safety

| Method          | Dependency Handling            |
| --------------- | ------------------------------ |
| taint           | Full graph respected           |
| apply -replace  | Full graph respected           |
| destroy -target | Partial graph execution (risk) |

HashiCorp warns against frequent use of `-target` because it can lead to partial infrastructure execution.

---

# 6️⃣ Which One Should You Use?

## ✅ Recommended (Modern)

```
terraform apply -replace=<resource>
```

Use when:

* Resource unhealthy
* Need clean rebuild
* Production-safe approach

---

## ⚠ Legacy

```
terraform taint
```

Use only in older Terraform versions.

---

## ⚠ Emergency / Advanced

```
terraform destroy -target
```

Use only if:

* Immediate deletion required
* You understand dependencies
* You know impact on environment

---

# 7️⃣ Summary (Short Version)

## 🔹 Difference Between Them?

* `terraform taint` marks a resource in state for recreation.
* `terraform apply -replace` forces replacement in one step (preferred).
* `terraform destroy -target` deletes a specific resource immediately and removes it from state.

## 🔹 Which is Best?

Use `terraform apply -replace` because it is safer and respects full dependency graph.

---

# 💪 One-Line Answer

> Terraform taint marks a resource for recreation, apply -replace forces replacement in a single step, and destroy -target immediately deletes a specific resource. The recommended modern and safest approach is apply -replace because it preserves dependency integrity.

---

# 🔥 Final Takeaway

All three methods can recreate a resource.

But:

* `apply -replace` = clean and modern
* `taint` = legacy state flagging
* `destroy -target` = partial execution, use carefully

```
