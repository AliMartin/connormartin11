# Rebuild and maintenance guide

Connor Martin website, documented **19 October 2026** after the local Drupal 11
upgrade, theme work, maintenance-page redesign, fresh export, and cleanup.
Production was not changed. Commands run from the project root unless stated
otherwise. Replace `/absolute/path/...` placeholders before running them.

## Sources and data required

A complete rebuild needs **matching application sources, database, and files**.
The configuration export cannot recreate articles, blog posts, users, uploaded
images, aliases stored as content, or custom block content referenced by UUID.

Preserve in version control or a complete source backup:

- `composer.json` and **`composer.lock`**.
- `themes/custom/cjm/`: templates, SCSS, compiled CSS, schema, images, and README.
- The entire `sync/` export, including any configuration collections.
- `.ddev/config.yaml` and intentional DDEV customizations, excluding generated
  certificates, secrets, and machine-specific files.
- Documentation and any intentional scaffold or deployment customizations.
- Future custom modules or patches. None are currently configured.

Separately preserve a matching database dump, `sites/default/files/`, private
files, environment settings/credentials, and host-specific mail/webserver/cron
configuration. Keep credentials and private data out of Git.

**Current repository limitation:** `git ls-files` listed only `.gitignore` and
`README.md` when this guide was prepared. Application sources exist locally but
are not yet tracked. Commit the required sources before relying on a clone for
recovery. The current `.gitignore` uses root-layout paths and excludes Composer
dependencies, uploads, private/temp data, and `.DS_Store` files. It intentionally
allows `sites/default/settings.local.php`; keep that file free of credentials.
The whole `.ddev/` directory is currently ignored, so preserve its configuration
separately or explicitly decide which portable DDEV files to version.

## Verified environment

| Component | Baseline |
| --- | --- |
| Drupal | 11.4.10; four root core package requirements are `^11` |
| PHP | DDEV 8.3; inspected runtime 8.3.30 |
| Database | MariaDB 11.8; inspected runtime 11.8.6 |
| Composer / Drush | 2.9.5 / 13.8.0 |
| DDEV | 1.25.1; `drupal11`; Nginx/FPM |
| Project / URL | `connormartin11` / https://connormartin11.ddev.site |
| Document root | `./`, Drupal legacy-project layout; no `web/` directory |

Composer recreates `vendor/`, `core/`, contributed extensions, and managed
libraries. Use **`composer install`** with the lock file to reproduce this
release. `composer update` resolves new versions and is not a rebuild command.

### Front-end libraries

All current `libraries/` folders are Composer-managed and excluded from Git:

| Installed folder | Composer package | Locked version |
| --- | --- | --- |
| `libraries/fitvids/` | `bower-asset/fitvids` | 1.2.0 |
| `libraries/mmenu/` | `bower-asset/mmenu` | 8.5.24 |
| `libraries/superfish/` | `npm-asset/superfish` | 1.7.10 |
| `libraries/jquery/` | Transitive `bower-asset/jquery` and `npm-asset/jquery` | Both 4.0.0 |

The existing Asset Packagist repository and installer configuration in
`composer.json` place these packages under `libraries/`. Run
`ddev composer install` to restore them; do not download or commit duplicate
copies. Responsive Menu expects the mmenu and optional Superfish paths there.
The asset-installed jQuery is separate from Drupal's `core/jquery` library.

Only these four generated folders are ignored, so a future manually maintained
library remains visible to Git. Prefer declaring future dependencies through
Composer and adding corresponding ignore rules. Keep both Composer files tracked.

The lock file includes Bootstrap5 4.0.8, Responsive Favicons 4.0.0, Responsive
Menu 4.4.5, Admin Toolbar 3.6.3, Backup and Migrate 5.1.5, Honeypot 2.2.2,
Manage Display 3.0.2, Pathauto 1.15.0, Token 1.17.0, and Ctools 4.1.1.
Upgrade Status 4.3.10 and PHPStan are development dependencies. The official
Symfony Runtime Composer plugin is allowed in this project's Composer config,
with explicit user approval recorded during the upgrade.

