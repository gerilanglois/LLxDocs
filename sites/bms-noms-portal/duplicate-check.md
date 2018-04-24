<!-- TITLE: Duplicate Check -->
<!-- SUBTITLE: A detailed view of the BMS Noms Portal Duplicate Check process -->

# BMS Noms Portal Duplicate Check Integration with CRM
When a service provider is add via the SP Add functionality on the site the contract for the process they are being added to is marked with an internal status of PendingDuplicateCheck. From there, this following process is utilized:

1. **CRM:** Workflow initiates WCF Pull Request (scheduled hourly)

2. **WCF:** Receives PullNewHCPsFromNomToCRM Request 

3. **WCF:** Using NominationsApi calls nominationsApiClient.GetAwaitingSpeakers

4. **Site:** Using a Web Hook calls NominationsApiController.SpeakersAwaiting, which returns Speakers with a pending contract internal status of PendingDuplicateCheck

5. **WCF:** Using NominationsApi calls nominationsApiClient.MarkAsExported

6. **Site:** Using a Web Hook calls NominationsApiController.MarkAsExported (sets contract.SetToCrmUtc = DateTime.UtcNow)

7. **WCF:** Returns a PullNewHCPsFromNomToCRM Response

8. **CRM:** write potential duplicates to the potential duplicates table

9. **CRM User:** Manual determines if a potential duplicate is actually a duplicate

10. **CRM:** Contract status changes event is raised

11. **CRM:** web request is made to ContractStatusChangeNotification.aspx with notification type of BmsNomsCrmDuplicateCheckReturn

12. **WCF:**  Using NominationsApi calls nominationsApiClient.SetCrmGuids(list);

13. **Site:** Using a Web Hook calls NominationsApiController.SetCrmGuids(list); site sets crmcontactguid, crmcontractguid and llid

14. **CRM:** web request is made to ContractStatusChangeNotification.aspx with notification type of  bmsnomscontractchange

15. **WCF:** uses CRMAccess proxy to read contract status from CRM

16. **WCF:** invokes BmsNomsContractStatusCompleteController.SetContractStatus

17. **Site:** Crm Info Complete workflow traps incoming crm status change of New_Speaker_Submitted_For_Review  and sets the internal status to CrmInfoComplete

18. **Site:** using CrmContractEventHandler detects the internal status change, checks IA status, checks regional allocations, and sets InternalContractStatus to NominationAdd or OnHold as appropriate
