# Forms

The Form element (`{template_dir}/includes/elements/form.php`, 4035 lines) is a full form builder. A form has **fields** (the inputs) and **actions** (what runs after a successful submission).

## Top-level form settings shape

```jsonc
{
  "name": "form",
  "settings": {
    "fields":   [ /* see Field Types below */ ],
    "actions":  [ "email", "redirect", "save-submission" ],
    "submitButtonText":  "Send message",
    "submitButtonStyle": "primary",
    "submitButtonSize":  "md",
    "submitButtonBlock": false,

    /* per-action settings -- only the keys for actions you list above */
    "emailTo":         "admin_email",
    "emailToCustom":   "you@site.com,sales@site.com",
    "emailSubject":    "Contact form",
    "emailContent":    "From {{form-name}} ({{form-email}}):\n\n{{all_fields}}",
    "fromName":        "Site",
    "fromEmail":       "noreply@site.com",
    "replyToEmail":    "{{form-email}}",
    "htmlEmail":       true,

    /* spam protection */
    "enableRecaptcha":  true,
    "enableHCaptcha":   "invisible",
    "enableTurnstile":  true,
    "turnstileSize":    "flexible",
    "turnstileTheme":   "light",

    /* misc */
    "successMessage":   "Thanks!",
    "errorMessage":     "Something went wrong.",
    "ajax":             true,
    "scrollToFormOnError": true,
    "saveSubmission":   true
  }
}
```

## Field shape (each item in `fields[]`)

```jsonc
{
  "id":           "name",                // referenced by {{name}} in templates and as form data key
  "type":         "text",
  "label":        "Your name",
  "placeholder":  "Jane Doe",
  "value":        "",                    // default
  "name":         "fullname",            // optional explicit name attr
  "required":     true,
  "minLength":    2,
  "maxLength":    80,
  "errorMessage": "Please enter your name",
  "autocomplete": "name",
  "pattern":      "^[A-Za-z\\s]+$",
  "title":        "First and last name",
  "width":        100,                    // % column width within form grid
  "_conditions":  [ /* show/hide based on other fields or context */ ]
}
```

The **id** is critical -- it's the JSON key in submission data and the placeholder used in `{{id}}` interpolation in email templates and webhook bodies.

## Field types

### Text-like
- **`text`** -- single-line.
- **`email`** -- single-line with email validation.
- **`tel`** -- telephone, no built-in pattern. Add `pattern` if needed.
- **`url`** -- URL validation.
- **`password`** -- password input. Optional `passwordToggle` (boolean), `passwordShowIcon`, `passwordHideIcon` (icon controls).
- **`number`** -- `min`, `max`, `step` allowed.
- **`textarea`** -- multi-line. `height` (number/px), `resize` = `none|vertical|horizontal|both`.
- **`richtext`** -- TinyMCE editor. Extra: `richtextToolbar`, `richtextSkin` (`oxide|oxide-dark`), `richtextContentStyle`, `richtextMenuBar`, `richtextStatusBar`, `richtextHeight`.

### Choice
- **`select`** -- dropdown. `options` is a textarea, one per line. Use `value:Label` per line for distinct values, set `valueLabelOptions: true`.
- **`radio`** -- radio group. Same `options` format.
- **`checkbox`** -- checkbox group. `options` same format. Submitted value is comma-separated string of selected `value`s. Single-checkbox usage: one option line.

### Date/time
- **`datepicker`** -- Flatpickr. `time` (boolean), `l10n` (language code, e.g. `en`, `de`), `minDate`, `maxDate`, `minTime`, `maxTime`, `dateFormat` (Flatpickr format).

### Files & media
- **`file`** -- file upload. `fileUploadButtonText`, `fileUploadLimit` (max files), `fileUploadSize` (MB cap), `fileUploadAllowedTypes` (CSV: `pdf,jpg,png`), `fileUploadStorage` = `attachment|directory`, `fileUploadStorageDirectory` (relative path under uploads if `directory`).
- **`image`** -- single image upload via media library (logged-in users only).
- **`gallery`** -- multi-image upload via media library.

### Special
- **`hidden`** -- pre-filled, no UI. Use with dynamic data: `value: "{post_id}"` to pass context.
- **`html`** -- static HTML inside the form layout (informational text, dividers).
- **`rememberme`** -- for the login action only.

## Spam protection

- **Honeypot:** add ANY field, set `isHoneypot: true`. The field is rendered hidden; bots that fill it get rejected. Cheap, no API.
- **reCAPTCHA v2:** `enableRecaptcha: true` + site/secret key in `bricks_global_settings.apiKeyGoogleRecaptcha`.
- **hCaptcha:** `enableHCaptcha: 'visible'|'invisible'` + `apiKeyHCaptcha` global. `hCaptchaSize`, `hCaptchaTheme`.
- **Turnstile:** `enableTurnstile: true` + `apiKeyTurnstile`. `turnstileSize` = `normal|compact|flexible`, `turnstileTheme` = `light|dark|auto`, `turnstileLabel`.

## Actions (`settings.actions[]`)

The list specifies which action handlers run on submit, in order. Per-action settings are flat keys on the same `settings` object:

