# BAU-STAT
db.getCollection('PACKAGE').aggregate([
    {
     $lookup:
       {
         from: "TEAM_MEMBER",
           let: { executorId: "$executorId"},
           pipeline: [
              { $match:
                 {  userMemberId: { $exists: true},
                   $expr:
                    { $and:
                       [
                        { $eq: [ "$userMemberId",  "$$executorId" ] },
                         { $eq: [ "$status",  "ADDED" ] }
                       ]
                    }
                 }
              },
              { $project: {teamId: 1 } }
           ],
           as: "userTeam"
       }
   },
   {
       $group : {
           _id : { userTeam:"$userTeam",packageVersion:"$packageVersion" , day : {$dateToString: { format: "%Y-%m-%d", date: "$create_time" }}},
           count: { $sum: 1 }
       }
   },
   { $merge: {
         into: { db: "cortex5", coll: "ViewTeamPackageDay" },
         on: "_id",
         whenMatched:  "merge",
         whenNotMatched: "insert"
   } }
])
db.getCollection('from_wallet').aggregate([
    {
        
    }
])


db.getCollection('PACKAGE').aggregate([
    {
       $group : {
           _id : { packageVersion:"$packageVersion" , day : {$dateToString: { format: "%Y-%m-%d", date: "$create_time" }}},
           count: { $sum: 1 }
        }
    },

   { $merge: {
         into: { db: "cortex5", coll: "ViewPackageDay" },
         on: "_id",
         whenMatched:  "replace",
         whenNotMatched: "insert"
   } }
])
db.getCollection('ViewPackageDay').find({"_id.userTeam":{$elemMatch:{"teamId":"5de1c96db24af400014a48a7"}}})


db.getCollection('BILL').aggregate([
    {
        $match : { "isPackageBill": true,  "method":false}
    },
    {
       $group : {
           _id : {  currency: "$currency", day : {$dateToString: { format: "%Y-%m-%d", date: "$create_time" }}},
            subAmount: { 
               $sum : "$amount"
           }
        }
    },

   { $merge: {
         into: { db: "cortex5", coll: "ViewBillPackageDay" },
         on: "_id",
         whenMatched:  "merge",
         whenNotMatched: "insert"
   } }
])
db.getCollection('BILL').aggregate([
    {
        $match : { "isPackageBill": true,  "method":true}
    },
    {
       $group : {
           _id : {  currency: "$currency", day : {$dateToString: { format: "%Y-%m-%d", date: "$create_time" }}},
            addAmount: { 
               $sum : "$amount"
           }
        }
    },

   { $merge: {
         into: { db: "cortex5", coll: "ViewBillPackageDay" },
         on: "_id",
         whenMatched:  "merge",
         whenNotMatched: "insert"
   } }
])

db.getCollection('BILL').aggregate([
    {
        $match : { "isWorkerIncome": true,  "method":true}
    },
    {
       $group : {
           _id : {  currency: "$currency", day : {$dateToString: { format: "%Y-%m-%d", date: "$create_time" }}},
           addAmount: { 
               $sum : "$amount"
           }
        }
    },

   { $merge: {
         into: { db: "cortex5", coll: "ViewBillWorkerIncomeDay" },
         on: "_id",
         whenMatched:  "merge",
         whenNotMatched: "insert"
   } }
])

db.getCollection('BILL').aggregate([
    {
        $match : { "isWorkerIncome": true,  "method":false}
    },
    {
       $group : {
           _id : {  currency: "$currency", day : {$dateToString: { format: "%Y-%m-%d", date: "$create_time" }}},
           subAmount: { 
               $sum : "$amount"
           }
        }
    },

   { $merge: {
         into: { db: "cortex5", coll: "ViewBillWorkerIncomeDay" },
         on: "_id",
         whenMatched:  "merge",
         whenNotMatched: "insert"
   } }
])

db.getCollection('PACKAGE').aggregate([
    {
        $match : { "codAmount": {$gt :0}}
    },
    {
       $group : {
           _id : {  day : {$dateToString: { format: "%Y-%m-%d", date: "$create_time" }}},
           total: { 
               $sum : 1
           },
           amount:{$sum : "$codAmount"}
        }
    },

   { $merge: {
         into: { db: "cortex5", coll: "ViewCODDay" },
         on: "_id",
         whenMatched:  "replace",
         whenNotMatched: "insert"
   } }
])

db.getCollection('PACKAGE').aggregate([
    { 
      $group : {
           _id : {   day : {$dateToString: { format: "%Y-%m-%d", date: "$create_time" }}},
           infoSenderAccountIdList: { $addToSet: "$infoSenderAccountId" }
        }
    },
   { $merge: {
         into: { db: "cortex5", coll: "ViewSenderAccountDay" },
         on: "_id",
         whenMatched:  "replace",
         whenNotMatched: "insert"
   } }
])