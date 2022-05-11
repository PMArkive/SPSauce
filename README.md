SPSauce
======

This is intended as dependency loader to help build-automation tools build sourcemod plugins with a lot of dependencies.
It will not help you set up a sourcemod server!

Why might this be necessary? There are a lot of consecutive libraries like smlib, morecolors, stocksoup, tf2items and many more
that can quickly increase the number of dependencies. Packing these libraries into your own repository might not always be 
permitted by the projects license. On the other hand packing old versions of libraries that may be buggy is also not a good 
idea and additionally convolutes your repository with dependency files.

On the other hand lot of plugins and libraries are hosted on the allied modders forums, where they are most accessible
to users, but buildtools usually have a hard time accessing resources from the forums. This tool fetches attachments
from plugin topics and patch comments and tries to place them into the correct directory within the build cache.

How it works
-----

SPsauce pulls an instance of sourcemod and all declared dependencies into a cache directory within the project.
The compiler will be located in ./spcache/addons/sourcemod/scripting/ and dependencies will be placed around it.
A built in `spcomp`-Task automatically add some common include directories, and can run in paralell with other compiles.

**Archives are best structured with an addons/sourcemod directory no more than 1 directory deep.** This allows the tool to 
easily find your project root. A single subdirectory is automatically generated by GitHub when downloading a zip or 
tarball or a repository branch, but more subdirectories should work as well.

Additionally you can call other applications from withing the build-script, if the provided functionallity is not enough.
Keep in mind that all file targets are limited to subdirectories wherever possible.

Execution is done in three phases: Parse, dependency resolution, task execution. The second phase  will already 

