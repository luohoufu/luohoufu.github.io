---
title: Fixing Abnormal System Cluster Status in INFINI Console 1.29.0/1.29.1
description: Guide on resolving an issue where newly initialized multi-node system clusters incorrectly display an abnormal status in INFINI Console versions 1.29.0 and 1.29.1 using an update_by_query command.
slug: fix-infini-console-cluster-status
date: 2025-04-02 08:00:00+0000
image: cover.png
categories: [ Troubleshooting, DevOps ]
tags: [ INFINI Console, Easysearch ]
weight: 1
---

## Introduction

Users running **INFINI Console versions 1.29.0 and 1.29.1** might encounter a specific issue after performing a **fresh initialization** of the platform. If the underlying system Elasticsearch cluster (the one storing Console's metadata, often named `.infini_cluster` or similar) consists of **more than one node**, the Console UI might incorrectly report the system cluster's health status as abnormal (e.g., yellow or red).

This appears to be a display or status detection artifact specific to these versions under the condition of a newly initialized, multi-node system cluster. The underlying Elasticsearch cluster itself is usually healthy (green).

This post provides a straightforward workaround to correct the status displayed in the INFINI Console.

## Symptoms

*   The health status indicator for the "System Cluster" in the INFINI Console UI shows yellow or red.
*   Accompanying text might indicate an "Abnormal" or unhealthy status.
*   Checking the actual Elasticsearch system cluster's health directly (e.g., via `GET _cluster/health`) shows a `status: green`.
*   This issue is observed only on **newly initialized** deployments of versions 1.29.0 or 1.29.1 with **more than one node** in the system Elasticsearch cluster.

## Solution

The fix involves updating a specific document within the `.infini_cluster` index (or the equivalent index storing your Console's cluster configuration). This document represents the system cluster entity within the Console, and we need to manually set its health status label correctly.

You can achieve this by running the following `_update_by_query` command using Kibana Dev Tools, `curl`, or any other tool capable of sending requests to your Elasticsearch cluster.

**Command:**

```json
POST /.infini_cluster/_update_by_query?conflicts=proceed
{
  "query": {
    "term": {
      "id": {
        "value": "infini_default_system_cluster"
      }
    }
  },
  "script": {
    "source": "ctx._source.labels = ['health_status': params.status]",
    "lang": "painless",
    "params": {
      "status": "green"
    }
  }
}
```

**Explanation:**

1.  **`POST /.infini_cluster/_update_by_query?conflicts=proceed`**: Targets the `.infini_cluster` index (adjust if your system cluster index has a different name) and uses the update-by-query API. `conflicts=proceed` ensures that if the document is modified between the query and update phases, the operation skips that document instead of failing.
2.  **`query`**: This finds the specific document representing the default system cluster, identified by `id: "infini_default_system_cluster"` (verify this ID if you use a custom name).
3.  **`script`**: This section performs the update using a Painless script.
    *   **`source: "ctx._source.labels = ['health_status': params.status]"`**: **Crucially**, this line attempts to **set or overwrite** a field named `labels` with a map containing only the key `health_status` set to the value provided in `params.status`.
    *   **`lang: "painless"`**: Specifies the scripting language.
    *   **`params: { "status": "green" }`**: Passes the desired status ("green") securely into the script.

**Important Considerations Regarding the Script:**

*   **Verify Field Names:** The provided script uses `labels` and `health_status`. **Please double-check if these are the exact field names INFINI Console 1.29.x uses internally to store the health status label.** It's possible the intended fields might be `labels` and `health_status` (without the '1'). If using `labels` doesn't work, try the alternative script below which targets `labels.health_status`:

    ```json
    POST /.infini_cluster/_update_by_query?conflicts=proceed
    {
      "query": {
        "term": {
          "id": {
            "value": "infini_default_system_cluster"
          }
        }
      },
      "script": {
        "source": """
          if (ctx._source.labels == null) {
            ctx._source.labels = [:];
          }
          ctx._source.labels.health_status = params.status;
        """,
        "lang": "painless",
        "params": {
          "status": "green"
        }
      }
    }
    ```
*   **Overwriting vs. Merging:** The original script *replaces* the entire `labels` field. If `labels` might contain other important data, using the alternative script (targeting `labels.health_status`) is safer as it only adds/updates the specific key.

**Running the Command:**

*   **Kibana:** Paste the command into Kibana > Dev Tools and run it.
*   **curl:**
    ```bash
    curl -X POST "http://YOUR_ELASTICSEARCH_HOST:9200/.infini_cluster/_update_by_query?conflicts=proceed" -H 'Content-Type: application/json' -d'
    {
      "query": {
        "term": {
          "id": {
            "value": "infini_default_system_cluster"
          }
        }
      },
      "script": {
        "source": "ctx._source.labels = ['health_status': params.status]",
        "lang": "painless",
        "params": {
          "status": "green"
        }
      }
    }'
    ```
    (Replace `YOUR_ELASTICSEARCH_HOST:9200` and potentially the script content based on the verification step above).

## Verification After Fix

After successfully running the `_update_by_query` command:

1.  Wait a few moments for potential caching to expire.
2.  Refresh the INFINI Console web interface.
3.  The system cluster status indicator should now correctly display as **green** (or normal).

## Conclusion

This workaround addresses the specific cosmetic issue of an incorrect system cluster status display in INFINI Console 1.29.0 and 1.29.1 for newly initialized multi-node setups. By manually updating the relevant document using `_update_by_query`, you can restore the correct status representation in the UI. Remember to verify the exact field names (`labels.health_status` vs. `labels.health_status`) if the initial command doesn't yield the expected result. Future versions of INFINI Console are likely to contain a permanent fix for this initialization behavior.