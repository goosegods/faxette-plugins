---
name: send-a-fax
description: Send a fax with Faxette to the US, Canada, the EU, Norway, Iceland or Liechtenstein, or check on one. Use when the person asks to fax a document or form, or asks how a Faxette fax is doing.
---

# Send a fax with Faxette

Faxette's tools, `create_document`, `create_fax` and `get_fax_status`, come from the Faxette MCP server in this plugin. The person pays on Faxette's status page, never in this chat. Faxette faxes to the US, Canada, the EU, Norway, Iceland and Liechtenstein.

## 1. Find and confirm the fax number

Find the fax number on the official source, such as the agency's or office's own website, never a directory or a search snippet. Show the person the number and the page it came from, and have them confirm it before you call `create_fax`. Pass it in international format, like `+14155552671`.

If a form needs a handwritten signature, tell the person to print, sign and scan it before you upload it.

## 2. Ask for their email

Ask the person for the email address where Faxette sends the receipt and updates. Never guess it.

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

Call `create_fax` with the fax number, the email and the `document_id`s in order. Give the person the `status_url`. They check the fax number, documents and price there, and pay to send it. Nothing is sent or charged until they do.

Never say a fax is sent or paid until `get_fax_status` says so.

## 5. Check on it

Call `get_fax_status` with the `status_url` only when the person asks, or at most once a minute while the fax is sending.

- `document_errors` lists files Faxette couldn't use. Tell the person what's wrong; they fix it on the status page.
- `checkout_available: true` means the person can pay on the status page.
- A `failed` fax with `failure_reason: wrong_number` means no fax machine answered. If `resend_by` is set, the person can correct the fax number on the status page and send it again until then.

## Privacy

Don't repeat the contents of medical or legal documents in the chat beyond what's needed.
