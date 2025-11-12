---
description: >-
  The Everclear API contains endpoints to create new intents, track intent
  statuses, and more.
---

# API

{% hint style="success" %}
Testnet API: `https://api.testnet.everclear.org`

Mainnet API: `https://api.everclear.org`
{% endhint %}

{% openapi-operation spec="staging-updated" path="/intents" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/batched-intents" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/batched-intents/{batchId}" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/solana/intents" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/solana/create-lookup-table" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/tron/intents" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/intents/{intentId}" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/intents/{intentId}/execute" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/intents/{intentId}/return-unsupported" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/invoices" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/invoices/{intentId}" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/invoices/{intentId}/min-amounts" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/configs/assets" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/routes/quotes" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/routes/limits" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/inventory/{chain}/{asset}" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/inventory/release" method="post" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/intents" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}

{% openapi-operation spec="staging-updated" path="/history/{intentId}/invoice-processing" method="get" %}
[OpenAPI staging-updated](https://4401d86825a13bf607936cc3a9f3897a.r2.cloudflarestorage.com/gitbook-x-prod-openapi/raw/d4013e37db8295cdc75edccb4bcf12e27f33d16157c6e82cd5a93b4cec7d83c4.yaml?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=dce48141f43c0191a2ad043a6888781c%2F20251112%2Fauto%2Fs3%2Faws4_request&X-Amz-Date=20251112T164223Z&X-Amz-Expires=172800&X-Amz-Signature=22b7cafab8874c715dbc2d3621aea6b46b3b918e6ef02cc895ceb94eb8f308a4&X-Amz-SignedHeaders=host&x-amz-checksum-mode=ENABLED&x-id=GetObject)
{% endopenapi-operation %}
