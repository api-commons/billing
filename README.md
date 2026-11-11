# Billing and Invoicing

The common billing vocabulary -- invoices and their lines, subscriptions, plans, prices,
credit notes, coupons and metered usage -- distilled from the APIs that already exist. Part
of [Schema Commons](https://github.com/api-commons/schema-commons), published by the
[API Commons](https://apicommons.org).

Built from the providers tagged Billing, Invoicing, E-Invoicing, Subscription (in its
variants), Recurring, Usage-Based Billing or Pricing, Accounts Receivable, API Monetization
or Payments in [APIs.io](https://apis.io) that publish their own OpenAPI. Only harvested
contracts count -- nothing API Evangelist generated. Every number below is a count of
providers.

## What the market already agrees on

<!-- commons:summary -->
| Concept | Expose it | Readable schema | Core properties (share) | Collection named |
| --- | ---: | ---: | --- | --- |
| invoice | 184 | 162 | id 78%, status 70%, due_date 54%, number 51% | invoices 146, invoice 43, bills 21 |
| invoice_line | 16 | 14 | description 86%, id 71%, quantity 71% | line-items 5, lines 5, lineitems 3, items 3 |
| subscription | 138 | 124 | id 82%, status 64%, created_at 53% | subscriptions 125, subscription 22 |
| plan | 59 | 54 | id 91%, name 85%, description 65%, created_at 61%, is_active 52%, amount 50% | plans 42, products 32, product 5, plan 4 |
| price | 17 | 15 | id 60%, plan_id 60%, currency 53%, pricing_model 53% | prices 13, price 3, pricing 2, item_prices 1 |
| credit_note | 29 | 25 | id 72%, total 68%, status 64%, description 60%, currency 56%, created_at 52%, customer_id 52%, updated_at 52% | credit-notes 11, creditnotes 6, credit_notes 5, creditmemos 4 |
| coupon | 27 | 19 | code 90%, id 68%, created_at 58%, max_redemptions 58% | coupons 16, discounts 12, vouchers 2, coupon 2 |
| usage_record | 22 | 19 | id 58%, quantity 58% | usage 15, usages 4, events 1, usage_records 1 |

Cohort: 837 providers with harvested OpenAPI, 270 crosswalks. Of the providers exposing a invoice: 66% create, 84% list, 85% fetch, 45% update, 29% delete. Generated 2026-11-10 by build.py.
<!-- /commons:summary -->

Shares are of the providers with a readable resource schema for that concept. The
extended tier and the long tail of every concept are in [profile.yml](profile.yml).

## What it does not agree on

As in commerce, the money is where the market diverges. An invoice's `status` (70%),
`due_date` (54%) and `number` (51%) are core, but its `total` sits at 41%: providers split
totals into subtotal, tax, amount due and amount paid in different ways.
A price's `amount` is in only 20% of price resources, because tiered, volume and
usage-based pricing replace a single amount with a `pricing_model` and its tiers -- the
model, not the amount, is what price resources share. And only a third of subscriptions
reference their plan directly; the rest carry it inside their items.

## Artifacts

- [openapi.yml](openapi.yml) -- OpenAPI 3.1 of the shared operations. No one serves it; it
  is the shape to map to. Lints clean against the
  [naming ruleset](https://github.com/api-commons/schema-commons/blob/main/ruleset/schema-commons-naming.yaml).
- [schema/](schema) -- JSON Schema 2020-12 per concept; each property carries
  `x-commons-tier`, `x-commons-share`, `x-commons-providers` and the spellings seen.
- [crosswalk/](crosswalk) -- one per provider, common name -> their name, with the harvested
  spec and operations each mapping was read from. Stripe and Chargebee map fully; Zuora's
  credit notes are `creditmemos`, Paddle's and Polar's plans are `products`.
- [adopters.yml](adopters.yml) -- who exposes which concept, linked to their APIs.io page.
- [concepts.yml](concepts.yml) -- the only hand-written input: concepts, path segments with
  their qualifiers, aliases, reviewed exclusions, merged duplicates and shared specs.
- [census/](census) -- the cohort and the raw census the profile is built from.

## Decisions

These are calls, not measurements, recorded so they can be revisited as the method matures.

- **Healthcare billing, utility metering, price data and invoice financing are out.**
  Medical Billing and Revenue Cycle Management are claims, Smart Metering is utility meters,
  Pricing is price-data feeds, Invoice Financing is lending.
- **Payments is in the cohort.** Processors bill too -- Stripe Billing, PayPal and Square
  invoicing -- and a processor's invoices and subscriptions are billing. Plans, prices,
  coupons and usage `require: [subscription]`, so a processor without subscriptions adds
  nothing to them.
- **A plan is `/plans`, though some providers say `/products`.** Plans lead (44 providers to
  36). A billing product (Paddle, Polar, Creem, Recharge, Zuora) counts as a plan; a commerce
  catalog, bank account product or menu item does not, and is excluded by name.
- **`/rates` is not a price.** It was FX, tax or hourly rates at every provider reviewed.
- **A bare `/events` is not usage.** Reviewed, it was a webhook-notification feed almost
  everywhere (Chargebee, Paddle, GoCardless, Coinbase), and `/subscriptions/{id}/events` is a
  subscription's lifecycle. Events count only beneath a usage or metering path. This costs
  Lago, whose usage genuinely is `POST /events`; the path cannot tell it apart.
- **Invoice lines are `/invoices/{invoice_id}/lines`.** `lines` and `line-items` tie (5 each);
  `lines` is Stripe's and the shorter.
- **Known gaps, and why.** Three billing platforms are absent, each for a different reason
  the method is right to respect: Recurly's specs are in the catalog but their provenance was
  never recorded, so they cannot be confirmed as Recurly's own; Orb's contracts were authored
  by API Evangelist, which never counts; Maxio publishes none we hold. Recording Recurly's
  provenance would bring it in.

## Rebuild

```bash
../schema-commons/scripts/run.sh .
```

## License

Artifacts CC BY-NC-SA 4.0 ([LICENSE](LICENSE)); code Apache 2.0 ([LICENSE-CODE](LICENSE-CODE)).
