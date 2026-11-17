
### Competition Submission Guide: GitHub Releases and GHCR Images

Once a participating team completes **one-time registration**, a submission notification is automatically sent to the organizing team **each time a GitHub release is published**. This document explains what participants need to prepare and what the system verifies.

### 1. Overall Process

1. **One-time registration**: Enter your team information through the registration link in the competition announcement, configure your repository as instructed, and pass the automatic connection test.
2. **Automatic notification for each release**: When a release is published, the system locates and verifies the image and notifies the organizing team.

### 2. Participant Requirements

| Item | Description |
|---|---|
| GitHub repository | The repository must contain at least one commit. An empty repository (zero commits) cannot be registered. |
| Collaborator | Invite `AISystemChallenge` with **Write access or higher**. For organization repositories, use **Collaborators and teams**. The invitation will be accepted automatically by the system. |
| Webhook | Select **Releases only** as the event. Copy the Payload URL and Secret from the registration page. |
| GHCR image | The image must be hosted at `ghcr.io/<owner>/…` under the **same account** that owns the registered repository and must be **linked** to that repository. |

<h5 style="margin-top: 48px;">Registration Procedure</h5>
<!-- #### Registration Procedure -->

1. Enter your team information, GitHub repository, and GHCR image address in the registration form. The image address must include a tag or digest (e.g., `ghcr.io/<owner>/<image>:v1.0.0`) and belong to the same account as the registered repository.
2. Follow the instructions on the registration page to complete the following three tasks.
   - **Invite a collaborator**: Invite `AISystemChallenge` to your repository with Write access or higher.
   - **Add a webhook**: Go to **Settings → Webhooks → Add webhook** in your repository and configure the following:
     - **Payload URL** and **Secret**: Copy and paste the values exactly as shown on the registration page.
     - **Content type**: `application/json`
     - **SSL verification**: Enabled (Enable SSL verification)
     - **Events**: Select `Let me select individual events` and check **Releases only**. Uncheck **Pushes**, which is selected by default.
     - **Active**: Checked
   - **Link the image**: Ensure that the image is linked to the registered repository (see "How to Link an Image" below).
3. Click **"Setup Complete · Start Automatic Connection Test"**. Do not refresh the page or close the browser window while the test is running.
4. The system accepts the invitation, verifies Write access, and checks whether the image is linked to the repository and has a tag. It then creates a temporary draft release to verify webhook delivery and deletes the draft afterward. The temporary draft release is not published, does not create a tag, and does not trigger GitHub Actions. Registration is completed when all checks pass.

The Payload URL and Secret are **fixed for each repository**. Registering the same repository again will produce the same values. If the competition webhook already exists, leave it unchanged. **Only one webhook is needed.** Creating multiple webhooks will result in duplicate notifications.

#### One-Click Copy Page

Click the **"Open One-Click Copy Page"** button on the registration page to open `https://aisystemchallenge.github.io/submission-guide/copy/` in a new tab. This page allows you to copy the collaborator username, Payload URL, and Secret with a single click.

- The values are passed in the URL fragment after `#`. The fragment is not sent to the server.
- This page displays only values that match the competition's official format. If a warning appears, do not enter those values into GitHub. Instead, reopen the page using the button on the registration page.
- Always open this page using the button on the registration page. Opening the URL directly will display empty values.

#### How to Link an Image

There are three ways to link an image to the registered repository. **Only one method is required.**

1. **(Recommended) Push using GitHub Actions when publishing a release**: Pushing an image to `ghcr.io` using `GITHUB_TOKEN` in a workflow automatically links the image to that repository. See [release.yml](examples/release.yml) for an example.
2. **Add a label to the Dockerfile**: Add `LABEL org.opencontainers.image.source=https://github.com/<owner>/<repo>` and push the image.
3. **Link directly from the package page**: Select the repository under **Package settings → Connect repository**.

### 3. What Happens When a Release Is Published

1. A participant **publishes** a release. Pre-releases are also included. Saving, editing, or deleting a draft does not trigger a notification.
2. The system receives the webhook and verifies its signature and repository.
3. The system queries GitHub for an image version whose **tag matches the release tag** among the GHCR packages linked to the repository. The image address does not need to be included in the release description.
4. If a `ghcr.io/…` address is included in the release description, that address is also verified using the same rules.
5. The organizing team receives a notification containing the pull address and digest.

If the image is not yet available, the system checks again approximately five minutes later. Therefore, it is acceptable if the image push has not finished immediately after the release is published.

#### Image Requirements

All three conditions must be satisfied: **Same account + Linked repository + Existing tag**.

- "Same account" refers to the **account that owns the repository**, not the individual who publishes the release. Therefore, any team member can publish a release.
- "Linked repository" means that the image is associated with the repository as a package (see "How to Link an Image" above).

### 4. What Is a Digest?

A digest is the `sha256` fingerprint (content address) of an image manifest. A tag (such as `v1.0.0`) can later be reassigned to a different build, but a digest uniquely identifies a specific image manifest. Therefore, notifications include the digest so that the exact image can be pulled as shown below.

```
docker pull ghcr.io/<owner>/<image>@sha256:…
```

For multi-architecture images, the index digest is used.

### 5. Rejection Conditions

Images are rejected under the following conditions. Rejected image addresses appear **only as warnings** in notifications to the organizing team and are excluded from the list of images to pull.

- Images belonging to a different account.
- Packages not linked to the registered repository. Granting an individual direct read access does not count as linking the package to the repository.
- Missing tags, including cases where the tag is still unavailable after the five-minute retry.
- Failure to retrieve image information.

### 6. FAQ

**Is release.yml alone sufficient?**  
No. One-time registration (collaborator invitation and webhook setup) must be completed separately. The `release.yml` workflow handles image building, pushing, and automatic repository linking when a release is published.

**Does the image have to be public?**  
No. What matters is whether the image is linked to the repository, not whether it is public.

**Do I have to include the pull address in the release description?**  
No. The system searches packages linked to the repository for a tag matching the release tag.

**Are there any rules for release tags?**  
Release tags must follow Docker tag naming rules. Only letters, numbers, `_`, `.`, and `-` are allowed, with a maximum length of 128 characters. Example: `v1.0.0`
