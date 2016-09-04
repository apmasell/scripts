# Backups
No one likes making backups. Backups are usually an unpleasant system administration task. I have discovered that most people do not make backups and many who do make useless backups. I have seen much and thought about much and now I am prepared to speak.

## Modes of Failures

When considering backups, the first thing is to consider how things will fail and what level of paranoia is needed.

<dl>
<dt>Wanton Self-Destruction</dt><dd> Presumably, this is when you have a friend edit a paper and then realise their edits were mind-bogglingly stupid and now you have no previous copy.</dd>

<dt>System Inaccessibility</dt><dd> Especially if you have a laptop, there may be a situation where you data is stored safely on your computer, but you no longer have a way to access it. Having a backup on a system that is running properly all the time allows you to get at it.</dd>

<dt>Software Failure</dt><dd> This is when you intend to do `rm -rf /tmp` and accidentally do `rm -rf / tmp`. All of you hardware is working correctly, but your software is toast.</dd>

<dt>Hardware Failure</dt><dd> This is the one with sparks and the smell of burning plastic. When this happens, you find yourself in _System Inaccessibility_ until replacing the damaged hardware and then are simply in _Software Failure_.</dd>

<dt>Big Meteor</dt><dd> This is when a large meteor lands on you computer destroying it and any backups in a 1000km radius. You are likely screwed, but, if lucky, can get to _Hardware Failure_ if you stored a copy of your backup sufficiently off-site.</dd>

<dt>Theft</dt><dd> This is unlikely in the data centre, but a laptop bag can be swiped fairly easily. If you keep your backup in the same case or even same room as your computer, it is at risk to be stolen at the same time.</dd>

<dt>Chronic Disease</dt><dd> Your files were recently destroyed by a computer virus or worm that you have had for a long time and thus probably lives in your backups. You should probably do a clean install and then try to dissect your backups for safe files. If you have a friend with a different operating system (even Windows), they are probably going to be immune to your virus and thus helpful for safely handling diseased files.</dd>
</dl>

## The Restore

This is often neglected when people prepare backups. How are you going to restore your data?

Microsoft's built-in Backup utility is a perfect example of this. Sure, it makes backups, but there is no way to do a system restore without first reinstalling Windows and the Backup utility. If I keep my backups as `tar` archives, I can do a restore with a <a href="http://www.knoppix.org/">Knoppix</a> disc.

If I intend to restore using a Knoppix disc, I have to consider if I can even get to my backups if they are stored on a strange file-system or over a network. If I choose to use a strange file-system for my root file system, will my Knoppix CD be able to reformat it? Is there a backup of my partition table? Do I need a strange driver for my disk controller. The only way to be sure how all this will work is to do a restore. That's probably not practical, but something to consider if the opportunity, such as a new machine, comes along to test a bare metal restore. Going through he motions of a restore are also useful. Listing the files in the archive is good enough to make sure you can really access the archive. To set up new systems, I have restored from backups and then changed the identity of the system. This has proved to be a good exercise.

The other type of restore to consider is the partial restore. If I lose a file that I'm working on, can I restore just that one file without severe pain? If the files are loose on disk, this is trivially easy. As you add more layers of complexity and compression, it becomes more time consuming. It also becomes clear that there is a big advantage to separating the backups of software from documents. Also, if a file is corrupted Wednesday and I discover this Thursday, can I retrieve the backup from Tuesday?

## The Archive Format

How the backed up files are stored determines so much about how you can restore. First, I'd like to state what my boss Frank from <l>envcan</l> said: “We will never backup to tape again. Only to spinning harddisk.” I'm mostly in agreement. Tape is not a practical backup medium because it is painfully slow to do a restore and restoring one file involves reading gobs of tape. A certain downtown Toronto hospital I worked for also did some entertaining off-site tape backups. After the backup was done, they would ship the tapes off-site. If you needed to restore, you had to put in a requisition to get the tape found from the right box (assuming it was labelled correctly), shipped back and then you could start the restore. That's crazy. Having a disk array holding the backups connected to a single machine or a storage network makes a hell of a lot more sense. An off-site storage array can then be made to synchronise to that one. Effectively, take a backup of the backup. This way, the data is off-site in the case of catastrophic failure and quickly available in the case of minor failure. At Environment Canada, we had access to another site's tape array in another city. We would backup to a local disk array and then push all the data to a machine in Montr&#xE9;al. The array actually had some disk and it would page modified chunks of disk out to tape. Although it was an extremely expensive piece of hardware, it could provide a huge quantity of storage that was mostly disk-fast on cheap tape.

