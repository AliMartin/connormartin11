# Connor Martin website

Drupal portfolio and trackside journal with the custom Bootstrap5 subtheme `cjm`.

**[Rebuild and maintenance guide](docs/REBUILD.md)** covers setup, backups,
restoration, configuration, upgrade history, theme changes, and recovery.

## Project baseline

Verified 19 October 2026: Drupal **11.4.10** (core requirements `^11`), PHP 8.3,
MariaDB 11.8, Composer 2.9.5, Drush 13.8.0, DDEV 1.25.1. The document root is the
project root, **not `web/`**. DDEV project: `connormartin11`, type `drupal11`.
Local URL: https://connormartin11.ddev.site.

- Dependencies: `composer.json` and `composer.lock`; reproduce with `composer install`.
- Theme: `themes/custom/cjm/`; see its [README](themes/custom/cjm/README.md).
- Configuration: `sync/`, freshly exported with 282 configuration items.
- Public uploads: `sites/default/files/`; restore these with the matching database.
- The local site remains in maintenance mode with the custom pit-stop page.

For a complete source checkout and a matching recovery bundle, start with:

```sh
ddev start
ddev composer install
ddev import-db --file=/absolute/path/to/database.sql.gz
```

Then restore uploaded/private files, configure environment settings, and run
database/cache checks following the [rebuild guide](docs/REBUILD.md).
Do not run `site:install` on a restored database.

Current caveats: `settings.local.php` exists but its include is commented out,
so Drupal still points at the empty `sites/default/files/sync` directory rather
than the root export. The fresh export enables Upgrade Status, a Composer
development dependency. At documentation time only `.gitignore` and this README
were tracked by Git, so a clone alone cannot yet rebuild the application.
The guide explains each issue and the production implications.

## Upstream Drupal reference

The original Drupal reference information is retained below.

<img alt="Drupal Logo" src="https://www.drupal.org/files/Wordmark_blue_RGB.png" height="60px">

Drupal is an open source content management platform supporting a variety of
websites ranging from personal weblogs to large community-driven websites. For
more information, visit the Drupal website, [Drupal.org][Drupal.org], and join
the [Drupal community][Drupal community].

## Contributing

Drupal is developed on [Drupal.org][Drupal.org], the home of the international
Drupal community since 2001!

[Drupal.org][Drupal.org] hosts Drupal's [GitLab repository][GitLab repository],
its [issue queue][issue queue], and its [documentation][documentation]. Before
you start working on code, be sure to search the [issue queue][issue queue] and
create an issue if your aren't able to find an existing issue.

Every issue on Drupal.org automatically creates a new community-accessible fork
that you can contribute to. Learn more about the code contribution process on
the [Issue forks & merge requests page][issue forks].

## Usage

For a brief introduction, see [USAGE.txt](/core/USAGE.txt). You can also find
guides, API references, and more by visiting Drupal's [documentation
page][documentation].

You can quickly extend Drupal's core feature set by installing any of its
[thousands of free and open source modules][modules]. With Drupal and its
module ecosystem, you can often build most or all of what your project needs
before writing a single line of code.

## Changelog

Drupal keeps detailed [change records][changelog]. You can search Drupal's
changes for a record of every notable breaking change and new feature since
2011.

## Security

For a list of security announcements, see the [Security advisories
page][Security advisories] (available as [an RSS feed][security RSS]). This
page also describes how to subscribe to these announcements via email.

For information about the Drupal security process, or to find out how to report
a potential security issue to the Drupal security team, see the [Security team
page][security team].

## Need a helping hand?

Visit the [Support page][support] or browse [over a thousand Drupal
providers][service providers] offering design, strategy, development, and
hosting services.

## Legal matters

Know your rights when using Drupal by reading Drupal core's
[license](/core/LICENSE.txt).

Learn about the [Drupal trademark and logo policy here][trademark].

[Drupal.org]: https://www.drupal.org
[Drupal community]: https://www.drupal.org/community
[GitLab repository]: https://git.drupalcode.org/project/drupal
[issue queue]: https://www.drupal.org/project/issues/drupal
[issue forks]: https://www.drupal.org/drupalorg/docs/gitlab-integration/issue-forks-merge-requests
[documentation]: https://www.drupal.org/documentation
[changelog]: https://www.drupal.org/list-changes/drupal
[modules]: https://www.drupal.org/project/project_module
[security advisories]: https://www.drupal.org/security
[security RSS]: https://www.drupal.org/security/rss.xml
[security team]: https://www.drupal.org/drupal-security-team
[service providers]: https://www.drupal.org/drupal-services
[support]: https://www.drupal.org/support
[trademark]: https://www.drupal.com/trademark
