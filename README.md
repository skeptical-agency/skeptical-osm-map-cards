# skeptical-osm-map-cards (Grok skill)

Location map cards for videos and posts from OpenStreetMap tiles in pure Python: zoom picking, tile mosaic math, and the PIL black-strip bounds guard.

## Install

This repo is the skill folder: `SKILL.md` sits at the top level. Clone it into a skills directory.

User scope (all your projects), macOS or Linux:

    mkdir -p ~/.grok/skills
    git clone https://github.com/skeptical-agency/skeptical-osm-map-cards.git ~/.grok/skills/skeptical-osm-map-cards

User scope, Windows PowerShell (assumes Grok reads skills from `$HOME\.grok\skills`):

    New-Item -ItemType Directory -Force "$HOME\.grok\skills"
    git clone https://github.com/skeptical-agency/skeptical-osm-map-cards.git "$HOME\.grok\skills\skeptical-osm-map-cards"

Project scope (shared via git):

    mkdir -p .grok/skills
    git clone https://github.com/skeptical-agency/skeptical-osm-map-cards.git .grok/skills/skeptical-osm-map-cards

No git? Download the repo as a zip from GitHub and unzip it into a folder named `skeptical-osm-map-cards` inside the skills directory.

More skills: [skeptical-agency/grok-skills](https://github.com/skeptical-agency/grok-skills).

## Requirements

- Python 3 with `Pillow` (`python3 -m pip install Pillow`) and `curl` (included on macOS, current Windows 10 and 11, and most Linux systems)

## Use

Ask Grok things like:

> make a map card for <place> from OpenStreetMap tiles in Python, 5x5 mosaic, with the black-strip bounds assert

SKILL.md has a plain-English intro, a worked example, the exact commands and the known pitfalls.

## Responsible use

Respect the Nominatim and OpenStreetMap tile usage policies (about 1 request per second, real user agent, cache tiles). Credit OpenStreetMap contributors on your card or post.

## License

MIT, see LICENSE.
