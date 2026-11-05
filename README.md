Watch me explain this lab: [LoomLink](https://www.loom.com/share/cc4a9188c99f4234b87d4653586f0ce7)

# azure-static-website-blob-storage

Hosted a public website on Azure with no server at all: a Storage Account with static website hosting serves `index.html` and a custom `404.html` straight from the `$web` blob container over HTTPS. The site loaded on the first try, and a fake path returned my custom 404 page instead of Azure's default error.

# Azure Cloud Series, Lab 01: Static Website on Azure Blob Storage

**Azure Storage Account · Blob Storage · Static Website Hosting · HTML · Azure Portal**

My earlier labs were all about servers: domain controllers, file servers, VMs to patch and lock down. This lab starts from the other direction. If all you need is to serve a few files to the internet, you don't need a server to build, patch, or secure. Azure Storage can host the site directly, and Azure handles the infrastructure underneath. This is the first lab of a six week Azure series that ends in a capstone project.

| | |
|---|---|
| **Platform** | Azure Portal and a text editor (I used VS Code) on a local Windows PC, against East US on my personal Azure subscription |
| **Time** | About 30 minutes |

---

## What this lab is for

The business problem: a team needs a simple public page (a status page, a landing page, internal documentation) up today. Standing up a VM means picking an OS, opening ports, installing a web server, and patching it forever, all to serve a few HTML files.

Three ideas make the serverless approach work:

- **PaaS, not IaaS.** With a VM, I own everything from the OS up. With a Storage Account, Azure owns the platform and I only manage the content.
- **The `$web` container is the website.** Turning on static website hosting creates a special container. Whatever is in it gets served at a public endpoint, separate from the normal blob endpoint.
- **Index and error documents.** Azure serves `index.html` when someone visits the root, and `404.html` when they request something that doesn't exist. Both names are case sensitive.

---

## Architecture

```
Visitor (Internet)
        │
        │  HTTPS
        ▼
Primary endpoint: https://stlab01richard.<zone>.web.core.windows.net/
        │
        ▼
Azure subscription (personal)
        │
        └── rg-lab01-richard (East US)
                │
                └── stlab01richard  (StorageV2, Standard, LRS)
                        │
                        └── Static website: Enabled
                                index document: index.html
                                error document: 404.html
                                │
                                └── $web container
                                        ├── index.html
                                        └── 404.html
```

| Component | Role |
|---|---|
| **`rg-lab01-richard`** | Resource group holding everything, so cleanup is one delete |
| **`stlab01richard`** | Storage account: Standard performance, locally redundant storage (cheapest option for a lab) |
| **Static website** | Feature that serves the `$web` container at a public web endpoint |
| **`$web` container** | Created automatically when static website hosting is enabled; holds the site files |
| **`index.html`** | Home page |
| **`404.html`** | Custom error page for any path that doesn't exist |

---

## What I built

### 1. Created the resource group
Created `rg-lab01-richard` in East US on my personal subscription. Everything in this lab lives in it.

<img width="1484" height="383" alt="StaticLabResourceGroup" src="https://github.com/user-attachments/assets/eddfff57-1d1a-493c-a523-b2aa8156c3ee" />

### 2. Created the storage account
Created `stlab01richard` with Standard performance and locally-redundant storage (LRS). Storage account names have to be globally unique across all of Azure, lowercase, and letters and numbers only. They become part of a public URL.

<img width="1438" height="266" alt="StaticLabStorageAccount" src="https://github.com/user-attachments/assets/b3b5b253-382f-4646-93eb-9fdde3edbb61" />

### 3. Enabled static website hosting
Under **Data management → Static website**, I switched hosting to Enabled, set the index document to `index.html` and the error document to `404.html`, and saved. Azure then showed the **Primary endpoint**, the site's public address, and created the `$web` container.

<img width="1933" height="731" alt="StaticLabEnableStaticWebsite" src="https://github.com/user-attachments/assets/569a4114-425f-40a6-9193-0d211895eb43" />

### 4. Wrote the site, including a custom 404 page
`index.html` is a simple styled home page with my name on it. Setting `404.html` as the error document only works if the file actually exists. Without it, a bad link shows Azure's generic error instead of your page. So I wrote one, styled to match the home page, with a link back home:

```html
<body>
  <h1>404 - Page Not Found</h1>
  <p>That page doesn't exist on this site.</p>
  <p><a href="/">Back to home</a></p>
</body>
```
<img width="634" height="256" alt="StaticLabIndexHTML" src="https://github.com/user-attachments/assets/f9658b08-27a0-476c-94da-014ae4d42f90" />
<img width="635" height="269" alt="StaticLab404HTML" src="https://github.com/user-attachments/assets/43145a53-f3b8-45d3-945f-c90669b69742" />

I wrote both files in VS Code, which saves with the exact name and extension you give it. Notepad can quietly save a file as `index.html.txt`, which is the most common cause of a "content does not exist" error with static websites.

### 5. Uploaded to `$web`
Containers → `$web` → Upload, both files at once.

<img width="1919" height="301" alt="StaticLab$webContents" src="https://github.com/user-attachments/assets/d0d291c6-d116-41b7-bb17-1f03249ba5b0" />

### 6. Validated
- **Home page:** the primary endpoint loaded "Hello from the Cloud!" with my name.
- **Custom 404:** a fake path (`/nope`) returned my 404 page, not Azure's default, and the "Back to home" link worked.
- **HTTP blocked:** the same endpoint over `http://` failed. "Secure transfer required" is on by default, so the site is only reachable over HTTPS.
- **Bonus Azure Default Error without Custom 404:** without my custom 404 file, Azure defaults to its own error page.

<img width="1777" height="185" alt="StaticLabSiteRunning" src="https://github.com/user-attachments/assets/f699c376-c5df-49b1-aed6-531ec64956d9" />
<img width="1776" height="186" alt="StaticLabSite404" src="https://github.com/user-attachments/assets/53bc5fc3-d7c6-4e7b-97ff-133b4e881d3d" />
<img width="1775" height="170" alt="StaticLabHTTPFail" src="https://github.com/user-attachments/assets/f5983f61-d352-4607-a8e4-20fb3a9857ed" />
<img width="1775" height="158" alt="StaticLabDefaultError" src="https://github.com/user-attachments/assets/bc74bf02-3d9d-49d8-a7b3-5673be003102" />

> No server, no OS, no web server software, no inbound ports to open. The whole "infrastructure" is one storage account and two files.

---

## Common issues

| Symptom | Cause | Fix |
|---|---|---|
| "The requested content does not exist" at the root | File named `Index.html` or saved as `index.html.txt` | Exact lowercase name; save as "All files" in Notepad |
| Same error, file name correct | Uploaded to a container other than `$web` | Upload to `$web`, the only container the site serves |
| Bad links show Azure's default error | Error document set, but `404.html` not uploaded | Upload a `404.html` to `$web` |
| "Storage account name is already taken" | Names are unique across all of Azure | Add digits, e.g. `stlab01richard99` |

---

## Key takeaways

1. **Not everything needs a server.** For static content, a storage account is cheaper, simpler, and has far less to secure than a VM.
2. **Configured isn't the same as working.** Azure accepts an error document name whether or not the file exists. Only testing a bad path proves it works.
3. **Names matter in cloud.** Storage account names are global and become a public URL. File names are case-sensitive. Most of this lab's troubleshooting section comes down to a name being slightly wrong.
4. **Resource groups make cleanup one step.** Putting everything in one group means one delete removes the whole lab and stops the billing.

If I did this again, I would deploy it with the Azure CLI or Terraform after the portal run, so the same site can be rebuilt in one command.

---

## Security notes

This is a lab configuration. Some of it is solid and some of it would not be acceptable in production:

**What the lab does well**
- **HTTPS only.** Secure transfer is required on the storage account, and I confirmed a plain `http://` request to the site fails.
- **Nothing to patch or log into.** No OS, no web server, no open management ports.
- **Only the `$web` container is public.** Serving the site doesn't make the rest of the storage account public.
- **Personal subscription, one resource group,** deleted after the lab.

**What I would change for production**
- **Custom domain behind Azure Front Door or a CDN,** with a managed TLS certificate, caching, and a WAF in front.
- **Deploy from Git with a pipeline** (GitHub Actions) instead of uploading in the portal, so every change is versioned and reviewed.
- **Lock down management access:** disable shared key access and use Entra ID with least-privilege roles (Storage Blob Data Contributor only for whoever deploys).
- **Set minimum TLS 1.2** and turn on diagnostic logging for the storage account.
- **Zone- or geo-redundant storage** if the site actually matters to someone.

---

## Verification

| Check | How I verified it |
|---|---|
| Resource group created | `rg-lab01-richard` in East US |
| Storage account created | `stlab01richard`, Standard, LRS |
| Static website enabled | Static website blade: Enabled, `index.html` / `404.html`, primary endpoint shown |
| Files uploaded | `$web` container lists `index.html` and `404.html` |
| Site live | Primary endpoint loads "Hello from the Cloud!" with my name |
| Custom error page works | `/nope` returns my 404 page; "Back to home" link works |
| HTTPS enforced | `http://` request to the endpoint fails |

<img width="1933" height="731" alt="StaticLabEnableStaticWebsite" src="https://github.com/user-attachments/assets/8ee01213-6845-4a86-bdd8-9247c666705a" />
<img width="1919" height="301" alt="StaticLab$webContents" src="https://github.com/user-attachments/assets/ffc727b1-231f-445a-a77e-35973f86f725" />
<img width="1777" height="185" alt="StaticLabSiteRunning" src="https://github.com/user-attachments/assets/89abd373-1c60-4071-82b2-805379820699" />
<img width="1776" height="186" alt="StaticLabSite404" src="https://github.com/user-attachments/assets/0d2ac1fd-6f2c-45ea-a95e-d73b19ed6d75" />
<img width="1775" height="170" alt="StaticLabHTTPFail" src="https://github.com/user-attachments/assets/928916c9-5f63-46ee-bd94-a31eac6d60b9" />


---

## Teardown

Deleted the resource group, which removes the storage account, the `$web` container, and the site in one step.

```
Resource groups → rg-lab01-richard → Delete resource group → type the name → Delete
```

Or with the Azure CLI:

```powershell
az group delete --name rg-lab01-richard --yes
az group show --name rg-lab01-richard   # expect ResourceGroupNotFound
```

---

## Repository structure

```
azure-static-website-blob-storage/
├── site/
│   ├── index.html
│   └── 404.html
└── README.md
```

---

## Next steps

- Redeploy the same site with **Terraform** (`azurerm_storage_account` + `azurerm_storage_account_static_website` + `azurerm_storage_blob`) so it's one command to rebuild
- Add a **GitHub Actions** workflow that uploads `site/` to `$web` on every push
- Put **Azure Front Door** in front with a custom domain
- Carry the pattern into the series capstone

---

**Status: complete.** Created a resource group and storage account → enabled static website hosting → uploaded `index.html` and a custom `404.html` to `$web` → home page and custom 404 both verified live → resource group deleted.
