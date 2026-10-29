---
name: send-a-fax
description: Send a fax with Faxette to the US, Canada, the EU, the UK, Switzerland or elsewhere in Europe, or check on one. Use when the person asks to fax a document or form, asks how a Faxette fax is doing, or asks about faxes they've sent.
---

# Send a fax with Faxette

This plugin has two Faxette MCP servers with the same tools. Use the one that covers the fax number's country for every step of that fax:

- `faxette` for the US and Canada
- `faxette-eu` for the EU, the UK, Switzerland, Norway, Iceland, Liechtenstein, Monaco, Montenegro, Ukraine and the Isle of Man

Faxette doesn't fax anywhere else yet. The person pays on Faxette's status page. If they're signed in and have credit, they can instead send it from their credit once they agree to the price (step 5).

Sending a fax doesn't need an account. People with a Faxette account can sign in so their faxes stay in it: `get_account`, `get_faxes`, `get_fax` and `send_fax` need that, and calling one while they aren't signed in asks your app to sign them in. Accounts belong to one server: an account on `faxette` doesn't work on `faxette-eu`.

## 1. Find and confirm the fax number

Find the fax number on the official source, such as the agency's or office's own website, never a directory or a search snippet. Show the person the number and the page it came from, and have them confirm it before you call `create_fax`. Pass it in international format, like `+14155552671`.

If a form needs a handwritten signature, tell the person to print, sign and scan it before you upload it.

## 2. Ask for their email

If the person is signed in, Faxette uses their account's email: don't ask, and leave `email` out of `create_fax`. Otherwise ask for the email address where Faxette sends the receipt and updates. Never guess it.

## 3. Upload each file

Faxette takes PDFs and photos saved as JPG, PNG, WEBP or HEIC, up to 20 MB each and 10 per fax. Upload them in the order they should be faxed. For each file:

1. Get its size in bytes and its base64 MD5 checksum:

   ```sh
   wc -c < referral.pdf
   openssl md5 -binary referral.pdf | base64
   ```

2. Call `create_document` with the fax number, the filename, the size, the checksum and the content type.
3. Within 5 minutes, PUT the file's exact bytes to `upload_url`, with one `-H` for each entry in `upload_headers`:

   ```sh
   curl -sS -f -X PUT --data-binary @referral.pdf -H "Content-Type: application/pdf" "<upload_url>"
   ```

4. Keep the `document_id` for `create_fax`.

If you can't read the files, skip this step and tell the person they'll add their files on the status page.

## 4. Start the fax

Call `create_fax` with the fax number, the email (unless they're signed in) and the `document_id`s in order. Give the person the `status_url`. They check the fax number, documents and price there, and pay to send it. Nothing is sent or charged until they do, unless they're signed in and choose to send it from their credit (step 5).

Never say a fax is sent or paid until `get_fax_status` or `get_fax` says so.

## 5. Send it from credit

Only when the person is signed in and asks to send from their credit:

1. Call `get_fax` with the fax's `id`. Its `status` must be `documents_ready`.
2. Call `get_account`. If their credit doesn't cover the fax's `credit_price`, give them the `status_url` instead, where they can buy credit or pay by card.
3. Tell them the `credit_price` and that it comes from their credit, and wait for a clear yes.
4. On `faxette-eu`, also tell them sending starts right away and their right to cancel ends once the fax is delivered, and get their explicit agreement to that. Agreeing to the price isn't enough. Then pass `immediate_sending_consent: true`.
5. Call `send_fax` with the `id` and the `credit_price` `amount_cents` they agreed to.

If `send_fax` answers `price_changed`, call `get_fax` again and confirm the new price. If it answers `guest_fax`, the fax was started without signing in: give them the `status_url` to pay by card, or start a new fax now that they're signed in.

## 6. Check on it

Call `get_fax_status` on the same server, with the `status_url`, only when the person asks, or at most once a minute while the fax is sending. If they're signed in, `get_fax` with the fax's `id` also gives the documents' file names and the fax's history.

- `document_errors` lists files Faxette couldn't use. Tell the person what's wrong; they fix it on the status page.
- `checkout_available: true` means the person can pay on the status page.
- A `failed` fax with `failure_reason: wrong_number` means no fax machine answered. If `resend_by` is set, the person can correct the fax number on the status page and send it again until then.

## Their faxes

When a signed-in person asks about faxes they've sent, use `get_faxes`. It returns 25 at a time, newest first: pass `next_page` back as `page` for more, and filter with `status`, like `["failed"]`. Use `get_account` to see which account they signed in with.

## Privacy

Don't repeat the contents of medical or legal documents in the chat beyond what's needed.
