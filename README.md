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