## Take a current recovery bundle

Create a new backup directory **outside the document root**, then run, adapting
the paths to the actual active storage configuration:

```sh
ddev export-db --file=/absolute/path/recovery-bundle/database.sql.gz
tar -czf /absolute/path/recovery-bundle/site-files.tar.gz sites/default/files private-files
tar -czf /absolute/path/recovery-bundle/source-and-config.tar.gz composer.json composer.lock themes/custom sync .ddev/config.yaml
```

The file archive assumes private storage is `private-files/`; confirm the active
setting and adapt it if different. Preserve environment settings securely and
record the source commit and runtime versions. Copy the bundle to durable
storage and test restoration. A scheduled-backup checkbox is not evidence of a
working backup. Historical backups listed below are **not current Drupal 11
recovery bundles**.

## Restore an existing site locally

1. Restore the matching complete sources. Keep the root-document layout. Install
   Docker and DDEV and start Docker.
2. Create `sites/default/settings.php` if absent, using the matching Drupal
   scaffold's `default.settings.php` as a starting point. DDEV manages local
   database settings through `settings.ddev.php`; do not copy production database
   credentials into the local environment.
3. Start the environment and install the locked packages, including development
   dependencies required by the current configuration:

   ```sh
   ddev start
   ddev composer install
   ddev import-db --file=/absolute/path/recovery-bundle/database.sql.gz
   ```

4. Restore files. With the archive format shown above, extract from the project
   root:

   ```sh
   tar -xzf /absolute/path/recovery-bundle/site-files.tar.gz
   ```

   That archive contains `sites/default/files/` and `private-files/`, not just
   public-file contents at its top level. Inspect another archive's structure
   before choosing its extraction destination. Verify PHP can write required
   upload/temp directories.
5. Configure the environment paths described below, then run:

   ```sh
   ddev drush updatedb -y
   ddev drush cache:rebuild
   ddev drush status
   ddev drush updatedb:status
   ```

6. A matching database restore may already contain all required configuration.
   If an import is intended, review its changes interactively before accepting:

   ```sh
   ddev drush config:import --source=/var/www/html/sync
   ddev drush cache:rebuild
   ```

   A dump from another site may have a different site UUID. Resolve a source/
   backup mismatch rather than changing UUIDs blindly.
7. Complete the verification checklist below. Keep mail pointed at a local test
   mailbox and avoid production integrations during testing.

**Do not run `site:install` on the restored database.** A fresh installation from
existing configuration into an empty database is a different workflow and will
not restore content, including the custom LinkedIn block referenced by UUID.

## Settings and the configuration-directory discrepancy

The existing `sites/default/settings.local.php` sets:

```php
$settings['update_free_access'] = FALSE;
$settings['file_private_path'] = DRUPAL_ROOT . '/private-files';
$settings['file_temp_path'] = DRUPAL_ROOT . '/tmp';
$settings['config_sync_directory'] = DRUPAL_ROOT . '/sync';
```

It also allows the `connormartin.co.uk` and `connormartin11.ddev.site` domains
and their subdomains as trusted hosts. Adapt the allowlist for a different
project name or staging domain.

**The include at the end of `settings.php` is commented out, so these overrides
are not active.** Runtime verification during documentation still showed:

- Config sync: `sites/default/files/sync`, containing zero configuration items.
- Private path: unset; explicit temporary-path setting: unset.
- Trusted hosts: DDEV's permissive `.*` pattern.

To apply the intended overrides during a rebuild, enable this block at the end
of `settings.php`, after the conditional DDEV settings include:

```php
if (file_exists($app_root . '/' . $site_path . '/settings.local.php')) {
  include $app_root . '/' . $site_path . '/settings.local.php';
}
```

Rebuild caches and verify `ddev drush status --fields=config-sync` points to the
root `sync/` folder. Verify private/temp directories exist and are writable.
Writing this guide did not change the include or runtime settings.