### `email`
Send an email to admin (and optionally a confirmation to the user).
- `emailTo` = `admin_email | custom`
- `emailToCustom` (CSV emails)
- `emailBcc`
- `emailSubject`
- `emailContent` -- supports `{{field-id}}` and `{{all_fields}}` placeholders
- `fromName`, `fromEmail`, `replyToEmail`
- `htmlEmail` (boolean)
- Confirmation email (sends to a form field that contains user's email):
  `confirmationEmailSubject`, `confirmationEmailTo`, `confirmationFromEmail`, `confirmationFromName`, `confirmationReplyToEmail`, `confirmationEmailContent`, `confirmationEmailHTML`.

### `redirect`
- `redirect` -- URL or absolute path.
- `redirectAdminUrl` (boolean) -- redirect into `/wp-admin`.
- `redirectTimeout` (ms).

### `webhook`
- `webhooks[]` -- array of `{ name, url, contentType: 'json'|'form-data', dataTemplate, headers }`.
  - `dataTemplate` is a string, often `{{all_fields}}` or `{ "x":"{{field-id}}" }`.
  - `headers` is a JSON string.
- `webhookMaxSize` (KB), `webhookRateLimit` (boolean) + `webhookRateLimitRequests` (per-hour cap).
- `webhookErrorIgnore` (boolean), `webhookErrorMessage`.

### `mailchimp`
- `mailchimpList`, `mailchimpGroups[]`, `mailchimpEmail` (= field-id of the email input), `mailchimpFirstName`, `mailchimpLastName`, `mailchimpDoubleOptIn`, `mailchimpPendingMessage`, `mailchimpErrorMessage`.

### `sendgrid`
- `sendgridList`, `sendgridEmail`, `sendgridFirstName`, `sendgridLastName`, `sendgridErrorMessage`.

### `registration`
Creates a WP user from the form.
- `registrationEmail`, `registrationUserName`, `registrationFirstName`, `registrationLastName`
- `registrationPassword` (field-id; auto-generates if missing)
- `registrationPasswordMinLength`
- `registrationRole` = `subscriber|contributor|author|editor|administrator`
- `registrationWPNotification` (sends standard WP notification)
- `registrationErrorMessage`.

### `login`
Logs the user in.
- `loginRedirect`, `loginRememberMe`, `loginErrorMessage`.

### `lost-password` / `reset-password`
Standard WP lost-password flow / token-based reset.

### `create-post` / `update-post`
Creates or updates a post from form fields.
- `createPostType`, `createPostStatus` = `draft|pending|publish`, `createPostAuthor`, `createPostTitle` (field-id), `createPostContent`, `createPostExcerpt`, `createPostFeaturedImage` (image field id).
- `createPostMeta`: `[ { metaKey, metaValue: '{field-id}' } ]`.
- `createPostTaxonomies`: `[ { taxonomy, fieldId } ]`.
- `update-post` adds `updatePostId`, `updatePostDisableCapabilityCheck`.

### `custom`
- `customCode` -- PHP code executed after submit. Receives form data via globals; **runs on the server, no eval()** -- Bricks compiles this into a closure on save.

### `save-submission`
Saves the submission to a custom table; viewable under WP Admin -> Bricks -> Form Submissions.
- `submissionFormName` -- display label.

### `unlock-password-protection`
Used for password-protected pages -- unlocks the gate when the form succeeds.

## Conditional fields

Each field's `_conditions` follows the same shape as element conditions (see `references/conditions.md`). Common conditions for fields:
- Show field B only if field A's value is X.
- Show only for logged-in users.

## File uploads

- Storage `attachment` puts the file in WP Media Library and links to the resulting attachment in submission data.
- Storage `directory` writes to `wp-content/uploads/{fileUploadStorageDirectory}/`. Use this for stuff that shouldn't pollute the media library (resumes, etc.).
- `fileUploadAllowedTypes` is a CSV of extensions, no dots.
- `fileUploadSize` in MB.

## Submission data shape

What a form action handler receives (also what's saved in submissions and posted to webhooks):

```jsonc
{
  "form_id":   "abc123",
  "form_name": "Contact form",
  "post_id":   42,
  "fields": {
    "name":  { "value": "Jane Doe",        "type": "text" },
    "email": { "value": "jane@example.com","type": "email" },
    "message":{ "value": "Hello",          "type": "textarea" }
  },
  "files":  [ /* uploaded file metadata */ ],
  "ip":     "1.2.3.4",
  "useragent": "..."
}
```

`{{all_fields}}` in templates expands to a formatted text dump of all field labels + values.

## Programmatic hooks

```php
add_action( 'bricks/form/before_send_email',     $cb, 10, 2 ); // $form, $args
add_action( 'bricks/form/after_email_send',      $cb, 10, 2 );
add_action( 'bricks/form/custom_action',         $cb, 10, 1 ); // for action: 'custom'
add_filter( 'bricks/form/email_attachments',     $cb, 10, 2 );
add_filter( 'bricks/form/before_validate',       $cb, 10, 1 );
add_filter( 'bricks/form/validation_errors',     $cb, 10, 2 );
add_action( 'bricks/form/submission_saved',      $cb, 10, 1 );
```

Hook names and signatures: grep `{template_dir}/includes/elements/form.php` for `do_action(` and `apply_filters(` to confirm.

## Recipes

**Newsletter signup with redirect:**
```jsonc
"actions": ["mailchimp","redirect"],
"mailchimpList": "abc123",
"mailchimpEmail": "email",
"redirect": "/thanks"
```

**Multi-recipient contact form with webhook to Slack:**
```jsonc
"actions": ["email","webhook","save-submission"],
"emailTo": "custom",
"emailToCustom": "sales@x.com,support@x.com",
"emailContent": "{{all_fields}}",
"webhooks": [{
  "name": "Slack",
  "url":  "https://hooks.slack.com/services/...",
  "contentType":"json",
  "dataTemplate": "{\"text\":\"New lead from {{name}} ({{email}}): {{message}}\"}"
}]
```

**Hidden context fields:**
```jsonc
{ "id":"page_url",  "type":"hidden", "value":"{post_url}" },
{ "id":"campaign",  "type":"hidden", "value":"{utm_campaign:fallback:none}" }
```
