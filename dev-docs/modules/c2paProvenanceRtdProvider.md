---
layout: page_v2
title: C2PA Content Provenance RTD Module
display_name: C2PA Content Provenance RTD Module
description: Injects C2PA content provenance signals into OpenRTB bid requests, enabling DSPs and brand-safety vendors to evaluate publisher content authenticity.
page_type: module
module_type: rtd
module_code: c2paProvenanceRtdProvider
enable_download: true
vendor_specific: false
sidebarType: 1
---

# C2PA Content Provenance RTD Module
{:.no_toc}

* TOC
{:toc}

## Overview

The C2PA Content Provenance RTD module injects [C2PA](https://c2pa.org/) (Coalition for Content Provenance and Authenticity) content provenance signals into OpenRTB bid requests. C2PA is an open standard that provides a chain of provenance for digital content, allowing consumers and systems to verify where content came from and whether it has been tampered with.

For programmatic advertising, content provenance signals enable DSPs and brand-safety vendors to evaluate the authenticity of publisher content before bidding. This helps advertisers make more informed decisions about where their ads appear, strengthening brand safety and rewarding publishers who invest in content authenticity.

This module reads provenance signals that are already published by the **WordPress AI** plugin on the publisher's site and forwards them into the OpenRTB `site.ext.data.c2pa` object so that downstream buyers can consume them.

## Prerequisites

To use this module, the publisher's site must expose C2PA provenance metadata. The recommended path is:

1. Install and activate the **WordPress AI** plugin on the publisher's WordPress site.
2. Enable the **Image Provenance** (or **Content Provenance**) experiment in the plugin settings.

Once enabled, the plugin will:

- Inject a `<meta name="c2pa-manifest-url" content="...">` tag into every post and page where provenance data is available.
- Expose a REST endpoint at `/wp-json/c2pa-provenance/v1/images/manifest/{id}` that returns the signed C2PA manifest for a given image.

No additional server-side configuration is required beyond activating the experiment.

## OpenRTB Signal

The module injects provenance data into `site.ext.data.c2pa` on each bid request. The shape of the injected object is:

```json
{
  "manifest_url": "https://publisher.com/wp-json/c2pa-provenance/v1/images/manifest/42",
  "verified": true,
  "signer_tier": "local",
  "action": "c2pa.created",
  "signed_at": "2026-03-12T00:00:00Z"
}
```

{: .table .table-bordered .table-striped }
| Name           | Type      | Description                                                                                      |
|:---------------|:----------|:-------------------------------------------------------------------------------------------------|
| `manifest_url` | String    | URL of the C2PA manifest endpoint for the primary image on the page.                             |
| `verified`     | Boolean   | Whether the manifest signature was successfully verified at read time.                           |
| `signer_tier`  | String    | The tier of the signing key used to produce the manifest. See [Signer Tiers](#signer-tiers).     |
| `action`       | String    | The C2PA action claim (e.g. `c2pa.created`, `c2pa.edited`).                                     |
| `signed_at`    | String    | ISO 8601 timestamp indicating when the manifest was signed.                                      |

## Configuration

This module is configured as part of the `realTimeData.dataProviders` object.

```javascript
pbjs.setConfig({
  realTimeData: {
    dataProviders: [{
      name: 'c2paProvenance',
      params: {
        manifestUrl: 'https://example.com/wp-json/c2pa-provenance/v1/images/manifest/42' // optional override
      }
    }]
  }
});
```

{: .table .table-bordered .table-striped }
| Name                 | Scope    | Description                                                                                                           | Example                                                                        | Type     |
|:---------------------|:---------|:----------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------|:---------|
| `name`               | required | Real time data module name. Always `'c2paProvenance'`.                                                                | `'c2paProvenance'`                                                             | `String` |
| `params`             | optional | Module configuration object.                                                                                          |                                                                                | `Object` |
| `params.manifestUrl` | optional | Override URL for the C2PA manifest endpoint. By default the module reads the `<meta name="c2pa-manifest-url">` tag.   | `'https://example.com/wp-json/c2pa-provenance/v1/images/manifest/42'`          | `String` |

## Signer Tiers

The `signer_tier` field indicates how the C2PA manifest was signed. DSPs may use this value to weight the trustworthiness of the provenance claim.

{: .table .table-bordered .table-striped }
| Tier        | Description                                                                                          |
|:------------|:-----------------------------------------------------------------------------------------------------|
| `local`     | Signed by the WordPress server's own key. This is the default when no external signing service is configured. |
| `connected` | Signed via a connected C2PA signing service integrated with the publisher's infrastructure.           |
| `byok`      | Bring-your-own-key. The publisher manages their own certificate for signing manifests.                |

## Installation

This is an open-source module available in the Prebid.js package. No vendor account is required.

1. Build the Prebid.js package with the C2PA Provenance RTD module included:

    * **Option 1:** Use the Prebid [Download](https://docs.prebid.org/download.html) page to build the package. Check **C2PA Content Provenance RTD Module** under Real-Time Data Modules.

    * **Option 2:** From the command line, run `gulp build --modules=rtdModule,c2paProvenanceRtdProvider,...`

2. Configure the module using `pbjs.setConfig` as shown in the [Configuration](#configuration) section above.
