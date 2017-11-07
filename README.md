# Dreamforce 17 - Salesforce Integration and API Overview Code Snippets

## Rest SOQL
Select all Brokers
```
/services/data/v41.0/query/?q=SELECT id,name,email__c FROM Broker__c
```

## Rest CRUD
Describe the Broker object
```
/services/data/v37.0/sobjects/Broker__c/describe
```
GET a specific Broker
```
/services/data/v37.0/sobjects/Broker__c/a00410000056sPvAAI
```
PATCH a broker to update some fields
```JSON
{
"Mobile_Phone__c" : "+1-617-244-3672",
"Phone__c" : "+1-617-244-3673"
}
```
## SLACK integration
```Apex
public class SlackPub {

    /*
    General notes. This version of the "publish to Slack" class will work with any object. The developer 
    passes in a PubRecord object with fields that represent an SObject type, the value of the name field
    (or some other identifying field like Case subject), and a single field name and value. 
    
    A text template is used to format the message. 
    
    A more complex version could accept additional field name/value pairs or an array of name/value pairs perhaps for an indeterminate number
    of fields to report as updated. 
    
    The class is intended for us to invoke from a process builder process. 
    */

    
    // DEMO POINT 1: Web hook URL for callout: 
    // Hard coded for demo, discuss alternative options.
    private static final String slackHookURL = 'https://hooks.slack.com/services/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';
    
    private static final String msgTemplate = 'The following {0} has changed:\n{1}\nNew {2}: *{3}*';

    public class PubRecord {
        @InvocableVariable(label='Object Type')
        public String objName;
        @InvocableVariable(label='Name Value')
        public String nameValue;
        @InvocableVariable(label='Field 1 Name')
        public String field1Name;
        @InvocableVariable(label='Field 1 Value')
        public String field1Value;
    }
    
    @InvocableMethod(label='Post Record Changes to Slack')
    public static void postToSlack(List<PubRecord> records) {
        PubRecord record = records[0];
        List<String> inputVals = getListFromPubRecord(record);
        
        String msgText = String.format(msgTemplate,inputVals);

        Map<String, Object> msg = new Map<String,Object>();
        msg.put('text', msgText);
        msg.put('mrkdwn', true);
        
        String body = JSON.serialize(msg);
        
        //DEMO POINT 4: Here's where we actually instantiate and 
        //pass the Queueable object to the queue, in order to execute it.
        System.enqueueJob(new QueueableSlackCall(slackHookURL, 'POST', body));
    }
    
    private static List<String> getListFromPubRecord(PubRecord record){
        List<String> fieldList = new List<String>();
        
        fieldList.add(record.objName);
        fieldList.add(record.nameValue);
        fieldList.add(record.field1Name);
        fieldList.add(record.field1Value);
        return fieldList;
    }
 
    // DEMO POINT 2: We are using an async callout via a Queuable class. Means asynchronous and can be monitored. 
    // AllowsCallouts interface tells the Apex runtime that a callout will be permitted from this bit of code.
    public class QueueableSlackCall implements System.Queueable, Database.AllowsCallouts {
        
        private final String url;
        private final String method;
        private final String body;
        public QueueableSlackCall(String url, String method, String body){
            this.url = url;
            this.method = method;
            this.body = body;
        }
        
        //DEMO POINT 3: In the execute method we see a straightforward HTTP request
        public void execute(System.QueueableContext ctx){
            //An instance of HTTP is used to invoked the request
            HttpRequest req = new HttpRequest();
            req.setEndpoint(url);
            req.setMethod(method);
            req.setBody(body);
            Http http = new Http();
            //Returns instance of HTTPResponse with any response data (none in this case, so we just leave it).
            HttpResponse resp = http.send(req);
        }
    }
}
```
