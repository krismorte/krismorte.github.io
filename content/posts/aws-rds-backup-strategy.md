---
title: "AWS RDS Backup Strategy"
date: "2020-04-13T18:55:33Z"
tags: ["java","database"]

---

This post will talk about backup and disaster recovery strategy on AWS RDS been mande by lambdas. First of all let's talk about the default snapshot that comes enable on every RDS and from my sight is a really good solution with its point-in-time option that takes log backups each every 5 minutes which allow us huge coverage on failures. There's 2 types of snapshot automated and manual, automated are taken daily and  incremented all day long to delivery the point-in-time solution these snapshot are holding for 35 days max, and this isn't configurable, the manual are made manually, throuth the CLI or by application and there's no expire date and are limited to 100 per instance and region but you can increase this number opening a support ticket with AWS. An important detail here is the possibility to take copy of these snapshots for both but from automated the result always became a manual, and this will be our strategy today.

Depending on which is you business and legal implications you have to keep copies for you data for amount of time. In this case automated copies from the existing backups is pretty handful, although is interesting put a a limit date to keep all these backups. For this reason we have 2 pair for lambda one for made the backup and another to remove it. When is comes to disaster recovery I won't present a full action automated plan to take a minimum downtime possible today, instead will show the base action of do the backups and copy in a different region in case of failure of the main one. Remind yourself that we are taking care only of your RDS snapshot for now, to a full disaster recovery your application has to be copied as well as all the dependencies, and if you want to come backup to the main region all the steps until now have to made in the opposite direction. 

After this brief explanation let go to the code, although I'm a DBA still have to dominate python so this time will be javascript other tools would be AWS CLI configured, nodejs >=10 with [serverless framework](https://www.serverless.com/) installed , this framework is used to deploy on AWS. Full version of this code is on [github](https://github.com/krismorte/lambda-rds-snapshot)

AWS SDK is huge so I got just the function I need on the applications in a single file `rds-func.js`

```javascript
const defaultTag = {
    Key: 'ClonedBy',
    Value: 'Lambda'
  }
const regionTag = { Key: 'SourceRegion',Value: ''}

class RDSFunc{
  constructor(AWS) {
    this.rds = new AWS.RDS();
  }

  describeClusters(){
    var clusters = this.rds.describeDBClusters({}).promise()
    .then((data)=>{
      return data.DBClusters;
    })    
    return clusters;
  }

  describeInstances(){
    var instances = this.rds.describeDBInstances({}).promise()
    .then((data)=>{
      return data.DBInstances
    })
    return instances;
  }

  describeClustersAutomatedSnapshot(cluster){

    const params = {
      DBClusterIdentifier: cluster,
      SnapshotType: "automated",
      MaxRecords: 50
    }

    var snaps = this.rds.describeDBClusterSnapshots(params).promise()
    .then((data)=>{
      return data.DBClusterSnapshots;
    })    
    return snaps;
  }

  describeClustersManualSnapshot(cluster){

    const params = {
      DBClusterIdentifier: cluster,
      SnapshotType: "manual",
      MaxRecords: 50
    }

    var snaps = this.rds.describeDBClusterSnapshots(params).promise()
    .then((data)=>{
      return data.DBClusterSnapshots;
    })    
    return snaps;
  }

  describeInstanceAutomatedSnapshot(instance){

    const params = {
      DBInstanceIdentifier: instance,
      SnapshotType: "automated",
      MaxRecords: 50
    }

    var snaps = this.rds.describeDBSnapshots(params).promise()
    .then((data)=>{
      return data.DBSnapshots;
    })    
    return snaps;
  }

  describeInstanceManualSnapshot(instance){

    const params = {
      DBInstanceIdentifier: instance,
      SnapshotType: "manual",
      MaxRecords: 50
    }

    var snaps = this.rds.describeDBSnapshots(params).promise()
    .then((data)=>{
      return data.DBSnapshots;
    })    
    return snaps;
  }

  copyClusterSnapshot(snap,region){
    
    var paramClone = {
      SourceDBClusterSnapshotIdentifier: snap,
      TargetDBClusterSnapshotIdentifier: this.renameSnapshot(snap),
      Tags: [
        defaultTag,
      ]
    };
    if(region){
      paramClone.SourceRegion=region
      regionTag.Value = region
      paramClone.Tags.push(regionTag)
    }

    var newSnapName = this.rds.copyDBClusterSnapshot(paramClone).promise()
    .then((data)=>{
      return paramClone.TargetDBClusterSnapshotIdentifier
    });
    return newSnapName
  }

  copyInstanceSnapshot(snap,region){
    
    var paramClone = {
      SourceDBSnapshotIdentifier: snap,
      TargetDBSnapshotIdentifier: this.renameSnapshot(snap),
      Tags: [
        defaultTag,
      ]
    };
    if(region){
      paramClone.SourceRegion=region
      regionTag.Value = region
      paramClone.Tags.push(regionTag)
    }
    console.log(paramClone)

    var newSnapName = this.rds.copyDBSnapshot(paramClone).promise()
    .then((data)=>{
      return paramClone.TargetDBSnapshotIdentifier
    });
    return newSnapName
  }

  deleteClusterSnapshot(snap){
    this.rds.deleteDBClusterSnapshot({DBClusterSnapshotIdentifier:snap}).promise()
    .then((data)=>{
    });
  }

  deleteInstanceSnapshot(snap){
    this.rds.deleteDBSnapshot({DBSnapshotIdentifier:snap}).promise()
    .then((data)=>{
    });
  }

  renameSnapshot(snapName){
    if(snapName.includes('arn:')){
      var i = snapName.indexOf("snapshot:rds");
      var tmpName = snapName.substring(i, snapName.length)
      return tmpName.replace('snapshot:','').replace("rds:", "manual-");
    }else{
      return snapName.replace("rds:", "manual-")
    }
  }

}


module.exports = {
  RDSFunc
}
```

