# regsrv

A modern, fast, and lightweight WHOIS/RDAP client for Node.js.

`regsrv` queries domain information using the modern **RDAP (Registration Data Access Protocol)**, the official successor to the legacy WHOIS protocol. It automatically finds the correct authoritative server for any given TLD based on IANA's official list and returns a clean, standardized JSON object.

## Features

- **Modern Protocol:** Uses JSON-based RDAP, not legacy text-based WHOIS.
- **Authoritative Servers:** Automatically finds the correct RDAP server for over 1200+ TLDs.
- **Standardized Output:** Returns a consistent, predictable object structure for all lookups.
- **Lightweight:** Zero runtime dependencies. Uses native Node.js `fetch`.
- **URL & Domain Parsing:** Accepts a full URL and automatically extracts the domain.
- **TypeScript Ready:** Fully written in TypeScript with type definitions included.

## Installation

```bash
npm install regsrv
```

Requires Node.js v18.0.0 or higher.

## Usage

```javascript
import whois from "regsrv";

async function getDomainInfo() {
  try {
    const data = await whois("eib.org");
    console.log(JSON.stringify(data, null, 2));
  } catch (error) {
    console.error(error);
  }
}

getDomainInfo();
```

### Example Output

A successful lookup for `eib.org` will return an object like this:

```json
{
  "domainName": "eib.org",
  "createdAt": "1997-09-15T04:00:00Z",
  "updatedAt": "2019-09-09T15:39:04Z",
  "expiresAt": "2028-09-14T04:00:00Z",
  "registrar": {
    "name": "MarkMonitor Inc.",
    "ianaId": "292"
  },
  "registrant": {
    "organization": "Clean Oceans"
  },
  "nameservers": ["ns1.eib.org", "ns2.eib.org"],
  "status": [
    "client delete prohibited",
    "client transfer prohibited",
    "client update prohibited",
    "server delete prohibited",
    "server transfer prohibited",
    "server update prohibited"
  ],
  "raw": {
    "...": "The full, raw RDAP response from the server"
  }
}
```

## More Usage Examples

### Error Handling

```javascript
import whois from "regsrv";

async function safeLookup(domain) {
  try {
    const data = await whois(domain);
    console.log("Domain info:", data);
  } catch (err) {
    if (err.message.includes("Unsupported TLD")) {
      console.error("This TLD is not supported.");
    } else if (err.message.includes("Domain not found")) {
      console.error("Domain does not exist.");
    } else {
      console.error("Unexpected error:", err);
    }
  }
}

safeLookup("example.com");
safeLookup("invalid.tld");
```

### Using with Promise.then

```javascript
import whois from "regsrv";

whois("github.com")
  .then((data) => {
    console.log("Got data:", data);
  })
  .catch((err) => {
    console.error("Error:", err);
  });
```

### Using in Parallel (Promise.all)

```javascript
import whois from "regsrv";

const domains = ["eib.org", "npmjs.com", "github.com"];
Promise.all(domains.map(whois))
  .then((results) => {
    results.forEach((data) => console.log(data.domainName, data));
  })
  .catch((err) => {
    console.error("One of the lookups failed:", err);
  });
```

---

## CLI Usage

You can also use the CLI:

```sh
npx regsrv eib.org
```

---

## Updating the TLD/RDAP Data

The list of TLDs and their authoritative RDAP servers is based on the official IANA bootstrap file. To update this data (for example, when new TLDs are added):

1. Run the update script:

   ```bash
   node scripts/tlds.mjs
   ```

2. This will fetch the latest RDAP bootstrap from IANA and update `src/data/dns.json`.

3. Commit the updated file to keep your package up to date.

---

## TypeScript Types

The main lookup function returns a strongly-typed object. Here are the types:

```typescript
export interface WhoisContact {
  name?: string;
  organization?: string;
  email?: string;
}

export interface WhoisData {
  domainName: string;
  // Dates in ISO 8601 format (YYYY-MM-DDTHH:mm:ssZ)
  createdAt?: string;
  updatedAt?: string;
  expiresAt?: string;

  registrar: {
    name?: string;
    ianaId?: string;
    url?: string;
  };

  registrant?: WhoisContact;

  nameservers: string[];

  status: string[];

  // The raw RDAP response for advanced use cases
  raw: any;
}
```

## API

### `whois(domainOrUrl: string): Promise<WhoisData>`

Looks up domain information.

- **`domainOrUrl`**: The string to look up. Can be a simple domain (`github.com`) or a full URL (`https://www.`).
- **Returns**: A `Promise` that resolves to a `WhoisData` object.
- **Throws**: An `Error` if the TLD is unsupported, the domain is not found (404), or a network error occurs.

## License

[MIT](LICENSE)
