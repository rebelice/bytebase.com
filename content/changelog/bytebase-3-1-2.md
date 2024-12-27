---
title: Bytebase 3.1.2
author: Ningjing
updated_at: 2024/12/19 17:00:00
feature_image: /content/changelog/3-1-2-banner.webp
description: 'Tooltips for tables, columns, and PostgreSQL view comments in SQL Editor'
---

## 🚀 New Features

- Add tooltips for tables, columns, and PostgreSQL view comments in SQL Editor.
- Support IM and webhook integration for Lark.
- Display table and view definition for Redshift.

## 🔔 Breaking Changes

- Remove the masking policy API `v1/{instance}/{database}/policies/masking`. Use `v1/{instance}/{database}/metadata` instead and set the `columnConfigs` parameter.

## 🎄 Enhancements

- Move the masking column page to the database detail page.
- Support popup login modal on token expiration.

## 🐞 Bug fix

- Fix the backup table name conflict in multi-tenant database group within the same instance.
- Fix the cross-origin issue in SSO authentication (OIDC).
- Fix bug causing QuotaExceededError in SQL Editor.

<IncludeBlock url="/docs/get-started/install/install-upgrade"></IncludeBlock>