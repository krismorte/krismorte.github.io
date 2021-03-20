---
title: "AWS RDS Backup Strategy"
date: "2020-04-13T18:55:33Z"
tags: ["java","database"]
---

Esse post vai tratar de uma estratégia de backup e disaster recovery voltado ao AWS RDS feito através de lambdas. Primeiramente vamos falar sobre as snapshots que vem habilitadas por default no RDS é uma solução ao me ver bem robusta com a opção de point-in-time que realiza backups de logs a cada 5 minutos o que permite uma cobertura enorme em casos de falhas. Existem 2 tipos de snapshot automática e manual, as automáticas são feitas diariamente e incrementais durante o decorrer do dia para fornecer o point-in-time elas são mantidas por no máximo 35 dias, e isso não é configurável, já as manuais são feitas por console, cli ou aplicações e não tem data de validade e são limitadas a 100 por instância e região mais esse número pode ser incrementados via chamado com a AWS. Um detalhe importante aqui é a possibilidade de você copiar essas snapshot tanto manual como automática sendo que toda snapshot automática copiada se tornar manual, essa será nossa estrategia aqui.

Dependendo do seu negócio e das implicações legais você terá de manter cópias de dados por um determinado tempo. Nesse caso automatizar cópias dos backups já feitos para que se tornem manuais é bem útil, porém é interessante por um limite (em dias) para manter essas snapshots. Por isso temos 2 pares de lambdas uma para a realização do backup e outra pra remoção. 
Se tratando do disaster recovery aqui não será apresentada uma solução 100% automatiza com o menor tempo de downtime, mas a função mais básica que é ter backups e cópias desses backups em outras regiões em caso de falha da região principal. Mais uma vez através de lambdas nesse caso recomendo manter essas snapshopts por pelo menos uma semana. Lembrando que aqui estamos tratando somente de snapshot de RDS, sua aplicação deve ser também copiada para a nova região e caso queira retornar para a região original todos os passos deverão ser repetidos no sentido contrário.

Depois dessa breve explicação vamos ao código, apesar de ser DBA ainda não domino python então vamos de javascript  ferramentas necessárias serão AWS CLI configurada, nodejs >=10 com o [serverless framework](https://www.serverless.com/) instalado esse framework é usado pro deploy na AWS. Versão completa do código está no [github](https://github.com/krismorte/lambda-rds-snapshot)

A SDK da AWS é imensa, então separei apenas as funções que preciso nas minhas aplicações em um único arquivo `rds-func.js`

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

Primeiro detalhe que salta aos olhos é que cluster e intâncias são tratados por chamadas diferentes na SDK assim como suas respectivas snapshots

>>describeDBClusters
describeDBInstances
describeDBClusterSnapshots
describeDBSnapshots

Outro detalhe é que para buscar as snapshots é obrigatório saber o nome do recurso (cluster ou instância), por isso em todos os lambdas foi necessário listar os recursos tanto para copiar as snapshots quanto para removê-las.

No trecho abaixo temos o backup diário onde são listados os cluster, verificado a data a partir de uma função que recebe quantos dias a partir de hoje serão removidos e logo depois é feita a chamada de cópia. Como todas as chamadas são assícronas depois que o lambda for executado você poderá acompanhar a criação da snapshot pelo console, dependendo do tamanho do server isso pode ser bem rápido. Logo depois o mesmo processo é repetido para instâncias.

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

A remoção segue a mesma lógica trocando a chamada de cópia pela de remoção

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

Na função que faz a copia para outra região entra um detalhe interessante, as buscas  são realizadas a partir da região original, mas o pedido de cópia deve ser feito da região de destino mas obrigatoriamente informado a região de origem, outro detalhe é que aqui a snapshot será referenciada pelo ARN e não pelo nome.

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

No lambda delete a lógica é a mesma onde é trocada a chamada de cópia pela de remoção e a snapshot volta a ser referênciada pelo nome.

Veja que dentro das funções toda a parametrização vem por meio de env todas são declaradas no arquivo principal `serverless.yml` usado pelo framework. No trecho abaixo informamos o provider a região onde iremos deployar o profile configurado para o seu aws cli, uma IAM com poderes full sobre os RDS (você pode alterar por uma mais limitado) e as variáveis.

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

__daysBefore:__ _diz respeito a quantidades de dias para a busca de backups automaticos e torna-los manual, você pode tornar esse valor zero para pegare sempre o mais atual, mas lembre-se que as snapshot são incrementais então dependendo do horário a snapshot poderá nçao estar completa._
__keepDays & keepLongDays:__ _são referentes a quantos dias as snapshot devem ser mantidas a primeira se refere as snapshot em outra região e segunda na mesma região_

__mainRegion & copyRegion:__ _as duas últimas se referem a regiões onde os RDS estão e a região secundária visando o disaster recovery_

O empacotamento pode ser customizado nesse caso eu apenas o utilo para exluir alguns arquivos

```yaml
package:
  exclude:
    - README.md
    - .gitignore
    - .vscode
    - node_modules
    - __test__
```

Nesse trecho as funções propriamente ditas onde declaro nome, arquivo + function, um timeout e o evento disparador que nesse caso é um agendamento cron. Eu prefiro essa abordagem de declarar e deployar todas as funções de uma vez, porém caso queria é possivel deployar uma por vez como será visto abaixo.

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

Pronto tudo feito agora nos resta de fato manda-las pra nuvem. Basta usar a linha de comando pra isso

`serverless deploy`

ou por função

`serverless deploy function --function manual-backup`

Pra remover

`serverless remove`



Aqui podemos dizer que poderemos dormir mais tranquilamente sabendo que nossos RDS estão protegidosde eventuais falhas. O básico das snapshots e disponibilidades em outras regiões ficam cobertas aqui, óbvio que uma solução de disaster recovery do seu ambiente será mais compliado e envolverá mais tecnologias e outras formas de automação.