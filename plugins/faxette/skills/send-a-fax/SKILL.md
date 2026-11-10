---
name: send-a-fax
description: Send a fax with Faxette to the US, Canada, Mexico, Puerto Rico, Costa Rica, the EU, the UK, Switzerland or elsewhere in Europe, check on one, or read the faxes an organization received. Use when the person asks to fax a document or form, asks how a Faxette fax is doing, asks about faxes they've sent, or asks what came into their organization's inbox.
---

# Send a fax with Faxette

This plugin adds Faxette's MCP server, `faxette`. It faxes to the US, Canada, Mexico, Puerto Rico, Costa Rica, the EU, the UK, Switzerland, Norway, Iceland, Liechtenstein, Monaco, Montenegro, Ukraine and the Isle of Man, through Faxette's US service or its European one. Without signing in, the fax number decides which: a fax to the US or Canada goes through the US service unless you pass `region: "eu"` to `create_document` and `create_fax`, which you do when the person is in Europe.

Faxette doesn't fax anywhere else yet. `get_capabilities` lists each service's countries, languages, file limits and prices. The person pays on Faxette's status page. If they're signed in and have credit, they can instead send it from their credit once they approve the exact fax and its price (step 5).

Sending a fax doesn't need an account. People with a Faxette account can sign in so their faxes stay in it: `get_account`, `get_faxes`, `get_fax`, `open_fax_review`, `send_fax` and the organization tools need that, and calling one while they aren't signed in asks your app to sign them in. A signed-in account only faxes to its own service's countries: a US account to the US, Canada, Mexico, Puerto Rico and Costa Rica, and a European account to Europe, the US and Canada.

## 1. Find and confirm the fax number

Look the office up with `lookup_fax_number` first: it searches Faxette's checked directory and returns each number with its official source and the date it was checked. If it finds nothing, find the number on the official source, such as the agency's or office's own website, never a directory or a search snippet. Either way, show the person the number and where it came from, and have them confirm it before you call `create_fax`. Pass it in international format, like `+14155552671`.

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

If you can't read the files, skip this step and tell the person they'll add their files on the review page or status page.

To fax a letter or note you wrote together, pass it to `create_fax` as `content` (plain text or Markdown, up to 50,000 characters) instead of uploading a file. Faxette prints it as the first pages of the fax. Agree the wording with the person first.

## 4. Start the fax

