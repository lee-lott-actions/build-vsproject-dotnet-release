# Build Visual Studio .Net Project Release Action

This GitHub Action builds a Visual Studio .NET project, packages the build artifacts into a zip file, and creates a GitHub release with the specified tag and release notes. It is designed to automate the process of building and releasing .NET projects, with support for both standard and pre-release versioning.

## Features
- Builds a Visual Studio .NET project (e.g., .csproj, .vbproj) using the `dotnet publish` command.
- Packages the build artifacts into a zip file.
- Creates a GitHub release with automated release notes and optional file uploads.
- Supports pre-release versioning with a custom suffix.
- Adds a job summary with release details.
- Includes the release date and time in Central Daylight Time (CDT).

## Inputs
| Name                | Description                                                                 | Required |
|---------------------|-----------------------------------------------------------------------------|----------|
| `project-file-path` | The path to the Visual Studio .NET project file (e.g., .csproj, .vbproj) to be built. | Yes      |
| `prerelease`        | Mark the release as a pre-release (`true`/`false`).                          | Yes      |
| `prerelease-suffix` | The suffix to append to the version number for pre-releases. For non-pre-releases, the version is based on the latest pre-release with this suffix. | Yes      |
| `package-name`      | The base name for the zip file containing the build artifacts.               | Yes      |
| `configuration`     | The build configuration to use for the .NET project (e.g., Debug, Release).  | Yes      |
| `runtime`           | The target runtime identifier for the .NET project build (e.g., win-x64, linux-x64). | Yes      |
| `token`             | The GitHub token used for authentication to create the release.              | Yes      |

## Outputs
| Name              | Description                                                          |
|-------------------|----------------------------------------------------------------------|
| `version-tag`     | The new tag created for the release (e.g., `v1.0.0`).                 |
| `version-number`  | The new tag with the "v" prefix removed (e.g., `1.0.0`).              |
| `zip-file-name`   | The name of the zip file containing the build artifacts, including the `.zip` extension (e.g., `my-package.1.0.0.zip`). |
| `zip-file-path`   | The full path to the zip file containing the build objects.           |
| `build-status`    | The build/release outcome: Release Created, No Release, or Release Failed. |

## Usage
**Important**: 
- This action must be run on a runner that supports the `dotnet` CLI (e.g., `windows-latest`, `ubuntu-latest`) and must be triggered by a pull request that has been closed and merged.
- The `actions/setup-dotnet` action must be called before this action to set up the .NET environment. Ensure the required .NET SDK version is specified.
- If your project relies on external NuGet sources, you must configure them (e.g., using `dotnet nuget add source`) before running this action.

To use this action in your GitHub workflow, include it as a step in your workflow file. Below is an example workflow that sets up the .NET environment, configures a NuGet source (if needed), and uses this action to build and release a .NET project.

### Example Workflow
```yaml
name: Build and Release .NET Project

on:
  pull_request:
    types: [closed]

jobs:
  build-and-release:
    if: github.event.pull_request.merged == true
    name: Create Release
    runs-on: windows-latest
    steps:
      - name: Checkout repository
        uses: actions/checkout@v6

      - name: Setup .NET
        uses: actions/setup-dotnet@v5
        with:
          dotnet-version: '10.0.x' # Specify the required .NET SDK version

      - name: Add NuGet Source (if needed)
        run: dotnet nuget add source --username ${{ github.repository_owner }} --password ${{ env.GITHUB_TOKEN }} --store-password-in-clear-text --name nuget.org "https://api.nuget.org/v3/index.json"
        shell: pwsh

      - name: Build .NET Project and Create Release
        uses: lee-lott-actions/build-vsproject-net-release@v1
        with:
          project-file-path: 'path/to/your/project.csproj'
          prerelease: 'false'
          prerelease-suffix: 'beta'
          package-name: 'my-net-package'
          configuration: 'Release'
          runtime: 'win-x64'
          token: ${{ secrets.GITHUB_TOKEN }}
```

---

## Notes

- The action will only proceed if the PR is merged.  
- Tags and releases follow [semantic versioning](https://semver.org/) and support pre-release naming.
- A summary of the release, including the release tag and URL, is added to the job summary.
- For pre-releases, an annotated tag is created but a GitHub Release is not published.
- For standard releases, an annotated tag is created and a GitHub Release is published with release notes and timing information.

---
