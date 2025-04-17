# VoltScript Logging

Classes for managing logging in VoltScript.

## How to use

Download the repository or use dependency management.

## Using dependency management

Dependency management is available in the documentation for each project, but also aggregated here:

### Authentication

You'll need a [Personal Access Token](https://help.hcltechsw.com/docs/voltscript/early-access/howto/writing/archipelago.md#github-personal-access-token) to use GitHub REST APIs. You'll then need to add this to the JSON object in your [atlas-settings.json](https://help.hcltechsw.com/docs/voltscript/early-access/howto/writing/archipelago.md#atlas-settingsjson), in the .vss directory of your user home directory:

```json
    "hcl-github": {
        "type": "github",
        "token": "YOUR_TOKEN"
    }
```

### Repository

You'll need to add to your **repositories** object in the atlas.json of your project:

```json
        {
            "id": "hcl-github",
            "type": "github",
            "url": "https://api.github.com/repos/HCL-TECH-SOFTWARE"
        }
```

### Dependency

You'll need the relevant dependency to add to your **dependencies** or **testDependencies** object in the atlas.json of your project:

```json
        {
            "library": "voltscript-logging",
            "version": "1.0.1",
            "module": "VoltScriptLogging.vss",
            "repository": "hcl-github"
        }
```

## Contributing

See [CONTRIBUTING.md](contributing.md).

## Code of Conduct

See [CODE_OF_CONDUCT.md](code_of_conduct.md).

## Issues and discussions

Let's chat on [OpenNTF Discord](https://openntf.org/discord).

For long-running discussions, use Discussions area in GitHub. For bugs and feature requests **specific to VoltScript Testing Framework** use, Issues area.