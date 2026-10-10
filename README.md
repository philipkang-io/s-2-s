# spec-to-spechub-demo

Prototype for the Autodesk PWS ask: spec changes committed to GitHub flow into Postman Spec Hub
**without pushing anything back to Git**, with an alert in Postman and a review step.

Two modes, picked with the repo variable `SPEC_MODE` (`auto` or `gate`; default `gate`).

## Auto mode (the flow Autodesk chose)

```
spec commit -> diff vs Staging spec (oasdiff) -> Working updated -> announcement in Staging -> Pull Changes -> Update Collection
```

1. Engineering commits a spec change to `main`.
2. The Action diffs it against the **Staging** spec, so the diff is what the reviewing team does not have yet.
3. The Action updates the **Working** workspace (spec and collection) with no approval.
4. The Action posts an announcement to the **Staging** workspace Updates tab (3 changes, 0 breaking, how to review). Anyone watching the Staging workspace gets a bell entry and an email from Postman. The poster does not. No GitHub comment or email is sent.
5. The team opens the Staging spec, runs **Pull Changes**, reviews the diff, and accepts.
6. In Spec Hub the team clicks **Update Collection**, which regenerates the Staging collection from the merged spec. It shows no diff for the collection, so the spec diff is the review.

Staging only changes when the team pulls. Working always matches `main`.

## Gate mode (alternative)

```
spec commit -> diff vs Spec Hub (oasdiff) -> alert (commit comment) -> approve -> update Spec Hub -> regenerate collection
```

Nothing changes in Postman until a reviewer approves the `spechub-approval` GitHub environment.

## Setup

- Repo secret `POSTMAN_API_KEY`; optional `SLACK_WEBHOOK_URL`.
- Environments: `spechub-auto` (no reviewers) and `spechub-approval` (required reviewer, gate mode).
- Workflow `env` IDs: `SPEC_ID` and `COLLECTION_UID` point at the Working workspace `[philip]spec-to-spechub-demo`. `STAGING_SPEC_ID` and `STAGING_WORKSPACE_ID` point at `[philip]spec-to-spechub-staging`.
- Staging, for auto mode:
  - the spec is a **fork** of the Working spec (auto-pull unchecked);
  - the collection is **generated from the Staging spec**, not a fork. A forked collection makes Spec Hub offer Generate Collection instead of Update Collection;
  - reviewers have access to the workspace and **watch** it (eye icon) to get the notification.
- The diff uses the pinned oasdiff release binary, not the Docker Hub image. Runners hit Docker Hub's anonymous pull limit, and the workflow fails loudly if the tool errors.

## Demo (about 5 minutes, auto mode)

1. **Baseline:** Working and Staging spec and collection match `main` (4 requests each).
2. **Engineering commits** a non-breaking change (new optional field plus a new `GET`).
3. **Alert:** a watcher of Staging gets the bell and email. Open the announcement in the Updates tab.
4. **Review:** Staging spec, Pull Changes, accept. Then Update Collection. The collection has 5 requests, and the examples show the new field.

## Replaying Act 2

`demo/act2-spec-change.patch` is the engineering change: adds `overdraftLimit` to the Balance schema **and** to the response examples for `GET /accounts/{accountId}` and `GET /accounts/{accountId}/balance`, plus a new `GET /accounts/{accountId}/statements`. The examples matter: the collection's saved responses are built from spec examples, so a schema-only change would not show up in the collection.

```
git apply demo/act2-spec-change.patch && git commit -am "feat: add statements endpoint" && git push
```

## Reset (auto mode)

Do the Postman side first, then the repo, so the reset commit finds no diff and posts nothing.

1. **Working:** PATCH the baseline spec into the Working spec, re-sync the collection, then delete the `List account statements` request. Sync never removes endpoints.
2. **Staging:** PATCH the baseline spec into the Staging spec, delete the Staging collection, and generate a new `Accounts API (generated)` from the Staging spec. The Staging spec stays a fork.
3. **Repo:** commit the baseline `specs/accounts.yaml` to `main`. The run's `publish` job is skipped.

Old announcements stay in the Staging Updates tab. Delete them in the app for a clean list.

## Limits

- Spec is OpenAPI 3.0.3. Collection sync only supports OpenAPI 3.0.
- Sync does not remove deleted endpoints.
- Postman does not alert a fork when its original changes, so the pipeline posts the announcement. A native upstream-changed alert is the product ask.
