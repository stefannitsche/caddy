# Custom Caddy image

A custom Caddy image that adds some security modules as well as a couple DNS providers. Do not count on this list being permanent, as services get evaulated stuff might and will probably be removed. The most like things to get removed first is DNS providers.

The following DNS providers are added:

- [Cloudflare](https://github.com/caddy-dns/cloudflare)
- [Desec](https://github.com/caddy-dns/desec)
- [Dynu](https://github.com/caddy-dns/dynu)
- [Loopia DNS](https://github.com/caddy-dns/loopia)
- [Hetzner](https://github.com/caddy-dns/hetzner)

The following security modules are added:

- [fail2ban](https://github.com/Javex/caddy-fail2ban)
- [geolocation](https://github.com/porech/caddy-maxmind-geolocation)
- [Crowdsec Bouncer](https://github.com/hslatman/caddy-crowdsec-bouncer)
  - [caddy-l4](https://github.com/mholt/caddy-l4)
  - [transform-encoder](https://github.com/caddyserver/transform-encoder)
  - [http](https://github.com/hslatman/caddy-crowdsec-bouncer/http)
  - [appsec](https://github.com/hslatman/caddy-crowdsec-bouncer/appsec)
  - [layer4](https://github.com/hslatman/caddy-crowdsec-bouncer/layer4)

## DNS Providers for ACME

### Cloudflare module for Caddy

#### Configuration

Single API Token

- **API Token:** `Zone.Zone:Read` and `Zone.DNS:Edit` permissions for the domain(s) you're managing with Caddy

##### JSON Example

To use this module for the ACME DNS challenge, [configure the ACME issuer in your Caddy JSON](https://caddyserver.com/docs/json/apps/tls/automation/policies/issuers/acme/) like so:

```json
{
"module": "acme",
  "challenges": {
    "dns": {
      "provider": {
        "name": "cloudflare",
        "api_token": "{env.CF_API_TOKEN}"
      }
    }
  }
}
```

##### Caddyfile Examples

```Caddyfile
tls {
  dns cloudflare {env.CF_API_TOKEN}
}
```

You can replace the `{env.CF_*}` placeholders with the actual auth token if you prefer to put it directly in your config instead of an environment variable, however it is less secure.

### [deSEC](https://desec.io) module for Caddy

#### Config examples

To use this module for the ACME DNS challenge, [configure the ACME issuer in your Caddy JSON](https://caddyserver.com/docs/json/apps/tls/automation/policies/issuer/acme/) like so:

```json
{
  "module": "acme",
  "challenges": {
    "dns": {
      "provider": {
        "name": "desec",
        "token": "YOUR_TOKEN"
      }
    }
  }
}
```

or with the Caddyfile:

```Caddyfile
# globally
{
  acme_dns desec {
    token "YOUR_TOKEN"
  }
}
```

```Caddyfile
# one site
tls {
  dns desec {
    token "YOUR_TOKEN"
  }
}
```

### [Loopia DNS](https://www.loopia.se/api/) module for Caddy

**WARNING:** This will only properly work if you set `propagation_timeout`.
Loopia can use **up to 15 minutes**, but usually less, to propagate
the changes so a high enough timeout is needed, default is 2 minutes.
You can set the [propagation_timeout](https://caddyserver.com/docs/caddyfile/directives/tls#acme) value
to compensate for this.

#### Loopia API account requirements

You will need to [create a API user account](https://www.loopia.se/api/authentication/)
that is separate from the normal Loopia user account.

The API user will need to have access to the following methods.

- getSubdomains
- addSubdomain
- removeSubdomain
- getZoneRecords
- addZoneRecord
- removeZoneRecord

#### Loopia Config examples

This can be used together with the ACME DNS challenge.

##### Caddyfile

You can put the config in a [Caddyfile tls block](https://caddyserver.com/docs/caddyfile/directives/tls).
Note that caddyserver/caddy#4177 must be solved before `propagation_timeout` is supported in Caddyfile.

```Caddyfile
tls {
    issuer acme {
        propagation_timeout "15m0s"
        dns loopia {
                username "<your user>@loopiaapi"
                password "<your password>"
        }
    }
}
```

##### Full example Caddyfile

```Caddyfile
somehost.example.org
tls {
        issuer acme {
                email "<your email for acme notifications>"
                propagation_timeout "15m0s"
                dns loopia {
                        username "<api user>@loopiaapi"
                        password "<api password>"
                }
        }
}
respond "Hello, world!"
```

##### JSON

Using JSON notation with the [ACME module](https://caddyserver.com/docs/json/apps/tls/automation/policies/issuers/acme/).

```json
{
    "module": "acme",
    "challenges": {
        "dns": {
            "propagation_timeout": "15m0s",
            "provider": {
                "name": "loopia",
                "username": "<your user>@loopiaapi",
                "password": "<your password>"
            }
        }
    }
}
```

### [Hetzner](https://docs.hetzner.cloud/reference/cloud#dns) Module for Caddy

#### Hetzner Configuration

To use this module for the ACME DNS challenge, configure the [ACME issuer](https://caddyserver.com/docs/json/apps/tls/automation/policies/issuers/acme) in your Caddy JSON as follows:

```json
{
    "module": "acme",
    "challenges": {
        "dns": {
            "propagation_delay": "30s",
            "provider": {
                "name": "hetzner",
                "api_token": "YOUR_HETZNER_AUTH_API_TOKEN"
            }
        }
    }
}
```

Or in the Caddyfile:

```Caddyfile
your.domain.com {
    respond "Hello World"

    tls {
        dns hetzner {env.YOUR_HETZNER_AUTH_API_TOKEN}
        propagation_delay 30s
    }
}
```

The Hetzner plugin can also be configured globally:

```Caddyfile
{
    acme_dns hetzner {env.YOUR_HETZNER_AUTH_API_TOKEN}
}
```

If you prefer to put the actual auth token directly in your config instead of an environment variable, you can replace `{env.YOUR_HETZNER_AUTH_API_TOKEN}` with it.

Setting propagation_delay to 30s causes Caddy to wait 30 seconds before starting the DNS TXT record propagation checks.  This resolves an issue that was occurring due to slow DNS propagation (see issue [#11](https://github.com/caddy-dns/hetzner/issues/11) for details).

### Dynu module for Caddy

#### Caddyfile definition

```Caddyfile
dynu [<api_token>] {
    api_token <api_token>
    own_domain <own_domain>
}
```

- `api_token` may be specified as an argument to the `dynu` directive, or in the body.
- `own_domain` should be set to the root domain returned from /dns/getroot/{hostname} of the Dynu API. For example, if you have a hostname `[my_host.]my_subdomain.dynu.com`, this should be set to `my_subdomain.dynu.com` so that the correct hostname is used to update Dynu. This is needed as the zone used by Caddy DNS challenge is different from the subdomain provided by Dynu.

#### Dynu Config examples

To use this module for the ACME DNS challenge, [configure the ACME issuer in your Caddy JSON](https://caddyserver.com/docs/json/apps/tls/automation/policies/issuer/acme/) like so:

```json
{
  "module": "acme",
  "challenges": {
    "dns": {
      "provider": {
        "name": "dynu",
        "api_token": "YOUR_DYNU_API_TOKEN",
        "own_domain": "YOUR_OWN_DYNU_DOMAIN"
      }
    }
  }
}
```

or with the Caddyfile:

```Caddyfile
tls {
  dns dynu {env.DYNU_API_TOKEN} {
    own_domain {env.OWN_DYNU_DOMAIN}
  }
}
```

You can replace `{env.DYNU_API_TOKEN}` with the actual auth token if you prefer to put it directly in your config instead of an environment variable.

## Security Modules

### caddy-fail2ban

A simple package to add [fail2ban](https://github.com/fail2ban/fail2ban) support to [caddy](https://caddyserver.com/). This simple module adds a `fail2ban` HTTP matcher based on a text file of IP addresses.

Insert this into your `Caddyfile`:

```Caddyfile
@banned {
  fail2ban ./banned-ips
}
handle @banned {
  abort
}
```

The right place for it depends on your setup, but you can find more complete examples in the [examples/](https://github.com/Javex/caddy-fail2ban/tree/main/examples/) directory.

Next, you will need to create the fail2ban action. You can copy the suggested one if you like:

```bash
cp fail2ban/caddy-banfile.conf /etc/fail2ban/actions.d/caddy-banfile.conf
```

Now in any of your jails if you want to block requests at the HTTP layer, you can use the action:

```ini
action = caddy-banfile[banfile_path="/etc/caddy/banned-ips"]
```

The above path is the default so you can omit the `banfile_path` parameter if you like.

### caddy-maxmind-geolocation

Caddy v2 module to filter requests based on source IP geographic location. This was a feature provided by the V1 `ipfilter`
middleware.

#### Requirements

To be able to use this module you will need to have a Maxmind GeoLite2 database, that can be downloaded for free
by creating an account. More information about this are available on the
[official website](https://dev.maxmind.com/geoip/geoip2/geolite2/).

You will specifically need the `GeoLite2-Country.mmdb` file, or the `GeoLite2-City.mmdb` if you're matching on subdivisions and metro codes.

#### Usage

You can use this module as a matcher to blacklist or whitelist a set of countries, subdivisions or metro codes.

You'll find the detailed explanation of all the fields on the [Caddy website's plugin page](https://caddyserver.com/docs/modules/http.matchers.maxmind_geolocation).

Here are some samples:

##### Geolocation Caddyfile

1. Allow access to the website only from Italy and France:

```Caddyfile
test.example.org {
  @mygeofilter {
    maxmind_geolocation {
      db_path "/usr/share/GeoIP/GeoLite2-Country.mmdb"
      allow_countries IT FR
    }
  }

   file_server @mygeofilter {
     root /var/www/html
   }
}

```

2. Deny access to the website from Russia or from IPs with an unknown country:

```Caddyfile
test.example.org {
  @mygeofilter {
    maxmind_geolocation {
      db_path "/usr/share/GeoIP/GeoLite2-Country.mmdb"
      deny_countries RU UNK
    }
  }

   file_server @mygeofilter {
     root /var/www/html
   }
}

```

3. Allow access from US and CA, but exclude the NY subdivision (note that you'll need the City database here):

```Caddyfile
test.example.org {
  @mygeofilter {
    maxmind_geolocation {
      db_path "/usr/share/GeoIP/GeoLite2-City.mmdb"
      allow_countries US CA
      deny_subdivisions NY
    }
  }

   file_server @mygeofilter {
     root /var/www/html
   }
}

```

4. Allow access from US, but only to TX subdivision excluding the metro code 623 and the not-recognized metro codes:

```Caddyfile
test.example.org {
  @mygeofilter {
    maxmind_geolocation {
      db_path "/usr/share/GeoIP/GeoLite2-City.mmdb"
      allow_countries US
      allow_subdivisions TX
      deny_metro_codes 623 UNK
    }
  }

   file_server @mygeofilter {
     root /var/www/html
   }
}

```

5. Deny access from AS64496 (note that you'll need the ASN database here):

```Caddyfile
test.example.org {
  @mygeofilter {
    maxmind_geolocation {
      db_path "/usr/share/GeoIP/GeoLite2-ASN.mmdb"
      deny_asn 64496
    }
  }

   file_server @mygeofilter {
     root /var/www/html
   }
}

```

### CrowdSec Bouncer for Caddy

#### Description

The Caddy CrowdSec Bouncer consists of five components:

- **Caddy App**: Responsible for communicating with CrowdSec via the *Local API* and keeping track of its decisions. It supports both *StreamBouncer* (HTTP polling) and *LiveBouncer* (a request is made on every incoming connection).
- **Bouncer HTTP Handler**: Checks client IPs of incoming HTTP requests against the decisions stored by the App. Multiple independent HTTP Handlers and Connection Matchers can share the storage exposed by the App.
- **Layer 4 Connection Matcher**: Matches TCP and UDP IP addresses against the CrowdSec *Local API*. Uses the [Caddy Layer 4 app](https://github.com/mholt/caddy-l4).
- **AppSec HTTP Handler**: Communicates with an AppSec component configured on your CrowdSec deployment, seamlessly checking incoming HTTP requests against configured rulesets.
- **`caddy crowdsec` Command**: Offers useful Caddy CLI commands for your CrowdSec integration.

##### What is CrowdSec?

CrowdSec is a free and open source security automation tool that uses local logs and a set of scenarios to infer malicious intent. 
In addition to operating locally, an optional community integration is also available, through which crowd-sourced IP reputation lists are distributed.

The architecture of CrowdSec is very modular.
At its core is the CrowdSec Security Engine, which keeps track of all data and related systems.
Bouncers are pieces of software that perform specific actions based on the decisions of the Security Engine.

#### Crowdsec Module Usage

> [!TIP]
> You can find full setup examples at [`examples`](https://github.com/hslatman/caddy-crowdsec-bouncer/tree/main/examples)

You can use the bouncer by either building a custom Caddy image with Docker or by fetching the required Go modules directly into your own build.

> **Note:** You will need a recent version of **Caddy (v2.7.3+)** and **Go (1.20+)**.

##### Crowdsec Module Configuration

Configuration using a Caddyfile is supported for HTTP handlers and Layer 4 matchers.

###### Configuration Options

| Directive               | Description                                                                                                                                                         | Default                  |
|:------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------|
| `api_url`               | The URL of the CrowdSec Local API.                                                                                                                                  | `http://127.0.0.1:8080/` |
| `api_key`               | The API key to authenticate with the Local API.                                                                                                                     | `<empty>` *(required)*   |
| `disable_streaming`     | Falls back to LiveBouncer mode (queries API per request).                                                                                                           | `false`                  |
| `metrics_interval`      | Interval for pushing metrics to the Local API.                                                                                                                      | `0s` *(disabled)*        |
| `enable_caddy_metrics`  | Enables emitting bouncer metrics at Caddy's `/metrics` endpoint.                                                                                                    | `false`                  |
| `ticker_interval`       | Interval for pulling decisions from the Local API.                                                                                                                  | `60s`                    |
| `enable_hard_fails`     | Caddy fails to start if CrowdSec API is unreachable.                                                                                                                | `false`                  |
| `appsec_url`            | The URL of the CrowdSec AppSec component.                                                                                                                           | `<empty>` *(disabled)*   |
| `appsec_max_body_bytes` | Maximum request body size sent to AppSec.                                                                                                                           | `0` *(full request)*     |
| `appsec_max_timeout`.   | Maximum time for request to AppSec component.                                                                                                                       | `2s`                     |
| `appsec_fail_open`      | Ignore AppSec component connection errors.                                                                                                                          | `false`                  |
| `enable_caddy_error`    | Propagates decisions as Caddy errors to allow custom error pages. **Warning:** Ensure `handle_errors` routes are strictly static to avoid resource exhaustion (DoS).| `false`                  |

###### Variables

When a request is blocked, the HTTP and AppSec handlers record why in the request context, so that Caddyfile matchers and logs can tell blocks apart:

| Variable                           | Description                           | Examples                          |
|------------------------------------|---------------------------------------|-----------------------------------|
| `{http.vars.crowdsec.module}`      | The handler that blocked the request. | `http`, `appsec`                  |
| `{http.vars.crowdsec.remediation}` | The remediation that was applied.     | `ban`, `captcha`, `throttle`      |
| `{http.vars.crowdsec.origin}`      | The origin of the decision            | `appsec`, `crowdsec`, `lists:foo` |
| `{http.vars.crowdsec.duration}`    | The duration of the decision.         | `3h59m58s`                        |

```Caddyfile
handle_errors {
  log_append blocked-by {http.vars.crowdsec.module}
}
```

###### Example

```Caddyfile
{
  debug

  crowdsec {
    api_url http://localhost:8080
    api_key <api_key>
    ticker_interval 15s
    appsec_url http://localhost:7422
    #disable_streaming
    #enable_hard_fails
    #enable_caddy_error
  }

  layer4 {
    localhost:4444 {
      @crowdsec crowdsec
      route @crowdsec {
        proxy {
          upstream localhost:6443
        }
      }
    }
  }
}

localhost:8443 {
  route {
    crowdsec
    respond "Allowed by Bouncer!"
  }
}

localhost:7443 {
  route {
    appsec
    respond "Allowed by AppSec!"
  }
}

localhost:6443 {
  route {
    crowdsec
    appsec
    respond "Allowed by Bouncer and AppSec!"
  }
}
```

Run the Caddy server

```bash
# with a Caddyfile
caddy run --config Caddyfile 
```

#### Demo

This repository also contains an example using Docker inside the examples/demo folder.
Steps to run this demo are as follows:

```bash
# run CrowdSec container
docker compose up -d crowdsec

# add the Caddy bouncer, generating an API key
docker compose exec crowdsec cscli bouncers add caddy-bouncer

# copy and paste the API key in the ./examples/demo/config.json file

# run Caddy; at first run a custom build will be created using xcaddy
docker compose up -d caddy

# tail the logs
docker compose logs -tf
```

You can then access https://localhost:9443 and https://localhost:8443.
The latter is an example of using the [Layer 4 App](https://github.com/mholt/caddy-l4) and will simply proxy to port 9443 in this case. 

#### Utilities

When Caddy is built with this module enabled, a new `caddy crowdsec` command will be enabled.
Its subcommands allow you to interact with your CrowdSec integration at runtime using [Caddy's Admin API](https://caddyserver.com/docs/api).
This is useful to verify the status of your integration, and check if it's configured and working properly.
The command requires the Admin API to be reachable from the system it is run from.

Support for the command is currently experimental, and will be improved and extended in future releases.
Output of the commands should not be relied upon in automated processes (yet).

##### Usage

```console
$ caddy crowdsec

Commands related to the CrowdSec integration (experimental)

Usage:
  caddy crowdsec [command]

Available Commands:
  check       Checks an IP to be banned or not
  health      Checks CrowdSec integration health
  info        Shows CrowdSec runtime information
  ping        Pings the CrowdSec LAPI endpoint

Flags:
  -a, --adapter string   Name of config adapter to apply (when --config is used)
      --address string   The address to use to reach the admin API endpoint, if not the default
  -c, --config string    Configuration file to use to parse the admin address, if --address is not used
  -h, --help             help for crowdsec
  -v, --version          version for crowdsec

Use "caddy crowdsec [command] --help" for more information about a command.

Full documentation is available at:
https://caddyserver.com/docs/command-line
```

#### Client IP

When your Caddy server is deployed behind a proxy (like a CDN or load balancer), the actual client IP is masked. Starting with `v0.3.1`, this module relies on Caddy's native logic (introduced in Caddy `v2.7.0` via [caddy#5104](https://github.com/caddyserver/caddy/pull/5104)) to determine the correct client IP before checking it against CrowdSec decisions.

> **Important:** Caddy uses the `X-Forwarded-For` header by default. To trust this header, you **must** configure the [`trusted_proxies`](https://caddyserver.com/docs/json/apps/http/servers/#trusted_proxies) global directive in your Caddyfile.

You can override the default header using the [`client_ip_headers`](https://caddyserver.com/docs/json/apps/http/servers/#client_ip_headers) directive.
