# game-server-operator
[![Build Status](https://travis-ci.org/fabianvf/game-server-operator.svg?branch=master)](https://travis-ci.org/fabianvf/game-server-operator)

This operator deploys game servers to OpenShift or Kubernetes

Currently supported games are:
- minecraft

## Deploying the Operator
To deploy it, ensure your kubeconfig is properly set up (ie, that you can use `kubectl` or `oc` to connect to it),
ensure you have `cluster-admin` permissions (you will need this to create CustomResourceDefinitions),
install [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) and
the [openshift python client](https://github.com/openshift/openshift-restclient-python/#installation), and run

```bash
ansible-playbook deploy.yml
```

If you'd rather deploy it by hand, you can instead run:

```bash
kubectl apply -f deploy/crds/crds.yaml
kubectl apply -f deploy/role.yaml
kubectl apply -f deploy/role_binding.yaml
kubectl apply -f deploy/service_account.yaml
kubectl apply -f deploy/operator.yaml
```

Once this is done, your operator will be running.

## Deploying a Minecraft server
all you need to do is create an instance of the `games.fabianism.us/v1alpha1` `Minecraft` resource. A basic example
Minecraft definition exists in `deploy/crds/minecraft.yaml`. You can create it with

```bash
kubectl apply -f deploy/crds/minecraft.yaml
```

This will kick off the deployment of the Minecraft server. To monitor the status of the deployment, run

```bash
kubectl describe minecraft
```

and monitor the status field. After some amount of time, a field called `URL` should appear in the status,
this is the URL that the operator has generated for your server. You should be able to access the minecraft
server by connecting to that URL.


### Deployment Options

The example deployment will only give you a very basic, ephemeral, vanilla Minecraft. In order to get a real deployment,
you will at least need to add storage options. The following options are supported in the Minecraft spec:

| name | description | default | required |
|-|-|-|-|
| image | The Minecraft image to deploy | docker.io/itzg/minecraft-server:latest | no |
| port | The port on the host that Minecraft should listen on. | Random available port | no |
| host | If running openshift, the host that Minecraft will be accessible from | `http://minecraft-{namespace}.{openshift subdomain}` | no |
| volumes | The volumes to create/mount into your container. See [VolumeSpec](#volumespec) for more detail. | | yes |
| server | The server configuration options. See [ServerSpec](#serverspec) for more detail. | | no |
| world | The world configuration options. See [WorldSpec](#worldspec) for more detail. | | no |
| mods | The mod configuration options. See [ModSpec](#modspec) for more detail. | | no |
| jvm | The jvm configuration options. See [JvmSpec](#jvmspec) for more detail. | | no |


#### VolumeSpec

The `volumes` field is a map of key:value pairs, where each key is a volume name and the value is a map of key:value pairs defining
how the volume is to be created/mounted/accessed.

For configuration persistence, only one volume is required, the `data` volume. It must be mounted to `/data` with read-write permissions.

All volumes accept the following options:

| name | description | default | required |
|-|-|-|-|
| type | The type of volume. Choices are [`PersistentVolumeClaim`, `EmptyDir`] | | yes |
| mountPath | The full path to mount in the container. | The name of the volume (ie, `config` -> `/config`). | no |
| claimName | The name of the `PersistentVolumeClaim` to use. | The name of the volume. | no |
| size | The size of the volume to be created |  | When `type` is `PersistentVolumeClaim` and `create` is `true`. |
| create | Whether the `PersistentVolumeClaim` be created if it does not exist. | no | no |
| storageClassName | The name of the storage class to set for a `PersistentVolumeClaim` | The default storageClass for the cluster. | no |
| accessModes | The list of accessModes for a volume. Options are [`ReadWriteOnce`, `ReadWriteMany`, `ReadOnlyMany`] | [`ReadWriteOnce`] | no |
| readOnly | Whether the volume should be mounted read only | false | no |

#### ServerSpec

The `server` field contains configuration options for server administration.

| name | description | default | required |
|-|-|-|-|
| version | The minecraft version to install | latest | no |
| name | The name of your Minecraft server | My Minecraft Server | no |
| type | The type of server to run. Choices are BUKKIT, SPIGOT, PAPER, FORGE, FTB, CURSEFORGE, VANILLA, SPONGEVANILLA, CUSTOM | VANILLA |
| icon | The icon to display for your server | | no |
| difficulty | The difficulty level. Choices are peaceful, easy, normal, hard | easy | no |
| maxPlayers | The maximun number of players your server will support | 20 | no |
| whitelist | A list of usernames to allow to connect to the server | everyone | no |
| ops | A list of usernames to grant op permissions to | noone | no |
| announcePlayerAchievements | Whether to announce achievements globally to the server | True | no |
| enableCommandBlock | Enables command blocks | False | no |
| forceGamemode | Force players to join in the default game mode. | False | no |
| hardcore | If set to true, players will be set to spectator mode if they die. | False | no |
| snooperEnabled | If set to false, the server will not send data to snoop.minecraft.net server | True | no |
| maxBuildHeight | The maximum height in which building is allowed. Terrain may still naturally generate above a low height limit. | | no |
| maxTickTime | The maximum number of milliseconds a single tick may take before the server watchdog stops the server with the message, A single server tick took 60.00 seconds (should be max 0.05); Considering it to be crashed, server will forcibly shutdown. Once this criteria is met, it calls System.exit(1). Setting this to -1 will disable watchdog entirely | 60000 | no |
| viewDistance | Sets the amount of world data the server sends the client, measured in chunks in each direction of the player (radius, not diameter). It determines the server-side viewing distance. | 10 | no |
| mode | The gamemode of the server. One of creative, survival, adventure, spectator | survival  | no |
| motd | The message of the day for the server. | | no |
| pvp | If set to False, PVP will not be allowed on the server | True | no |
| enableRcon | Whether to enable RCON | False | no |
| rconPassword | The password for RCON |  | no |
| onlineMode | Whether to check that users are logged in. | True | no |
| allowNether | Allows players to travel to the Nether. | True | no |
| allowFlight | Allows players to fly. | False | no |

#### WorldSpec

The `world` field contains options for generating or loading your world.

| name | description | default | required |
|-|-|-|-|
| maxWorldSize | This sets the maximum possible size in blocks, expressed as a radius, that the world border can obtain. | 10000 | no |
| generateStructures | Defines whether structures (such as villages) will be generated. | True | no |
| spawnAnimals | Determines if animals will be able to spawn. | True | no |
| spawnMonsters | Determines if monsters will be spawned. | True | no |
| spawnNpcs | Determines if villagers will be spawned. | True | no |
| seed | The level seed to generate the world with | | no |
| levelType | One of default, flat, largebiomes, amplified, customized, buffet | default | no |
| generatorSettings | When using a levelType of FLAT, CUSTOMIZED, and BUFFET, you can further configure the world generator by passing custom generator settings. |  | no |
| level | Determines the World Save Name to load | world | no |
| world | URL or path to world | | no |

#### ModSpec

The `mods` field contains options for loading and configuring your modpacks.
Bukkit, Spigot, Paper, Forge, Feed The Beast, CurseForge, Vanilla, SpongeVanilla and Custom modpacks
are all supported. The configuration options that will be loaded are based on the `spec.server.type` field.


| name | description | default | required |
|-|-|-|-|
| modpack | |  | no |
| removeOldMods | | False | no |
| ftbServerMod | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-server-with-a-feed-the-beast-ftb--curseforge-modpack | | no |
| cfServerMod | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-server-with-a-feed-the-beast-ftb--curseforge-modpack | | no |
| forgeversion | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-forge-server | latest stable | no |
| forgeInstaller | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-forge-server | installer corresponding to forgeversion | no |
| forgeInstallerURL | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-forge-server |  | no |
| bukkitDownloadURL | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-bukkitspigot-server |  | no |
| spigotDownloadURL | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-bukkitspigot-server |  | no |
| buildFromSource | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-bukkitspigot-server | False | no |
| paperDownloadURL | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-paperspigot-server | | no |
| manifest | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#using-a-client-made-curseforge-modpack | | no |
| spongeBranch | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-a-spongevanilla-server |  | no |
| customServer | see https://github.com/itzg/dockerfiles/tree/master/minecraft-server#running-with-a-custom-server-jar | | no |

#### JvmSpec

The `jvm` field contains options for configuring the JVM. These include memory/heap settings as well as arbitrary arguments you may want to pass to the JVM.

| name | description | default | required |
|-|-|-|-|
| memory | can be used to adjust both initial (Xms) and max (Xmx) memory settings of the JVM. | 1G | no |
| initMemory | independently sets the initial heap size. |  | no |
| maxMemory | Independently sets the max heap size. |  | no |
| jvmOpts | General JVM options that will be passed to the Minecraft Server invocation |  | no |
| jvmXxOpts | Options like `-X` that need to proceed general JVM options  |  | no |
| jvmDdOpts | For some cases, if e.g. after removing mods, it could be necessary to startup minecraft with an additional `-D` parameter like `-Dfml.queryResult=confirm`. If you add `fml.queryResult:confirm`, it will be converted to `-Dfml.queryResult=confirm` |  | no |

#### Examples
TODO
