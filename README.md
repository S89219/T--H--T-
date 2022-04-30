
<!doctype html>
<html>
<head>
<!-- Copyright 2019 The Chromium Authors. All rights reserved.
     Use of this source code is governed by a BSD-style license that can be
     found in the LICENSE file. -->
<meta charset="utf-8">
<script type="module" src="autofill_and_password_manager_internals.js"></script>
<link rel="stylesheet" href="chrome://resources/css/chrome_shared.css">
<!--
  The style sheets are inlined to get a prettier export if the user presses
  Ctrl/Cmd + S to save the site or presses the download button.
-->
<style>
html {
  scroll-behavior: smooth;
}

.sticky-bar {
  background-color: white;
  border-bottom: 1px solid black;
  color: black;
  overflow: auto;
  padding-bottom: 1.5ex;
  position: sticky;
  top: 0;
}

#log-display-config {
  display: none; /* only visible for Autofill, not for Password Manager */
  font-size: 120%;
  padding: 1ex;
}

#log-display-config label {
  padding-inline-end: 1em;
}

.fake-button {
  background-color: lightgray;
  border: 1px solid black;
  margin-inline-end: 1em;
  padding: .5ex;
}

#logging-note {
  font-style: italic;
}

#logging-note-incognito {
  font-style: italic;
}

/* Initially, nothing is visible, to avoid flicker. */
#log-entries,
#logging-note,
#logging-note-incognito {
  display: none;
}

/* Visibility settings for non-Incognito tabs. */
[data-incognito=false] #log-entries,
[data-incognito=false] #logging-note {
  display: block;
}

/* Visibility settings for Incognito tabs. */
[data-incognito=true] #logging-note-incognito {
  display: block;
}

#version-info {
  margin: 3px;
  padding: 3px;
}

.version {
  font-family: monospace;
  max-width: 430px;
  padding-inline-start: 5px;
  vertical-align: top;
  word-break: break-word;
}

.label {
  font-family: monospace;
  font-weight: 200;
  vertical-align: top;
}

.log-entry,
.marker {
  padding: 3px;
}

.marker {
  background-color: red;
  font-size: 200%;
  overflow-wrap: break-word;
  white-space: normal;
  word-wrap: break-word;
}

.marker::before {
  content: 'Position marked: ';
}

/*
 * Colors can be taken from
 * https://material.io/design/color/#tools-for-picking-colors
 * Pick the rows of entries labeled with 100
 */

.log-entry[scope='Context'] {
  background-color: #F5F5F5;
}

.log-entry[scope='Parsing'] {
  background-color: #FFECB3;
}

.log-entry[scope='AbortParsing'] {
  background-color: #FFCDD2;
}

.log-entry[scope='Filling'] {
  background-color: #D1C4E9;
}

.log-entry[scope='Submission'] {
  background-color: #BBDEFB;
}

.log-entry[scope='AutofillServer'] {
  background-color: #D7CCC8;
}

.log-entry[scope='Metrics'] {
  background-color: #B2EBF2;
}

.log-entry[scope='AddressProfileFormImport'] {
  background-color: #BFFBF2;
}

.log-entry[scope='CreditCardUploadStatus'] {
  background-color: #4DB6AC;
}

.log-entry[scope='CardUploadDecision'] {
  background-color: #4DD0E1;
}

.log-entry[scope='Rationalization'] {
  background-color: #F8BBD0;
}

/*
 * Checkboxes add/remove hide-<Scope> classes to the #log-entries. Hiding of the
 * relevant <div>'s and adjacent <hr>'s is implemented by these classes.
 */

.hide-Context .log-entry[scope='Context'],
.hide-Context .log-entry[scope='Context'] + hr {
  display: none;
}

.hide-Parsing .log-entry[scope='Parsing'],
.hide-Parsing .log-entry[scope='Parsing'] + hr {
  display: none;
}

.hide-AbortParsing .log-entry[scope='AbortParsing'],
.hide-AbortParsing .log-entry[scope='AbortParsing'] + hr {
  display: none;
}

.hide-Filling .log-entry[scope='Filling'],
.hide-Filling .log-entry[scope='Filling'] + hr {
  display: none;
}

.hide-Submission .log-entry[scope='Submission'],
.hide-Submission .log-entry[scope='Submission'] + hr {
  display: none;
}

