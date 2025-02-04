## Create a batch apex which should check all the open not contacted leads created in the current week and update the lead as Closed Lost. The batch apex should be scheduled to run on every Sunday at 10:00 pm

### Batch  Class
```jsx
/**
 * This batch job processes Lead records in Salesforce.
 * It updates the status of leads created last week and marked as 'Open - Not Contacted' to 'Closed Lost'.
 * The job follows the standard Batchable interface with start, execute, and finish methods.
 */
public class LeadBatch Implements Database.Batchable<sobject>{
 
    Public Database.QueryLocator Start (Database.BatchableContext bc){ //get Reorod 
        return Database.getQueryLocator([Select id,
                                         Name,
                                        Status from lead
                                        where Status='Open - Not Contacted'
                                        AND createdDate =Last_Week]);
    } 
    public void execute(Database.BatchableContext bc,list<Lead> listLeads){ // Process Record
              for(lead leads:listLeads){
                  leads.Status='Closed Lost';           
              }
        update listLeads;     
    }
public void Finish(Database.BatchableContext bc){
     System.debug('update record Completed');
    }   
}
```
### Scheduled Class
```jsx
public class LeadUpdateWeedlyScheduler implements Schedulable {
    
    public void execute(SchedulableContext sc){
        // Call the Batch Class
        LeadBatch leadB= new leadBatch(); 
        Database.executeBatch(leadB);
    }

}
```
## Execute Window 
### Schedule Time Of Schedule Class 
```jsx
  String Times='0 0 22 ? * Sun *';
  system.schedule('Weekly update lead',times, new LeadUpdateWeedlyScheduler());
```
## Test Class
```jsx
@isTest
public class TestClassBatchAndSchedule {
    
@TestSetup  static void TestLead(){
       list<Lead> listLead=new list<lead>();
        
        for(integer i=0;i<5;i++){
            if(i<2){
               listLead.add(New Lead (LastName='Test'+i,Company='Fibler'+i,Status='Open - Not Contacted'));
            }else{
                listLead.add(New Lead (LastName='Test'+i,Company='tibiler'+i,Status='Working - Contacted'));
            }
        } 
      insert listLead;
           for(lead ld:listLead){
             Test.SetCreatedDate(ld.id,Date.today()-7); //  it set CreatedDate
                }
       }
 @Istest static void TestBatchClass(){
             
               test.StartTest();
               LeadBatch BatchTest=new LeadBatch();
               Database.executeBatch(BatchTest);
               Test.StopTest();
               
         List<Lead> updatedLeads = [SELECT Id, Status FROM Lead where Status='Closed Lost'AND CreatedDate = LAST_WEEK];
             System.assertEquals(2,updatedLeads.size());   //test Batch Class
}
    @IsTest Static void ScheduleClass(){
        
             Test.StartTest();
             String Times='0 0 22 ? * Sun *';
             system.schedule('Weekly update lead',times, new LeadUpdateWeedlyScheduler());
             Test.stopTest();

            for(Lead Lead:[Select Status from lead where status='Closed Lost']){
                    system.assertEquals('Closed Lost',lead.Status);    //test Schedule Class
            }
         }
    }
```
