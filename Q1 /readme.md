## Create a batch apex which should check all the open not contacted leads created in the current week and update the lead as Closed Lost. The batch apex should be scheduled to run on every Sunday at 10:00 pm

# Wirte Batch 
```jsx
public class LeadBatch Implements Database.Batchable<sobject>{
 
    
    Public Database.QueryLocator Start (Database.BatchableContext bc){
        return Database.getQueryLocator([Select id,Name,Status from lead where Status='Open - Not Contacted' AND createdDate =Last_Week]);
    } 
    public void execute(Database.BatchableContext bc,list<Lead> listLeads){
        for(lead leads:listLeads){
            leads.Status='Closed Lost';           
        }
        update listLeads;
      
        
    }
public void Finish(Database.BatchableContext bc){
     System.debug('update record Completed');
    }   
} ```
