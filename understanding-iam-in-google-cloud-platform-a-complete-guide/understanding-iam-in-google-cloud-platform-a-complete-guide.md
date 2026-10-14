---
title: "Understanding IAM in Google Cloud Platform: A Complete Guide"
author: "Sunke Nithin"
date: 2025-08-09
categories: ["GCP"]
tags: ["AI", "AWS", "AZURE", "CLOUD", "technology"]
source: https://nithinsunke.wordpress.com/2025/08/09/understanding-iam-in-google-cloud-platform-a-complete-guide/
---

# Understanding IAM in Google Cloud Platform: A Complete Guide

In the world of cloud computing, controlling **who can access what** is foundational for security and governance. **Google Cloud IAM (Identity and Access Management)** provides a flexible and powerful system for managing access to your cloud resources.

Whether you're running a small proof-of-concept or managing enterprise workloads, understanding IAM is **essential for securely operating in GCP**.

## What is IAM in GCP?

IAM in Google Cloud is a framework that defines:

- **Who (identity)** can
- **Do what (role/permission)**
- **On which resource**

IAM controls access across all GCP services such as Compute Engine, Cloud Storage, BigQuery, Cloud Functions, and more.

## Core Components of IAM

1.  **Identities (Who is accessing?)**

Identities are the entities that interact with GCP.

| Identity Type                 | Description              | Example                                                 |
|-------------------------------|--------------------------|---------------------------------------------------------|
| Google Account                | Individual user account  | `user:john@example.com`                                 |
| Google Group                  | A group of users         | `group:devs@example.com`                                |
| Service Account               | Used by apps or services | `serviceAccount:my-app@project.iam.gserviceaccount.com` |
| G Suite/Cloud Identity domain | Entire domain            | `domain:example.com`                                    |
| All users                     | Anyone on the internet   | `allUsers`                                              |
| Authenticated users           | Any signed-in user       | `allAuthenticatedUsers`                                 |

2\. **Resources (What is being accessed?)**

Resources are GCP services and components like:

- Projects
- Virtual Machines (Compute Engine)
- Cloud Storage Buckets
- BigQuery Datasets
- Cloud Pub/Sub Topics
- APIs

3\.  **Roles (What access is granted?)**

Roles are collections of **permissions**.

#### 🔹 Predefined Roles[](https://github.com/NithinSunke/MY_KB_ARTICALS/blob/main/GCP/Understanding%20IAM%20in%20Google%20Cloud%20Platform%20A%20Complete%20Guide.md#-predefined-roles)

GCP provides **fine-grained, service-specific roles**.

- `roles/viewer` – read-only access to all resources
- `roles/editor` – read-write access
- `roles/storage.objectViewer` – read-only access to Cloud Storage objects
- `roles/compute.admin` – full control over Compute Engine resources

####  Custom Roles[](https://github.com/NithinSunke/MY_KB_ARTICALS/blob/main/GCP/Understanding%20IAM%20in%20Google%20Cloud%20Platform%20A%20Complete%20Guide.md#-custom-roles)

You can create custom roles with only the permissions your team needs.

#### ⚠️ Basic Roles *(Not recommended for production)*[](https://github.com/NithinSunke/MY_KB_ARTICALS/blob/main/GCP/Understanding%20IAM%20in%20Google%20Cloud%20Platform%20A%20Complete%20Guide.md#%EF%B8%8F-basic-roles-not-recommended-for-production)

- `Owner` – full access, including IAM (dangerous!)
- `Editor` – full access without IAM
- `Viewer` – read-only access

**Use predefined or custom roles instead of basic roles to follow the principle of least privilege.**

4\. **Policy**

IAM policy is a collection of **bindings**: each binding associates **members** with **roles**.

**Example Policy JSON:**

``` wp-block-code
{
  "bindings": [
    {
      "role": "roles/storage.admin",
      "members": [
        "user:alice@example.com",
        "group:devops@example.com"
      ]
    }
  ]
}
```

## IAM Policy Hierarchy

IAM policies in GCP are **hierarchical** and can be applied at the following levels:

``` wp-block-code
Organization (optional)
   └── Folders (optional)
         └── Projects
               └── Resources
```

- **Policies are inherited down** the hierarchy.
- This allows broad policies at the org level and specific exceptions at lower levels.

### How to Manage IAM

### Using the Google Cloud Console:

1.  Go to **IAM & Admin → IAM**
2.  Click **“Grant Access”**
3.  Enter the **principal (user/service account)**
4.  Assign a **role**
5.  Click **Save**

### Using gcloud CLI:

``` wp-block-code
gcloud projects add-iam-policy-binding [PROJECT_ID] \
  --member="user:john@example.com" \
  --role="roles/compute.viewer"
```

## Using Terraform:

``` wp-block-code
resource "google_project_iam_binding" "viewer_binding" {
  project = "my-project-id"
  role    = "roles/viewer"

  members = [
    "user:john@example.com",
  ]
}
```

## Common Use Cases

| Task                           | Recommended Role                                      |
|--------------------------------|-------------------------------------------------------|
| Read-only access to a project  | `roles/viewer`                                        |
| Deploy code to Cloud Functions | `roles/cloudfunctions.developer`                      |
| Admin access to VMs            | `roles/compute.admin`                                 |
| Grant access to storage bucket | `roles/storage.objectViewer` or `roles/storage.admin` |
| Audit IAM changes              | `roles/iam.securityReviewer`                          |

## IAM in Action: Scenario

**Scenario**: You want to give a developer read-only access to BigQuery datasets in a project.

- Go to **IAM & Admin → IAM**
- Add member: `user:dev@example.com`
- Assign role: `roles/bigquery.dataViewer`

Now, the developer can query but not modify or delete datasets.

## IAM Best Practices

| Practice                                     | Why It Matters                                |
|----------------------------------------------|-----------------------------------------------|
| Use **least privilege**                      | Minimizes risk if credentials are compromised |
| Prefer **groups over individuals**           | Easier to manage at scale                     |
| Audit IAM policies regularly                 | Detect overly broad permissions               |
| Use **custom roles** for sensitive workflows | Fine-grained control                          |
| Avoid `Owner` role                           | Too much power, especially in production      |
| Use **service accounts** for automation      | Secure and trackable                          |

## Tools for IAM Governance

- **IAM Recommender**: Suggests removing unused permissions.
- **Policy Analyzer**: Finds risky IAM configurations.
- **Cloud Audit Logs**: Tracks all IAM activity.
- **Cloud Asset Inventory**: View IAM across org/project.

## IAM vs Organization Policies

| Feature          | IAM                          | Organization Policies                    |
|------------------|------------------------------|------------------------------------------|
| What it controls | Who can access what          | Enforce constraints on resource behavior |
| Scope            | Roles and permissions        | Service usage policies                   |
| Example          | Assign `roles/compute.admin` | Block external IPs on VMs                |

## Final Thoughts

IAM in Google Cloud is a **cornerstone of security and compliance**. Mastering it helps you:

- Prevent data breaches
- Enforce principle of least privilege
- Simplify user and service account access
- Audit and monitor access effectively

> Whether you're building in the free tier or scaling to enterprise workloads, understanding IAM is critical for a secure and manageable cloud foundation.
