# Welcome to GeoGebra!


This repository contains source code of [GeoGebra](https://www.geogebra.org)'s math apps.
It is available on a private GitLab instance and mirrored to GitHub.

Please read https://www.geogebra.org/license about GeoGebra's
licensing.

## Running the web version
To start the web version from command line, run

```
./gradlew -p source/web :web:run
```

This will start a development server on your machine where you can test the app. 
If you need to access the server from other devices, you can specify a binding address

```
./gradlew -p source/web :web:run -Pgbind=A.B.C.D
```

where `A.B.C.D` is your IP address. 
Then you can access the dev server through `http://A.B.C.D:8888`.
You can also run `./gradlew :web:tasks` to list other options.

## Hosting GeoGebra Classic in Azure

Use [Build and deploy GeoGebra](.github/workflows/build-geogebra.yml) to compile
the web client in GitHub Actions and optionally publish it to an
[Azure Storage static website](https://learn.microsoft.com/azure/storage/blobs/storage-blob-static-website).
No Docker, App Service, Java server, or continuously running `:web:run` is needed
in Azure. Storage and bandwidth charges still apply.

### 1. Build without Azure first

After the workflow is on the repository's default branch, open **Actions →
Build and deploy GeoGebra → Run workflow**. Leave `deploy` unchecked.
Download the `geogebra-site` artifact when the build succeeds; it is retained
for seven days.

This checkout uses Gradle 9.4.1, which cannot run on Java 8. The workflow installs
Java 21 to run Gradle and Java 17 for the project's compilation toolchain.
It runs `:web:copyHtml` and `:web:gwtCompile`, selecting the production
`org.geogebra.web.Web3D` module (output name `web3d`) needed by Classic.
The source build output is `source/web/web/war`, not a Java application to
deploy. Only `web3d/`, `css/`, and `classic.html` are packaged, with Classic also
copied to `index.html`. Development/test pages and `WEB-INF` are not published.
The other calculator apps and the embedding `deployggb.js` bundle are not
included.

You can extract the artifact and serve its root with a local static HTTP server
to check Classic, graphing, 3D, CAS, and opening/saving a local `.ggb` file.
Do not test by opening `index.html` directly through `file://`.
The build needs access to the Gradle, Maven, Google Maven, GeoGebra, and Sass
dependency download services. If a restricted runner cannot reach them, fix
network access rather than changing dependency versions.

### 2. Create the Azure website

In Azure Portal, create a dedicated **Storage account** (Standard,
StorageV2/general-purpose v2 is sufficient). Under **Data management → Static
website**, enable static website hosting and set **Index document name** to
`index.html`. Leave the error document unset; missing JavaScript must return
404, not the application HTML. Record the **Primary endpoint**, for example
`https://<account>.<zone>.web.core.windows.net/`.

Use the static website endpoint, not the blob container URL. Azure creates the
`$web` container automatically. The website is publicly readable; do not upload
private files. The account's network rules must allow the GitHub-hosted runner
to upload and your users to reach the website. A private endpoint-only account
requires a different runner/network arrangement.

### 3. Configure passwordless deployment

1. In **Microsoft Entra ID → App registrations**, register an application for
   this deployment. Record its **Application (client) ID** and **Directory
   (tenant) ID**, plus the Azure **Subscription ID** containing the account.
   Do not create a client secret.
2. On the storage account, open **Access control (IAM)** and assign **Storage
   Blob Data Contributor** to that application's service principal, scoped
   to this dedicated account. Subscription-wide Contributor/Owner is not needed.
   Allow time for the role assignment to propagate.
3. In this GitHub repository, create **Settings → Environments →
   azure-production**. Restrict its deployment branches to your trusted
   production branch; enable required reviewers if available.
4. In the Entra application, add a **Certificates & secrets → Federated
   credentials → GitHub Actions deploying Azure resources** credential:
   - Organization: `TordRu` (or your fork's owner).
   - Repository: `geogebra`.
   - Entity type: **Environment**.
   - Environment: `azure-production`.
   - Issuer: `https://token.actions.githubusercontent.com`.
   - Subject: `repo:TordRu/geogebra:environment:azure-production`
     (adjust the owner/repository for forks).
   - Audience: `api://AzureADTokenExchange`.
5. In the GitHub environment, add these **environment variables**, not secrets:

   | Variable | Value |
   | --- | --- |
   | `AZURE_CLIENT_ID` | Application (client) ID |
   | `AZURE_TENANT_ID` | Directory (tenant) ID |
   | `AZURE_SUBSCRIPTION_ID` | Subscription ID |
   | `AZURE_STORAGE_ACCOUNT` | Storage account name, not its URL |

See [Azure Login with OIDC](https://learn.microsoft.com/azure/developer/github/connect-from-azure-openid-connect)
for federation details. Only the deployment job can request an OIDC token;
the build job has read-only repository permissions and needs no Azure access.

### 4. Deploy and verify

Run the workflow from the trusted production branch with `deploy` checked,
approve the environment if required, and open the primary endpoint after
deployment. Check the browser network panel: `web3d/web3d.nocache.js`, its
compiled chunks, CSS, translations, and CAS assets should load successfully.
Azure CLI determines content types from file extensions; in particular, check
that `.wasm` responses have `Content-Type: application/wasm`.
Also test graphing, 3D, CAS, and local file import/export on the hosted site.

Deployments overwrite existing files but retain older compiled assets to avoid
breaking already-open clients. They are not atomic releases: use a maintenance
window for this simple setup, and periodically clean unused old assets after
clients have refreshed. All uploaded files use `Cache-Control: no-cache`.
For a custom domain with HTTPS, caching, or more advanced release routing, add
Azure Front Door separately.

### Scope and limitations

This hosts the browser client, **not the GeoGebra backend**. Sign-in, cloud
materials, sharing, collaboration, fonts, and other online services can still
depend on external GeoGebra services and their origin/CORS policies. Some
upstream page features assume geogebra.org routes, such as
`/api/material_preview.php`; those routes do not exist on a static website.
The generated page also attempts to register `/sworker.js`, which this
checkout does not supply; offline/service-worker support is not configured.
Use `classic.html` or the site root, rather than geogebra.org-style pretty URLs.
This is not a fully offline or independently hosted GeoGebra service.

Review [GeoGebra's licensing](https://www.geogebra.org/license) before publishing,
especially for commercial use. Azure resources, IAM roles, federation, and
GitHub environment settings must be created in your accounts; the workflow
does not provision them.

## Running the desktop version (Classic 5)
To start the desktop version from command line, run

```
./gradlew -p source/desktop :desktop:run
```
You can also run `./gradlew -p source/desktop :desktop:tasks` to list other options.

## Setup the development environment

* Open IntelliJ. If you don't have IntelliJ on your computer yet,
you can download and install it from [here](https://www.jetbrains.com/idea/download)
* In the menu select File / New / Project from Version Control / Git
* In the new window add the following path: `https://git.geogebra.org/ggb/geogebra.git`
* Click on ‘Checkout’, select your preferred destination folder, select Java 1.8 as the SDK, 
click on OK and wait…
* After the project is checked out, select the root folder of the project, 
open the Run Anything tool (Double ^ on Mac) and run the following command: 
`./gradlew -p source/web :web:run`
* After a minute or two the GWT UI will appear
* After the Startup URLs are loaded on the UI, select the app that you wish start. For example, 
if you select `graphing.html` and click on Launch Default Browser 
then the Graphing Calculator app with the newest features 
will load and start in your default browser 