.hide-AutofillServer .log-entry[scope='AutofillServer'],
.hide-AutofillServer .log-entry[scope='AutofillServer'] + hr {
  display: none;
}

.hide-Metrics .log-entry[scope='Metrics'],
.hide-Metrics .log-entry[scope='Metrics'] + hr {
  display: none;
}

.hide-AddressProfileFormImport .log-entry[scope='AddressProfileFormImport'],
.hide-AddressProfileFormImport .log-entry[scope='AddressProfileFormImport'] + hr {
  display: none;
}

.hide-CreditCardUploadStatus .log-entry[scope='CreditCardUploadStatus'],
.hide-CreditCardUploadStatus .log-entry[scope='CreditCardUploadStatus'] + hr {
  display: none;
}

.hide-CardUploadDecision .log-entry[scope='CardUploadDecision'],
.hide-CardUploadDecision .log-entry[scope='CardUploadDecision'] + hr {
  display: none;
}

.form {
  border: 1px black solid;
  margin: 3px;
  padding: 3px;
}

.form td {
  vertical-align: text-top;
}

.profile_import_from_form_section {
  border: 1px black solid;
  margin: 3px;
  padding: 3px;
}

.profile_import_from_form_section td {
  vertical-align: text-top;
}

.country_data {
  border: 1px black solid;
  margin: 3px;
  padding: 3px;
}

.country_data td {
  vertical-align: text-top;
}

.modal-dialog {
  background-color: rgb(255, 255, 255);
  border: 1px solid rgb(0, 0, 0);
  display: block;
  height: 100px;
  left: 10%;
  overflow: auto;
  position: fixed;
  right: 10%;
  top: 10%;
  width: 80%;
  z-index: 1;
}

.modal-dialog-content {
  padding: 20px;
}

.modal-dialog-close-button {
  bottom: 20px;
  position: absolute;
  right: 20px;
}

</style>
</head>
<body>
<div>
  <h1 id="h1-title"></h1>
  <div id="log-display-config">
    <span id="marker-fake-button" class="fake-button">Add Marker</span>
    <input type="checkbox" id="enable-autoscroll" checked><label for="enable-autoscroll">Enable autoscroll</label>
    <span id="checkbox-placeholder"></span>
    <span id="reset-cache-fake-button" class="fake-button" style="display: none">Reset Cache</span>
    <span id="download-fake-button" class="fake-button">Download Log</span>
  </div>
