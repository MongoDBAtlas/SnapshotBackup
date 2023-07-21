<img src="https://companieslogo.com/img/orig/MDB_BIG-ad812c6c.png?t=1648915248" width="50%" title="Github_Logo"/> <br>


# MongoDB Atlas Snapshot S3 backup

### [&rarr; Atlas CLI](#atlascli)
### [&rarr; AWS Integration](#aws)
### [&rarr; Bucket Configuration](#bucket)
### [&rarr; Create export Job](#job)
### [&rarr; Import Data from S3](#import)


<br>

### AtlasCLI

Atlas CLI를 다운로드 후 설치 하여 줍니다.
https://www.mongodb.com/try/download/atlascli


Atlas와 연결을 위해서는 설정을 하여야 하며 API Key, Organization ID, Project ID가 필요 합니다.   

발급 받은 API Key, Organization ID, Project ID를 입력 하여 줍니다.

````
~ % atlas config init        
You are configuring a profile for atlas.

All values are optional and you can use environment variables (MONGODB_ATLAS_*) instead.

Enter [?] on any option to get help.

? Public API Key: biyqu***
? Private API Key: [? for help] ************************************
There was an error fetching your organizations: https://cloud.mongodb.com/api/atlas/v2/orgs GET: HTTP 403 (Error code: "ORG_REQUIRES_ACCESS_LIST") Detail: This organization requires access through an access list of ip ranges. Reason: Forbidden. Params: [175.196.74.**]
? Do you want to enter the Organization ID manually? Yes
? Default Org ID: 61357268d1361******
There was an error fetching your projects: This organization requires access through an access list of ip ranges.
? Do you want to enter the Project ID manually? Yes
? Default Project ID: 6271da22e663051*****
Skipping default project setting
? Default Output Format: json

Your profile is now configured.
You can use [atlas config set] to change these settings at a later time.

A new version of atlascli is available 'v1.9.1'!
To upgrade, see: https://dochub.mongodb.org/core/install-atlas-cli.

To disable this alert, run "atlas config set skip_update_check true".

````

연결 테스트를 위해 Project 내에 데이터 베이스 리스트를 조회 하여 봅 니다.


````

~ % atlas clusters list                         
{
  "links": [
    {
      "href": "https://cloud.mongodb.com/api/atlas/v2/groups/6271da22e66****/clusters?includeCount=true\u0026pageNum=1\u0026itemsPerPage=100",
      "rel": "self"
    }
  ],
  "results": [
    {
      "backupEnabled": false,
      "biConnector": {
        "enabled": false,
        "readPreference": "secondary"
      },
      "clusterType": "REPLICASET",
      "connectionStrings": {
        "standard": "mongodb://ac-ysmyzo7-shard-***.tp**.mongodb.net:27017,ac-ysmyzo7-shard-***.tp**.mongodb.net:27017,ac-ysmyzo7-shard-***.tp**.mongodb.net:27017/?ssl=true\u0026authSource=admin\u0026replicaSet=atlas-p9f785-shard-0",
        "standardSrv": "mongodb+srv://cluster0.tp**.mongodb.net"
      },
      "createDate": "2023-07-15T15:03:57Z",
      "diskSizeGB": 0.5,
      "encryptionAtRestProvider": "NONE",
      "groupId": "6271da22e6630****",
      "id": "64b2b55d9dcb2***",
      "mongoDBMajorVersion": "6.0",
      "mongoDBVersion": "6.0.6",
      "name": "Cluster0",
      "paused": false,
      "pitEnabled": false,
      "replicationSpecs": [
        {
          "id": "6271da22e6630****",
          "numShards": 1,
          "regionConfigs": [
            {
              "electableSpecs": {
                "instanceSize": "M0"
              },
              "priority": 7,
              "providerName": "TENANT",
              "regionName": "AP_NORTHEAST_2",
              "backingProviderName": "AWS"
            }
          ],
          "zoneName": "Zone 1"
        }
      ],
      "rootCertType": "ISRGROOTX1",
      "stateName": "IDLE",
      "terminationProtectionEnabled": false,
      "versionReleaseSystem": "LTS"
    }
  ],
  "totalCount": 1
}

A new version of atlascli is available 'v1.9.1'!
To upgrade, see: https://dochub.mongodb.org/core/install-atlas-cli.

To disable this alert, run "atlas config set skip_update_check true".
````

### AWS