On production, prefer private and temporary directories outside the document
root. Existing `.htaccess` protection in these directories works with Apache,
not Nginx; configure server restrictions or relocate them. Apply production host
patterns and prevent HTTP access to configuration, credentials, and backups.

Until paths are corrected, use explicit destinations/sources:

```sh
ddev drush config:export --destination=/var/www/html/sync -y
ddev drush config:import --source=/var/www/html/sync
```

The user requested an empty-and-refresh export on 19 October 2026. All **282**
root export items matched Drupal's exportable active configuration. This fresh
export enables `upgrade_status` in `core.extension` and includes its settings;
it supersedes earlier selective exports that omitted local module enablement.

## Work completed

### Drupal 10 to Drupal 11

- Appraised Drupal 10.6.19 and installed/enabled Upgrade Status 4.3.10 as a
  development dependency.
- Upgraded Responsive Favicons 2.0.2 to 4.0.0 (`^4.0`).
- Switched Responsive Menu 5.0.0-alpha2 to stable 4.4.5 (`^4.4`), which supports
  Drupal 11. Preserved existing settings and stored schema version `95002`.
  No uninstall or schema-version reset was performed.
- Changed all four root `drupal/core-*` requirements from `^10.3` to `^11`.
  Included Drush in resolution to update its code generator to 4.2.0 alongside
  Symfony 7.4; the first narrower update failed and was reverted by Composer.
- Finished installation after approving the Symfony Runtime plugin. Applied all
  database and post-update hooks and rebuilt caches.
- Refreshed migrated CKEditor 5, Views, asset compression, and other exports;
  removed obsolete `field.settings` and `system.rss` exports.
- Changed DDEV's project type to `drupal11` and restarted it. Result: **11.4.10**.

Do not repeat upgrade commands when restoring the completed release. Use the
matching database and `composer install` from its lock file.

### `cjm` theme and maintenance page

- Preserved the existing Drupal 10/11 declaration, normal site templates,
  regions, CSS, and Bootstrap5 library integration. Stable9 remains the base
  theme inherited through Bootstrap5 and must remain installed for this chain.
- Added `config/schema/cjm.schema.yml`, extending core `theme_settings` with
  Bootstrap5 and saved subtheme-generator settings. Normalized four integer
  flags to booleans in active configuration and the export.
- Removed unused CKEditor 4 metadata and retained CKEditor 5 styling.
- Added `templates/layout/maintenance-page.html.twig`, the `cjm/maintenance`
  library, and separately authored `css/maintenance.css`.
- Created the dark blue “A quick pit stop” design using the site's Goodwood
  Alfa Romeo photo, with no portrait of Connor. The copied theme image is
  `images/maintenance-goodwood.jpeg`, so it survives without uploaded-file paths.
- Updated the editable Drupal maintenance message and its export. Reviewed
  desktop and phone layouts in Chrome and corrected mobile image sizing.

See [the theme README](../themes/custom/cjm/README.md). For the normal site/editor
CSS, install Sass and run from `themes/custom/cjm/`:

```sh
sass scss/style.scss css/style.css
sass scss/ck5style.scss css/ck5style.css
```

The Sass compiler version was not pinned. Keep the compiled CSS for exact
restoration and review regenerated output. Maintenance CSS is edited directly,
not generated by these commands.

### Cleanup

Removed temporary favicon processing files, temporary analysis artifacts, 46
`.DS_Store` files, and the unused custom `style-52.css` and its source map.
Archived the two original upgrade reports outside the installation. Preserved
`tmp/.htaccess`, current CSS/maps, uploads, settings, dependencies, and recovery
backups. This guide consolidates the rebuild information from those reports.

## Production deployment considerations

The fresh export enables Upgrade Status but Composer lists it under `require-dev`.
A `composer install --no-dev` build will omit its code and cannot boot a database
that still enables it. Decide on a production configuration policy first. For
example, uninstall it on a release/staging copy while its code is installed,
export/review that copy's configuration, then build without development packages.
Keep local monitoring enabled if useful; do not change the working site's
configuration just to prepare a production artifact.