First detail that's pops-up is that cluster and instances are treat by different call on SDK following by its snapshots

>>describeDBClusters
>>describeDBInstances
>>describeDBClusterSnapshots
>>describeDBSnapshots

Another detail is to search snapshots you have to know the resource's names (cluster or instance), because of that all lambdas perform a query to list all resources for both copy or delete.

Below we have a list of all daily cluster backups, checked by date from a function that will remove days from today and right after will call copy function. As all calls are asynchronous after lambda execution you can follow the copy through the console, depend of the database size of course. On the second piece all this process is repeated to instances.

```javascript
    var cluster = await rdsFunc.describeClusters();

    cluster.forEach(async (cluster)=>{
        var snaps = await rdsFunc.describeClustersAutomatedSnapshot(cluster.DBClusterIdentifier)
        if(snaps){
          snaps.forEach(async (snap)=>{
            var copyDate = dateFunc.minusDaysFromToday(daysBefore);
            var snapshotDate = dateFunc.removeTimeFromDate(snap.SnapshotCreateTime);
            if (copyDate == snapshotDate) {
                var copy = await rdsFunc.copyClusterSnapshot(snap.DBClusterSnapshotIdentifier)
                console.log(copy+" Rds cluster snapshot cloned")
            }
        })
        }        
    })

//RDS Instance
    var instances = await rdsFunc.describeInstances();
    instances.forEach(async (instance)=>{
      var snaps = await rdsFunc.describeInstanceAutomatedSnapshot(instance.DBInstanceIdentifier)
      if(snaps){
        snaps.forEach(async (snap)=>{
          var copyDate = dateFunc.minusDaysFromToday(daysBefore);
          var snapshotDate = dateFunc.removeTimeFromDate(snap.SnapshotCreateTime);
          if (copyDate == snapshotDate) {
              var copy = await rdsFunc.copyInstanceSnapshot(snap.DBSnapshotIdentifier)
              console.log(copy+" Rds instance snapshot cloned")
          }
        })
      }      
    })
```

Delete follow the same structure but instead of copy function you have delete