Project의 설정 화면에서 Integration에서 AWS를 선택 합니다.   

<img src="/images/image01.png" width="90%" height="90%">     

이후 Authorize Atlas to Access your AWS account 에서 Authorize WAS IAM Role을 클릭 합니다.

AWS와 연계를 위한 설정으로 이를 실행 하여 줍니다. 

실행해야 할 Script를 실행 하여 줍니다.

Create New Role with the AWS CLI

````
% cat role-trust-policy.json 
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::53672772**:root"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "66e6b9ac-2a33-4cac-ba54-2***f"
        }
      }
    }
  ]
}

````

생성할 Role 이름을 입력 하고 이를 AWS cli 를 이용하여 실행 하여 줍니다.

````
~ % aws iam create-role \
 --role-name sa-korea-pov \
 --assume-role-policy-document file://role-trust-policy.json
{
    "Role": {
        "Path": "/",
        "RoleName": "sa-korea-pov",
        "RoleId": "AROA6IESFAO***",
        "Arn": "arn:aws:iam::979559**:role/sa-korea-pov",
        "CreateDate": "2023-07-15T15:19:26+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "AWS": "arn:aws:iam::53672772**:root"
                    },
                    "Action": "sts:AssumeRole",
                    "Condition": {
                        "StringEquals": {
                            "sts:ExternalId": "66e6b9ac-2a33-4cac-ba54-2**"
                        }
                    }
                }
            ]
        }
    }
}
````

실행 후 생성된 Role 의 ARN 정보를 입력 하여 줍니다.   

<img src="/images/image02.png" width="90%" height="90%">     

Validate and Finish 로 확인 하여 줍니다.

완료되면 생성된 Role 정보와 성공 메시지를 볼 수 있습니다.   

<img src="/images/image03.png" width="90%" height="90%">     


### Bucket

생성한 Role 에 bucket 에 대한 업로드 권한이 있어야 합니다. 다음 Permission을 role에 줍니다.

````
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetBucketLocation",
      "Resource": "arn:aws:s3:::<<bucket-name>>"
    },
    {
      "Effect": "Allow",
      "Action": "s3:PutObject",
      "Resource": "arn:aws:s3:::<<bucket-name>>/*"
    }
  ]
}
````

Role에 생성한 Permission 입니다.

<img src="/images/image04.png" width="90%" height="90%">     


### Job

우선 Bucket을 등록 하여야 합니다. 등록을 위한 Command는 다음과 같습니다.  

https://www.mongodb.com/docs/atlas/cli/stable/command/atlas-backups-exports-buckets-create/

<img src="/images/image05.png" width="90%" height="90%">    

필요한 값으로 iamRoleId가 필요 합니다. 값은 다음 명령어로 얻을 수 있습니다.   

https://www.mongodb.com/docs/atlas/cli/stable/command/atlas-cloudProviders-accessRoles-list/

````
~ % atlas cloudProviders accessRoles list                                                                     
{
  "awsIamRoles": [
    {
      "atlasAWSAccountArn": "arn:aws:iam::5367277****:root",
      "atlasAssumedRoleExternalId": "66e6b9ac-2a33-4cac-ba54-2****",
      "authorizedDate": "2023-07-15T15:27:21Z",
      "createdDate": "2023-07-15T15:14:42Z",
      "featureUsages": [
        {
          "featureId": {
            "exportBucketId": "64b3352e1f35****",
            "groupId": "6271da22e6630***"
          },
          "featureType": "EXPORT_SNAPSHOT"
        }
      ],
      "iamAssumedRoleArn": "arn:aws:iam::9795***:role/sa-korea-pov",
      "roleId": "64b2b7e279d5e11****",
      "providerName": "AWS"
    }
  ]
}

````

결과 중 roleId를 iamRoleId로 입력 하여 줍니다.  


````
~ % atlas backups exports buckets create kdja** --cloudProvider AWS --iamRoleId 64b2b7e279d5e11****
{
  "_id": "64b3352e1f35b2****",
  "bucketName": "kd***",
  "cloudProvider": "AWS",
  "iamRoleId": "64b2b7e279d5e11****"
}

````

생성한 bucket에 대한 정보를 조회 하는것 입니다.

