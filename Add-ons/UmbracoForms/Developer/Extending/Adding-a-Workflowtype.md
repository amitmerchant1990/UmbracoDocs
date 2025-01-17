---
versionFrom: 7.0.0
---

# Adding a workflow type to Umbraco Forms
*This builds on the "[adding a type to the provider model](Adding-a-Type.md)" chapter*

Add a new class to your project and have it inherit from Umbraco.Forms.Core.WorkflowType, implement the class. For this sample we will focus on the execute method. This method process the current record (the data submitted by the form) and have the ability to change data and state.

```csharp
using System;
using System.Collections.Generic;
using umbraco.BusinessLogic;
using Umbraco.Forms.Core;
using Umbraco.Forms.Core.Enums;
using Umbraco.Forms.Data.Storage;
using Umbraco.Forms.Web.Services;

/// <summary>
/// Summary description for Class1
/// </summary>
public class TestWorkflow : WorkflowType
{

    public TestWorkflow()
    {
        this.Id = new Guid("ccbeb0d5-adaa-4729-8b4c-4bb439dc0202");
        this.Name = "TestWorkflow";
        this.Description = "This workflow is just for testing";
        this.Icon = "icon-chat-active";
        this.Group = "Services";
    }
    public override WorkflowExecutionStatus Execute(Record record, RecordEventArgs e)
    {
        // first we log it
        Log.Add(LogTypes.Debug, -1, "the IP " + record.IP + " has submitted a record");

        // we can then iterate through the fields
        foreach (RecordField rf in record.RecordFields.Values)
        {
            // and we can then do something with the collection of values on each field
            List<object> vals = rf.Values;

            // or get it as a string
            rf.ValuesAsString();
        }

        // If we altered a field, we can save it using the record storage
        RecordStorage store = new RecordStorage();
        store.UpdateRecord(record, e.Form);
        store.Dispose();

        // we then invoke the recordservice which handles all record states // and make the service delete the record.
        RecordService rs = new RecordService();
        rs.Delete(record, e.Form);

        return WorkflowExecutionStatus.Completed;
    }

    public override List<Exception> ValidateSettings()
    {
        return new List<Exception>();
    }
}
```

The Execute() method gets a Record and a RecordEventArgs argument. These 2 arguments contains all information related to the workflow. The record contains all data and meta data submitted by the form. The RecordEventArgs contains references to what form the record is from, what state it is in and a reference to the current HttpContext

The sample above uses 2 different areas to work with the record. The first is the RecordStorage class which handles all storage of the record data. The second is the RecordService, this handles record state changes and record events.
