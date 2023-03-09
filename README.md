<img style="max-width:840px;" src="https://raw.githubusercontent.com/wiki/Unpackerr/unpackerr/images/unpackerr-logo-text.png">

## About

This application runs as a daemon on your download host. 
It checks for completed downloads and extracts them so 
[Lidarr](http://lidarr.audio), 
[Radarr](http://radarr.video), 
[Readarr](http://readarr.com), and
[Sonarr](http://sonarr.tv) may import them.
There are a handful of options out there for extracting and deleting files after
your client downloads them. I just didn't care for any of them, so I wrote my own. I
wanted a small single-binary with reasonable logging that can extract downloaded
archives and clean up the mess after they've been imported.

Not a starr app user, and just need to extract files? We do that too.
This application can run standalone and extract files found in a "watch" folder.
In other words, you can configure this application to watch your download folder, and
it will happily extract everything you download. This has nothing to do with the four 
Starr apps mentioned in the previous paragraph. This Folder-watch feature may be used
with or without Starr apps.

## What's it extract?

Pretty much everything. It also extracts recursively, meaning deep within folders, and archives within archives.
**Tars, Rars, Zips, 7-Zips, Gzips, Tarred gzips and bzips; encrypted rars and 7zips. And ISO disc images.**
Need something else? Ask. Does it do too much? Let me know what knobs you need. [Open a request!](https://github.com/Unpackerr/unpackerr/issues/new)

## Installation

-   **Note**: Requires access to your download location.
    Make sure you set the `path` variables correctly in the configuration.
    Even if they're set incorrectly this app makes a best effort attempt to
    locate your downloads.

### Docker

This project [builds automatically](https://github.com/Unpackerr/unpackerr/blob/main/init/docker/hooks/build)
in [Docker Cloud](https://hub.docker.com/r/golift/unpackerr) and creates
[ready-to-use multi-architecture images](https://hub.docker.com/r/golift/unpackerr/tags) images.
The `latest` tag is always a [tagged release on GitHub](https://github.com/Unpackerr/unpackerr/releases).

Use the methods below to install using Docker.

#### unRAID (Docker)

-   Unpackerr is available in the
    [Community Applications](https://github.com/selfhosters/unRAID-CA-templates/blob/main/templates/unpackerr.xml)
    on unRAID.
-   Note that your mount `/mnt/HostDownloads:/downloads` **MUST** align with the Starr's paths for Unpackerr to work.
    - This path likely best to be the same as your download client.
    - E.g. if Qbit is given `/mnt/user/data/downloads:/data/downloads` then Unpackerr shall get the same mount

#### Docker Config File

-   Copy the [example config file](https://github.com/Unpackerr/unpackerr/blob/main/examples/unpackerr.conf.example) from this repo.
-   Then grab the image from docker hub and run it using an overlay for the config file's directory.
-   The config file must be at `/config/unpackerr.conf`. 
-   Recommend bind-mounting `/config` as an app-data directory. Example Follows.
-   Note that your mount `/mnt/HostDownloads:/downloads` **MUST** align with the Starr's paths for Unpackerr to work.
    - This path likely best to be the same as your download client.
    - E.g. if Qbit is given `/mnt/user/data/downloads:/data/downloads` then Unpackerr shall get the same mount

```shell
docker pull golift/unpackerr
docker run -d -v /mnt/HostDownloads:/downloads -v /folder/with/config/file:/config golift/unpackerr
docker logs <container id from docker run>
```

#### Docker Compose

-   Copy the [example docker-compose.yml](https://github.com/Unpackerr/unpackerr/blob/main/examples/docker-compose.yml) from this repo.
-   Edit the docker-compose.yml file with your environment variables and save.
-   Recommend removing variables you are not using; the defaults are shown below.
-   Note that your mount `/mnt/HostDownloads:/downloads` **MUST** align with the Starr's paths for Unpackerr to work.
    - This path likely best to be the same as your download client.
    - E.g. if Qbit is given `/mnt/user/data/downloads:/data/downloads` then Unpackerr shall get the same mount

```shell
docker-compose up -d
```

#### Docker Env Variables

-   Instead of a config file, you may configure the docker container
    with environment variables.
-   Any variable not provided takes the default.
-   Must provide URL and API key for Sonarr or Radarr or Lidarr or any combination.
-   You may provide multiple Sonarr, Radarr or Lidarr instances using
    `UN_SONARR_1_URL`, `UN_SONARR_2_URL`, etc.

##### Global Settings
|Config Name|Variable Name|Default / Note|
|---|---|---|
debug|`UN_DEBUG`|`false` / Turns on more logs|
log_file|`UN_LOG_FILE`|None by default. Optionally provide a file path to write logs|
log_files|`UN_LOG_FILES`|`10` / Log files to keep after rotating. `0` disables rotation|
log_file_mb|`UN_LOG_FILE_MB`|`10` / Max size of log files in megabytes|
interval|`UN_INTERVAL`|`2m` / How often apps are polled, recommend `1m` to `5m`|
start_delay|`UN_START_DELAY`|`1m` / Files are queued at least this long before extraction|
retry_delay|`UN_RETRY_DELAY`|`5m` / Failed extractions are retried after at least this long|
max_retries|`UN_MAX_RETRIES`|`3` / Times to retry failed extractions. `0` = unlimited.|
parallel|`UN_PARALLEL`|`1` / Concurrent extractions, only recommend `1`|
file_mode|`UN_FILE_MODE`|`0644` / Extracted files are written with this mode|
dir_mode|`UN_DIR_MODE`|`0755` / Extracted folders are written with this mode|
passwords|`UN_PASSWORD_0`|No default; empty list. Provide a list of RAR passwords to try.

_Note about about providing `passwords`. If a wrong password is provided, the entire archive must
be read before we know it's a bad password. Providing many passwords here can drastically slow down
extractions and cause extra disk IO. You may also specify a password file by prefixing it with `filepath:`_

##### Sonarr

|Config Name|Variable Name|Default / Note|
|---|---|---|
sonarr.url|`UN_SONARR_0_URL`|No Default. Something like: `http://localhost:8989`|
sonarr.api_key|`UN_SONARR_0_API_KEY`|No Default. Provide URL and API key if you use Sonarr|
sonarr.paths|`UN_SONARR_0_PATHS_0`|`/downloads` List of paths where content is downloaded for Sonarr. Used as fallback if the path Sonarr reports does not exist or is not accessible.|
sonarr.protocols|`UN_SONARR_0_PROTOCOLS`|`torrent` Protocols to process. Alt: `torrent,usenet`|
sonarr.timeout|`UN_SONARR_0_TIMEOUT`|`10s` / How long to wait for the app to respond|
sonarr.delete_orig|`UN_SONARR_0_DELETE_ORIG`|`false` / Delete archives after import? Recommend not setting this to true|
sonarr.delete_delay|`UN_SONARR_0_DELETE_DELAY`|`5m` / Extracts are deleted this long after import, `-1s` to disable|
sonarr.syncthing|`UN_SONARR_0_SYNCTHING`|`false` / Setting this to true makes unpackerr wait for syncthing to finish|

##### Radarr

|Config Name|Variable Name|Default / Note|
|---|---|---|
radarr.url|`UN_RADARR_0_URL`|No Default. Something like: `http://localhost:7878`|
radarr.api_key|`UN_RADARR_0_API_KEY`|No Default. Provide URL and API key if you use Radarr|
radarr.paths|`UN_RADARR_0_PATHS_0`|`/downloads` List of paths where content is downloaded for Radarr. Used as fallback if the path Radarr reports does not exist or is not accessible|
radarr.protocols|`UN_RADARR_0_PROTOCOLS`|`torrent` Protocols to process. Alt: `torrent,usenet`|
radarr.timeout|`UN_RADARR_0_TIMEOUT`|`10s` / How long to wait for the app to respond|
radarr.delete_orig|`UN_RADARR_0_DELETE_ORIG`|`false` / Delete archives after import? Recommend not setting this to true|
radarr.delete_delay|`UN_RADARR_0_DELETE_DELAY`|`5m` / Extracts are deleted this long after import, `-1s` to disable|
radarr.syncthing|`UN_RADARR_0_SYNCTHING`|`false` / Setting this to true makes unpackerr wait for syncthing to finish|

##### Lidarr

|Config Name|Variable Name|Default / Note|
|---|---|---|
lidarr.url|`UN_LIDARR_0_URL`|No Default. Something like: `http://localhost:8686`|
lidarr.api_key|`UN_LIDARR_0_API_KEY`|No Default. Provide URL and API key if you use Lidarr|
lidarr.paths|`UN_LIDARR_0_PATHS_0`|`/downloads` List of paths where content is downloaded for Lidarr. Used as fallback if the path Lidarr reports does not exist or is not accessible|
lidarr.protocols|`UN_LIDARR_0_PROTOCOLS`|`torrent` Protocols to process. Alt: `torrent,usenet`|
lidarr.timeout|`UN_LIDARR_0_TIMEOUT`|`10s` / How long to wait for the app to respond|
lidarr.delete_orig|`UN_LIDARR_0_DELETE_ORIG`|`false` / Delete archives after import? Recommend not setting this to true|
lidarr.delete_delay|`UN_LIDARR_0_DELETE_DELAY`|`5m` / Extracts are deleted this long after import, `-1s` to disable|
lidarr.syncthing|`UN_LIDARR_0_SYNCTHING`|`false` / Setting this to true makes unpackerr wait for syncthing to finish|

##### Readarr

|Config Name|Variable Name|Default / Note|
|---|---|---|
readarr.url|`UN_READARR_0_URL`|No Default. Something like: `http://localhost:8787`|
readarr.api_key|`UN_READARR_0_API_KEY`|No Default. Provide URL and API key if you use Readarr|
readarr.paths|`UN_READARR_0_PATHS_0`|`/downloads` List of paths where content is downloaded for Readarr. Used as fallback if the path Readarr reports does not exist or is not accessible|
readarr.protocols|`UN_READARR_0_PROTOCOLS`|`torrent` Protocols to process. Alt: `torrent,usenet`|
readarr.timeout|`UN_READARR_0_TIMEOUT`|`10s` / How long to wait for the app to respond|
readarr.delete_orig|`UN_READARR_0_DELETE_ORIG`|`false` / Delete archives after import? Recommend not setting this to true|
readarr.delete_delay|`UN_READARR_0_DELETE_DELAY`|`5m` / Extracts are deleted this long after import, `-1s` to disable|
readarr.syncthing|`UN_READARR_0_SYNCTHING`|`false` / Setting this to true makes unpackerr wait for syncthing to finish|

##### Folder

Folders are a way to watch a folder for things to extract. You can use this to
monitor your download client's "move to" path if you're not using it with an Starr app.

|Config Name|Variable Name|Default / Note|
|---|---|---|
folder.path|`UN_FOLDER_0_PATH`|No Default; folder to watch for archives. **Not for Starr apps**|
folder.extract_path|`UN_FOLDER_0_EXTRACT_PATH`|Where to extract to. Default is the same as `path`|
folder.delete_after|`UN_FOLDER_0_DELETE_AFTER`|`10m` Delete extracted files and/or archives after this duration; `0` disables|
folder.disable_log|`UN_FOLDER_0_DISABLE_LOG`|`false` Turns off creation of extraction logs files for this folder|
folder.delete_original|`UN_FOLDER_0_DELETE_ORIGINAL`|`false` Delete archives after successful extraction|
folder.delete_files|`UN_FOLDER_0_DELETE_FILES`|`false` Delete extracted files after successful extraction|
folder.move_back|`UN_FOLDER_0_MOVE_BACK`|`false` Move extracted items back into original folder|
folder.extract_isos|`UN_FOLDER_0_EXTRACT_ISOS`|`false` Setting this to true enables .iso file extraction|

##### Command Hooks

Unpackerr can execute commands (or scripts) before and after an archive extraction.
The only thing required is a command. Name is optional, and used in logs only.
Setting `shell` to `true` executes your command after `/bin/sh -c` or `cmd.exe /c`
on Windows.

|Config Name|Variable Name|Default / Note|
|---|---|---|
cmdhook.command|`UN_CMDHOOK_0_COMMAND`|No Default; command to run|
cmdhook.name|`UN_CMDHOOK_0_NAME`|Defaults to first word in command; provide an optional name for logs|
cmdhook.timeout|`UN_CMDHOOK_0_TIMEOUT`|Defaults to global timeout, usually `10s`|
cmdhook.silent|`UN_CMDHOOK_0_SILENT`|`false` / Hide command output from logs|
cmdhook.shell|`UN_CMDHOOK_0_SHELL`|`false` / Run command inside a shell|
cmdhook.exclude|`UN_CMDHOOK_0_EXCLUDE`|`[]` / List of apps to exclude: radarr, sonarr, folders, etc|
cmdhook.events|`UN_CMDHOOK_0_EVENTS`|`[0]` / List of event IDs to send (shown under Webhooks)|

All extraction data is input to the command using environment variables, see example below.
Extracted files variables names begin with `UN_DATA_FILES_`.
Try `/usr/bin/env` as an example command to see what variables are available.

```none
UN_DATA_OUTPUT=folder/subfolder_unpackerred
UN_PATH=folder/subfolder
UN_DATA_START=2021-10-04T23:04:27.849216-07:00
UN_REVISION=
UN_EVENT=extracted
UN_GO=go1.17
UN_DATA_ARCHIVES_2=folder/subfolder/files.zip
UN_DATA_ARCHIVES_1=folder/subfolder_unpackerred/Funjetting.r00
UN_DATA_ARCHIVES_0=folder/subfolder_unpackerred/Funjetting.rar
UN_DATA_FILES_1=folder/subfolder/Funjetting.r00
UN_DATA_BYTES=2407624
PWD=/Users/david/go/src/github.com/Unpackerr/unpackerr
UN_DATA_FILES_0=folder/subfolder/Funjetting.mp3
UN_OS=darwin
UN_DATA_FILES_3=folder/subfolder/_unpackerred.subfolder.txt
UN_DATA_FILES_2=folder/subfolder/Funjetting.rar
UN_BRANCH=
UN_TIME=2021-10-04T23:04:27.869613-07:00
UN_VERSION=
UN_DATA_QUEUE=0
SHLVL=1
UN_APP=Folder
UN_STARTED=2021-10-04T23:03:22.849253-07:00
UN_ARCH=amd64
UN_DATA_ELAPSED=20.365752ms
UN_DATA_ERROR=
```

##### Webhooks

This application can send a POST webhook to a URL when an extraction begins, and again
when it finishes. Configure 1 or more webhook URLs with the parameters below.
Works great with [notifiarr.com](https://notifiarr.com). You can use
[requestbin.com](https://requestbin.com/r/) to test and _see_ the payload.

|Config Name|Variable Name|Default / Note|
|---|---|---|
webhook.url|`UN_WEBHOOK_0_URL`|No Default; URL to send POST webhook to|
webhook.name|`UN_WEBHOOK_0_NAME`|Defaults to URL; provide an optional name to hide the URL in logs|
webhook.nickname|`UN_WEBHOOK_0_NICKNAME`|`Unpackerr` / Passed into templates for telegram, discord and slack hooks|
webhook.channel|`UN_WEBHOOK_0_CHANNEL`|`""` / Passed into templates for slack.com webhooks|
webhook.timeout|`UN_WEBHOOK_0_TIMEOUT`|Defaults to global timeout, usually `10s`|
webhook.silent|`UN_WEBHOOK_0_SILENT`|`false` / Hide successful POSTs from logs|
webhook.ignore_ssl|`UN_WEBHOOK_0_IGNORE_SSL`|`false` / Ignore invalid SSL certificates|
webhook.exclude|`UN_WEBHOOK_0_EXCLUDE`|`[]` / List of apps to exclude: radarr, sonarr, folders, etc|
webhook.events|`UN_WEBHOOK_0_EVENTS`|`[0]` / List of event IDs to send (shown below)|
webhook.template_path|`UN_WEBHOOK_0_TEMPLATE_PATH`|`""` / Instead of an internal template, provide your own|
webhook.template|`UN_WEBHOOK_0_TEMPLATE`|`""` / Instead of auto template selecton, force one; options below|
webhook.content_type|`UN_WEBHOOK_0_CONTENT_TYPE`|`application/json` / Content-Type header sent to webhook|

Event IDs (not all of these are used in webhooks): `0` = all,
`1` = queued, `2` = extracting, `3` = extract failed, `4` = extracted,
`5` = imported, `6` = deleting, `7` = delete failed, `8` = deleted

###### Webhook Notes

1. _`Nickname` should equal the `chat_id` value in Telegram webhooks._
1. _`Channel` is used as destination channel for Slack. It's not used in others._
1. _`Nickname` and `Channel` may be used as custom values in custom templates._
1. _`Name` is only used in logs, but it's also available as a template value as `{{name}}`._
1. Built-In Templates: `pushover`, `telegram`, `discord`, `notifiarr`, `slack`, `gotify`

##### Example Usage

```shell
docker pull golift/unpackerr
docker run -d -v /mnt/HostDownloads:/downloads -e "UN_SONARR_0_URL=http://localhost:8989" -e "UN_SONARR_0_API_KEY=kjsdkasjdaksdj" golift/unpackerr
docker logs <container id from docker run>
```

#### More Dockers!

 If you want a container that has a bit more to it, you can try a third party option.
 The container provided by golift is from scratch so it has nothing more than a binary
 and a config file (with our defaults).

-   **[@hotio](https://github.com/hotio) maintains a
    [Custom Docker Container](https://hub.docker.com/r/hotio/unpackerr)
    for Unpackerr.** ([repo](https://github.com/hotio/unpackerr))

### Linux Install

Linux repository hosting provided by
[![packagecloud](https://docs.golift.io/integrations/packagecloud-full.png "PackageCloud.io")](http://packagecloud.io)

On Linux, unpackerr runs as `user:group` `unpackerr:unpackerr`. You will need to give that
user or group read and write access to your archives. That may mean adding the `unpackerr`
user, for example, to the `debian-transmission` group.

Run this to install the golift repo and unpackerr:
```
curl -s https://golift.io/repo.sh | sudo bash -s - unpackerr
```

After install, edit the config and start the service:

```
sudo nano /etc/unpackerr/unpackerr.conf
sudo systemctl restart unpackerr
```

### FreeBSD Install

-   Download a package from the [Releases](https://github.com/Unpackerr/unpackerr/releases) page.
-   Install it, edit config, start it.

On FreeBSD the app runs as `nobody`. That's not very good and will probably change in the future.

### macOS Install

-   Use homebrew.
-   Edit config file at `/usr/local/etc/unpackerr/unpackerr.conf`
-   Start it.
-   Like this:

```shell
brew install golift/mugs/unpackerr
vi /usr/local/etc/unpackerr/unpackerr.conf
brew services start unpackerr
```

You can also use a GUI app on a Mac instead of CLI via Homebrew:

-   Download a `.dmg` file from [the Releases page](https://github.com/Unpackerr/unpackerr/releases).
-   Copy the `Unpackerr.app` to `/Applications`.
-   Run it. It starts in the menu bar as an icon.
-   Click the menu bar icon and select `Config` -> `Edit`.
-   Edit the config to suit your system and save.
-   Click the menu bar icon again and select `Quit`. Then open the app again.
-   View the logs by clicking the menu bar icon and `Logs` -> `View`.
-   You can add it to login items to run it automatically when you login.

The `.app` and the Homebrew version are the same application, but one runs in GUI mode and one does not.

### Windows Install

-   Extract a `.exe.zip` file from [the Releases page](https://github.com/Unpackerr/unpackerr/releases) into a folder like `C:\Program Files\unpackerr\`.
-   Run the `unpackerr.amd64.exe` binary. This starts the app in the system tray.
-   Click the systray icon and select `Config` -> `Edit`.
-   Edit the config to suit your system and save.
-   Click the systray icon again and select `Quit`. Then open the app again.
-   View the logs by clicking the systray icon and `Logs` -> `View`.
-   Make a shortcut to the application in your Startup menu to run it when you login.

## Integrations

The following fine folks are providing their services, completely free! These service
integrations are used for things like storage, building, compiling, distribution and
documentation support. This project succeeds because of them. Thank you!

<p style="text-align: center;">
<a title="PackageCloud" alt="PackageCloud" href="https://packagecloud.io"><img src="https://docs.golift.io/integrations/packagecloud.png"/></a>
<a title="GitHub" alt="GitHub" href="https://GitHub.com"><img src="https://docs.golift.io/integrations/octocat.png"/></a>
<a title="Docker Cloud" alt="Docker" href="https://cloud.docker.com"><img src="https://docs.golift.io/integrations/docker.png"/></a>
<a title="Homebrew" alt="Homebrew" href="https://brew.sh"><img src="https://docs.golift.io/integrations/homebrew.png"/></a>
<a title="Go Lift" alt="Go Lift" href="https://golift.io"><img src="https://docs.golift.io/integrations/golift.png"/></a>
</p>

## Troubleshooting

- Make sure your Downloads location matches on all your applications!
- Most issues can be resolved by ensuring the user and group Unpackerr runs as has Read and Write permissions to the path trying to be extracted.
- Most Docker (and Unraid) issues can be resolved by ensuring your paths align between containers
  - See [TRaSH's Guides](https://trash-guides.info/Hardlinks/Hardlinks-and-Instant-Moves/) and the [Servarr Team's Docker Guide](https://wiki.servarr.com/docker-guide) for details on Docker/Unraid path practices
- Common Bad Path Examples that will not work
  Ex 1.
  - Sonarr: `/mnt/user/data:/data`
  - Qbittorrent: `/mnt/user/data/downloads:/data/downloads`
  - Unpackerr: `/mnt/user/data/:/downloads`
  Ex 2.
  - Sonarr: `/mnt/user/data:/data`
  - Qbittorrent: `/mnt/user/data/downloads:/downloads`
  - Unpackerr: `/mnt/user/data/:/downloads`
- Common Good Path Examples
  Ex 1.
  - Sonarr: `/mnt/user/data:/data`
  - Qbittorrent: `/mnt/user/data/downloads:/data/downloads`
  - Unpackerr: `/mnt/user/data/:/data/downloads`
- [Find help on Discord](https://golift.io/discord).

Log files:

-   Linux: `/var/log/messages` or `/var/log/syslog` (w/ default syslog)
-   FreeBSD: `/var/log/syslog` (w/ default syslog)
-   macOS: `/usr/local/var/log/unpackerr.log` or `~/.unpackerr/unpackerr.log`
-   Windows: `~/.unpackerr/unpackerr.log`

If transfers are in a Warning or Error state they will not be extracted.
If Unpackerr prints information about transfers you do not see in your Starr app.

**Permissions** tend to mess things up too, so make sure the user unpackerr runs as can read 
and write to your download location.

Still having problems?
[Let me know!](https://github.com/Unpackerr/unpackerr/issues/new)

## Archive Support

Unpackerr will decompress archives of these types:

-   `rar`, `tar`, `tgz`, `gz`, `zip`, `7z`, `bz2`, `tbz2`, `iso`
-   Multi-file archives are supported with RAR and 7ZIP archives.
-   Password protected archived are supported with RAR and 7ZIP archives.
-   Archives are detected by the file extension. ISO is disabled by default.

## Logic

The application polls radarr, readarr, sonarr and lidarr at the interval configured.
The queued items are inspected for completeness. The interval of these pollers is set
in the config file. 1-10 minutes is generally sufficient.

When Unpackerr finds an item in Sonarr or Radarr or Lidarr the download location
is checked for an archive file. If an extractable archive exists, and **Radarr/Sonarr/etc
has `status=Completed` from your download client** Unpackerr will extract the file.
Files are extracted to a temporary folder, and then moved back into the download
location for Completed Download Handling to import them. When the item falls out of the
Lidarr/Radarr/Readarr/Sonarr queue, the extracted files are deleted.

## Contributing

Yes, please. Just make a pull request and lets chat about it in the PR or on Discord.

## License

[MIT](https://github.com/Unpackerr/unpackerr/blob/main/LICENSE)
