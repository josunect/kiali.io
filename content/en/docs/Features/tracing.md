---
title: "Tracing"
date: 2018-06-20T19:04:38+02:00
draft: false
weight: 4
---

## Traces

Kiali has now a Span duration legend item on Service Metrics tab, when enabled, correlates Span and Metrics on the same chart.

<a class="image-popup-fit-height" href="/images/documentation/features/traces-metrics-v1.22.0.png" title="Metrics and Trace Spans">
    <img src="/images/documentation/features/traces-metrics-thumb-v1.22.0.png" style="display: block; margin: 0 auto;" />
</a>

User can navigate to the traces tab to browse filtered traces for a given service in the time interval or to show details for a single trace.

The tracing toolbar offers some control over the data to fetch, to facilitate the user experience. In the tracing view, as shown in the image below, it’s possible to select the traces interval, results limit, status code, errors, adjust time (expand results on time), last Xm traffic (Traces from last minutes) and refresh interval.

After selecting a trace, Kiali shows the information related to that trace like number of spans, spans grouped by operation name, duration, date.

<div style="display: flex;">
    <span style="margin: 0 auto;">
      <a class="image-popup-fit-height" href="/images/documentation/features/traces-view-v1.22.0.png" title="Traces Timeline for a Service">
          <img src="/images/documentation/features/traces-view-thumb-v1.22.0.png" style="width: 660px;display:inline;margin: 0 auto;" />
      </a>
      <a class="image-popup-fit-height" href="/images/documentation/features/traces-view-details-v1.22.0.png" title="Trace Details">
          <img src="/images/documentation/features/traces-view-details-thumb-v1.22.0.png" style="width: 660px;display:inline;margin: 0 auto;" />
      </a>
    </span>
</div>

## Jaeger Configuration

### In cluster URL

In order to fetch data from Jaeger, Kiali needs to get an URL that can be resolved from inside the cluster, typically using Kubernetes DNS. This is the `in_cluster_url` configuration. For instance, for a Jaeger service named `tracing` within `istio-system` namespace, Kiali config would be:

```yaml
  external_services:
    tracing:
      in_cluster_url: 'http://tracing.istio-system/jaeger'
```

{{% alert color="warning" %}}
If you use the Kiali operator (recommended), this config can be set in the Kiali CR. But in most cases, the Kiali operator will set a valid default `in_cluster_url` so you wouldn't have to change anything. If you don't use the Kiali operator, this config can be set in Kiali config map.
{{% /alert %}}

### External URL

Configuring an external URL for Jaeger will enable links from Kiali to Jaeger UI. This URL needs to be accessible from the browser (it's used for links generation). Example:

```yaml
  external_services:
    tracing:
      in_cluster_url: 'http://tracing.istio-system/jaeger'
      url: 'http://my-jaeger-host/jaeger'
```

Once this URL is set, Kiali will show an additional item to the main menu:

![Distributed Tracing View](/images/documentation/tracing/menu_external_link.png)

{{% alert color="warning" %}}
You may have `url` configured and not `in_cluster_url`, for instance, if Jaeger is not accessible from Kiali pod. In this situation, Kiali will not show its own traces chart but will display external links to the Jaeger UI instead.
{{% /alert %}}

### Other configuration

For advanced configuration on Jaeger integration, please refer to [the Kiali CR 'external_services.tracing' section](https://github.com/kiali/kiali-operator/blob/master/deploy/kiali/kiali_cr.yaml). It is relevant for config map as well, if you don't use the Kiali operator.