````
~ % atlas backups exports buckets list                                                         
{
  "links": [
    {
      "href": "https://cloud.mongodb.com/api/atlas/v2/groups/6271da22e66305***/backup/exportBuckets?includeCount=true\u0026pageNum=1\u0026itemsPerPage=100",
      "rel": "self"
    }
  ],
  "results": [
    {
      "_id": "64b3352e1f35b27***",
      "bucketName": "kd****",
      "cloudProvider": "AWS",
      "iamRoleId": "64b2b7e279d5e11***"
    }
  ],
  "totalCount": 1
}
````


Export Job을 생성 하여 줍니다.  

https://www.mongodb.com/docs/atlas/cli/stable/command/atlas-backups-exports-jobs-create/


필요한 파라미터 중 snapshotId가 필요 합니다. 이를 얻기 위해 다음 명령어를 실행 하여 줍니다.  

Snapshot list 보기

https://www.mongodb.com/docs/atlas/cli/stable/command/atlas-backups-snapshots-list/

````
~ % atlas backups snapshots list <<clusterName>> 
{
  "links": [
    {
      "href": "https://cloud.mongodb.com/api/atlas/v2/groups/6271da22e6630***/clusters/woowabro/backup/snapshots?includeCount=true\u0026pageNum=1\u0026itemsPerPage=100",
      "rel": "self"
    }
  ],
  "results": [
    {
      "cloudProvider": "AWS",
      "createdAt": "2023-07-15T22:32:52Z",
      "expiresAt": "2023-07-17T22:35:13Z",
      "frequencyType": "hourly",
      "id": "64b31e47c40e886d5ea****",
      "links": [
        ...
      ],
      "mongodVersion": "6.0.6",
      ...
    },
    {
      "cloudProvider": "AWS",
      "createdAt": "2023-07-15T16:52:48Z",
      "description": "aaa",
      "expiresAt": "2023-07-18T16:55:12Z",
      "frequencyType": "ondemand",
      "id": "64b2ce8a49388c05***",
      "links": [
        ...
      ],
      "mongodVersion": "6.0.6",
      ...
    }
  ],
  "totalCount": 2
}

````

Export 대상이 되는 snapshot의 id를 사용 합니다.  


````
~ % atlas backups exports jobs create --bucketId 64b3352e1f35b2** --clusterName woowabro --snapshotId 64b2ce8a49388***
{
  "createdAt": "2023-07-16T00:28:35Z",
  "exportBucketId": "64b3352e1f35b2***",
  "exportStatus": {
    "exportedCollections": 0,
    "totalCollections": 0
  },
  "id": "64b339b360878171**",
  "prefix": "exported_snapshots/61357268d136197***/6271da22e6630***/woowabro/2023-07-15T1652/1689467312",
  "snapshotId": "64b2ce8a49388***",
  "state": "Queued"
}
````

작업이 완료되면 다음과 같이 S3에 데이터가 적재 되어 있는 것을 확인 할 수 있습니다. 

<img src="/images/image07.png" width="90%" height="90%">   

적재 되는 데이터 형태는 json.gz으로 생성 되며 완료가 되면 컬렉션 별로 .complete 파일이 생성 됩니다.

### Import

데이터 Import 는 S3의 데이터를 다운로드 후에 import 하는 방법 입니다.    
우선 데이터를 다운로드 하는 하는 것이 필요 합니다. 다운로드 후 zip 압축을 해제 합니다. 



````
woowabro % aws s3 cp s3://kdja**/exported_snapshots/61357268d1361978***/6271da22e663051***/woowabro/2023-07-20T0622/1689835315/  . --recursive
download: s3://kdja**/exported_snapshots/61357268d1361978***/6271da22e663051***/woowabro/2023-07-20T0622/1689835315/.complete to ./.complete
download: s3://kdja**/exported_snapshots/61357268d1361978***/6271da22e663051***/woowabro/2023-07-20T0622/1689835315/woowahan/datacol/metadata.json to woowahan/datacol/metadata.json
download: s3://kdja**/exported_snapshots/61357268d1361978***/6271da22e663051***/woowabro/2023-07-20T0622/1689835315/woowahan/datacol/atlas-98ztan-shard-0.0.json.gz to woowahan/datacol/atlas-98ztan-shard-0.0.json.gz

woowabro % gunzip -r *
gunzip: woowahan/datacol/metadata.json: unknown suffix -- ignored

````

import를 위해 다음 명령어를 생성 하여 줍니다.

```` massimport.sh

