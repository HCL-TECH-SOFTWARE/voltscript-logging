# VoltScript Logging

Classes for managing logging in VoltScript.

## How to use

Download the repository. The structure is individual projects for specific areas, each as subdirectories of the "samples" directory (e.g. hashvse, input etc).

## Reviewing the code

To look at and run the code, open each project directory in a separate Visual Studio Code window - the VoltScript Language Support extension for Visual Studio Code uses a seti.ini at the root of the workspace. So if you open a folder at a higher level, you will get `UseVSE` errors.

The "language" samples have no dependencies on VoltScript Extensions or VoltScript Library Modules, so can be run without any additional effort.

All other samples have dependencies on VoltScript Extensions and VoltScript Library Modules. In order to compile, you will need to build the dependencies using VoltScript Build Manager.

### Building Dependencies

Dependencies can be found in two locations:

- VoltScript Library Modules are open source and can be found on [HCL's public GitHub organization](https://opensource.hcltechsw.com/volt-mx-go).
- VoltScript Extensions are closed source and hosted on the [Volt MX Marketplace](https://marketplace.demo-hclvoltmx.com/).

#### Atlas-settings.json

**NOTE:** A starter atlas-settings.json is provided in the root of this repository. After copying to the .vss directory, you will need to add your GitHub Personal Access Token and Volt MX Marketplace username and password. See further explanation below.

When you open a project that has dependencies in Visual Studio Code, VoltScript Build Manager will run setup to look for and, if necessary, create a .vss directory in your user home directory. This is where dependencies will be cached locally to be shared across projects. It is also where you need to create an atlas-settings.json file for authentication to the remote repositories on GitHub and Volt MX Marketplace.

When you create the file and edit it in Visual Studio Code, you can either create the JSON configuration manually or, by typing "atlas-settings", get access to a complete snippet including comments. Further details on the whole process can be found in the [how-to guide](https://help.hcltechsw.com/docs/voltscript/early-access/howto/writing/archipelago.html#atlas-settingsjson).

- For VoltScript Library Modules, you'll need a GitHub Personal Access Token. Follow the instructions in the [how-to guide](https://help.hcltechsw.com/docs/voltscript/early-access/howto/writing/archipelago.html#github-person-access-token).
- For VoltScript Extensions, you'll need to sign up for the Volt MX Demo Marketplace. Instructions are in the [how-to-guide](https://help.hcltechsw.com/docs/voltscript/early-access/howto/writing/archipelago.html#volt-mx-marketplace-credentials). **NOTE:** Ensure you log into the Volt MX Marketplace via a browser after confirming your account, to ensure the account is properly activated. Otherwise authentication to gain an access token will fail.

Update the atlas-settings.json with the details or, for a more advanced approach, store in environment variables. **NOTE:** If storing in environment variables, you may need to restart Visual Studio Code to pick up the environment variable changes.

**NOTE:** If using a Visual Studio Code dev container, the atlas-settings.json will be within the container. If you clone the samples repository, the same container - and thus the same atlas-settings.json - should be shared across all samples. If you just downloaded the repository, you will need to copy it into each container. It is recommended to keep a local copy of either the atlas-settings or credentials outside of the container.

#### Building dependencies

- Open the atlas.json in the relevant sample.
- Open the Command Palette (Ctrl + Shift + P on Windows, Cmd + Shift + P on Mac).
- Run the command "VoltScript: Install Dependencies". **NOTE:** This command is only available in an atlas.json or atlas-settings.json file.

Alternatively, you can click the cloud icon in the top-right of the screen on the atlas.json editor's header bar.

Dependencies should now be downloaded, project directories built, and a seti.ini and effective-atlas.json created. seti.ini is used to map UseVSE statements to the relevant VoltScript Extension. The effective-atlas.json provides detailed information of the build process, including in which repository dependencies were found.

Opening the "libs/functions.vss" file, you can now see the sample code. Some functions will be triggered from unit tests in "test" directory. In some cases, scripts will be in "src" directory for manual running.

Scripts can be run from Visual Studio Code using "VoltScript" Save & Run Script" command from the Command Palette or Ctrl + F5. Alternatively, they cn be run from the Terminal view using `VoltScript ` + path to file, for example `VoltScript src/main.vss`.

## CONTRIBUTING

The samples here map to documentation.
