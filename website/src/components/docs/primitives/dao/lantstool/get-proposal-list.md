import { TryOutOnLantstool } from "@site/src/components/lantstool/TryOutOnLantstool";


<TryOutOnLantstool path="docs/2.build/5.primitives/dao/query-exist-proposals.json" />

<details>
<summary>Example response</summary>

```bash
[
  {
    "id": 9262,
    "proposer": "pasternag.near",
    "description": "NEAR, a top non-EVM blockchain, has gone live on Router’s Testnet Mandara. With Router Nitro, our flagship dApp, users in the NEAR ecosystem can now transfer test tokens to and from NEAR onto other supported chains. $$$$https://twitter.com/routerprotocol/status/1727732303491961232",
    "kind": {
      "Transfer": {
        "token_id": "",
        "receiver_id": "pasternag.near",
        "amount": "500000000000000000000000",
        "msg": null
      }
    },
    "status": "Approved",
    "vote_counts": {
      "council": [1, 0, 0]
    },
    "votes": {
      "brzk-93444.near": "Approve"
    },
    "submission_time": "1700828277659425683"
  },
  {
    "id": 9263,
    "proposer": "fittedn.near",
    "description": "How to deploy BOS component$$$$https://twitter.com/BitkubAcademy/status/1728003163318563025?t=PiN6pwS380T1N4JuQXSONA&s=19",
    "kind": {
      "Transfer": {
        "token_id": "",
        "receiver_id": "fittedn.near",
        "amount": "500000000000000000000000",
        "msg": null
      }
    },
    "status": "Expired",
    "vote_counts": {
      "Whitelisted Members": [2, 0, 0]
    },
    "votes": {
      "trendheo.near": "Approve",
      "vikash.near": "Approve"
    },
    "submission_time": "1700832601849419123"
  }
]
```

</details>

