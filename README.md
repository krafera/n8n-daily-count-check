# Daily count check for n8n

An n8n workflow that compares yesterday's orders at a source and a destination, matches them on `order_id`, and sends one email listing what is missing, what is extra and which fields differ. The email comes even on clean days, so a morning without it means the check itself has stopped.

## What arrives

```text
Subject: Count check 2026-10-22: 2 missing, 2 extra, 2 different

Daily count check for 2026-10-22.
Source: 10 orders. Destination: 10 rows (read up to 02:00 today).

Missing at the destination (2): 2004, 2007
Only at the destination (2): 2005 (second copy), 2999 (not in the source)
Different (2):
- 2003: amount 245 -> 254
- 2006: status refunded -> paid

This message is sent every day, also when everything matches. If it does not arrive, the check itself has stopped.
```

## How it works

1. **Every morning at 07:00** starts the run in your instance's time zone.
2. **Set the window** covers yesterday for the source, and yesterday 00:00 to 02:00 today for the destination, so a row written just after midnight still counts.
3. **Read the source** and **Read the destination** get the rows from two data tables.
4. **Compare on order_id** matches the rows and skips the columns the destination adds itself (`id, createdAt, updatedAt, written_at`).
5. **Count and write the message** reads outputs 0, 2 and 3 of Compare Datasets (In A only, Different, In B only), drops rows for other days and writes the email.
6. **Send the daily message** sends it.

## Setup

1. Create two data tables: `count_check_source` with `order_id` (string), `order_created_at` (date), `amount` (number) and `status` (string); `count_check_destination` with the same columns plus `written_at` (date).
2. Import `daily-count-check.json` into an empty workflow.
3. In **Set the window**, set `alert_to` and `alert_from`; in **Send the daily message**, pick your SMTP credential.
4. Publish.

To use your own systems, replace the two Read nodes, keep `order_id` and `order_created_at`, and add the columns your destination fills in by itself to Fields to Skip Comparing.

## Why Always Output Data is on

Without it the check itself can go quiet. Two control runs showed it:

- Off on Compare Datasets, clean day: the execution succeeded, the Code node never ran, no email.
- Off on the two reads, empty day: the execution succeeded, no email.

## Tests

Tested on self-hosted n8n 2.43.6 (Docker) with a local mail catcher: mixed day, clean day, only an extra row, both sides empty, destination empty, and the two control runs above.

## License

MIT. See `LICENSE`.

---

A paid version that emails only on bad days, watched by a second workflow, is in Krafera Silent Failure Watch: https://krafera.gumroad.com/l/lltue
