# FS3 Client Quickstart Guide
[![Chat on discord](https://discord.com/assets/cb48d2a8d4991281d7a6a95d2f58195e.svg)](https://discord.com/invite/KKGhy8ZqzK) [![Go Report Card](https://goreportcard.com/badge/minio/mc)](https://goreportcard.com/report/minio/mc) [![Docker Pulls](https://img.shields.io/docker/pulls/minio/mc.svg?maxAge=604800)](https://hub.docker.com/r/minio/mc/) [![license](https://img.shields.io/badge/license-AGPL%20V3-blue)](https://github.com/filswan/fs3-mc/blob/master/LICENSE)


FS3 Client (mc) provides a modern alternative to UNIX commands like ls, cat, cp, mirror, diff, find etc. It supports filesystems and Amazon S3 compatible cloud storage service (AWS Signature v2 and v4).

```
alias       set, remove and list aliases in configuration file
ls          list buckets and objects
mb          make a bucket
rb          remove a bucket
cp          copy objects
mirror      synchronize object(s) to a remote site
cat         display object contents
head        display first 'n' lines of an object
pipe        stream STDIN to an object
share       generate URL for temporary access to an object
find        search for objects
sql         run sql queries on objects
stat        show object metadata
mv          move objects
tree        list buckets and objects in a tree format
du          summarize disk usage recursively
retention   set retention for object(s)
legalhold   set legal hold for object(s)
diff        list differences in object name, size, and date between two buckets
rm          remove objects
encrypt    manage bucket encryption config
event       manage object notifications
watch       listen for object notification events
undo        undo PUT/DELETE operations
policy      manage anonymous access to buckets and objects
tag         manage tags for bucket(s) and object(s)
ilm         manage bucket lifecycle
version     manage bucket versioning
replicate   configure server side bucket replication
admin       manage MinIO servers
update      update mc to latest release
list        list swan info
car         generate car file for filecoin offline deal
send        send filecoin deal
```

## Install from Source
Source installation is only intended for developers and advanced users. If you do not have a working Golang environment, please follow [How to install Golang](https://golang.org/doc/install). Minimum version required is [go1.13](https://golang.org/dl/#stable)


```sh
# get submodules
git submodule update --init --recursive
make ffi
GO111MODULE=on go get github.com/filswan/fs3-mc
go build -o ./mc
```

## Add a Cloud Storage Service
If you are planning to use `mc` only on POSIX compatible filesystems, you may skip this step and proceed to [everyday use](#everyday-use).

To add one or more Amazon S3 compatible hosts, please follow the instructions below. `mc` stores all its configuration information in ``~/.mc/config.json`` file.

```
./mc alias set <ALIAS> <YOUR-S3-ENDPOINT> <YOUR-ACCESS-KEY> <YOUR-SECRET-KEY> --api <API-SIGNATURE> --path <BUCKET-LOOKUP-TYPE>
```

<ALIAS> is simply a short name to your cloud storage service. S3 end-point, access and secret keys are supplied by your cloud storage provider. API signature is an optional argument. By default, it is set to "S3v4".

Path is an optional argument. It is used to indicate whether dns or path style url requests are supported by the server. It accepts "on", "off" as valid values to enable/disable path style requests.. By default, it is set to "auto" and SDK automatically determines the type of url lookup to use.

### Example - MinIO Cloud Storage
MinIO server displays URL, access and secret keys.

```
./mc alias set minio http://192.168.1.51 BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12
```

### Example - Amazon S3 Cloud Storage
Get your AccessKeyID and SecretAccessKey by following [AWS Credentials Guide](http://docs.aws.amazon.com/general/latest/gr/aws-security-credentials.html).

```
./mc alias set s3 https://s3.amazonaws.com BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12
```

**Note**: As an IAM user on Amazon S3 you need to make sure the user has full access to the buckets or set the following restricted policy for your IAM user

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowBucketStat",
            "Effect": "Allow",
            "Action": [
                "s3:HeadBucket"
            ],
            "Resource": "*"
        },
        {
            "Sid": "AllowThisBucketOnly",
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::<your-restricted-bucket>/*",
                "arn:aws:s3:::<your-restricted-bucket>"
            ]
        }
    ]
}
```

### Example - Google Cloud Storage
Get your AccessKeyID and SecretAccessKey by following [Google Credentials Guide](https://cloud.google.com/storage/docs/migrating?hl=en#keys)

```
./mc alias set gcs  https://storage.googleapis.com BKIKJAA5BMMU2RHO6IBB V8f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12
```

## Test Your Setup
`mc` is pre-configured with https://play.min.io, aliased as "play". It is a hosted MinIO server for testing and development purpose.  To test Amazon S3, simply replace "play" with "s3" or the alias you used at the time of setup.

*Example:*

List all buckets from https://play.min.io

```
mc ls play
[2016-03-22 19:47:48 PDT]     0B my-bucketname/
[2016-03-22 22:01:07 PDT]     0B mytestbucket/
[2016-03-22 20:04:39 PDT]     0B mybucketname/
[2016-01-28 17:23:11 PST]     0B newbucket/
[2016-03-20 09:08:36 PDT]     0B s3git-test/
```

Make a bucket
`mb` command creates a new bucket.

*Example:*
```
./mc mb play/mybucket
Bucket created successfully `play/mybucket`.
```

Copy Objects
`cp` command copies data from one or more sources to a target.

Note

*Example:*
```
./mc cp myobject.txt play/mybucket
myobject.txt:    14 B / 14 B  ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓  100.00 % 41 B/s 0
```

## How to use    

#### Prepare your environment
 - A running lotus node at local
 - A filecoin wallet with sufficient balance to send deal, set as environment variable $FIL_WALLET
 - FS3 Server credentials set as environment variables $ENDPOINT, $ACCESS_KEY, $SECRET_KEY

### Upload files to FS3 server
#### Export environment variables
Environment variables `ENDPOINT`, `ACCESS_KEY`, `SECRET_KEY` need to be set for FS3 Server authorization and usage.
``` bash 
# export FS3 Serve Endpoint
export ENDPOINT=<FS3_SERER_ENDPOINT>

# export FS3 Server Access Key
export ACCESS_KEY=<FS3_SERVER_ACCESS_KEY>

# export FS3 Server SECRET Key
export SECRET_KEY=<FS3_SERVER_SECRET_KEY>
```

Example: 
``` bash 
# export FS3 Serve Endpoint
export ENDPOINT=http://FS3_SERVER_URL:PORT

# export FS3 Server Access Key
export ACCESS_KEY=minioadmin

# export FS3 Server SECRET Key
export SECRET_KEY=minioadmin
```

#### Upload local files to FS3 Server

Note: The FS3 server must be running when doing the uploading process. Make sure it is runing before uploading files.

You can simply upload a local file to the designated bucket of FS3 Server using `mc` with `cp`
``` bash 
./mc cp PATH/TO/LOCAL/FILE minio/BUCKET
```

Example:
``` bash 
# Upload test.txt to bucket 'test_bucket'
./mc cp /tmp/test/test.txt minio/test_bucket
```

### Send online deals
you may send an online deal to a miner

#### Export environment variables
A wallet address is a must for sending deals to miner. You can change it via setting environment variable `FIL_WALLET`. Or you can set it up as an augment passing to `mc` when sending deals.
``` bash 
# export wallet address
export FIL_WALLET=<MY_FIL_WALLET>
```



#### Import file stored in FS3
Import the file stored in FS3 to Filecoin, then you can share it to your miner

```bash
./mc import --bucket [bucket] --object [object]
```

For example:
```bash
./mc import --bucket testBucket --object test.zip
```

Note:
The `defaultVolumeAddress` where fs3 store the uploaded data is `~/minio-data`. It can be changed in file `fs3-mc/cmd/Config-v10.go`

A message that contains `Bucket`,`Object` and `Datacid` will be returned if successful.

#### Send online deal to miner
`sendonline` command can send an online deal to a designated miner, a fully synchronized lotus node at local is required

```
--from: (optional) specify filecoin wallet to use, default: $FIL_WALLET
--verified-deal: specify whether deal is verified, default: "false" ('true' if verified)
--fast-retrieval: specify data retrieval type, defalult: 'true' ('false' if not using fast retrieval approach)   
--data-cid: specify the valid data-cid for sending deal
--miner-id: specify which miner to send deal to
--price: specify the deal price for each GiB of file, default: 0
--duration: specify length in day to store the file, default: 1036800
```

*Example:*    
    
```bash
./mc sendonline --from nusx7m3exqsfkxezncpefsf6fmian --verified-deal false --fast-retrieval true --data-cid m7xmefllqsixl5 --miner-id t00001 --price 0.00005 --duration 1036800
```

A message that contains all the deal information and `Dealcid` will be returned if successful.
<a name="everyday-use"></a>
## Everyday Use

### Shell aliases
You may add shell aliases to override your common Unix tools.

```
alias ls='mc ls'
alias cp='mc cp'
alias cat='mc cat'
alias mkdir='mc mb'
alias pipe='mc pipe'
alias find='mc find'
```

### Shell autocompletion
In case you are using bash, zsh or fish. Shell completion is embedded by default in `mc`, to install auto-completion use `mc --autocompletion`. Restart the shell, mc will auto-complete commands as shown below.

```
mc <TAB>
admin    config   diff     find     ls       mirror   policy   session  sql      update   watch
cat      cp       event    head     mb       pipe     rm       share    stat     version
```