Call `create_fax` with the fax number, the email (unless they're signed in) and the `document_id`s in order. Also pass:

- `idempotency_key`: a new random UUID for each fax. If the call times out or answers `request_in_progress`, call it again with the same key and the same details. Never change the key to retry, or you'll start a second fax.
- `cover_page`, only if the person asks for one, with wording they've confirmed. It counts toward the pages and the price.
- `language`, if the person wants Faxette's pages and emails in another language this server offers.

Apps that can show interactive screens open Faxette's review screen, where the person adds files and sees the exact PDF that will be faxed. Otherwise give them the `status_url`, or the `review_url` when there is one (it works for 30 minutes). They check the fax number, documents and price there, and pay to send it. Nothing is sent or charged until they do, unless they're signed in and choose to send it from their credit (step 5).

Never say a fax is sent or paid until `get_fax_status` or `get_fax` says so.

## 5. Send it from credit

Only when the person is signed in and asks to send from their credit:

1. Call `get_fax` with the fax's `id`. Its `status` must be `documents_ready`. Keep its `credit_quote`.
2. Call `get_account`. If their credit doesn't cover the fax's `credit_price`, give them the `status_url` instead, where they can buy credit or pay by card.
3. Show them the exact prepared PDF with `open_fax_review`, or its `review_url`. Tell them the fax number, the documents, the `credit_price` and that it comes from their credit, and wait for a clear yes. A price alone isn't approval: they have to see what will be faxed.
4. For a European account (`get_account` shows `region: "eu"`), also tell them sending starts right away and their right to cancel ends once the fax is delivered, and get their explicit agreement to that. Agreeing to the price isn't enough. Then pass `immediate_sending_consent: true`.
5. Call `send_fax` with the `id`, the `credit_quote` as `quote` and the `credit_price` `amount_cents` they agreed to.

The quote covers the exact number, PDF and price they approved, and lasts 30 minutes. If `send_fax` answers `quote_changed`, `quote_required` or `price_changed`, something changed or the quote expired: call `get_fax` again, show the new PDF and price, and get a fresh yes. If it answers `guest_fax`, the fax was started without signing in: give them the `status_url` to pay by card, or start a new fax now that they're signed in.

## 6. Check on it

Call `get_fax_status` on the same server, with the `status_url`, only when the person asks, or at most once a minute while the fax is sending. If they're signed in, `get_fax` with the fax's `id` also gives the documents' file names and the fax's history.

- `document_errors` lists files Faxette couldn't use. Tell the person what's wrong; they fix it on the status page.
- `checkout_available: true` means the person can pay on the status page.
- A `failed` fax with `failure_reason: wrong_number` means no fax machine answered. If `resend_by` is set, it can be sent again until then.

To resend a failed fax, check it with `get_fax` or `get_fax_status`, confirm the fax number with the person (and any corrected number), then call `resend_fax`. It uses the same fax and payment, not a new one.

When the person asks for proof of delivery, call `get_fax_report`. Its links work for 5 minutes. A delivered fax means the machine received it, not that someone read it.

## Their faxes

When a signed-in person asks about faxes they've sent, use `get_faxes`. It returns 25 at a time, newest first: pass `next_page` back as `page` for more, and filter with `status`, like `["failed"]`. Use `get_account` to see which account they signed in with.

## Organizations

An office that uses Faxette as a team is an organization, with a fax number and an inbox. When the person connects your app, they pick which of their organizations it can use. `get_organizations` lists those, each with its `id`, the person's role, its plan and its fax numbers. Pass the `id` as `organization` to `create_fax`, `get_faxes`, `get_fax` and the inbox tools. If there are several and the person hasn't said which, ask; never guess. If the list is empty, they picked none: tell them to reconnect the app and select the organization. A connection made before organizations existed can't use them yet: the first inbox tool asks your app to connect again.

### Send as an organization

Follow steps 1 to 4 with `organization` on `create_fax`. Then, instead of step 5:

1. Call `get_fax`. Its `payment_option` says how `send_fax` would pay: `plan`, when the organization's plan covers the fax at no charge, or `saved_card`, when `amount_cents` is held on the organization's card ending `card_last4` and charged once the fax is delivered.
2. Show the person the exact prepared PDF with `open_fax_review` or its `review_url`, tell them which way it's paid, and wait for a clear yes.
3. Call `send_fax` with the `id` and the `payment_method`, plus the `quote` for `saved_card`.

If `payment_option` is null, the organization has neither a plan that covers the fax nor a card: an owner adds a card or starts a plan in Billing. If `send_fax` answers `payment_changed`, call `get_fax` again and confirm with the person again.

### The inbox

- `get_inbound_faxes` lists the faxes an organization received, 25 at a time, newest first, with who sent each, which number it came to and whether the person has read it. Pass `unread: true` for only the unread ones.
- `get_inbound_fax` gives one fax's details and `pdf_url`, a link to its PDF for 5 minutes, for apps that can download files.
- `read_inbound_fax` returns its pages as images, up to 10. Use it only when the person asks what a fax says or wants it summarized.

Opening a fax either way marks it read for the person. A fax that arrived after the organization's plan ended stays locked until an owner restarts the plan, and a fax whose PDF was already deleted has no pages: both return their details only.

### Blocking a sender

`block_sender` blocks a number for the whole organization: pass `inbound_fax_id` for the number a received fax came from, or `number` for one the person gives you. Before calling it, confirm the number with the person and tell them that future faxes from it will be deleted as they arrive, and that nobody will be emailed. Faxes already in the inbox stay. `get_blocked_senders` lists the blocked numbers, and `unblock_sender` undoes a block.

## Privacy

Don't repeat the contents of medical or legal documents in the chat beyond what's needed.