My preference for the actual file format is loose files on disk. If not, `tar` or `cpio` archives, optionally compressed, are the next best solution. Odd-ball formats made by so many commercial backup software programs are to be avoided in my opinion unless they come with a rescue disc capable of starting restore without bending over backward to deliver the archive to them. Again, this is why I like loose files or a `tar`/`cpio` archive and a Knoppix CD. A word on `tar` versus `cpio`: it seems that `cpio` has fallen into obscurity by modern Linux users in favour of `tar`. Dust of your `cpio` because it has an excellent backup feature: `find`. Because `find` allows you to select a very precise file set and `cpio` will read this file set through a pipe, it becomes very nice to make incremental backups. It also allows you to have all your backups relative to the root directory without any trouble.

Also, if the backup agent fails during the backup process, it should not destroy the old backup. In the case of `tar`, that means outputting to a temporary location and then moving the new archive over the old one so that if anything happened during the backup, there is one full backup present.

A large concern is the size of backups. Doing a full backup is expensive in both time and space. My solution has been the `rsync --backup-dir=foo` method which will synchronise from my live copy to a backup and store all the old files it would delete of replace into _foo_. Effectively, I always have a full backup in a folder I name “current” and any changed or deleted files form yesterday get pushed into a folder with the current date. If the backup is interrupted, the backup is not an accurate reflection of the live copy, but it is complete. It's also quite disk efficient. I have a script which occasionally goes and deletes these old incrementals.

If `rsync` is not to your liking, then using something like: an archive of the full system once a month, an archive since the last full backup once a week, and an archive since the last partial backup once a day. This makes for a manageable collection of incremental files.

## Databases

Neglecting databases is also quite common. Any binary file that is held open is, for the purposes of backups, a database. You cannot guarantee that copying the files while the database is active will result in a coherent, usable backup. On Linux, the obvious offenders are OpenLDAP, Postgres, mySQL, BIND (when using dynamic DNS) and anything else holding on to Berkley DataBases when running. Most of these utilities provide commands to dump the entire database to a text file. The format of this text file is always easily re-loadable by the database. Storing the dumps though, especially daily, can take a fair bit of space. My solution has been to create an undo patch. I have a script (available at the end of this document) that accepts the database output from standard input and writes it to a temporary file. It then produces a patch from the temporary file to the current backup and moves the temporary file in place of the current backup. This means the latest version of the database is always available. If an older one is needed, applying the undo patches serially will allow you to backtrack to any point in the history of the database. This also means that the old patches can be deleted once they expire.

BIND is a pleasantly special case because `rndc freeze` and `rndc thaw` can bracket a purely file-based backup to handle its database. Freezing all zones at once is only allowed in BIND 9.4.0. In older versions, you'll have to freeze and thaw each zone one at a time.

## Automation

If a backup is not done automatically, it is unlikely that a human will do it. `cron` is your friend. Backups should happen automatically at night while you slumber. An external harddisk makes this fairly trivial. If you are a mobile user, backing up through a method like `rsync` is very bandwidth effective allowing an easy backup from a mobile client to a home system both on the Internet. I find my nightly backup pushes 20-40MB.

## My Tools

I use two scripts to do most of my backup magic. One is `backup` which wraps `rsync` and backups up a file tree to a remote or local directory. `undo-delta` takes a stream from standard input and creates an undo patch for my databases. I run it once using `slapcat` for LDAP and once with `pg_dumpall` for Postgres.

Since I occasionally make backups to my external Firewire harddisk, `backup` operates in two modes. The `-i` switch will produce incremental backups which I do regularly and I omit it when backing up to Firewire. After dealing with a very, very flakey wireless connection, I added `-b limit` to limit bandwidth. It can also take a `-e filespec` to exclude files in addition to my standard `*.nb` (no-backup), `.xsession*` and `Junk`. I store my backup in a format like this `backuproot/hostname/type`. The host name is alterable with the `-h` switch if the value from the `hostname` command is not correct. The _type_ will be the name of the directory for file backups, converted to title case, and can be optionally specified via the `-n` switch to the `undo-delta` script. `undo-delta` also takes `-e` switch to specify an optional file extension (e.g., ldif for LDAP and psql for Postgres).

The _backuproot_ must have a file `.backup` so the scripts can verify they do indeed have a valid backup medium.

`backup-prune` will walk through a backup root and delete any backups older than a specified window. It uses the names of the files, not timestamps, to determine the age of a backup.

This works on Linux and MacOS. I have not tested it on other platforms.

## Source Control as Backup

Source control is not backup. I've tried. It doesn't work. It's not forgiving enough to make unmonitored backups that are meaningful. Moreover, it doesn't prevent all the necessary modes of failure. There might be useful cases for keeping subsets of configurations in source control, but only if they are very source-like, in that they are updated only by humans and the updates can have meaningful commit messages.
