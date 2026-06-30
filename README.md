# cloud-itonami-4920

Open Business Blueprint for **ISIC Rev.5 4920**: freight transport by road
(community logistics — last-mile, regional, rural freight).

This repository designs a forkable OSS business for community freight:
shipment booking, tracking, multi-modal routing, and consignment settlement —
run by a qualified operator so a local carrier keeps its own shipment and
proof-of-delivery data instead of renting a closed TMS SaaS.

## Core Contract

```text
intake + identity + shipment + route
        |
        v
Freight Advisor -> Freight Governor -> book, dispatch, or human approval
        |
        v
tracking record + route legs + consignment + proof-of-delivery + audit
```

No automated advice can dispatch a shipment with an invalid tracking number,
suppress a delivery exception, or settle a consignment without governor
approval and audit evidence.

## Capability layer

Resolves via [`kotoba-lang/industry`](https://github.com/kotoba-lang/industry)
(ISIC `4920`). Implemented by:

- [`kotoba-lang/logistics`](https://github.com/kotoba-lang/logistics) — shipment, tracking, route, consignment

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