**On Authentication:** I know Git Hub PATs are a bit iffy, but without the API allows for only [60 requests per hour](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#rate-limiting) and that's abysmal! If you can't use
`clone` or want to use spsauce with another build system, you can always create a read only PAT (without scopes).

Build Script
-----
There are the platform prefixes, @windows, @linux and @mac that allow you to perform actions for a single platform.
Lines with that start with // or # are comments. Spaces are trimmed from each line, so you can indent as you please.

### Commands
| Name                                 | Arguments                                                 | Description                                                                                                                                                                                                                                                                                                                                      |
|--------------------------------------|-----------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| auth `ARGS`                          | ['try'] 'github' \<PAT²> [login²]                         | Log in to the GitHub API with a personal access token. The login parameter is for organisations and increasing the request limit even more. GitHub auth is required for `dependency github` only. The try keyword will prevent the script from failing if authentication failed. This might be useful if you have mutual exclusive auth sources. |
| sourcemod `ARGS`                     | \<branch> (\<build>&#124;['latest'])                      | Download the specified sourcemod branch, with the given build number or the latest build                                                                                                                                                                                                                                                         |
| dependency `ARGS`                    | ('am'&#124;'forum'&#124;'forums') \<plugin thread id>     | Download the files from a plugin posted on the forums. Directories will be guessed by file extension                                                                                                                                                                                                                                             |
| dependency `ARGS`                    | ('am'&#124;'forum'&#124;'forums') 'patch' \<post id>      | Download the files from a single forum post id. Directories will be guessed again, and the file has to be replaced                                                                                                                                                                                                                               |
| dependency `ARGS`                    | 'github' \<project slug> \<tag name> [archive name]       | Looks up the release tag in the repo. Will download the sources if no other archive was specified                                                                                                                                                                                                                                                |
| dependency `ARGS`                    | 'github' \<project slug> 'latest'                         | Download the archive for the main branch                                                                                                                                                                                                                                                                                                         |
| dependency `ARGS`                    | 'github' \<project slug> \<branch name>'-SNAPSHOT'        | Download the archive for the specified branch. The -SNAPSHOT suffix for the branch name is just to distinguish between release tags                                                                                                                                                                                                              |
| dependency `ARGS`                    | 'limetech' \<project id> (\<version>&#124;\<build>)       | Download a version or build of one of asherkins plugins hosted on limetech.org. Take the project id from the url in the version list                                                                                                                                                                                                             |
| dependency `ARGS`                    | 'raw' \<url>                                              | Has to point to an archive or known file type to be placed into the cache folder                                                                                                                                                                                                                                                                 |
| clone `ARGS`                         | \<url> [branch²] 'into' \<dir²>                           | Requires git to be installed, clones the specified branch into specified directory within spcache                                                                                                                                                                                                                                                |
| compilepool `ARGS`                   | \<size>                                                   | When encountering `spcomp` tasks, collect all consecutive `spcomp` tasks and execute the specified amount of tasks at the same time. Defaults to the amount of CPUs you have                                                                                                                                                                     |
| spcomp `ARGS`                        | ...²                                                      | Args are the same as for the sp compiler. Automatically adds the include directories spcache/addons/sourcemod/scripting/include/; addons/sourcemod/scripting/include/; scripting/include/; include/                                                                                                                                              |
| exec `ARGS`                          | \<commandline²>                                           | Run a command, script cancelles on exit value != 0                                                                                                                                                                                                                                                                                               |
| echo `ARGS`                          | \<message²>                                               | Write a message to std out during dependency resolution phase                                                                                                                                                                                                                                                                                    |
| echo! `ARGS`                         | \<message²>                                               | Write a message to std out during task execution phase                                                                                                                                                                                                                                                                                           |
| die `ARGS`                           | \<message²>                                               | Writes a message to std out during dependency resolution phase and exits with error-level                                                                                                                                                                                                                                                        |
| die! `ARGS`                          | \<message²>                                               | Writes a message to std out during task execution phase and exits with error-level                                                                                                                                                                                                                                                               |
| mkdir `ARGS`                         | \<path²>                                                  | Creates a directory within cwd                                                                                                                                                                                                                                                                                                                   |
| delete/erase/remove `ARGS`           | \<path²>                                                  | Delete a file or directory recursively within cwd                                                                                                                                                                                                                                                                                                |
| move `ARGS`                          | \<from²> ':' \<to²> ['replace' \<mode>]                   | Move a file or directory within cwd.<br>Use `mode` to specify how to handle duplicate files. Possible modes are `All`, `Older`, `Skip` and `Error`(default).                                                                                                                                                                                     |
| copy `ARGS`                          | \<from²> ':' \<to²> ['replace' \<mode>]                   | Copy a file or directory within cwd (like move)                                                                                                                                                                                                                                                                                                  |
| set `ARGS`                           | \<variable> 'to' \<value²>                                | Sets a environment variable or 'argument' variable to the specified value                                                                                                                                                                                                                                                                        |
| set `ARGS`                           | \<variable> ['as' \<format>] from \<file²> \<regex>       | Sets a environment variable or 'argument' variable to a value read from a file. The file is read whole and matched against the regex (Java Regex, MULTILINE is set by default). Format supports capture groups \0 through \9. As this command is more complex, you may quote arguments. Quotes are escaped with double quotes.                   |
| script `ARGS`<br>...<br>end script   | 'lua'                                                     | Execute a block of lua code to manipulate variables. These sandboxes are quite limited and do not get os/io access!                                                                                                                                                                                                                              |
| with files<br>...<br>:release `ARGS` | 'github' \<owner²/repo²>[@branch²] \<tag²>                | Create a new Release on GitHub (or append if the release already exists) with the files listed in the lines between `with files` and `:release ...`                                                                                                                                                                                              |
| with files<br>...<br>:release `ARGS` | ('am'&#124;'forum'&#124;'forums') \<threadid> \<version²> | Update the attachments on an AlliedMods forum thread with the files listed in the lines between `with files` and `:release ...`. Only unpacked files are remove (sp,inc,smx,so,dll,cfg,txt).                                                                                                                                                     |
| with files<br>...<br>:release `ARGS` | 'zip' \<file²>                                            | Create a basic release archive with the specified path and name relative to the pwd using the files listed in the lines between `with files` and `:release ...`.                                                                                                                                                                                 |
| with files<br>...<br>:release `ARGS` | 'updater' \<updaterFile²> \<version²>                     | Patch the content of an updater file using the files listed in the lines between `with files` and `:release ...`. If a version change is detected it will create a patch block. Will create a separate file with a hash list to track file changes.                                                                                              |
| pucpatch `ARGS`                      | \<file²> ':' \<convar²> \<version²>                       | Tries to patch the specified file as 'Plugin Update Checker'-CSV, updating the version value for the specified version convar. Supports limited vim modeline to automatically realign columns.                                                                                                                                                   | 

²) denotes that variables are supported

Not all arguments, but most support replacements from environment variables or applications arguments. This is mainly inteded to not leak you auth tokens.
${NAME} is replaced with an environment variable, %{NAME} is replaced with the value from the argument `--<NAME> <VALUE>`. There is one exception however:
${CWD} and %{CWD} are replaced with the path of the loaded build-script.

The release tasks (`with files :release`) only support mod files, so you can not currently ship arbitrary files. This 
helps check file type and size limits on the forum as well as path extrapolation for the updater config, as that needs 
to know where files go, potentially even while you're using a flat project structure. 

Interactive mode has some special commands:
* `quit` or `exit` (or ^C) to exit
* `batch` to switch into batch mode until the next `run`
* `run` to execute the batch. Optional if `-I` was used and input ends.

Arguments
-----

`sps <Arguments> [--] [Build script]`
* If you do not specify a build script, it will default to `sp.sauce`
* `-x`|`--fulldeps`<br> Normally dependencies only extract .sp and .inc files as that's all that's requried for building, but you can let them exctract all files.
* `--offline`<br> Skips all `auth`, `sourcemod`, `dependency` and `clone` tasks.
* `-e`|`--no-exec`<br> Skips all `exec` tasks.
* `-s`|`--no-script`<br> Skips all `script` tasks.
* `-i` Interactive single mode. Will execute instructions from stdin every line, unless a batch is started.
* `-I` Interactive batch mode. Immediately start a batch in interactive mode. Can be used to pipe instructions in.
* `--<KEY> <VALUE>`<br> Passes values into the template system.
* `--stacktrace`<br> Used for debugging

Wrappers
-----
Since this application is written in java, invoking SPSauce can be a bit tedious with `java -jar <path to jar>`.
The release archive contains wrapper scripts for batch and bash that short this to `sps`. These wrappers require the
jar archive to be placed in ./spsauce/spsauce-version-all.jar.  
For non-technicall people it's best to pack the spsauce wrapper and script in your repository, and use the default
script name `sp.sauce`. This way you can tell your users to just execute `./sps` or `sps.bat` to build the plugin.

Automatic Artifacts
-----
While i think doing fully automatic releases might be a bit much, you're free to do so. But in this example I'll only pack up the compiled plugin with necessary files like translations and gamedata. The Archive will then be attached to the release.

First, let's write the sp.sauce script to compile and pack the plugin:
```spsauce
# specify the sourcemod version to use
sourcemod 1.10
# compile the plugin
spcomp plugin.sp -oplugin.smx
# get the plugin version (this is usually a define like this)
set %{PLUGIN_VERSION} as \1 from plugin.sp ^#define\s+PLUGIN_VERSION\s+"([^"]+)"
# authenticate agains github
auth github ${GITTOKEN}
# pack up the release archive
with plugins plugin.smx
# packs files in a zip, folder names will be recursed automatically
# intendation is optional but makes things more readable
with files
 translations
 gamedata
 scripting/include
:release zip Plugin_%{PLUGIN_VERSION}.zip
# patch meta files for `Updater` and `Plugin Update Checker`
with files
 translations
 gamedata
 plugins
:release updater www/static/updater.cfg %{PLUGIN_VERSION}
pucpatch www/static/versions.txt : version_convar %{PLUGIN_VERSION}
```

Now the GitHub Actions file:
```yaml
name: Plugin Release Archive
on:
  release:
    types: [published]
jobs:
  pack-plugin:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v2
        with:
          distribution: 'adopt'
          java-version: '8'
      - name: SPSauce
        run: ./sps
```