#!/bin/bash
regex='/(.+)/(.+)/.+'
dir=${1%/}
find $dir -type f -not -path '*/\.*' -not -path '*metadata\.json' | while read line ; do
  [[ $line =~ $regex ]]
  db_name=$database
  col_name=${BASH_REMATCH[2]}
  mongoimport --uri "$2" --mode=upsert -d $db_name -c $col_name --file $line --type json
done

````

다운받은 파일 폴더에서 다음과 같이 실행 하여 줍니다.  

```` 
woowabro % export database=woowahan
woowabro % ./massimport.sh . "mongodb+srv://admin:*****@serverless.9g***.mongodb.net/"
2023-07-21T11:26:37.753+0900	error parsing command line options: expected argument for flag `-c, --collection', but got option `--file'
2023-07-21T11:26:37.753+0900	try 'mongoimport --help' for more information
2023-07-21T11:26:38.684+0900	connected to: mongodb+srv://[**REDACTED**]@serverless.9g***.mongodb.net/
2023-07-21T11:26:41.684+0900	[........................] woowahan.datacol	264KB/24.6MB (1.0%)
2023-07-21T11:26:44.685+0900	[........................] woowahan.datacol	515KB/24.6MB (2.0%)
2023-07-21T11:26:47.684+0900	[........................] woowahan.datacol	765KB/24.6MB (3.0%)
2023-07-21T11:26:50.685+0900	[........................] woowahan.datacol	765KB/24.6MB (3.0%)
2023-07-21T11:26:53.684+0900	[........................] woowahan.datacol	1016KB/24.6MB (4.0%)
2023-07-21T11:26:56.684+0900	[#.......................] woowahan.datacol	1.24MB/24.6MB (5.0%)
2023-07-21T11:26:59.684+0900	[#.......................] woowahan.datacol	1.73MB/24.6MB (7.0%)
2023-07-21T11:27:02.684+0900	[#.......................] woowahan.datacol	1.97MB/24.6MB (8.0%)
2023-07-21T11:27:05.684+0900	[##......................] woowahan.datacol	2.22MB/24.6MB (9.0%)
2023-07-21T11:27:08.684+0900	[##......................] woowahan.datacol	2.71MB/24.6MB (11.0%)
2023-07-21T11:27:11.685+0900	[###.....................] woowahan.datacol	3.45MB/24.6MB (14.0%)
2023-07-21T11:27:14.685+0900	[####....................] woowahan.datacol	4.19MB/24.6MB (17.0%)
2023-07-21T11:27:17.684+0900	[####....................] woowahan.datacol	4.92MB/24.6MB (20.0%)
2023-07-21T11:27:20.684+0900	[######..................] woowahan.datacol	6.15MB/24.6MB (25.0%)
2023-07-21T11:27:23.684+0900	[#######.................] woowahan.datacol	7.38MB/24.6MB (30.0%)
2023-07-21T11:27:26.684+0900	[########................] woowahan.datacol	8.86MB/24.6MB (36.0%)
2023-07-21T11:27:29.685+0900	[##########..............] woowahan.datacol	10.3MB/24.6MB (42.0%)
2023-07-21T11:27:32.684+0900	[###########.............] woowahan.datacol	11.8MB/24.6MB (48.0%)
2023-07-21T11:27:35.684+0900	[############............] woowahan.datacol	13.0MB/24.6MB (53.0%)
2023-07-21T11:27:38.684+0900	[##############..........] woowahan.datacol	14.7MB/24.6MB (59.7%)
2023-07-21T11:27:41.684+0900	[###############.........] woowahan.datacol	16.2MB/24.6MB (66.0%)
2023-07-21T11:27:44.683+0900	[#################.......] woowahan.datacol	17.6MB/24.6MB (71.4%)
2023-07-21T11:27:47.684+0900	[##################......] woowahan.datacol	19.2MB/24.6MB (78.0%)
2023-07-21T11:27:50.684+0900	[###################.....] woowahan.datacol	20.4MB/24.6MB (83.0%)
2023-07-21T11:27:53.684+0900	[#####################...] woowahan.datacol	21.9MB/24.6MB (89.0%)
2023-07-21T11:27:56.684+0900	[######################..] woowahan.datacol	23.4MB/24.6MB (95.1%)
2023-07-21T11:27:59.315+0900	[########################] woowahan.datacol	24.6MB/24.6MB (100.0%)
2023-07-21T11:27:59.315+0900	100000 document(s) imported successfully. 0 document(s) failed to import.

````


