---
layout: post
title: "Restore large mogodb backup"
---

Recently, I wanted to restore a mongodb backup of a bigger production Database locally for testing. 
Some Data was missing and at first I was confused because there was no error message in the mongorestore logs:



Only a subset of the data was acutally restored.

`docker run -d -p 27017:27017 -v C:\Users\dechert\Projekte\qu4lity-data-management\data-test:/data-test -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin --name test-mongo mongo:4.4.3`


## Problem

First i tried to restore the backup from an archive 
`docker exec -i test-mongo mongorestore --verbose --username=admin --password=admin --authenticationDatabase=admin --gzip --archive=/data-test/qu4lity_backup_2021-07-09_171016.archive`.

docker exec -i test-mongo mongorestore -vvvvv --username=admin --password=admin --authenticationDatabase=admin --gzip --archive=/data-test/qu4lity_backup_2021-07-09_171016.archive

Even with the most verbose logging, there was no information hinting at the actual problem.

I also tried dumping the bson and importing that but the same problem seems to happen.



```
2021-07-12T16:21:24.593+0000    [........................]  test_database.test_collection  0B/2.34GB  (0.0%)
2021-07-12T16:21:27.593+0000    [........................]  test_database.test_collection  0B/2.34GB  (0.0%)
```

When observing `docker stats` it quickly became obvious, that the restore was using all of the RAM the Docker daemon was allowed to use.

```
CONTAINER ID   NAME            CPU %     MEM USAGE / LIMIT     MEM %     NET I/O       BLOCK I/O        PIDS
62b20a51e8b2   test-mongo      70.03%    1.782GiB / 2.676GiB   66.58%    1.01kB / 0B   752MB / 32.5MB   46
```


## Solution

The amount of usable RAM needed to be increased