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



{% swagger src="../.gitbook/assets/api-spec.yaml" path="/intents" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/intents" method="post" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/intents/{intentId}" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/intents/{intentId}" method="post" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/intents/{intentId}/execute" method="post" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/intents/{intentId}/return-unsupported" method="post" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/invoices" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/invoices/{intentId}" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/invoices/{intentId}/min-amounts" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/withdraw" method="post" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/configs/assets" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/metrics/liquidity-flow" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/metrics/clearing-volume" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/economy/{chain}/{tickerHash}" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/history/{intentId}" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/api-spec.yaml" path="/history/{intentId}/invoice-processing" method="get" %}
[api-spec.yaml](../.gitbook/assets/api-spec.yaml)
{% endswagger %}
