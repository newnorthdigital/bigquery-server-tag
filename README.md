# BigQuery Event - New North Digital

A server-side Google Tag Manager (sGTM) tag template that streams events to BigQuery in a GA4-shaped schema. Built by [New North Digital](https://newnorth.nl).

It takes the incoming sGTM event, builds a row close to the GA4 BigQuery export shape (event params, user properties, device, geo, traffic attribution, privacy/consent), and writes it with `BigQuery.insert()`.

## Configuration

| Field | Purpose |
|---|---|
| BigQuery Project ID | Destination project |
| BigQuery Dataset ID | Destination dataset |
| BigQuery Table ID | Destination table |
| User ID | Optional, written to `user_id` |
| Event Name | The event name written to the row |
| Event Parameters | Include all auto params or none; exclude or add/override specific keys |
| User Properties | Include all or none; exclude or add/override |
| Device Properties | Include auto-detected or none; exclude or add/override |
| Geo Properties | Include auto-detected or none; exclude or add/override |
| Exclude params with a null or undefined value | Drops empty params |
| Log to console for debugging | Verbose logging in preview |

## How it writes

The tag calls `gtmOnSuccess()` **before** `BigQuery.insert()`, so the container response returns in ~50ms and the insert runs in the background. Trade-off: a failed insert does not fail the client request. The insert retries once on failure to recover transient BigQuery errors.

### Cloud Run note (important)

Because the insert runs after the response is sent, the Cloud Run service must keep CPU allocated outside the request, otherwise background inserts can stall or drop at low traffic. Deploy the sGTM service with CPU always allocated:

```
gcloud run services update <service> --region <region> --no-cpu-throttling
```

## Permissions

- `logToConsole` (all environments, so insert failures are visible in production logs)
- `read_event_data` (any)
- `access_bigquery` (write, `*/*/*`)

## Changelog

### Unreleased
- Log insert failures in all environments (previously debug only), so production failures are no longer silent.
- `event_timestamp` now prefers the client time (`x-ga-timestamp_millis`) and falls back to server receive time.
- Single retry on transient BigQuery insert failures.
- `skipInvalidRows: true` so one malformed row cannot fail the whole insert.
- Skip null/undefined event params instead of writing null-valued params.
- Performance: read fields from the cached `getAllEventData()` snapshot with a `getEventData` fallback; compute device category and parse the page URL once per event instead of twice; removed a dead `streamIdOverride` reference.

## License

Apache-2.0. See [LICENSE](./LICENSE).
