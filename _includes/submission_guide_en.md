### Competition Submission Guide: GitHub Releases and GHCR Images

Once a participating team completes **one-time registration**, an official release is automatically created and a submission notification is sent to the organizing team **each time the team pushes a self-built image using `docker push` under the registered image name**. During registration, participants enter **only the final component** of the image name. If a package with that name does not exist, a workflow installed in the repository during registration creates a package using a `hello-world` placeholder image and links it to the repository. Registration is completed after the system verifies the connection. This document explains what participants need to prepare and what the system verifies and installs.

### 1. Overall Process

1. **One-time registration**: Through the registration link in the competition announcement, agree to the collection and use of personal information and **verify the team leader's email address** by entering the six-digit verification code sent by email. Then enter your team information, GitHub repository, and **the final component of the GHCR image name** (e.g., `my-server`). Configure your repository as instructed and pass the automatic connection test. During this test, the system commits a submission workflow file to the repository. If a package with the same name does not exist, the commit triggers the workflow's **link job** (if the file already has identical content, no commit is made, so the system requests an execution equivalent to **Run workflow**). The link job pushes a `hello-world` placeholder image (tagged `aisc-registration`) to create the image package and link it to the repository. The system verifies that the package is linked before proceeding. The placeholder tag is not a submission, and this process normally completes within one minute.
2. **`docker push` for each subsequent submission**: **Build the image yourself**, assign a **new tag** under the registered image name, and run `docker push`. Push only after completing registration. The process then proceeds as follows: push → automatic release creation → submission email (notification to the organizing team). This normally completes within one minute. There is no distinction between test and official submissions: every new tag other than `latest` and `aisc-registration` is treated as a submission (see "4. How to Submit" below).

GitHub Actions does not build submission images, and a Dockerfile is not required in the repository root. No `LABEL` is required in the image. Participants only need to run `docker push` for their locally built images. The repository workflow is responsible solely for linking the package during registration (link job) and creating releases for pushed tags (release job).

### 2. Participant Requirements

| Item | Description |
|---|---|
| GitHub repository | The repository must contain at least one commit. An empty repository (zero commits) cannot be registered. GitHub Actions must be enabled for the repository (see "5. Enabling Actions and Usage Limits"). The image does not need to be built in this repository. |
| Collaborator | Invite `AISystemChallenge` with **Write access or higher**. For organization repositories, use **Collaborators and teams**. The invitation will be accepted automatically by the system. |
| Webhook | Select **Releases only** as the event. Copy the Payload URL and Secret from the registration page. |
| GHCR image name | Enter **only the final component** (e.g., `my-server`). The system constructs the full name as `ghcr.io/<repository owner>/<entered name>`, where the account is the owner of the registered repository. Do not include a tag. Use lowercase letters and numbers; separator characters (`.`, `_`, `__`, `-`) may appear only between characters. The name must be no more than 100 characters (uppercase letters are converted to lowercase). Hierarchical names such as `a/b/c` or names belonging to another account are not allowed. Using the repository name is recommended. The image does not need to exist at registration time. |
| GitHub PAT (classic) | The `write:packages` permission is required to push images. Use the token only for `docker login ghcr.io` and do not commit it to the repository (see "4. How to Submit"). |
| Docker | An environment capable of building images and running `docker push`. |

<h5 style="margin-top: 48px;">Registration Procedure</h5>

1. In the registration form, agree to the collection and use of personal information and **verify the team leader's email address**. Then enter your team information, GitHub repository URL, and GHCR image name. Enter **only the final component** of the image name (e.g., `my-server`). The system constructs the full name as `ghcr.io/<owner>/my-server`.
   - **Team leader email verification**: Click **"Agree and Verify Email (동의하고 이메일 인증)"** to enter the team leader's email address. A six-digit verification code will be sent to that address. Enter the code on the next screen. Once verified, you will proceed to the team information form, and the verified address will be used as the team leader's email address. There is no separate email field on the team information form.
   - The verification code must be entered within **10 minutes**. If you do not see the email, check your spam folder. If you did not receive the code or entered the wrong email address, select **"Other Actions (다른 동작)"** on the same screen and choose **"Resend Code (코드 다시 받기)"** or **"Change Email Address (이메일 주소 고치기)"**.
   - You can enter an incorrect code up to **5 times**, and a maximum of **4 verification emails** can be sent per registration attempt (including resending codes and changing the email address). If either limit is reached, the verification process ends, and you must start registration again.
