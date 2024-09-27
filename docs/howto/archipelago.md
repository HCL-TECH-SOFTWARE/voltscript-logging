# Use dependency management

!!!info
    For generic how-to information about VoltScript Dependency Management, see [VoltScript documentation](https://help.hcltechsw.com/docs/voltscript/early-access/howto/writing/archipelago.html){: target="_blank" rel="noopener noreferrer"}.

Dependency management is available in the documentation for each project, but also aggregated here:

## Authentication

You'll need a [Personal Access Token](https://help.hcltechsw.com/docs/voltscript/early-access/howto/writing/archipelago.md#github-personal-access-token) to use GitHub REST APIs. You'll then need to add the following to the JSON object in your [atlas-settings.json](https://help.hcltechsw.com/docs/voltscript/early-access/howto/writing/archipelago.md#atlas-settingsjson) in the `.vss` directory of your user home directory:

```json
    "hcl-github": {
        "type": "github",
        "token": "${env.TOKEN}"
    }
```

## Repository

You'll need to add the following to your **repositories** object in the `atlas.json` of your project:

```json
        {
            "id": "hcl-github",
            "type": "github",
            "url": "https://api.github.com/repos/HCL-TECH-SOFTWARE"
        }
```

## Dependency

You'll need to add the following relevant dependency to your **dependencies** object in the `atlas.json` of your project:

```json
        {
            "library": "voltscript-logging",
            "version": "1.0.0",
            "module": "VoltScriptLogging.vss",
            "repository": "hcl-github"
        }
```