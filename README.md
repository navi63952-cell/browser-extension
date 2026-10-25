# AI Auto Sender

A plain JavaScript Chrome extension (Manifest V3) that repeats a message in the conversation already open in the current tab. Clicking its toolbar icon opens a compact Chrome action popup; the popup is not a standalone webpage or dashboard. It uses each site's webpage controls and does not call AI APIs or open conversations.

## Project structure

- manifest.json
- adapters/base.js, chatgpt.js, gemini.js, claude.js
- background/service-worker.js
- content/automation-engine.js, content.js
- popup/popup.html, popup.css, popup.js

## Install in Chrome

1. Save or extract this project to a folder.
2. Open chrome://extensions.
3. Turn on Developer mode.
4. Choose Load unpacked and select this project folder (the folder containing manifest.json).
5. Open a conversation on ChatGPT, Gemini, or Claude, then click AI Auto Sender in Chrome's toolbar. The toolbar action opens the compact popup anchored to the browser.

After editing files, use Reload on the extension card and refresh the AI website tab so Chrome loads the updated content scripts.

## Use and verify

For each supported site, open an existing conversation and test with a short prompt and a longer interval first. Confirm the first message appears, the counter increments, the next-message countdown appears, and subsequent messages arrive at the interval. Test PAUSE, RESUME, and STOP, then close the popup and confirm the loop continues. With Wait for response to finish checked, confirm the countdown starts after the response finishes. With Send immediately unchecked, confirm the initial interval elapses before the first message.

Repeat this on:

- https://chatgpt.com
- https://gemini.google.com
- https://claude.ai

The website interfaces are dynamic and may change their accessible labels or DOM structure. Each adapter has primary and fallback selectors, but a live logged-in smoke test is needed to confirm compatibility with the current UI and account rollout. If an interface change prevents detection, the popup reports the missing composer/send control or response-detection failure. The extension will not count a send if it cannot observe the composer clear, the composer disappear, or generation begin. If response tracking is enabled and generation cannot be detected, it stops rather than risking another send during an active response.

## How it works

The popup detects the active tab's hostname, stores user settings, and sends commands through the MV3 service worker. The service worker routes each command to that tab and stores a small per-tab state snapshot in chrome.storage.local. If the AI tab was already open when the extension loaded and has no content script yet, the worker injects the packaged adapter bundle into that supported active tab and retries the command. The automation engine and its one timer live in the content script on that tab, so closing the popup does not stop it. Closing the tab clears its saved tab state; changing to a different website stops the automation.

adapters/base.js provides shared composer insertion, submission verification, and DOM-change-based response waiting. Each website adapter owns that site's selectors and implements composer, send-button, and generation detection. The engine only uses the adapter interface, so its logic is independent of website selectors.

The extension stores the message, interval, adapter selection, checkbox settings, message count, and status. It does not read or store passwords, cookies, authentication tokens, API keys, or conversation history. Intervals are expressed in seconds; Chrome can throttle page timers when a tab is backgrounded, so exact timing is not guaranteed.

## Add another AI website

1. Create adapters/perplexity.js and implement a class extending AIAutoSenderBaseAdapter: matches(url), findComposer(), findSendButton(), and isGenerating(). Keep Perplexity-specific selectors in that file.
2. Register the adapter in window.AIAutoSenderAdapters with an id, display name, and class.
3. Add the site's HTTPS URL pattern to host_permissions and content_scripts.matches in manifest.json, and its script to the js list after adapters/base.js.
4. Add its id and label to the popup selector and URL detection in popup/popup.html and popup/popup.js.
5. Reload the extension and refresh the target tab, then verify composer insertion, send verification, and generation detection against the site's current UI.
