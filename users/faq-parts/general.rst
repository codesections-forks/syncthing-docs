General
=======

What is Syncthing?
------------------

Syncthing is an application that lets you synchronize your files across multiple
devices. This means the creation, modification or deletion of files on one
machine will automatically be replicated to your other devices. We believe your
data is your data alone and you deserve to choose where it is stored. Therefore
Syncthing does not upload your data to the cloud but exchanges your data across
your machines as soon as they are online at the same time.

Is it "syncthing", "Syncthing" or "SyncThing"?
----------------------------------------------

It's **Syncthing**, although the command and source repository is spelled
``syncthing`` so it may be referred to in that way as well. It's definitely not
:strike:`SyncThing`, even though the abbreviation ``st`` is used in some
circumstances and file names.

What things are synced?
-----------------------

The following things are *always* synchronized:

-  File contents
-  File modification times

The following may be synchronized or not, depending:

-  File permissions (when supported by file system; on Windows only the
   read only bit is synchronized)
-  Symbolic links (synced, except on Windows, but never followed)
-  File or directory owners and groups (when enabled)
-  Extended attributes (when enabled)
-  POSIX or NFS ACLs (as part of extended attributes)

The following are *not* synchronized;

-  Directory modification times (not preserved)
-  Hard links (followed, not preserved)
-  Windows junctions (synced as ordinary directories; require enabling in
   :stconf:opt:`the configuration <folder.junctionsAsDirs>` on a per-folder
   basis)
-  Resource forks (not preserved)
-  Windows ACLs (not preserved)
-  Devices, FIFOs, and other specials (ignored)
-  Sparse file sparseness (will become sparse, when supported by the OS & filesystem)

What does syncing mean for links?
---------------------------------

As mentioned above, `symbolic links
<https://en.wikipedia.org/wiki/Symbolic_link>`__ are synced but not followed
(except on Windows, where they're not synced). To see what this means, consider
the situation where you have a symbolic link at ``./Sync/symlink`` that points
to ``../some-file.txt``.  If Syncthing syncs the ``./Sync`` directory, then it
will sync ``./Sync/symlink`` *as a link* – that is, Syncthing will replicate the
link on every synced device. But, because ``../some-file.txt`` isn't part of the
synced directory, Syncthing won't sync ``some-file.txt`` (that's what "not
followed" means).  Thus, the ``./Sync/symlink`` file will point to whatever
(non-synced) file is at ``../some-file.txt`` on any particular device.  If no
such file exists, ``./Sync/symlink`` will be a broken symlink.

The situation for `hard links <https://en.wikipedia.org/wiki/Hard_link>`__ is
nearly reversed: Syncthing does not sync hard links but *does* sync the
underlying file (which is implied by the nature of hard links, which effectively
are the file).  For example, if you have a hard link at ``./Sync/hardlink`` that
points to a file at ``../some-file.txt``, then a sync of the ``./Sync``
directory will replicate the contents of ``../some-file.txt`` to
`./Sync/hardlink` on every device.  But it will *not* sync the fact that
``./Sync/hardlink`` was a link on the original device – on other devices,
``./Sync/hardlink`` will be completely independent from ``../some-file.txt`` (if
that file exists).

All of this results in a general rule of thumb: if Syncthing is syncing both a
link and its target, it's better to use a symbolic link.  However, if the link
points to a file that is *not* in a synced directory and you want to sync the
contents of that file, you should use a hard link instead.

Is synchronization fast?
------------------------

Syncthing segments files into pieces, called blocks, to transfer data from one
device to another. Therefore, multiple devices can share the synchronization
load, in a similar way to the torrent protocol. The more devices you have online,
the faster an additional device will receive the data
because small blocks will be fetched from all devices in parallel.

Syncthing handles renaming files and updating their metadata in an efficient
manner. This means that renaming a file will not cause a retransmission of
that file. Additionally, appending data to existing files should be handled
efficiently as well.

:ref:`Temporary files <temporary-files>` are used to store partial data
downloaded from other devices. They are automatically removed whenever a file
transfer has been completed or after the configured amount of time which is set
in the configuration file (24 hours by default).

How does Syncthing differ from BitTorrent/Resilio Sync?
-------------------------------------------------------

The two are different and not related. Syncthing and BitTorrent/Resilio Sync accomplish
some of the same things, namely syncing files between two or more computers.

BitTorrent Sync, now called Resilio Sync, is a proprietary peer-to-peer file
synchronization tool available for Windows, Mac, Linux, Android, iOS, Windows
Phone, Amazon Kindle Fire and BSD. [#resiliosync]_ Syncthing is an open source file
synchronization tool.

Syncthing uses an open and documented protocol, and likewise the security
mechanisms in use are well defined and visible in the source code. Resilio
Sync uses an undocumented, closed protocol with unknown security properties.

.. [#resiliosync] https://en.wikipedia.org/wiki/Resilio_Sync

Is there an iOS client?
-----------------------

There are no plans by the current Syncthing team to officially support iOS in the foreseeable future.

iOS has significant restrictions on background processing that make it very hard to
run Syncthing reliably and integrate it into the system.

However, there is a commercial packaging of Syncthing for iOS that attempts to work within these limitations. [#mobiussync]_

.. [#mobiussync] https://www.mobiussync.com

Should I keep my device IDs secret?
-----------------------------------

No. The IDs are not sensitive. Given a device ID it's possible to find the IP
address for that device, if global discovery is enabled on it. Knowing the device
ID doesn't help you actually establish a connection to that device or get a list
of files, etc.

For a connection to be established, both devices need to know about the other's
device ID. It's not possible (in practice) to forge a device ID. (To forge a
device ID you need to create a TLS certificate with that specific SHA-256 hash.
If you can do that, you can spoof any TLS certificate. The world is your
oyster!)

.. seealso::
    :ref:`device-ids`
