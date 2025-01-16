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

{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/intents" method="get" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}



{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/intents" method="post" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/intents/{intentId}" method="get" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/intents/{intentId}" method="post" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/intents/{intentId}/execute" method="post" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/intents/{intentId}/return-unsupported" method="post" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/invoices" method="get" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/invoices/{intentId}" method="get" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi.yaml" path="/withdraw" method="post" %}
[OpenApi.yaml](../.gitbook/assets/OpenApi.yaml)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi (1).yaml" path="/economy/{chain}/{tickerHash}" method="get" %}
[OpenApi (1).yaml](<../.gitbook/assets/OpenApi (1).yaml>)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi (1).yaml" path="/history/{intentId}" method="get" %}
[OpenApi (1).yaml](<../.gitbook/assets/OpenApi (1).yaml>)
{% endswagger %}

{% swagger src="../.gitbook/assets/OpenApi (1).yaml" path="/history/{intentId}/invoice-processing" method="get" %}
[OpenApi (1).yaml](<../.gitbook/assets/OpenApi (1).yaml>)
{% endswagger %}

