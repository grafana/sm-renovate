# Renovate presets

These are [configuration presets](https://docs.renovatebot.com/config-presets/) with useful configurations shared across the Grafana org.

You can use them by referencing them in your renovate config file as such:

```json
  "extends": [
    "config:best-practices",
    "github>grafana/sm-renovate//presets/grafana.json5",
  ]
```

Feel free to contribute presets that you feel can be useful for others, and not too specific for a single repository. If you are creating a preset with opinionated settings for your team, name it after such. Consider adding yourself to the `CODEOWNERS` file for presets you create or want to step up and maintain.

When changing presets, be mindful of breaking changes. If those are unavoidable consider [searching](https://github.com/search?q=org%3Agrafana+%22sm-renovate%2Fpresets%22&type=code) for repositories using the preset and messaging their maintainers to notify about the change.
