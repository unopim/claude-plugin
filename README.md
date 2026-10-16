# UnoPim for Claude

Connect Claude to your own [UnoPim](https://unopim.com) product information management (PIM) install. Search products, write and translate product content, audit catalog quality, and set up attributes, families and categories in conversation. Every action runs as your UnoPim admin user and is limited by that user's role permissions.

UnoPim is open source and self-hosted, so this plugin connects to the server you run. It has no central service of its own.

## What the plugin contains

- **An MCP server connection** (`.mcp.json`) to the remote MCP endpoint of your UnoPim install, over HTTPS with OAuth sign-in.
- **Three skills** that guide Claude through common catalog work:
  - `catalog-audit`: a read-only data-quality report covering missing required values, thin content, uncategorized and disabled products.
  - `product-enrichment`: drafts names, descriptions, SEO text and translations, shows a before and after view, and saves only after you approve.
  - `catalog-structure`: plans attributes, options, groups, families and categories, and creates them only after you approve.

The plugin contains no scripts, hooks, binaries or package installs. It runs nothing on your machine.

## Requirements

- A UnoPim install reachable over **HTTPS**.
- The [`unopim/mcp`](https://github.com/unopim/unopim-mcp) package, version 1.0.5 or later, installed on that UnoPim server. It is open source under the MIT license.
- A UnoPim admin account. Claude can do only what that account's role allows.

## Set up your UnoPim server

On the UnoPim server, run:

```bash
composer require unopim/mcp
php artisan mcp:install
```

`mcp:install` sets up OAuth if it is missing and publishes `config/mcp.php`. The package registers OAuth discovery, client registration and the consent screen itself.

Serve UnoPim over HTTPS at a public address, with `APP_URL` set to it, and make sure Passport keys exist. `unopim:install` creates them; on an install that has none, run `php artisan passport:keys`.

If UnoPim runs behind Nginx with a rule that blocks dot paths, such as the stock UnoPim Docker configuration, allow `/.well-known/` through so OAuth discovery works. Add this above the `location ~ /\.` rule and reload Nginx. Apache needs no change.

```nginx
location ^~ /.well-known/ {
    try_files $uri /index.php?$query_string;
}
```

The full server guide is in the [unopim-mcp README](https://github.com/unopim/unopim-mcp#readme).

## Install and connect the plugin

Every UnoPim install has its own address, so you tell Claude where yours is. Your endpoint is your UnoPim address followed by `/mcp/unopim`, for example `https://pim.example.com/mcp/unopim`.

### Claude Code

1. Add the UnoPim plugin from the Claude directory, or run `/plugin` and find it under the directory.
2. When asked for **UnoPim MCP server URL**, enter your endpoint. From a terminal, you can install and set it in one step:

   ```bash
   claude plugin install unopim --config mcp_url=https://pim.example.com/mcp/unopim
   ```

   To change it later, run `claude plugin configure unopim` to see the current value, and set a new one with:

   ```bash
   echo '{"mcp_url": "https://pim.example.com/mcp/unopim"}' | claude plugin configure unopim --values-stdin
   ```
3. Run `/mcp`, select `unopim` and sign in. Your browser opens the UnoPim sign-in and consent screen; sign in with your UnoPim admin account and approve access.

### Claude on the web, desktop and mobile, and Cowork

These apps don't ask for plugin settings, so connect your server once as a custom connector:

1. Add the UnoPim plugin from **Customize > Plugins**. Its skills load right away.
2. Open your connector settings, choose **Add custom connector**, name it `UnoPim` and enter your endpoint as the URL.
3. Select **Connect**, sign in with your UnoPim admin account and approve access.

The skills then use the tools of that connector. If you added the plugin in one of these apps and it appears in Claude Code without a server URL, set the URL with `claude plugin configure unopim` as above, or keep using the custom connector, which Claude Code can use too. On Team and Enterprise plans, an Owner adds the custom connector for the organization, and each member connects with their own UnoPim account.

## Tools

The tools come from the `unopim/mcp` package on your server:

| Area | Tools |
| --- | --- |
| Catalog overview | `get_catalog_schema` |
| Products | `search_products`, `get_product`, `upsert_products` |
| Categories | `search_categories`, `upsert_categories` |
| Attributes | `search_attributes`, `upsert_attributes`, `search_attribute_options` |
| Attribute families and groups | `search_families`, `upsert_families`, `search_attribute_groups`, `upsert_attribute_groups` |
| Settings | `search_settings`, `upsert_settings`, `search_currencies`, `upsert_currencies` |
| Import and export jobs | `search_jobs`, `get_job_execution` |

Search and get tools only read data and are annotated as read-only. Upsert tools create or update records in batches of up to 50 and are annotated as destructive, so Claude apps can ask for your approval before running them. The plugin's skills also show you the exact changes and wait for your approval before calling any upsert tool. Job tools only read job status; the plugin cannot start imports or exports. Developer tools such as file access, database queries and log reading are off on the remote endpoint by default; the server administrator turns them on only by setting `MCP_REMOTE_DEVELOPER_TOOLS=true`, and this plugin's skills never use them.

## Data handling and privacy

- **Where data goes:** the plugin sends requests only to the UnoPim URL you enter. That server is yours. The plugin sends nothing to the UnoPim project, Webkul or any other third party, and it has no analytics or telemetry.
- **What is sent:** the tool requests Claude makes on your behalf, such as search filters, SKUs and product values to save. Responses contain catalog data from your UnoPim install, which Claude reads to answer you.
- **Sign-in:** OAuth through your UnoPim server. The plugin stores no passwords or tokens in its files; Claude stores the OAuth token that your server issues.
- **Storage and retention:** the plugin stores nothing. Your catalog data stays in your UnoPim database under your own retention rules. Conversations follow your Claude account's data settings.
- **Permissions:** every tool checks the signed-in admin's UnoPim role, and the server refuses tokens of a disabled admin.

See the UnoPim [privacy policy](https://unopim.com/privacy-policy/), [terms of service](https://unopim.com/term-of-service/) and [sub-processors](https://unopim.com/sub-processors/).

## Support

- Issues and questions: [github.com/unopim/unopim-mcp/issues](https://github.com/unopim/unopim-mcp/issues)
- UnoPim documentation: [docs.unopim.com](https://docs.unopim.com)
- Security reports: see the [security policy](https://github.com/unopim/unopim-mcp/blob/main/SECURITY.md).

## License

MIT. See [LICENSE](LICENSE).
