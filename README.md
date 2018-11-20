# Minecraft Bedrock Server
Run a bedrock server in a Docker container.

## Introduction
This Docker image will download the Bedrock Server app and set it up, along with its dependencies.

## Usage
1. Prepare your data folder:
    1. Copy the data folder to some place where docker can use it as mounts (or create all the content manually).
    2. Configure the `server.properties` to your likings.
    3. Configure the `whitelist.json` in case you have set `white-list=true` in the above step. There is an example in the provided file. Just fill in the name and add more entries if wanted. The `xuid` is optional and will automatically be added as soon as a matching player connects. Here's also an example of the `whitelist.json` file:
        ```json
        [
            {
                "ignoresPlayerLimit": false,
                "name": "MyPlayer"
            },
            {
                "ignoresPlayerLimit": false,
                "name": "AnotherPlayer",
                "xuid": "274817248"
            }
        ]
        ```
    4. Configure the `ops.json` and add the operators. This file consists of a list of `permissions` and `xuid`s. The `permissions` can be `member`, `visitor` or `operator`. The `xuid` can be copied from the `whitelist.json` as soon as the user connected once. An example could look like:
        ```json
        [
            {
                "permission": "operator",
                "xuid": "274817248"
            }
        ]
        ```
    5. TODO: `permissions.json` (is this really needed?)
    6. In case you want your already existing world, put it into the worlds folder and adjust the name in the `server.properties`.
2. Start the Docker container:
```bash
docker run -d --name=minecraft\
    -v 'yourpath/server.properties:/bedrock-server/server.properties'\
    -v 'yourpath/whitelist.json:/bedrock-server/whitelist.json'\
    -v 'yourpath/ops.json:/bedrock-server/ops.json'\
    -v `yourpath/worlds:/bedrock-server/worlds'\
    -p 19132:19132\
    --restart=always\
    roemer/bedrock-server
```

## Commands
There are various commands that can be used in the console. To access the console, you need to attach to the container with the following command:
```
docker attach <container-id>
```
To leave the console without exiting the container, use `Ctrl`+`p` + `Ctrl`+`q`.

Here are the commands:

| Command syntax | Description |
| -------------- | ----------- |
| kick {`player name` or `xuid`} {`reason`} | Immediately kicks a player. The reason will be shown on the kicked players screen. |
| stop | Shuts down the server gracefully. |
| save {`hold` or `resume` or `query`} | Used to make atomic backups while the server is running. See the backup section for more information. |
| whitelist {`on` or `off` or `list` or `reload`} | `on` and `off` turns the whitelist on and off. Note that this does not change the value in the `server.properties` file!<br />`list` prints the current whitelist used by the server<br />`reload` makes the server reload the whitelist from the file.
| whitelist {`add` or `remove`} {`name`} | Adds or removes a player from the whitelist file. The name parameter should be the Xbox Gamertag of the player you want to add or remove. You don't need to specify a XUID here, it will be resolved the first time the player connects. |
| ops {`list` or `reload`} | `list` prints the current used operator list.<br />`reload` makes the server reload the operator list from the ops file. |
| changesetting {`setting`} {`value`} | Changes a server setting without having to restart the server. Currently only two settings are supported to be changed, `allow-cheats` (`true` or `false`) and `difficulty` (0, `peaceful`, 1, `easy`, 2, `normal`, 3 or `hard`). They do not modify the value that's specified in `server.properties`. |

## Backups
The server supports taking backups of the world files while the server is running. It's not particularly friendly for taking manual backups, but works better when automated. The backup (from the servers perspective) consists of three commands:

| Command | Description |
| ------- | ----------- |
| save hold | This will ask the server to prepare for a backup. It’s asynchronous and will return immediately. |
| save query | After calling `save hold` you should call this command repeatedly to see if the preparation has finished. When it returns a success it will return a file list (with lengths for each file) of the files you need to copy. The server will not pause while this is happening, so some files can be modified while the backup is taking place. As long as you only copy the files in the given file list and truncate the copied files to the specified lengths, then the backup should be valid. |
| save resume | When you’re finished with copying the files you should call this to tell the server that it’s okay to remove old files again. |