```javascript
    var cluster = await rdsFunc.describeClusters();

    cluster.forEach(async (cluster)=>{
        var snaps = await rdsFunc.describeClustersManualSnapshot(cluster.DBClusterIdentifier)
        
        if(snaps){
          snaps.forEach(async (snap)=>{
            var copyDate = dateFunc.minusDaysFromToday(daysBefore);
            var snapshotDate = dateFunc.removeTimeFromDate(snap.SnapshotCreateTime);
            if (copyDate == snapshotDate) {
                var copy = await rdsFunc.deleteClusterSnapshot(snap.DBClusterSnapshotIdentifier)
                console.log(copy+" Rds cluster snapshot deleted")
            }
        })
        }        
    })
```

On the function that copy to another region comes a interesting detail, the search has to be realized from the original region but the copy will be executed from the destiny region and informed the original region. Another detail here is that copy function will reference the snapshot by ARN instead the name.

```javascript
var cluster = await rdsFunc.describeClusters();

for (i=0;i<cluster.length;i++){
    var snaps = await rdsFunc.describeClustersAutomatedSnapshot(cluster[i].DBClusterIdentifier)
    if(snaps){
        snaps.forEach(async (snap)=>{
            AWS.config.update({
                region: process.env.copyRegion
            })
            var rdsFuncOtherRegion = new RDSFunc(AWS)
            var copyDate = dateFunc.minusDaysFromToday(daysBefore);
            var snapshotDate = dateFunc.removeTimeFromDate(snap.SnapshotCreateTime);
            if (copyDate == snapshotDate) {
var copy = await rdsFuncOtherRegion.copyClusterSnapshot(snap.DBClusterSnapshotArn,process.env.mainRegion)
            console.log(copy+" Rds cluster snapshot cloned")
            }
        })
     }        
}
```

Delete function is the same but one more time just replacing copy for delete

If you look inside the function all variables comes from environment variables that were declared on the mail file  `serverless.yml` , this file is used by the framework. Below we inform the provider, the region where we will deploy you local aws profile and an IAM with RDS access (you can change for one more restrict one) and the variables

```yaml
provider:
  name: aws
  runtime: nodejs10.x
  region: us-west-2
  profile: <YOUR-AWS-PROFILE>
  iamManagedPolicies:
    - "arn:aws:iam::aws:policy/AmazonRDSFullAccess"
  environment:
    daysBefore: 30
    keepDays: 6
    keepLongDays: 365
    mainRegion: us-west-2
    copyRegion: us-east-2
```

__daysBefore:__ _is about the amount of days on the snapshot research to turn it to manual, you can put 0 to get today but remember  that the snapshots are incremental depend of the time you can get an incomplete one._
__keepDays & keepLongDays:__ _are related to how many days snapshots have to be kept respectively in another region and in the same region._

__mainRegion & copyRegion:__ _the last two are related to the RDS region and the disaster recovery region_

The packaging can be customized in that case I just use it to exclude some files

```yaml
package:
  exclude:
    - README.md
    - .gitignore
    - .vscode
    - node_modules
    - __test__
```

Below we finally have the function declaration in this order name, file + function name, timeout and a event trigger in our case is a cron schedule . I prefer this approach of deploy all function at once but it's possible deploy one function at the time, you will see right below.

```yaml
functions:
  manual-backup:
    handler: rds-clone-snap.main
    timeout: 15
    events:
      - schedule: cron(0 10 1/1 * ? *)
  manual-backup-other-region:
    handler: rds-clone-snap-region.main
    timeout: 15
    events:
      - schedule: cron(0 10 1/1 * ? *)
  delete-backup:
    handler: rds-del-snap.main
    timeout: 15
    events:
      - schedule: cron(0 10 1/1 * ? *)
  delete-backup-other-region:
    handler: rds-del-snap-region.main
    timeout: 15
    events:
      - schedule: cron(0 10 1/1 * ? *)
```

Ok, everything is set now is the time to send all to the cloud. This code below is all you need

`serverless deploy`

Or function by function

`serverless deploy function --function manual-backup`

To remove 

`serverless remove`

Here we can say that we can sleep more peacefully knowing that our RDS are protected from possible failures. The basics of snapshots and availability in other regions are covered here, it is obvious that a disaster recovery solution for your environment will be more complete and will involve more technologies and other forms of automation.
