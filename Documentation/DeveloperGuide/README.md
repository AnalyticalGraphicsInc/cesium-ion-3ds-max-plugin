# Development

3ds Max runs only on Windows, therefore the following documentation is for Windows only.

## Project Overview

The project consists of two parts:

- A .NET Core project which handles all network interaction.
- MaxScripts which integrate the .NET project and create the user interface

This guide covers how to run maxScript and build .NET in Visual Studio Code.
Template task.json and launch.json files are included in the repository.

## Prerequisites

### Requirements for building, running and debugging of the .NET code

These are not required, if only MAXScript files are changed.

- **.NET Core SDK**: Follow [this link](https://dotnet.microsoft.com/download) select **Download .NET Core SDK** and follow the instructions.
- **AWS SDK for .NET** is already included in the project file via NuGet. Therefore, building the project should automatically download the AWS SDK. If the Amazon.* namespaces are unavailable nevertheless, follow [this link](https://aws.amazon.com/sdk-for-net/) select **Download MSI Installer** and install the AWS Tools for Windows.

### Visual Studio Code

1. Clone the repository.
2. Open the repository as workspace in VS Code.
3. In VS Code install the *Language MaxScript* extension (*ctrl + shift + x*).
4. [Optional] Set a shortcut to run maxScripts by going to *File->Preferences->Keyboard Shortcuts* and set a shortcut for *Tasks: Run Task*.

## Using the plugin

### Add plugin to 3ds Max

Create the Windows Environment Variable `ADSK_APPLICATION_PLUGINS` and set it to your repository.
The plugin should now be loaded on start-up.

To manually run the plugin:

1. Start 3ds Max (to run scripts from VS Code 3ds Max has to be open)
2. Open *PluginPackage/PreStartupScripts/cesiumPlugin.ms*.
3. Run it with the Task **Execute Script in 3ds Max**.
4. Open *PluginPackage/Widgets/nameRequiredWidget.ms* and run it. This creates a warning popup.
5. Open *PluginPackage/Widgets/mainWidget.ms* and run it. This creates the Exporter popup window.
6. Next open *PluginPackage/PostStartupScripts/addMenus.ms* and run it. This will add the menu item in 3ds Max under *File->Export*.

Running these files in a different order will create an error in 3ds Max.

### Updating GUI

To update the popup simply rerun the `.ms` MaxScript file which creates it (for example `mainWidget.ms`).

### Delete Old GUIs

When you close and reopen 3ds Max it can happen that the previously created export menu item will get lost. In that case it will still appear in there but with the text `Missing: exportButton'mxs docs` and without any functionality. To delete it open *Customize->Customize User Interface*.

![Customize User Interface](../resetUI.png)
<p align="center">
    The Customize User Interface Dialog
</p>

Open the *Menus* tab and delete it in the panel on the right under *File->File-Export* by selecting it and pressing *delete* on your keyboard or reset all menus by pressing the *Reset* button. Afterwards repeat the steps to run the plugin.

### Updating .NET

Press *ctrl + shift + b*. This builds the project for **Release** and places the binaries in the right folder (./PluginPackage/C#/).

## Debugging

The following guide helps in debugging the scripts.

### MaxScript

Code can be debugged using the `print "debug message"` command, or by creating a breakpoint using `break()`. in the code.

The later one opens the [MAXScript Debugger](http://help.autodesk.com/view/3DSMAX/2020/ENU/?guid=GUID-E04AB16E-D5C8-4B00-81A6-E3945E97A1EB).

![MAXScript Debugger](../debugger.png)
<p align="center">
    The MAXScript Debugger
</p>

While in the debugger enter `?` as command to see a list of available commands.

The MAXScript Debugger and the [MAXScript Listener](http://help.autodesk.com/view/3DSMAX/2020/ENU/?guid=GUID-C8019A8A-207F-48A0-985E-18D47FAD8F36) can also be opened via the *Scripting* menu in 3ds Max.

![MAXScript Listener](../listener.png)
<p align="center">
    The MAXScript Listener
</p>

The MAXScript Listener shows errors and can be used to run maxScript snippets (similar to a python console). The content of a variable can be displayed by typing the name and pressing *Enter*.

### .NET

1. Open *launch.json* and change the arguments in brackets `<>` with the correct path. The first argument is the function name, the others are the function parameters. Instead of using arguments you can also change the main function in *server.cs* to do the desired task. But don't forget to change it back afterwards.
2. Go to the Debug Panel (*ctrl + shift + d*) and run in Debug Mode (*F5*).

## Release Guide

### Create the release package

1. Pull down the latest master branch: `git pull origin master`.
2. Modify `PluginPackage/PackageContents.xml` and increment the minor version only:
   - `"AppVersion="1.0.0"` becomes `AppVersion="1.1.0"`
3. Proofread and update CHANGES.md to capture any changes since last release.
4. Commit and push these changes directly to master.
5. Make sure the repository is clean `git clean -xdf`. __This will delete all files not already in the repository.__
6. Pack the content of the *PluginPackage* folder and name it `io-cesium-ion-vx.x.x.zip` (where x.x.x will be the version).

### Testing

1. Reset the User Interface in 3ds Max as described in [Delete old menus](#delete-old-menus).
2. Close 3ds Max.
3. Delete the token file at `%LOCALAPPDATA%/Autodesk/3dsMax/\<ReleaseNumber> - 64bit/ENU/plugcfg_ln/cesiumIonToken`. *ReleaseNumber* is usually the year of the release like 2020.
4. Delete all files named `cesiumion\<number>.fbx` and `progress\<number>.log` in `%LOCALAPPDATA%/Autodesk/3dsMax/\<ReleaseNumber> - 64bit/ENU/temp/`
5. Unpack the created .zip file to `%ALLUSERSPROFILE%\Autodesk\ApplicationPlugins\` (e.g. `C:\ProgramData\Autodesk\ApplicationPlugins`) or `%APPDATA%\Autodesk\ApplicationPlugins\` (e.g. `C:\Users\<username>\AppData\Roaming\Autodesk\ApplicationPlugins`).
    - The location of `PackageContents.xml` must be `....\ApplicationPlugins\io-cesium-ion-vx.x.x\PackageContents.xml`. If this is not the case after unpacking, pack the release again accordingly.
6. Rename your repository or delete the path to your repository in the `ADSK_APPLICATION_PLUGINS` environment variable.
7. Start 3ds Max and open any model you would like to test.
8. Use the plugin to export the model to Cesium ion. Be sure to try all the model type options.
9. Try to force errors by interrupting the internet connection while uploading etc.
10. After testing remove the previously copied `PluginPackage` folder and restore your repository name or `ADSK_APPLICATION_PLUGINS` environment variable.

### Release on Github

1. Test the plugin.
2. Create and push a tag, e.g.,
   - `git tag -a 1.1 -m '1.1 release'`
   - `git push origin 1.1` (do not use `git push --tags`)
3. Publish the release zip file to GitHub
   - [Create new release](https://github.com/AnalyticalGraphicsInc/cesium-ion-3ds-max-plugin/releases/new).
   - Select the tag you use pushed
   - Enter `Cesium ion 3ds Max 1.x` for the title
   - In the description, include the date, list of highlights and permalink to CHANGES.md, which is in the format https://github.com/AnalyticalGraphicsInc/cesium-ion-3ds-max-plugin/blob/1.xx/CHANGES.md, where 1.xx is the version number.
   - Attach the `io-cesium-ion-vx.x.x.zip` you generated during the build process.
   - Publish the release
4. Tell the outreach team about the new release to have it included in the monthly release announcements/blog post and on social media.
5. Update cesium.com with a link to the latest release zip.

### Release on Autodesk App Store

1. Complete the steps for release to github.
2. Use VS Code Markdown PDF extension to create a PDF of the main [`README.md`](../../README.md).
3. Add the generated `README.pdf` to the zip file.
4. Go to the app page on Autodesk App Store using [this link](https://apps.autodesk.com/en/MyUploads/DetailPageForPublisher?appId=3653390948844719757&appLang=en&os=Win64). You will need to sign in using the `devops@cesium.com` account (login details in LastPass).
5. Under **Actions** click **Edit**.
6. Upload the new zip file under **App File** (remove any previous files).
7. Update the version number.
8. Update any text or screenshots as needed.
9. Click continue to go to the next page, which contains information about pricing, versions and tags. Usually nothing to update here. Click continue again.
10. Review the summary page. Click **Preview** to see how the app page will look on the app store. Go back and edit anything if needed, otherwise click continue. This will take you back to the page in step 4.
11. On the left hand side, click the **Submit** button for send it to Autodesk for review and creating the installer.
