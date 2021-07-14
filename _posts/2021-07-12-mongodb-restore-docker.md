---
layout: post
title: "Restore large mogodb backup in Docker"
---

Recently, I wanted to restore a mongodb backup of a bigger production Database locally for testing. 

The container was created with a standard docker run command: `docker run -d -p 27017:27017 -v {PATH_TO_FOLDER}:/data-test -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin --name test-mongo mongo:4.4.3`.


## Problem
First i tried to restore the backup from an archive.
`docker exec -i test-mongo mongorestore --username=admin --password=admin --authenticationDatabase=admin --gzip --archive=/data-test/backup.archive`.
Some Data was missing and at first I was confused because there was no error message in the mongorestore logs:

```
2021-07-12T16:21:24.593+0000    [........................]  test_database.test_collection  0B/2.34GB  (0.0%)
2021-07-12T16:21:27.593+0000    [........................]  test_database.test_collection  0B/2.34GB  (0.0%)
```

Turns out, only a subset of the data was acutally restored. Only aroung 800 of the expected 3100 objects inside the collection were present.

Even with the most verbose logging with the `-vvvvv`-option, there was no information hinting at the actual problem.

I also tried dumping the bson and importing that but the same problem seems to happen and the logging was also not helpful.
`docker exec -i test-mongo mongorestore --username=admin --password=admin --authenticationDatabase=admin -vvvvv --numParallelCollections 1 --drop /data-import/bson`

When observing `docker stats` it quickly became obvious, that the restore was using all of the RAM the Docker daemon was allowed to use.

```
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O        PIDS
62b20a51e8b2   test-mongo      70.03%    1.782GiB / 2.676GiB   66.58%    1.01kB / 0B   752MB / 32.5MB   46
```

## Solution

The amount of usable RAM needs to be increased. In Docker Desktop with Hyper-V Backend, go to the Resources Section in the Settings and increase the maximum memory. In my case, in order to restore the 2.34GB backup, aroung 7 GB of RAM were needed.

![Docker Desktop Memory Settings](/public/2021-07-12-mongodb-restore-docker/settings-resources.png)

When running Docker desktop with the WSL2 Backend this issue does not occur with the default settings, as Docker is allowed to use a lot of memory (in my case 12,36GB out of a total 16).
If you have limited the RAM to be used by WSL in a `.wslconfig` (located in the `%HOMEPATH%` directory, change the memory setting accordingly.

```
[wsl2]
memory=7GB
```
and than run `Restart-Service LxssManager` in an admin powershell to load the changed config.

