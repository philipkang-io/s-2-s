# spec-to-spechub-demo

Prototype for the Autodesk PWS ask: spec changes committed to GitHub flow into Postman Spec Hub
**without pushing anything back to Git**, with an alert and a review step first.

```
spec commit -> diff vs Spec Hub (oasdiff) -> alert -> approve -> update Spec Hub -> regenerate collection
```

- Spec: OpenAPI 3.0.3 (`specs/accounts.yaml`). Collection sync only supports OpenAPI 3.0.
- Spec Hub is the source of truth. The "approved" spec is whatever is in Spec Hub now.
- Approval = the `spechub-approval` GitHub environment (required reviewer).

## Demo (about 5 minutes)
1. **Baseline:** Spec Hub spec and collection match `main`.
2. **Engineering commits** a non-breaking change (new optional field plus a new `GET`). The alert shows exactly what changed.
3. **Approve:** Spec Hub updates and the collection regenerates. Show the new request.

## Setup
- Repo secret `POSTMAN_API_KEY`; optional `SLACK_WEBHOOK_URL`.
- Environment `spechub-approval` with a required reviewer.
- `SPEC_ID` and `COLLECTION_UID` in the workflow point at the Postman workspace `[philip]spec-to-spechub-demo`.

## Replaying Act 2
`demo/act2-spec-change.patch` is the engineering change: adds `overdraftLimit` to the Balance schema **and** to the response examples for `GET /accounts/{accountId}` and `GET /accounts/{accountId}/balance`, plus a new `GET /accounts/{accountId}/statements`. The examples matter: the collection's saved responses are built from spec examples, so a schema-only change would not show up in the collection.
```
git apply demo/act2-spec-change.patch && git commit -am "feat: add statements endpoint" && git push
```
Then approve the `spechub-approval` environment on the run. To reset: PATCH the baseline spec back into Spec Hub, re-sync the collection, and revert the spec in `main`.
