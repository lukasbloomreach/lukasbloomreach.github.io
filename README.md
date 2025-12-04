# Bloomreach Widget Webhook Example

A minimal, clean implementation of a Bloomreach Engagement Widget Webhook with comprehensive inline documentation.

## What is a Widget Webhook?

Widget webhooks are **custom UI components** that appear inside Bloomreach Engagement's Scenario editor. They allow you to create custom action nodes with your own interface, enabling complex integrations without users needing to understand technical details.

**Key points:**
- Widget webhooks are IFRAMEs served by your web service
- They communicate with Bloomreach using the `postMessage` API
- Users can configure them visually when building scenarios
- When a scenario runs, Bloomreach executes the configured webhook

## Quick Start

### Step 1: Host the Widget

You have several options to host the widget HTML file:

#### Option A: GitHub Pages (Recommended)

1. Create a new repository on GitHub
2. Add the `widget-webhook-example.html` file
3. Enable GitHub Pages in repository Settings → Pages
4. Your widget will be available at: `https://[username].github.io/[repo]/widget-webhook-example.html`

#### Option B: Raw GitHub Content

> ⚠️ Note: Raw GitHub URLs may have CORS limitations

1. Push the HTML file to a GitHub repository
2. Navigate to the file and click "Raw"
3. Copy the URL (looks like: `https://raw.githubusercontent.com/...`)

#### Option C: Any HTTPS Server

Host the HTML file on any server that supports HTTPS (required for IFRAMEs).

### Step 2: Configure in Bloomreach

1. In Bloomreach Engagement, go to **Data & Assets → Integrations**
2. Click **Create new Webhook preset**
3. Fill in:
   - **Name**: Give it a descriptive name (e.g., "Test Widget Webhook")
   - **Preset type**: Select **Widget Webhook**
   - **Widget URL**: Paste your hosted URL from Step 1
4. Save the preset

### Step 3: Use in a Scenario

1. Go to **Campaigns → Scenarios**
2. Create or edit a scenario
3. Add your webhook preset as an **Action Node**
4. The widget UI will appear - configure your webhook:
   - Set the URL (e.g., your webhook.site unique URL)
   - Set the HTTP method
   - Configure the request body
5. Save the scenario

### Step 4: Test

1. Go to [webhook.site](https://webhook.site) and copy your unique URL
2. Paste it in the widget's URL field
3. Use Bloomreach's "Test webhook" feature
4. Check webhook.site for incoming requests

## Communication Protocol

The widget and Bloomreach communicate via `postMessage`. Here's the flow:

```
┌─────────────────┐                      ┌─────────────────┐
│     Widget      │                      │   Bloomreach    │
│    (IFRAME)     │                      │   (Parent)      │
└────────┬────────┘                      └────────┬────────┘
         │                                        │
         │  1. widget_hello                       │
         │ ─────────────────────────────────────► │
         │                                        │
         │  2. app_hello (with saved data)        │
         │ ◄───────────────────────────────────── │
         │                                        │
         │  3. widget_initialized                 │
         │ ─────────────────────────────────────► │
         │                                        │
         │        [User edits form...]            │
         │                                        │
         │  4. app_request_state                  │
         │ ◄───────────────────────────────────── │
         │                                        │
         │  5. widget_state (form data)           │
         │ ─────────────────────────────────────► │
         │                                        │
         │  6. errors (if validation failed)      │
         │ ◄───────────────────────────────────── │
```

### Message Types

| Message | Direction | Description |
|---------|-----------|-------------|
| `widget_hello` | Widget → App | Initial handshake from widget |
| `app_hello` | App → Widget | Acknowledgment with saved data |
| `widget_initialized` | Widget → App | Widget is ready for interaction |
| `app_request_state` | App → Widget | User wants to save/test |
| `widget_state` | Widget → App | Current form configuration |
| `errors` | App → Widget | Validation errors |
| `app_reject` | App → Widget | Version not supported |

## Customization

### Changing the Bloomreach Instance

Update the `APP_ORIGIN` constant in the JavaScript:

```javascript
// Production EU
const APP_ORIGIN = 'https://app.exponea.com';

// Production US
const APP_ORIGIN = 'https://app-us1.exponea.com';

// CIS
const APP_ORIGIN = 'https://app-cis.exponea.com';
```

### Adding More Fields

To add more configuration options:

1. Add HTML form elements in the body
2. Get the element references in JavaScript
3. Read/write values in the `app_hello` and `app_request_state` handlers
4. Include the values in the `webhook` object

### Adding Authentication

```javascript
webhook: {
    // ... other fields ...
    auth: {
        type: 'basic',
        username: 'myuser',
        password: 'secret'  // Only send on save, never displayed back
    }
}
```

### Adding Custom Headers

```javascript
headers: [
    {
        name: 'Content-Type',
        value: 'application/json',
        type: 'public'  // Visible in UI
    },
    {
        name: 'Authorization',
        value: 'Bearer token123',
        type: 'secret'  // Hidden in UI, only sent on save
    }
]
```

### Using Event Properties

Track custom properties in campaign events:

```javascript
event_properties: {
    recipient: '{{ customer.email }}',
    campaign_type: 'webhook_test'
}
```

## Webhook Configuration Reference

The `webhook` object in `widget_state` supports these fields:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `url` | string | Yes | Target URL (supports Jinja2) |
| `method` | string | Yes | HTTP method: GET, POST, PUT, DELETE |
| `body` | string | No | Request body (supports Jinja2) |
| `headers` | array | No | HTTP headers |
| `response_handling` | string | Yes | `discard`, `text`, or `json` |
| `general_consent` | boolean | Yes | Use general consent |
| `consent_category` | string | No | Specific consent category |
| `auth` | object | No | Authentication config |
| `event_properties` | object | No | Properties to track |
| `frequency_policy` | string | No | Frequency policy name |

## Using Jinja2 Templates

The `url`, `body`, and `event_properties` values support Jinja2 templating:

```json
{
    "customer_id": "{{ customer.id }}",
    "email": "{{ customer.email }}",
    "first_name": "{{ customer.first_name | default('Guest') }}",
    "timestamp": "{{ time }}"
}
```

Common variables:
- `customer.*` - Customer attributes
- `event.*` - Triggering event properties (in scenarios)
- `time` - Current Unix timestamp
- `params.*` - Scenario parameters

## Troubleshooting

### Widget doesn't load

1. Ensure the URL is HTTPS
2. Check browser console for CORS errors
3. Verify the HTML file is publicly accessible

### Messages not received

1. Check `APP_ORIGIN` matches your Bloomreach instance exactly
2. Look for security errors in the browser console
3. Ensure you're testing from within Bloomreach (not standalone)

### Saved data doesn't populate

1. Check the `app_hello` handler for errors
2. Verify the saved webhook structure matches what you're reading
3. Add console.log to debug the incoming data

### Validation errors

1. Check the `errors` message in console
2. Ensure `url` is a valid HTTPS URL
3. Verify JSON body is valid
4. Check required fields are present

## Files

- `widget-webhook-example.html` - The widget webhook implementation
- `README.md` - This documentation
- `Setup of WebHook Widgets.pdf` - Original Bloomreach documentation
- `Widget webhook setup.docx` - Additional documentation

## Resources

- [Bloomreach Documentation](https://documentation.bloomreach.com/engagement/docs/documentation-overview)
- [Jinja2 in Bloomreach](https://documentation.bloomreach.com/engagement/docs/jinja)
- [Jinja2 Filters](https://documentation.bloomreach.com/engagement/docs/filters)
- [webhook.site](https://webhook.site) - Free webhook testing

