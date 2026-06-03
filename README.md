# GoVFS - Go Virtual File System

GoVFS is a full-stack simulation of an EXT2/EXT3 file system, built from scratch. The system emulates disk management, partitioning, journaling, user permissions, and file operations - all backed by a binary `.dsk` file written in Go and exposed through a Web Interface.

## Overview 

This project simulates a complete EXT2/EXT3 file system on top of binary disk image files (`.dsk`). It supports:
- Virtual disk creation and management
- MBR and EBR partition tables
- EXT2 and EXT3 formatting with superblock, bitmaps, inodes, and data blocks
- Journaling and crash recovery for EXT3
- User & group management with UGO permission enforcement
- Full file system operations: create, read, edit, move, copy, remove, rename, find
- A web-based GUI for visual exploration of the file system
- Graphviz reports for every internal structure

A detailed explanation of the functionality can be read [here](./docs/details.md)

## Features

### Screen 1 — Command Terminal
Execute any supported command directly from the browser. Output is streamed back in real time, mimicking a terminal experience.

### Screen 2 — File System Explorer
Navigate the virtual file system visually:

1. Select a disk (.dsk)
2. Select a partition
3. Log in with valid credentials
4. Browse directories and view file contents
5. Log out when done

### Screen 3 — Reports Viewer
View all Graphviz-generated reports (MBR, DISK, inode, block, tree, superblock, journaling, etc.) rendered inline in the browser.

## Architecture
All `.dsk` disk images are stored as regular binary files on the host, mounted into the container.

## Tech Stack
| Layer      | Technology        |
|------------|-------------------|
| Frontend   | React, TypeScript |
| Backend    | Go                |
| Reports    | Graphviz          |
| Containers | Docker            |

## Getting Started
### Prerequisites
- Docker
- Docker Compose

Clone the repository

```bash
git clone https://github.com/sa-larios22/govfs.git
cd govfs
```

Start the project

```bash
docker compose build
docker compose up -d
```

Stop the project

```bash
docker compose stop
```

Disk image files `.dsk` are persisted in the `./disks` directory on your host machine and survive container restarts.

## Usage
### Command Reference
All commands are case-insensitive. Parameters can appear in any order. Strings with spaces must be wrapped in double quotes.

### Disk Management
| Command                                                                                                                   | Description                       |
|---------------------------------------------------------------------------------------------------------------------------|-----------------------------------|
| `mkdisk -size=<n> [-unit=K\|M] [-fit=BF\|FF\|WF]`                                                                         | Create a new virtual disk         |
| `rmdisk -driveletter=<A-Z>`                                                                                               | Delete a virtual disk             |
| `fdisk -driveletter=<L> -name=<n> -size=<n> [-type=P\|E\|L] [-unit=B\|K\|M] [-fit=BF\|FF\|WF] [-delete=full] [-add=<n>]`  | Manage partitions                 |
| `mount -driveletter=<L> -name=<n>`                                                                                        | Mount a partition                 |
| `unmount -id=<id>`                                                                                                        | Unmount a partition               |
| `mkfs -id=<id> [-type=full] [-fs=2fs\|3fs]`                                                                               | Format a partition (EXT2 or EXT3) |

#### Examples
```bash
# Create a 3000 KB disk
mkdisk -size=3000 -unit=K

# Create a 300 KB primary partition on disk A
fdisk -size=300 -driveletter=A -name=Partition1

# Mount and format as EXT3
mount -driveletter=A -name=Partition1
mkfs -id=A118 -fs=3fs
```

### Session Management
 
| Command | Description |
|---------|-------------|
| `login -user=<u> -pass=<p> -id=<id>` | Start a session on a mounted partition |
| `logout` | End the current session |
 
---
 
### User & Group Management
*(Requires active session; most commands require `root`.)*
 
| Command | Description |
|---------|-------------|
| `mkgrp -name=<n>` | Create a group |
| `rmgrp -name=<n>` | Delete a group |
| `mkusr -user=<u> -pass=<p> -grp=<g>` | Create a user |
| `rmusr -user=<u>` | Delete a user |
| `chgrp -user=<u> -grp=<g>` | Change a user's group |
 
---
 
### File System Operations
*(Requires active session.)*
 
| Command | Description |
|---------|-------------|
| `mkfile -path=<p> [-r] [-size=<n>] [-cont=<src>]` | Create a file |
| `mkdir -path=<p> [-r]` | Create a directory |
| `cat -file1=<p> [-file2=<p> ...]` | Display file contents |
| `remove -path=<p>` | Remove a file or directory |
| `edit -path=<p> -cont=<src>` | Edit a file's content |
| `rename -path=<p> -name=<new>` | Rename a file or directory |
| `copy -path=<src> -destino=<dst>` | Copy a file or directory |
| `move -path=<src> -destino=<dst>` | Move a file or directory |
| `find -path=<p> -name=<pattern>` | Search by name (`?` = one char, `*` = many) |
| `chown -path=<p> -user=<u> [-r]` | Change owner |
| `chmod -path=<p> -ugo=<nnn> [-r]` | Change permissions (octal UGO) |
 
---
 
### EXT3 Recovery
 
| Command | Description |
|---------|-------------|
| `loss -id=<id>` | Simulate data loss (clears bitmaps + inode/block areas) |
| `recovery -id=<id>` | Recover file system from journaling |
 
---
 
### Scripting & Reports
 
```bash
# Run a script file
execute -path=/home/Desktop/setup.sdaa
 
# Generate a report
rep -id=A118 -path=/home/reports/report.jpg -name=<type>
```
 
Available report types: `mbr`, `disk`, `inode`, `block`, `bm_inode`, `bm_block`, `tree`, `sb`, `file`, `ls`, `journaling`