2. Follow the instructions on the registration page to complete the following tasks.
   - **Invite a collaborator**: Invite `AISystemChallenge` to your repository with Write access or higher.
   - **Add a webhook**: Go to **Settings → Webhooks → Add webhook** in your repository and configure the following:
     - **Payload URL** and **Secret**: Copy and paste the values exactly as shown on the registration page.
     - **Content type**: `application/json`
     - **SSL verification**: Enabled (Enable SSL verification)
     - **Events**: Select `Let me select individual events` and check **Releases only**. Uncheck **Pushes**, which is selected by default.
     - **Active**: Checked
   - **Check the following**: Verify that GitHub Actions is enabled for the repository (see "5. Enabling Actions and Usage Limits"). If a package with the same name already exists, verify that it is linked to the repository (see "6. Linking an Image to the Repository").
3. Click **"Setup Complete · Start Automatic Connection Test"**. Do not refresh the page or close the browser window while the test is running.
4. The system automatically performs the following steps in order.
   1. **Accept invitation**: `AISystemChallenge` accepts the repository invitation.
   2. **Verify Write access**: The system confirms that it has Write access or higher.
   3. **Verify image name**: The system checks that the name follows the naming rules. If no package with the same name exists, this check passes (the workflow's link job creates and links the package in a later step; no image `LABEL` is required). If a package with the same name is already linked to this repository, that package is reused. If a package with the same name exists but is not linked to this repository or is linked to another repository, the check fails.
   4. **Commit submission workflow**: `AISystemChallenge` commits `.github/workflows/aisc-submission.yml` to the default branch. If the file already has identical content, the commit is skipped. If its content differs, it is overwritten. When a commit is made, that commit (push) triggers the workflow's link job (see "3. The Submission Workflow Is Automatically Installed During Registration").
   5. **Verify package linkage**: If a package is already linked to this repository, this step is skipped. Otherwise, the system waits for the link job to push a `hello-world` placeholder image, create the package, and link it to the repository. If no commit was made in Step 4 because the file already had identical content, the system first requests an execution equivalent to **Run workflow**. If the connection cannot be verified within a few minutes, registration fails and the registration page explains the cause and required actions (see "6. Linking an Image to the Repository").
   6. **Create a temporary draft release**: The draft is not published, does not create a tag, and does not trigger GitHub Actions.
   7. **Verify webhook delivery**: The system checks whether the webhook for the temporary release is received.
   8. **Delete the temporary draft release**.
5. Registration is completed when all checks pass. The completion page explains how the package was linked (created during registration or already linked) and what to do next (build the image and run `docker push`).

The Payload URL and Secret are **fixed for each repository**. Registering the same repository again will produce the same values. If the competition webhook already exists, leave it unchanged. **Only one webhook is needed.** Creating multiple webhooks will result in duplicate notifications.

##### One-Click Copy Page

Click the **"Open One-Click Copy Page"** button on the registration page to open `https://aisystemchallenge.github.io/submission-guide/copy/` in a new tab. This page allows you to copy the collaborator username, Payload URL, and Secret with a single click.

- The values are passed in the URL fragment after `#`. The fragment is not sent to the server.
- This page displays only values that match the competition's official format. If a warning appears, do not enter those values into GitHub. Instead, reopen the page using the button on the registration page.
- Always open this page using the button on the registration page. Opening the URL directly will display empty values.

<a id="workflow-install"></a>

### 3. The Submission Workflow Is Automatically Installed During Registration

During the automatic connection test, `AISystemChallenge` commits `.github/workflows/aisc-submission.yml` to the default branch of the registered repository. The workflow content is identical for all teams except for the `AISC_IMAGE` value (the registered image name). You can preview the content in [aisc-submission.yml](examples/aisc-submission.yml). The example uses `ghcr.io/<owner>/<name>` as a placeholder for `AISC_IMAGE`, which the system replaces with the registered image name during registration.

The commit message is `chore: add AI System Challenge submission workflow (image ghcr.io/…)`.

Repositories registered under the previous instructions (where publishing a release triggered a workflow to build the image, or where adding a `LABEL` was described as sufficient to link a package) may still contain the previous workflow file. **Registering again** causes the automatic connection test to update the file with the new content described in this document (the commit message begins with `update` instead of `add`). That commit triggers the link job.

##### What the Workflow Does

This workflow contains two jobs. Which job runs depends on the event that triggered the workflow. It does not build submission images. The only image it builds is an empty placeholder image pushed by the link job.

| Job | When It Runs | What It Does |
|---|---|---|
| **link** | When this workflow file is committed (pushed), including the system's registration commit or subsequent edits to the file, and when **Run workflow** is selected in the Actions tab. When registering a repository that already contains the file, the system requests an equivalent execution (`workflow_dispatch`). It is not triggered by package events. | Pushes the placeholder image tagged `aisc-registration` to create the image package and link it to this repository. This is not a submission. |
| **release** | When a package event (`registry_package`) occurs. | Creates an official release named after the pushed tag. This constitutes a submission. |

<h5 style="margin-top: 48px;">link Job: Linking the Package to the Repository During Registration</h5>

The package is linked to this repository by this job, not by an image `LABEL`. The job performs the following steps.

1. Logs in to `ghcr.io` using the repository's `GITHUB_TOKEN`. The token is passed through input rather than as a command-line argument.
2. Builds a placeholder image (based on `hello-world`, with an image source label pointing to this repository) and pushes it under the registered name with the `aisc-registration` tag. This push creates the image package and links it to the repository. `aisc-registration` is a placeholder tag, not a submission.
3. If the push fails, the job stops and writes an `::error::` message to the Actions log. There are two possible causes.
   - A package with the same name already exists but is not linked to this repository, or this repository's Actions does not have permission to write to it (e.g., a package created by a direct push before registration). Link this repository under **Package settings → Connect repository** on the package page. If the package is already linked to this repository, this failure can be ignored, and pushing a new tag will still create a submission.
   - An organization policy prevents Actions from creating or writing packages using `GITHUB_TOKEN`. After correcting the policy, rerun the job using **Run workflow**.
4. On success, the job displays a message explaining that the package is linked if it appears in the **Packages** list on the right side of the repository's main page. You can now submit by pushing a new tag.

During registration, the system waits for and verifies the result of this job (whether the package is linked to the repository). Therefore, if registration passes, the package is linked. If you later delete the package, use **Run workflow** to execute this job again and recreate it (see "6. Linking an Image to the Repository").

##### release Job: Registering a Pushed Tag as a Submission

This job is triggered by a package event (`registry_package`) and performs the following checks in the order defined in the workflow file.

1. Events without a tag are skipped. A single push may generate multiple events (such as platform-specific manifests from the same push). Events without tags do not start the job; only tagged events proceed to release creation.
2. Packages that do not match the registered name are skipped. The job constructs `ghcr.io/<account>/<package>` using the repository owner's account and package name in lowercase, then compares it with `AISC_IMAGE`.
3. If the tag does not follow Docker tag naming rules, the job stops with an error and does not create a release. Tags may contain only letters, numbers, `_`, `.`, and `-`, must be no longer than 128 characters, and must begin with a letter, number, or `_` (e.g., `v1.0.0`).
4. The `latest` tag is not submitted. If `docker push` is used without an explicit tag, `latest` is pushed. This rule prevents accidental submissions. The job only logs an informational message and skips the event.
5. The `aisc-registration` tag is not submitted. It is the placeholder tag pushed by the link job to connect the package. The job only logs an informational message and skips the event.
6. If a release with the same tag already exists, the event is skipped. Pushing the same tag again does not create another submission.
7. In all other cases, the job creates an **official release** (neither a pre-release nor a draft) on the repository's default branch using the pushed tag name. The release title is the tag, and its description is `Submission image <AISC_IMAGE>:<tag> (<digest>) — automatically created by GHCR push`.

The workflow has only two permissions: `contents: write` (to create releases) and `packages: write` (for the link job to push the placeholder image). Both use the repository's `GITHUB_TOKEN`. Releases created this way do not trigger other workflows, but they do send webhook notifications, allowing the competition system to receive the release. The link job's push also does not trigger other workflows.

##### Workflow File Values

| Value | Description |
|---|---|
| `AISC_IMAGE` | The registered image name. Do not change it, as the workflow will no longer be able to find the submission image. |

This is the only value that differs between teams. Registering again restores the workflow file to the common version (overwriting any modifications). Editing and pushing the file triggers the link job again, which pushes the placeholder image again (not a submission).

<a id="branch-protection"></a>

##### When Branch Protection Rules Prevent Commits

If branch protection rules or rulesets prevent commits to the default branch, take one of the following actions and retry the test.

1. Allow `AISystemChallenge` to bypass the protection rules.
2. Manually add the public template [aisc-submission.yml](examples/aisc-submission.yml) to `.github/workflows/aisc-submission.yml` in your repository. Change **only** the `AISC_IMAGE` line to the registered image name. If the content matches, the system skips the commit. If other lines are modified, the system will attempt to overwrite them, which may be blocked again. The commit (push) of the manually added file also triggers the link job, which pushes the placeholder image (not a submission).

##### If Registration Is Not Completed

The workflow file is committed during the automatic connection test. Therefore, even if a later step fails or registration is not completed, the file and any placeholder package created by the link job remain in the repository. You may delete them if you do not intend to register. The registration commit may also trigger push-based CI workflows in your repository.

<a id="how-to-submit"></a>

### 4. How to Submit

After completing registration, submitting an entry requires only running `docker push` for an image you have built yourself. If registration has passed, the package is already linked to this repository, so you can push immediately (if you deleted the package, see "6. Linking an Image to the Repository").

##### Logging In to ghcr.io

Create a PAT (classic) with the `write:packages` permission under GitHub **Settings → Developer settings → Personal access tokens → Tokens (classic)**, then log in as follows.

```
echo <PAT> | docker login ghcr.io -u <GitHub username> --password-stdin
```

Treat the PAT like a password. Do not include it in a repository, image, or public document, and do not commit it.

##### Building and Pushing an Image

1. Build the image yourself. No `LABEL` is required. You may include one, but a `LABEL` alone does not link the package to the repository. The package is linked by the workflow's link job during registration (see "3. The Submission Workflow Is Automatically Installed During Registration").
2. **After completing registration**, build and push the image under the registered name with a **new tag**. All components of the image name must be lowercase.

   ```
   docker build -t ghcr.io/<owner>/<name>:v1.0.0 .
   docker push ghcr.io/<owner>/<name>:v1.0.0
   ```

   A package created by a direct push (before registration or after deleting the package) is not linked to the repository. As a result, the `registry_package` event is not delivered to this repository, and no release or submission is created (see "6. Linking an Image to the Repository").
3. Shortly afterward, a release with the same name as the tag appears under **Releases** in the repository, and a submission notification is sent to the organizing team. This normally completes within one minute of the push.

##### Submission Rules

- **Pushing a new tag constitutes a submission.** There is no distinction between test and official submissions. Every new tag other than `latest` and `aisc-registration` is treated as a submission. The workflow always creates an official release, never a pre-release or draft.
- **The `latest` tag is not submitted.** If `docker push` is used without an explicit tag, `latest` is pushed. This rule prevents accidental submissions. Always use a version tag (e.g., `v1.0.0`).
- **The `aisc-registration` tag is not submitted.** It is a placeholder tag pushed by the link job during registration to link the package. Use a different version tag for submissions.
- **Pushing the same tag again does not create another submission.** If a release with that tag already exists, the workflow skips it. To submit again, push a new tag (e.g., `v1.0.1`).
- **Tag naming rules**: Tags may contain only letters, numbers, `_`, `.`, and `-`, must be no longer than 128 characters, and must begin with a letter, number, or `_` (e.g., `v1.0.0`). Invalid tags cause the workflow to stop with an error without creating a release.

<a id="actions-usage"></a>

### 5. Enabling Actions and Usage Limits

- GitHub Actions must be enabled for the repository (**Settings → Actions → General**). If disabled, neither the link job nor the release job will run, so the package will not be linked and no release or submission will be created.
- Organization policies may restrict Actions. Some policies may also prevent Actions from creating or writing packages using `GITHUB_TOKEN`. In that case, the link job fails. After correcting the policy, rerun it using **Run workflow**.
- The system does not directly detect either of these conditions. However, if the link job does not run or fails and the package is not linked within a few minutes, registration fails. The registration page will instruct you to check the Actions settings first. After enabling Actions or correcting the policy, retry the test to run the link job again.
- Private repositories consume the account's free GitHub Actions minutes. This workflow performs only short operations to push a placeholder image and create releases. Usage limits depend on the GitHub plan and may change, so specific limits are not listed here. Refer to GitHub's documentation.
- If the package is not linked during registration or no release is created after a push, first check the runs and logs for `AI System Challenge submission` under the repository's Actions tab. The logs contain `::error::` messages from the link job and reasons for skipped submissions (`latest`, `aisc-registration`, existing tag, or unregistered image name).

<a id="link-package"></a>

### 6. Linking an Image to the Repository

For a push to trigger this repository's workflow, the package must be **linked** to this repository. If you push to an unlinked package, the `registry_package` event will not be delivered to this repository, so no workflow, release, or submission will be created.

##### Automatic Linking During Registration

If no package with the same name exists, the link job in the workflow committed during registration pushes a `hello-world` placeholder image (`aisc-registration`) using the repository's `GITHUB_TOKEN`. The resulting package is linked to this repository. This normally completes within one minute (see "3. The Submission Workflow Is Automatically Installed During Registration"). The system waits until the connection is verified before proceeding. Therefore, if registration passes, the package has been created and linked to this repository. If the file already has identical content and no commit is made, the system requests an execution equivalent to **Run workflow**. No `LABEL` is required. Including one is acceptable, but a `LABEL` alone does not establish the connection. A package created by a direct push remains unlinked even if the image contains a `LABEL`.

##### How to Verify the Connection

- If the package appears in the **Packages** list on the right side of the repository's main page, it is linked.
- If the repository name appears on the package page, it is linked.

##### If the Package Is Not Linked

1. If the message "The image package is not linked to the repository" appears during registration, the package has not been linked after several minutes. Open the latest run of `AI System Challenge submission` under the repository's **Actions** tab and check the link job logs. Possible causes include disabled Actions (see "5. Enabling Actions and Usage Limits"), organization policies blocking package writes through `GITHUB_TOKEN`, or an existing unlinked package with the same name. After resolving the issue, click **"Retry Test"** on the registration page to run the link job again.
2. If you deleted the package or its connection was removed after registration, open `AI System Challenge submission` under the repository's **Actions** tab and click **Run workflow** to rerun the link job. Actions must be enabled.
3. If the execution fails, follow the `::error::` message in the logs. The cause is either an existing package with the same name (see "Linking Manually" below) or an organization policy. After correcting the policy, rerun the job using **Run workflow**.
4. After linking the package, push using a **new tag**. Tags pushed before the package was linked did not generate events for this repository.

##### Linking Manually

1. On the package page, select this repository under **Package settings → Connect repository**.
2. After linking, push again using a **new tag**. Tags pushed before linking did not generate events for this repository.

##### If a Package with the Same Name Already Exists

If a package with the same name already exists (e.g., an image pushed directly before registration), link it to this repository using the "Linking Manually" procedure above. If you do not intend to link it, register a different image name.

- If a package with the same name exists but is not linked to this repository or is linked to another repository, the automatic connection test fails.
- A **private** package that is not linked to this repository may not be visible to the competition account (`AISystemChallenge`), allowing the image name check to pass. However, the link job's push will be rejected by that package (`::error::`), preventing the package from being linked. After waiting several minutes, registration fails with the message "The image package is not linked to the repository." It is not rejected immediately. If a package with the same name already exists, link it first or register a different name.
- If the package is already linked to this repository, a link job failure can be ignored. Pushing a new tag will still create a submission.

### 7. What Happens When an Image Is Pushed

1. A participant runs `docker push` with a new tag under the registered image name (the package must be linked to this repository).
2. GitHub sends a package event (`registry_package`) that triggers the release job in the repository's submission workflow (Actions). The release job creates an official release named after the tag.
3. The system receives the release webhook and verifies its signature and repository.
4. The system queries GitHub for an image version whose **tag matches the release tag** among the GHCR packages linked to the repository. It checks immediately after the release and retries approximately five minutes later if the image is not yet available. The image address does not need to be included in the release description.
5. If a `ghcr.io/…` address is included in the release description (as it is in automatically created releases), that address is also verified using the same rules. Rejected addresses appear **only as warnings** in notifications to the organizing team.
6. The organizing team receives a notification containing the pull address and digest.

If the tag cannot be found in the package list during the second check, the image may be omitted from the notification. In that case, contact the organizing team.

##### Image Requirements

All three conditions must be satisfied: **Same account + Linked repository + Existing tag**.

- "Same account" refers to the **account that owns the repository**, not the individual who pushes the image. Therefore, any team member can push an image.
- "Linked repository" means that the image is associated with the repository as a package. During registration, the workflow's link job establishes this connection by pushing a placeholder image. If the package is not linked, connect it manually from the package page (see "6. Linking an Image to the Repository"). The image `LABEL` is not used to establish the connection.
- The tag must match the release tag. Automatically created releases use the name of the pushed tag, so the two always match.

### 8. What Is a Digest?

A digest is the `sha256` fingerprint (content address) of an image manifest. A tag (such as `v1.0.0`) can later be reassigned to a different build, but a digest uniquely identifies a specific image manifest. Therefore, notifications include the digest so that the exact image can be pulled as shown below.

```
docker pull ghcr.io/<owner>/<image>@sha256:…
```

For multi-architecture images, the index digest is used.

### 9. Rejection Conditions

If an address included in the release description meets any of the following conditions, the image is rejected. Rejected image addresses appear **only as warnings** in notifications to the organizing team and are excluded from the list of images to pull.

- Images belonging to a different account.
- Packages not linked to the registered repository. Granting an individual direct read access does not count as linking the package to the repository.
- Missing tags, including cases where the tag is still unavailable after the five-minute retry.
- Failure to retrieve image information.

### 10. FAQ

**Does the image need to exist at registration time?**

No. You only register the final component of the image name. If no package with the same name exists, the check passes, and the workflow's link job committed during registration pushes a `hello-world` placeholder image (`aisc-registration`) to create the package and link it to the repository. Registration is completed after the system verifies the connection. Build and push your submission image yourself after registration.

**What should I enter as the image name?**

Enter only the final component (e.g., `my-server`). The system constructs the full name as `ghcr.io/<repository owner>/my-server`. Use lowercase letters and numbers, with separator characters (`.`, `_`, `__`, `-`) only between characters, and a maximum length of 100 characters. Uppercase letters are converted to lowercase. Tags (such as `:v1.0.0`), digests, hierarchical names such as `a/b/c`, and names belonging to another account are not allowed. Even if you enter a full name such as `ghcr.io/<owner>/my-server`, only the final component is used.

**I haven't received the verification code email. What should I do?**

First, check your spam folder. The email subject ends with "Email Verification Code," and the message contains a six-digit code and its expiration time. Make sure the email address shown on the verification screen is correct. If the email has not arrived, select **"Other Actions (다른 동작)"** on the same screen and choose **"Resend Code (코드 다시 받기)"**. A new code will be sent, and the previous code will no longer be valid. If the email address is incorrect, choose **"Change Email Address (이메일 주소 고치기)"**. A maximum of four verification emails can be sent per registration attempt.

**What happens if I enter the wrong verification code?**

You can enter an incorrect code up to five times, and the remaining attempts are displayed on the screen. If you enter an incorrect code for the fifth time, verification ends, and you must start registration again. Entries that are not six-digit numbers (including empty entries) do not count toward the limit. The code expires after 10 minutes. If it expires, select **"Resend Code (코드 다시 받기)"** to receive a new one.

**Does pushing the `latest` tag count as a submission?**

No. The `latest` tag is not submitted. If `docker push` is used without an explicit tag, `latest` is pushed. This rule prevents accidental submissions. Use a version tag (e.g., `v1.0.0`) when pushing.

**Does pushing the same tag again create another submission?**

No. If a release with that tag already exists, the workflow skips it. To submit again, push a new tag (e.g., `v1.0.1`).

**Are test pushes and official submission pushes handled differently?**

No. There is no distinction. Every new tag other than `latest` and `aisc-registration` is treated as a submission.

**What is the `aisc-registration` tag?**

It is a **placeholder** tag pushed by the workflow's link job during registration. This push creates the image package and links it to the repository. `aisc-registration` is not a submission. Pushing this tag does not create a release or notification because the release job skips it. Use a different version tag (e.g., `v1.0.0`) for submissions.

**Do I need to add a LABEL?**

No. A `LABEL` is not required. Including one is acceptable, but the package is linked to the repository by the workflow's link job, not by a `LABEL`. A package created by a direct push remains unlinked even if the image contains a `LABEL`.

**How can I verify that the package is linked to the repository?**

If registration passes, the package is linked. To verify manually, check whether the package appears in the **Packages** list on the right side of the repository's main page or whether the repository name appears on the package page. If the package was deleted or is not linked, open `AI System Challenge submission` under the repository's **Actions** tab and click **Run workflow** to rerun the link job. If it is still not linked, connect it under **Package settings → Connect repository** on the package page, then push again using a new tag. See "6. Linking an Image to the Repository" above for details.

**The link job failed during registration. What should I do?**

Check the `::error::` message in the Actions run logs. There are two possible causes. If a package with the same name already exists but is not linked to this repository (e.g., a package created by a direct push), link this repository under **Package settings → Connect repository** on the package page. If the package is already linked, the failure can be ignored, and pushing a new tag will create a submission. If an organization policy prevents Actions from writing packages using `GITHUB_TOKEN`, correct the policy and rerun the job using **Run workflow**. If you are still in the registration process, clicking **"Retry Test"** on the registration page also reruns the link job.

**I see "The image package is not linked to the repository" during registration. What does this mean?**

It means the system waited several minutes but the package was still not linked to the repository. Possible causes include disabled Actions, organization policies preventing package writes through `GITHUB_TOKEN`, or an existing unlinked package with the same name (such as one created by a direct push), which caused the link job's push to be rejected. Check the `::error::` message in the link job logs under `AI System Challenge submission` in the Actions tab. After resolving the issue, click **"Retry Test"**. If a package with the same name already exists, link it under **Package settings → Connect repository** or register again using a different name.

**I pushed an image, but no release was created.**

First, check the `AI System Challenge submission` runs under the repository's Actions tab. If no run exists, the package may not be linked to the repository (e.g., a package created by a direct push before registration), or Actions may be disabled. If a run exists, check the logs for the reason it was skipped (`latest`, `aisc-registration`, an existing release with the same tag, or an unregistered image name) or for a tag naming error.

**What happens if Actions is disabled?**

The system does not directly detect whether Actions is enabled. However, if Actions is disabled, the link job will not run and the package will not be linked during registration. Registration will fail after several minutes, and the registration page will explain the cause. Actions cannot remain disabled. The release job will also not run, so no release or submission will be created. Enable Actions under **Settings → Actions → General**, then click **"Retry Test"**. If the package was deleted after registration, rerun the link job using **Run workflow**.

**Does the image have to be public?**

No. What matters is whether the image is linked to the repository, not whether it is public.

**Do I have to include the pull address in the release description?**

No. The release is created automatically by the release job, and the system searches packages linked to the repository for a tag matching the release tag.

**Are there any rules for pushed tags?**

Tags must follow Docker tag naming rules. They may contain only letters, numbers, `_`, `.`, and `-`, must be no longer than 128 characters, and must begin with a letter, number, or `_`. Example: `v1.0.0`. Invalid tags cause the workflow to stop with an error without creating a release.

**Can I modify the workflow file?**

Modifying the file is not recommended. Do not change `AISC_IMAGE`, as the workflow will no longer be able to find the submission image. Modifying other parts may prevent releases from being created according to the submission rules. Editing and pushing the file triggers the link job again (which pushes only the placeholder image, not a submission). Registering again restores the file to the common version.

**Is manually adding the submission workflow sufficient?**

No. One-time registration (collaborator invitation, webhook setup, and automatic connection test) must be completed separately. Manual installation is intended only for cases where branch protection rules prevent the system from committing the workflow file.

**What if branch protection rules are enabled?**

Allow `AISystemChallenge` to bypass the protection rules or manually add the public template. See "When Branch Protection Rules Prevent Commits" above for detailed instructions.

**What should teams registered under the previous instructions do?**

Repositories registered under the previous instructions (where publishing a release triggered a workflow to build the image, or where adding a `LABEL` was described as sufficient to link the package) may still contain the previous workflow file. **Register again** using the same repository and image name. The automatic connection test updates the file with the new content, and the resulting commit triggers the link job to connect the package to the repository. The new workflow does not build submission images, so build your image yourself and run `docker push`. If a package has already been pushed, check whether it is linked in the Packages list. If it is not linked, follow "6. Linking an Image to the Repository" above.