</div>
<div id="logging-note"></div>
<div id="logging-note-incognito"></div>
<div id="version-info">
  <table>
    <tr>
      <td class="label">Version:</td>
      <td class="version"><span>100.0.4896.127</span>
        (<span>official</span>)
        <span></span></td>
    </tr>
    <tr>
      <td class="label">Revision:</td>
      <td class="version"><span>ff0d0695743e65305d7194f9bd309e5e1c824aa0-refs/branch-heads/4896_88@{#4}</span></td>
    </tr>
    <tr>
      <td class="label">User Agent:</td>
      <td class="version"><span>Mozilla/5.0 (Linux; Android 8.1.0; CPH1805) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.127 Mobile Safari/537.36</span></td>
    </tr>
    <tr>
      <td class="label">App Locale:</td>
      <td class="version"><span>vi</span></td>
    </tr>
    <tr>
      <td class="label">Variations:</td>
      <td class="version" id="variations-list"></td>
    </tr>
  </table>
</div>
<div id="log-entries">
</div>
</body>
</html>

# GitHub public feedback discussions

In this repository, you can find the [official GitHub public feedback discussions](https://github.com/github/feedback/discussions) for the following product areas, as well as an overall category for general product feedback:

| **Feedback Category** | **About the Product** 	|
|---	|---	|
| 🚢  [Actions and Packages](https://github.com/github/feedback/discussions/categories/actions-and-packages-feedback) 	| [GitHub Actions](https://github.com/features/actions) and [GitHub Packages](https://github.com/features/packages) |
| 🔎  [Code Search & Navigation](https://github.com/github/feedback/discussions/categories/code-search-and-navigation-feedback) 	| [Code Search & Navigation](https://cs.github.com/about) 	|
| 💻  [Codespaces](https://github.com/github/feedback/discussions/categories/codespaces-feedback) 	| [GitHub Codespaces](https://github.com/features/codespaces) 	|
| 👩‍✈️  [Copilot](https://github.com/github/feedback/discussions/categories/copilot-feedback)   	| [GitHub Copilot](https://copilot.github.com/) (Technical Preview) 	|
| 🤖  [Dependabot](https://github.com/github/feedback/discussions/categories/dependabot-feedback) 	| [GitHub Dependabot](https://github.com/features/security) 	|
| 🗣️  [Discussions](https://github.com/github/feedback/discussions/categories/discussions-feedback)  	| [GitHub Discussions](https://docs.github.com/en/discussions) 	|
| 🌐  [Feed](https://github.com/github/feedback/discussions/categories/feed-feedback)  	| [GitHub Feed](https://github.blog/2022-03-22-improving-your-github-feed/) 	|
| 🐙  [Issues](https://github.com/github/feedback/discussions/categories/issues-feedback) 	| [GitHub Issues](https://github.com/features/issues) 	|
| ⭐  [Lists](https://github.com/github/feedback/discussions/categories/lists-feedback) 	| [GitHub Lists](https://docs.github.com/en/get-started/exploring-projects-on-github/saving-repositories-with-stars#organizing-starred-repositories-with-lists) (Public Beta) 	|
| 📱  [Mobile](https://github.com/github/feedback/discussions/categories/mobile-feedback) 	| [GitHub Mobile](https://github.com/mobile) 	|
|  🖼️  [Profile](https://github.com/github/feedback/discussions/categories/profile-feedback)  	| [GitHub Profile](https://docs.github.com/en/account-and-profile/setting-up-and-managing-your-github-profile/customizing-your-profile/about-your-profile) 	|
| ✔️  [Pull Requests](https://github.com/github/feedback/discussions/categories/pull-requests-feedback) 	| [GitHub Pull Requests](https://docs.github.com/en/github/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) 	|
|  💖  [Sponsors](https://github.com/github/feedback/discussions/categories/sponsors-feedback) 	| [GitHub Sponsors](https://github.com/sponsors) 	|
| :octocat:  [General Feedback](https://github.com/github/feedback/discussions/categories/general-feedback) 	| [GitHub Docs](https://docs.github.com/en) |

These discussions are where you can share suggestions for how the products should be improved and discuss those improvements with the community, including members of the GitHub product team. Check out [Making suggestions](#making-suggestions) to learn how to provide feedback.

This repository works in conjunction with the [GitHub public product roadmap](https://github.com/github/roadmap), which is where you can learn about what features we're working on, what stage they're in, and when we expect to bring them to you. Accordingly, the Issues feature of this repository has been disabled. Discussion categories have been established for specific features listed above, as well as a general category for other topics. Additional categories may be added in the future. In the meantime, topics outside of the listed categories above, will be transferred into the general category. Please review the [CODE OF CONDUCT](CODE_OF_CONDUCT.md) before participating in discussions.

## Making suggestions

We encourage you to [open a discussion](https://github.com/github/feedback/discussions) if you have suggestions for how we can improve our products. You don't need to have a solution to the problem you are facing to kick off a discussion. We are hoping to foster productive and collaborative conversations, so please check out [how to give good feedback](https://github.com/github/feedback/discussions/1) if you want some guidance on how to kick off a successful discussion.

Prior to making a new discussion, please take a look at previous discussions to see if someone else has already brought to our attention your suggestions. If you find a similar discussion, reply with additional details or upvote the discussion to signal your support rather than creating a new discussion.

### From a suggestion to a shipped feature

Once you kick off a discussion, the GitHub product team will evaluate the feedback but will not be able to respond to every submission. From there, we will work with you, and the entire community, to ensure we understand the current capabilities GitHub doesn’t have and explore the space for potential solutions to your problem statement:

- If the product team determines that we are going to prioritize a feature to solve the problem you've identified, we may open an issue and track its development in the [public roadmap](https://github.com/github/roadmap).
- If the product team determines that we will not be working to solve the problem you have identified, we may comment on the discussion describing our reasoning so our decisions can remain transparent.

## Disclaimer

Any statement in this repository that is not purely historical is considered a forward-looking statement. Forward-looking statements included in this repository are based on information available to GitHub as of the date they are made, and GitHub assumes no obligation to update any forward-looking statements. The forward-looking comments in the public feedback discussions do not represent a commitment, guarantee, obligation or promise to deliver any product or feature, or to deliver any product and feature by any particular date, and are intended to outline the general development plans. Customers should not rely on these public feedback discussions to make any purchasing decision.
