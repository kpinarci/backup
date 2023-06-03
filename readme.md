## Install Borg Backup and Borgmatic

```sh
apt install -y software-properties-common zstd && \
add-apt-repository -y ppa:costamagnagianfranco/borgbackup && \
apt update && apt install -y borgbackup liblz4-tool && \
apt install -y python3-pip python3-setuptools && \
pip3 install --upgrade pip && \
pip3 install --upgrade borgmatic
```

## Create a new ssh-key

To enable passwordless authentication, generate a new SSH key and add the public key to the `authorized_keys` file.

```sh
ssh-keygen -t ed25519 -o -a 100 -C "$(whoami)@$(hostname)-borg-$(date -I)" -f ~/.ssh/borg_id_ed25519
```

## Restrict ssh-key
For better security, you can restrict the ssh-key on `authorized_keys`.
More details [here](https://www.thomas-krenn.com/de/wiki/Ausf%C3%BChrbare_SSH-Kommandos_per_authorized_keys_einschr%C3%A4nken)

```shell
# /root/.ssh/authorized_keys
command="borg serve --restrict-to-path /home/<reponame>" <set here your ssh-pub-key>
```

## Short way how to backup with borgmatic

## Configuration

After you install borgmatic, genereate a sample configuration file:

```sh
sudo generate-borgmatic-config
```

This genereate a sample configuration file at /etc/borgmatic/config.yaml
by default. If you'd like to use another path, use the `--destination` flag, for instance:

```sh
generate-borgmatic-config --destination ~/Github/borgmatic/config.yaml
```

## Validation

If you'd like to validate that you borgmatic configuration is valid, the following command is available for that:

```sh
sudo validate-borgmatic-config
```

## Repository creation

Before you can create backups with borgmatic, you first need to create a `Borg repository` so you have a destination for you backup archives.
To create a repository, run a command like the following with Borg `New in borgmatic version 1.7.0` or, with Borg `2.x.`:

```sh
sudo borgmatic rcreate --encryption repokey-aes-ocb
```

## Backups

Now that you've configured borgmatic and created a repository, it's a good idea to test that borgmatic is working. So to run borgmatic and start a
backup, you can invoke it like this:

```sh
sudo borgmatic create --verbosity 2 --files -stats
```

**--verbosity:** flag makes borgmatic show the steps it's performing.
**--files:** flag list each file that's new or changed since the last backup.
**--stats:** flag shows summary information about the created archive. All of these flags are optional 😃

## Default action

If you are omit `create` and other actions, borgmatic runs through a set of default actions: prune any old backups as per the configured retention policy, compact
segments to free up space, create backup, and check backups for consistency problems due to things like file damage. For instance:

```sh
sudo borgmatic --verbosity 2 --files --stats
```

## Autopilot Cron

If you're using cron, download the [sample cron file](https://projects.torsion.org/borgmatic-collective/borgmatic/src/main/sample/cron/borgmatic). Then, from the
directory where you downloaded it:

```sh
sudo mv borgmatic /etc/cron.d/borgmatic && sudo chmod +x /etc/cron.d/borgmatic
```

## Extract

When the worst happens-or you want to test your backups- the first step is to figure out which archive to extract. A good way to that is to use the `rlist`
action:

```sh
borgmatic rlist
```

That should yield output looking something like:

```sh
air-2023-05-21T15:36:57.109116       Sun, 2023-05-21 15:36:58 [....]
air-2023-05-21T16:28:37.113850       Sun, 2023-05-21 16:28:38 [....]
```

Assuming that you want to extract the archive with the most up-to-date files and therefore the latest timestamp, run a command like:

```sh
borgmatic extract --archive air-2023-05-21T16:28:37.113850
```

Or simplify this to:

```sh
borgmatic extract --archive latest
```

## Extract particular files

Sometimes, you want to extract a singe deleted file, rather than extract everything from an archive. To do that, tack on one or more --path values. For Instance:

```sh
borgmatic extract --archive latest --path path/1 path/2
```

For instance:

```sh
borgmatic extract --archive air-2023-05-21T15:36:57.109116 --path /home/user/Github --verbosity 2
```

Note that the specified restore paths should not have a leading slash. Like a whole- archive extract, this also extracts into the current directory by default.
So far example, if you happen to be in the directory /var and you run the extract command above, borgmatic will extract /var/path/1 and /var/path/2

## Searching for files

if you're not sure which archile contains the file you're looking for, you can [search across archives](https://torsion.org/borgmatic/docs/how-to/inspect-your-backups/#searching-for-a-file)

## Mount a filesystem (FUSE)

if instead of extracting files, you'd like explore the files from an archives as a FUSE filesystem, you can use the borgmatic mount action. Here`s an example:

```sh
borgmatic mount --archive latest --mount-point /mnt
```

When you're all done exploring your files, unmount your mount point. No --archives flag is needed:

```sh
borgmatic umount --mount-point /mnt
```

## Remove backup repository

List backups

```sh
borgmatic list
```

Example backups

```sh
hetzner-storagebox: Listing archives
air-2023-05-21T15:36:57.109116       Sun, 2023-05-21 15:36:58 [example]
air-2023-05-21T16:28:37.113850       Sun, 2023-05-21 16:28:38 [example]
air-2023-05-22T19:21:54.950761       Mon, 2023-05-22 19:21:57 [example]
```

Select and remove

```sh
borg delete ${BORG_REPO_URL}::'air-2023-05-21T15:36:57.109116'
```

## Change the repository password

```sh
borg key change-passphrase -v ${BORG_REPO_URL}
```

## Set Environments

I will not save my Borg passphrase and backup server in the config file. Instead, I will add them to the `.bashrc` file for better security. Here's an example:

```sh
export BORG_REPO_URL=sh://USER@USER.your-storagebox.de:23/./<reponame>
export BORG_PASSPHRASE='supersecretpasshrase'
```

## Set cronjob

For automatic backups move this file to `/etc/cron.d/` or set this command in sh:

```sh
sudo move borgmatic /etc/cron.d/borgmatic && chmod 644 /etc/cron.d/borgmatic
```

## Set logrotate

Move the logrotate file to `/etc/logrotate.d/borgmatic_logrotate`

```sh
sudo move borgmatic_logrotate /etc/logrote.d/borgmatic_logrote && chmod 644 /etc/logrotate/borgmatic_logrotate
```