Review `.cpanel.yml` before using it: the current wildcard copy omits dotfiles
such as `.htaccess`, retains obsolete target files, and has no database-update,
maintenance, or rollback procedure. It was not used for this upgrade. Deploy a
tested artifact, include required scaffold files, apply database updates and
reviewed imports, rebuild caches, and retain matching recovery data.

## Verification and maintenance mode

```sh
ddev composer validate --no-check-publish
ddev composer check-platform-reqs
ddev composer audit --locked
ddev drush status
ddev drush updatedb:status
ddev drush core:requirements
```

After upgrading: bootstrap passed, database/entity updates were complete, theme
schema validation and Twig compilation passed, homepage/article/blog/contact/
login returned HTTP 200, CSS/JS loaded, and unknown URLs returned 404. Content
counts remained 3 articles and 6 blog entries. Composer reported no vulnerability
advisories at the time; it still flagged the abandoned installers extender.

The site remains in maintenance mode. The redesigned page returns HTTP **503**;
its image and stylesheet return HTTP 200. Record the state before temporarily
viewing normal pages:

```sh
ddev drush state:get system.maintenance_mode
ddev drush state:set system.maintenance_mode 0 --input-format=integer
ddev drush cache:rebuild
```

If it was previously enabled, restore it afterward:

```sh
ddev drush state:set system.maintenance_mode 1 --input-format=integer
ddev drush cache:rebuild
```

The message is exported in `system.maintenance`; the on/off switch is **state**,
not exported configuration. It must be restored separately or from the database.

Before production, test authenticated editing, CKEditor 5 styling/image upload,
mail delivery, role permissions, sidebars, mobile navigation, and the signed-in
toolbar. Those interactions were not covered by the HTTP checks. The maintenance
page did receive browser review. The existing responsive-menu mobile toggle
block is disabled in configuration and was left unchanged.

Upgrade Status lives at `/admin/reports/upgrade-status`. On Drupal 11 it checks
the next major version, **Drupal 12**: the theme's Drupal 10/11 version declaration
warning therefore does not mean it is incompatible with Drupal 11. The installed
scanner also emitted theme `$origin` warnings on 11.4; review tool updates
separately. Do not claim Drupal 12 support without testing it.

## Outstanding work

- Activate/review local settings, verify private storage, and test backups. The
  earlier scheduled-backup error came from an unset private path; merely creating
  `settings.local.php` has not fixed it while the include is commented out.
- Track the application sources; keep the root-layout ignore rules aligned with
  future dependencies and environment files.
- Plan replacements for deprecated Contact, Field Layout, History, and Stable9
  before a later major upgrade; they still run on Drupal 11.
- Review production settings permissions/hosts and Drupal 12's future HTML5
  validation default. These were status warnings, not failed Drupal 11 migrations.
- Replace abandoned `oomphinc/composer-installers-extender` in a separate tested change.

## Historical recovery material

| Location | Contents / limitation |
| --- | --- |
| `/tmp/connormartin11-pre-core11/` | Pre-upgrade **Drupal 10.6.19** database, files, Composer files, settings, config archive, and logs; not a current Drupal 11 backup. |
| `/tmp/connormartin11-before-responsive-d11.sql.gz` | Database before the responsive module changes. |
| `/tmp/connormartin11-responsive-d11-baseline/` | Original Composer files and menu source. |
| System temp `connormartin11-sync-backup-92hgstf0/` | Export before the fresh full export. |
| System temp `connormartin11-cleanup-hc6n5_iq/` | `working-files.tar.gz` (old reports/favicon work) and `old-theme-css.tar.gz`. |

On this machine the last two folders were created beneath
`/private/var/folders/00/6hp39qcj74541hc6gmt5vncr0000gn/T/`.
Temporary storage may be purged: copy needed material to durable storage and
create a current matching Drupal 11 recovery bundle. Major-version rollback
requires matching code, database, configuration, and files; Composer downgrading
alone does not undo database migrations.
