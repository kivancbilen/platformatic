# Authorization

## Introduction

<!--

TODO:

- What authorization is in the context of Platformatic DB
- Authentication is handled by a third-party service (+ examples)
- Authorization strategies
- User Roles & Metadata
- Rules

-->

<!-- TODO: Rewrite introduction -->

Authorization in Platformatic DB is **role-based**. Platformatic delegate
authentication and assignment of roles to external authentication services.
The job of the authentication service is to authenticate users and assign
their roles correctly.

## Bypass authorization in development

<!-- TODO: Update HTTP Headers link -->

To make testing and developing easier, it's possible to bypass auhtorization checks
if an `adminSecret` is set. See [HTTP Headers](#http-headers).
