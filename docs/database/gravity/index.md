Pi-hole uses the well-known relational database management system SQLite3 for managing the various domains that are used to control the DNS filtering system. The database-based domain management has been added with Pi-hole v5.0.

## Domain tables (`domainlist`)
The database stores white-, and blacklists which are directly relevant for Pi-hole's domain blocking behavior. The `domainlist` table contains all domains on the white- and blacklists. It has a few extra fields to store data related to a given domain such as the `enabled` state, the dates when the domain were added and when it has last been modified, and an optional comment.

The date fields are defined as `INTEGER` fields as they expect numerical timestamps also known as *UNIX time*. The `date_added` and `date_modified` fields are initialized with the current timestamp converted to UNIX time. The `comment` field is optional and can be empty.

Pi-hole's *FTL*DNS reads the tables through the various view, omitting any disabled domains.

Label | Type | Uniqueness enforced | Content
----- | ---- | ------------------- | --------
`id` | integer | Yes | Unique ID for database operations
`type` | integer | No | `0` = exact whitelist,<br> `1` = exact blacklist,<br> `2` = regex whitelist,<br> `3` = regex whitelist
`domain` | text | Yes | Domain
`enabled` | boolean | No | Flag whether domain should be used by `pihole-FTL`<br>(`0` = disabled, `1` = enabled)
`date_added` | integer | No | Timestamp when domain was added
`date_modified` | integer | No | Timestamp when domain was last modified, automatically updated when a record is changed
`comment` | text | No | Optional field for arbitrary user comments, only field that is allowed to be `NULL`

## Adlist Table (`adlist`)
The `adlist` table contains all sources for domains to be collected by `pihole -g`. Just like the other tables, it has a few extra fields to store meta data related to a given source.

Label | Type | Uniqueness enforced | Content
----- | ---- | ------------------- | --------
`id` | integer | Yes | Unique ID for database operations
`address` | text | Yes | The URL of the list
`enabled` | boolean | No | Flag whether domain should be used by `pihole-FTL`<br>(`0` = disabled, `1` = enabled)
`date_added` | integer | No | Timestamp when domain was added
`date_modified` | integer | No | Timestamp when domain was last modified, automatically updated when a record is changed
`comment` | text | No | Optional field for arbitrary user comments

## Gravity Table (`gravity`)
The `gravity` table consists of the domains that have been processed by Pi-hole's `gravity` (`pihole -g`) command. The domain in this list are the collection of domains sourced from the configured sources (see the [`adlist` table](lists.md#adlist-table-adlist).

During each run of `pihole -g`, this table is flushed and completely rebuilt from the newly obtained set of domains to be blocked.

Label | Type | Content
----- | ---- | -------
`domain` | text | Blocked domain compiled from adlist referenced by `adlist_id`
`adlist_id` | integer | ID associated to adlists in table `adlist`

Uniqueness is enforced on pairs of (`domain`, `adlist_id`). In other words: domains can be added multiple times, however, only when they are referencing different adlists as their origins.

## Client table (`client`)

Clients are identified by their IP addresses. Each client automatically gets a unique identifier (`id`).

Label | Type | Content
----- | ---- | -------
`id` | integer | Client ID (autoincrementing)
`ip` | text | IP address of the client (IPv4 or IPv6)

## Audit Table (`domain_audit`)
The `domain_audit` table contains domains that have been audited by the user on the web interface.

Label | Type | Uniqueness enforced | Content
----- | ---- | ------------------- | --------
`id` | integer | Yes | Unique ID for database operations
`domain` | text | Yes | Domain
`date_added` | integer | No | Unix timestamp when domain was added

{!abbreviations.md!}