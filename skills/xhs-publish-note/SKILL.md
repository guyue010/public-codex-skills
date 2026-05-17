---
name: xhs-publish-note
description: Publish image-based Xiaohongshu notes through the web Creator Studio using Chrome automation. Use when the user asks Codex to turn a short idea, generated image, local image, or prepared image asset into a Xiaohongshu 图文笔记, upload the image, fill title/body/topics when appropriate, click publish, and confirm success. Requires the user to already be logged in to Xiaohongshu in Chrome and the Codex Chrome Extension to have file URL access enabled for automated file upload.
---

# Xiaohongshu Image Note Publishing

## Overview

Use Chrome to publish a Xiaohongshu image note from a concise user idea. Follow the successful browser path: prepare a real local image file, upload it through Creator Studio, fill title and body, publish, and verify the success page.

## Preconditions

- Use the `chrome:Chrome` skill for browser control.
- The user must already be logged in at `https://creator.xiaohongshu.com/`.
- The Codex Chrome Extension must have `Allow access to file URLs` enabled; otherwise `fileChooser.setFiles` may fail with `Not allowed`.
- Keep image files under the current workspace and use absolute paths for upload.
- Stop for the user only for login, CAPTCHA, SMS verification, or browser/extension permission blocks.

## Workflow

1. Clarify the publish payload only if necessary:
   - title
   - body text
   - image source or image prompt
   - optional topics, location, visibility, schedule, originality declaration
2. If the user asks for a generated image, use the `imagegen` skill and copy the final image into the workspace.
3. Open Chrome Creator Studio:
   - `https://creator.xiaohongshu.com/publish/publish?from=homepage&target=image`
4. Upload the image:
   - click the visible `上传图片` button
   - wait for `filechooser`
   - call `chooser.setFiles([absoluteImagePath])`
   - wait until the editor appears with `图片编辑` and title/body inputs
5. Fill the note:
   - title input placeholder: `填写标题会有更多赞哦`
   - body editor: `contenteditable="true"`
   - keep copy concise and natural
   - do not add random topics unless the user asks; accept platform suggestions only if useful
6. Publish:
   - scroll or use a screenshot if needed to locate the red `发布` button near the bottom
   - click `发布`
   - wait for and verify `发布成功`
7. Finalize browser tabs:
   - keep the publish/success tab as `deliverable` if useful
   - report title, body, image used, and success confirmation

## Proven Chrome Pattern

Use this pattern after Chrome runtime setup:

```js
const uploadBtn = tab.playwright.getByRole("button", { name: "上传图片", exact: true });
const chooserPromise = tab.playwright.waitForEvent("filechooser", { timeoutMs: 10000 });
await uploadBtn.click({ timeoutMs: 10000 });
const chooser = await chooserPromise;
await chooser.setFiles(["/absolute/path/to/image.png"]);
await new Promise(r => setTimeout(r, 8000));
```

Fill the editor:

```js
await tab.playwright
  .getByPlaceholder("填写标题会有更多赞哦", { exact: true })
  .fill(title, { timeoutMs: 10000 });

await tab.playwright
  .locator('[contenteditable="true"]')
  .fill(body, { timeoutMs: 10000 });
```

## Avoid These Paths

- Do not rely on pasting a public image URL into the upload area; the web publisher did not import URLs as images.
- Do not rely on image clipboard paste; the upload area did not accept image or HTML clipboard data.
- Do not use unofficial Xiaohongshu APIs unless the user explicitly provides credentials and accepts platform risk.
- Do not claim official server-side note publishing is available unless current official docs and credentials confirm it.
